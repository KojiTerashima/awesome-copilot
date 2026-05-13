# Projects V2

GitHub Projects V2 は GraphQL 経由で管理されます。MCP サーバーは GraphQL API をラップする 3 つのツールを提供しているため、通常は生の GraphQL は不要です。

## MCP ツールの使用（推奨）

**プロジェクトの一覧取得:**
`mcp__github__projects_list` を `method: "list_projects"`、`owner`、`owner_type`（`"user"` または `"organization"`）付きで呼び出します。

**プロジェクトのフィールド一覧取得:**
`mcp__github__projects_list` を `method: "list_project_fields"` と `project_number` 付きで呼び出します。

**プロジェクトのアイテム一覧取得:**
`mcp__github__projects_list` を `method: "list_project_items"` と `project_number` 付きで呼び出します。

**Issue/PR をプロジェクトに追加:**
`mcp__github__projects_write` を `method: "add_project_item"`、`project_id`（node ID）、`content_id`（issue/PR の node ID）付きで呼び出します。

**プロジェクトアイテムのフィールド値を更新:**
`mcp__github__projects_write` を `method: "update_project_item"`、`project_id`、`item_id`、`field_id`、`value`（`text`、`number`、`date`、`singleSelectOptionId`、`iterationId` のいずれか 1 つを持つオブジェクト）付きで呼び出します。

**プロジェクトアイテムを削除:**
`mcp__github__projects_write` を `method: "delete_project_item"`、`project_id`、`item_id` 付きで呼び出します。

## プロジェクト操作のワークフロー

