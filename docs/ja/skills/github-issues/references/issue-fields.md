# Issue Fields（GraphQL、Private Preview）

> **Private preview:** Issue fields は現在プライベートプレビューです。アクセス申請は https://github.com/orgs/community/discussions/175366 から行ってください。

Issue fields は、組織レベルで定義し、各 Issue ごとに設定するカスタムメタデータ（日時、テキスト、数値、単一選択）です。labels、milestones、assignees とは別物です。一般的な例: Start Date、Target Date、Priority、Impact、Effort。

**重要:** すべての issue field のクエリとミューテーションには `GraphQL-Features: issue_fields` HTTP ヘッダーが必要です。これがないと、スキーマ上で field が表示されません。

**project fields より issue fields を優先してください。** Issue に日時、優先度、ステータスのようなメタデータを設定する必要がある場合は、project fields（project item に属する）ではなく issue fields（Issue 自体に属する）を使用してください。issue fields は project やビューをまたいでも Issue と一緒に移動しますが、project fields は単一の project にスコープされます。issue fields が利用できない場合、または field が project 固有（例: スプリントのイテレーション）の場合にのみ project fields を使ってください。

## 利用可能な field を確認する

Field は org レベルで定義されます。値を設定しようとする前に一覧を取得してください。

```graphql
# Header: GraphQL-Features: issue_fields
{
  organization(login: "OWNER") {
    issueFields(first: 30) {
      nodes {
        __typename
        ... on IssueFieldDate { id name }
        ... on IssueFieldText { id name }
        ... on IssueFieldNumber { id name }
        ... on IssueFieldSingleSelect { id name options { id name color } }
      }
    }
  }
}
```

Field type: `IssueFieldDate`, `IssueFieldText`, `IssueFieldNumber`, `IssueFieldSingleSelect`。

single-select field では、値を設定するために option の `id`（name ではなく）が必要です。

## Issue 上の field 値を読む

```graphql
# Header: GraphQL-Features: issue_fields
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      issueFieldValues(first: 20) {
        nodes {
          __typename
          ... on IssueFieldDateValue {
            value
            field { ... on IssueFieldDate { id name } }
          }
          ... on IssueFieldTextValue {
            value
            field { ... on IssueFieldText { id name } }
          }
          ... on IssueFieldNumberValue {
            value
            field { ... on IssueFieldNumber { id name } }
          }
          ... on IssueFieldSingleSelectValue {
            name
            color
            field { ... on IssueFieldSingleSelect { id name } }
          }
        }
      }
    }
  }
}
```

## field 値を設定する

一度に 1 つ以上の field を設定するには `setIssueFieldValue` を使います。上記の確認クエリで取得した Issue の node ID と field ID が必要です。

```graphql
# Header: GraphQL-Features: issue_fields
mutation {
  setIssueFieldValue(input: {
    issueId: "ISSUE_NODE_ID"
    issueFields: [
      { fieldId: "IFD_xxx", dateValue: "2026-04-15" }
      { fieldId: "IFT_xxx", textValue: "some text" }
      { fieldId: "IFN_xxx", numberValue: 3.0 }
      { fieldId: "IFSS_xxx", singleSelectOptionId: "OPTION_ID" }
    ]
  }) {
    issue { id title }
  }
}
```

`issueFields` の各エントリには、`fieldId` に加えて値パラメータを**ちょうど 1 つ**指定します。

| Field type | Value parameter | Format |
|-----------|----------------|--------|
| Date | `dateValue` | ISO 8601 date string, e.g. `"2026-04-15"` |
| Text | `textValue` | String |
| Number | `numberValue` | Float |
| Single select | `singleSelectOptionId` | ID from the field's `options` list |

field 値をクリアするには、値パラメータの代わりに `delete: true` を設定します。

## field 設定のワークフロー

1. **field を確認する** - org の `issueFields` をクエリして field ID と option ID を取得する
2. **Issue の node ID を取得する** - `repository.issue.id` から取得する
3. **値を設定する** - Issue の node ID と field エントリを指定して `setIssueFieldValue` を呼び出す
4. **可能ならまとめて設定する** - 複数 field を 1 回の mutation 呼び出しで設定できる

## 例: Issue に日付と優先度を設定する

```bash
gh api graphql \
  -H "GraphQL-Features: issue_fields" \
  -f query='
mutation {
  setIssueFieldValue(input: {
    issueId: "I_kwDOxxx"
    issueFields: [
      { fieldId: "IFD_startDate", dateValue: "2026-04-01" }
      { fieldId: "IFD_targetDate", dateValue: "2026-04-30" }
      { fieldId: "IFSS_priority", singleSelectOptionId: "OPTION_P1" }
    ]
  }) {
    issue { id title }
  }
}'
```

## field 値で検索する

### GraphQL バルククエリ（推奨）

field 値で Issue を見つける最も確実な方法は、GraphQL で Issue を取得し、`issueFieldValues` でフィルタリングすることです。検索修飾子構文（`field.name:value`）は、まだすべての環境で安定していません。

```bash
# Find all open P1 issues in a repo
gh api graphql -H "GraphQL-Features: issue_fields" -f query='
{
  repository(owner: "OWNER", name: "REPO") {
    issues(first: 100, states: OPEN) {
      nodes {
        number
        title
        updatedAt
        assignees(first: 3) { nodes { login } }
        issueFieldValues(first: 10) {
          nodes {
            __typename
            ... on IssueFieldSingleSelectValue {
              name
              field { ... on IssueFieldSingleSelect { name } }
            }
          }
        }
      }
    }
  }
}' --jq '
  [.data.repository.issues.nodes[] |
    select(.issueFieldValues.nodes[] |
      select(.field.name == "Priority" and .name == "P1")
    ) |
    {number, title, updatedAt, assignees: [.assignees.nodes[].login]}
  ]'
```

**`IssueFieldSingleSelectValue` のスキーマに関する注意:**
- 選択された option の表示テキストは `.name` に入ります（`.value` ではありません）
- `.color`、`.description`、`.id` も利用可能です
- 親 field への参照は `.field` にあります（field 名を取得するには inline fragment を使用）

### 検索修飾子構文（実験的）

Issue fields は、検索クエリ内のドット記法でも検索できる場合があります。これには REST では `advanced_search=true`、GraphQL では `ISSUE_ADVANCED` search type が必要ですが、結果は一貫せず、一致する Issue が存在していても 0 件になることがあります。

```
field.priority:P0                  # Single-select equals value
field.target-date:>=2026-04-01     # Date comparison
has:field.priority                 # Has any value set
no:field.priority                  # Has no value set
```

Field 名には **slug**（小文字、空白はハイフン）を使用します。たとえば "Target Date" は `target-date` になります。

```bash
# REST API (may not return results in all environments)
gh api "search/issues?q=repo:owner/repo+field.priority:P0+is:open&advanced_search=true" \
  --jq '.items[] | "#\(.number): \(.title)"'
```

> **警告:** コロン記法（`field:Priority:P1`）は黙って無視されます。検索修飾子を使う場合は、必ずドット記法（`field.priority:P1`）を使ってください。ただし、上記の GraphQL バルククエリ方式のほうがより確実です。完全な検索ガイドは [search.md](search.md) を参照してください。