1. **プロジェクトを見つける** — 下の [名前でプロジェクトを見つける](#finding-a-project-by-name) を参照
2. **フィールドを把握する** - `projects_list` の `list_project_fields` を使ってフィールド ID とオプション ID を取得
3. **アイテムを見つける** - `projects_list` の `list_project_items` を使ってアイテム ID を取得
4. **変更する** - `projects_write` を使ってアイテムの追加・更新・削除を実行

## 名前でプロジェクトを見つける

> **⚠️ 既知の問題:** `projectsV2(query: "…")` は完全一致ではなくキーワード検索を行い、結果は更新日時の新しい順で返されます。"issue" や "bug" のような一般的な単語では、誤検出が何百件も返ることがあります。目的のプロジェクトが何十ページも奥に埋もれている場合があります。

次の優先順で進めてください。

### 1. 直接取得（番号が分かっている場合）
```bash
gh api graphql -f query='{
  organization(login: "ORG") {
    projectV2(number: 42) { id title }
  }
}' --jq '.data.organization.projectV2'
```

### 2. 既知の issue から逆引き（最も信頼性が高い）
ユーザーがそのプロジェクトに含まれる issue、epic、またはマイルストーンに言及している場合は、その issue の `projectItems` を問い合わせてプロジェクトを特定します。

```bash
gh api graphql -f query='{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      projectItems(first: 10) {
        nodes {
          id
          project { number title id }
        }
      }
    }
  }
}' --jq '.data.repository.issue.projectItems.nodes[] | {number: .project.number, title: .project.title, id: .project.id}'
```

これは、名前検索がうまく機能しない大規模 org で最も信頼できる方法です。

### 3. GraphQL の名前検索 + クライアント側フィルタ（フォールバック）
大きめのページを取得し、タイトル完全一致をクライアント側でフィルタします。

```bash
gh api graphql -f query='{
  organization(login: "ORG") {
    projectsV2(first: 100, query: "search term") {
      nodes { number title id }
    }
  }
}' --jq '.data.organization.projectsV2.nodes[] | select(.title | test("(?i)^exact name$"))'
```

これで何も返らない場合は、`after` カーソルでページネーションするか、正規表現を広げてください。結果は新しい順なので、古いプロジェクトはページネーションが必要です。

### 4. MCP ツール（小規模 org のみ）
`mcp__github__projects_list` を `method: "list_projects"` で呼び出します。50 未満のプロジェクトしかない org では有効ですが、名前フィルタがないため全結果を走査する必要があります。

## 進捗レポートのためのプロジェクト探索

ユーザーがプロジェクトの進捗更新（例: "Project X の進捗を教えて"）を求めたときは、次のワークフローに従ってください。

1. **プロジェクトを見つける** — 上記の [プロジェクト探索](#finding-a-project-by-name) 戦略を使います。名前検索が失敗した場合は、既知の issue 番号をユーザーに確認します。

2. **フィールドを把握する** - `projects_list` の `list_project_fields` を呼び出して Status フィールド（オプションがワークフロー段階を表す）と Iteration フィールド（現スプリントに絞り込むため）を確認します。

3. **全アイテムを取得する** - `projects_list` の `list_project_items` を呼び出します。大規模プロジェクト（100+ アイテム）の場合は、全ページをページネーションしてください。各アイテムにはフィールド値（status、iteration、assignees）が含まれます。

4. **レポートを作成する** - Status フィールド値ごとにアイテムをグループ化して件数を集計します。イテレーションベースのプロジェクトでは、先に現在のイテレーションに絞り込みます。次のような内訳で提示します。

   ```
   Project: Issue Fields (Iteration 42, Mar 2-8)
   15 actionable items:
     🎉 Done:        4 (27%)
     In Review:      3
     In Progress:    3
     Ready:          2
     Blocked:        2
   ```

5. **文脈を追加する** - アイテムに sub-issues がある場合は `subIssuesSummary` の件数を含めます。依存関係がある場合は、ブロックされているアイテムとそのブロッカーを明記します。

## OAuth スコープ要件

| Operation | Required scope |
|-----------|---------------|
| Read projects, fields, items | `read:project` |
| Add/update/delete items, change field values | `project` |

**よくある落とし穴:** 既定の `gh auth` トークンは `read:project` しか持たないことが多いです。ミューテーションは `INSUFFICIENT_SCOPES` で失敗します。書き込みスコープを追加するには:

```bash
gh auth refresh -h github.com -s project
```

これによりブラウザベースの OAuth フローが開始されます。ミューテーションを有効にするには完了が必要です。

## Issue の Project Item ID を見つける

issue は分かっているが project item ID（例: Status を更新するため）が必要な場合は、issue 側から問い合わせます。

```bash
gh api graphql -f query='
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      projectItems(first: 5) {
        nodes {
          id
          project { title number }
          fieldValues(first: 10) {
            nodes {
              ... on ProjectV2ItemFieldSingleSelectValue {
                name
                field { ... on ProjectV2SingleSelectField { name } }
              }
            }
          }
        }
      }
    }
  }
}' --jq '.data.repository.issue.projectItems.nodes'
```

この 1 クエリで、item ID、project 情報、現在のフィールド値を返せます。

## gh api 経由で GraphQL を使う（推奨）

GraphQL のクエリとミューテーションの実行には `gh api graphql` を使います。書き込み操作では MCP ツールより信頼性が高いです。

**プロジェクトと Status フィールドのオプションを取得:**
```bash
gh api graphql -f query='
{
  organization(login: "ORG") {
    projectV2(number: 5) {
      id
      title
      field(name: "Status") {
        ... on ProjectV2SingleSelectField {
          id
          options { id name }
        }
      }
    }
  }
}' --jq '.data.organization.projectV2'
```

**全フィールドを一覧取得（イテレーション含む）:**
```bash
gh api graphql -f query='
{
  node(id: "PROJECT_ID") {
    ... on ProjectV2 {
      fields(first: 20) {
        nodes {
          ... on ProjectV2Field { id name }
          ... on ProjectV2SingleSelectField { id name options { id name } }
          ... on ProjectV2IterationField { id name configuration { iterations { id startDate } } }
        }
      }
    }
  }
}' --jq '.data.node.fields.nodes'
```

**フィールド値を更新（例: Status を "In Progress" に設定）:**
```bash
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PROJECT_ID"
    itemId: "ITEM_ID"
    fieldId: "FIELD_ID"
    value: { singleSelectOptionId: "OPTION_ID" }
  }) {
    projectV2Item { id }
  }
}'
```

`value` には `text`、`number`、`date`、`singleSelectOptionId`、`iterationId` のいずれか 1 つを指定できます。

**アイテムを追加:**
```bash
gh api graphql -f query='
mutation {
  addProjectV2ItemById(input: {
    projectId: "PROJECT_ID"
    contentId: "ISSUE_OR_PR_NODE_ID"
  }) {
    item { id }
  }
}'
```

**アイテムを削除:**
```bash
gh api graphql -f query='
mutation {
  deleteProjectV2Item(input: {
    projectId: "PROJECT_ID"
    itemId: "ITEM_ID"
  }) {
    deletedItemId
  }
}'
```

## エンドツーエンド例: Issue の Status を "In Progress" に設定

```bash
# 1. Get the issue's project item ID, project ID, and current status
gh api graphql -f query='{
  repository(owner: "github", name: "planning-tracking") {
    issue(number: 2574) {
      projectItems(first: 1) {
        nodes { id project { id title } }
      }
    }
  }
}' --jq '.data.repository.issue.projectItems.nodes[0]'

# 2. Get the Status field ID and "In Progress" option ID
gh api graphql -f query='{
  node(id: "PROJECT_ID") {
    ... on ProjectV2 {
      field(name: "Status") {
        ... on ProjectV2SingleSelectField { id options { id name } }
      }
    }
  }
}' --jq '.data.node.field'

# 3. Update the status
gh api graphql -f query='mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PROJECT_ID"
    itemId: "ITEM_ID"
    fieldId: "FIELD_ID"
    value: { singleSelectOptionId: "IN_PROGRESS_OPTION_ID" }
  }) { projectV2Item { id } }
}'
```

