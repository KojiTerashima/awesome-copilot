# サブイシューと親イシュー

サブイシューを使うと、作業を階層的なタスクに分解できます。各親イシューには最大 100 件のサブイシューを設定でき、最大 8 階層までネストできます。サブイシューは同じオーナー配下であればリポジトリをまたいで設定できます。

## 推奨ワークフロー

サブイシューを作成する最も簡単な方法は、**2 ステップ**です。まずイシューを作成し、その後リンクします。

```bash
# ステップ 1: イシューを作成して数値 ID を取得
ISSUE_ID=$(gh api repos/{owner}/{repo}/issues \
  -X POST \
  -f title="Sub-task title" \
  -f body="Description" \
  --jq '.id')

# ステップ 2: 親のサブイシューとしてリンク
# 重要: sub_issue_id は整数である必要があります。JSON の送信には --input（-f ではなく）を使ってください。
echo "{\"sub_issue_id\": $ISSUE_ID}" | gh api repos/{owner}/{repo}/issues/{parent_number}/sub_issues -X POST --input -
```

**なぜ `-f` ではなく `--input` なのですか？** `gh api -f` フラグはすべての値を文字列として送信しますが、API は `sub_issue_id` に整数を要求します。`-f sub_issue_id=12345` を使うと 422 エラーが返ります。

または、GraphQL の `createIssue` に `parentIssueId` を指定して 1 ステップで実行することもできます（以下の GraphQL セクションを参照）。

## MCP ツールを使う場合

**サブイシューを一覧する:**
`mcp__github__issue_read` を `method: "get_sub_issues"`、`owner`、`repo`、`issue_number` とともに呼び出します。

**イシューをサブイシューとして作成する:**
サブイシューを直接作成するための MCP ツールはありません。上記ワークフローまたは GraphQL を使用してください。

## REST API を使う場合

**サブイシューを一覧する:**
```bash
gh api repos/{owner}/{repo}/issues/{issue_number}/sub_issues
```

**親イシューを取得する:**
```bash
gh api repos/{owner}/{repo}/issues/{issue_number}/parent
```

**既存のイシューをサブイシューとして追加する:**
```bash
# sub_issue_id は数値の issue ID（issue number ではない）です
# イシュー作成時または取得時の .id フィールドから取得します
echo '{"sub_issue_id": 12345}' | gh api repos/{owner}/{repo}/issues/{parent_number}/sub_issues -X POST --input -
```

すでに親を持つサブイシューを移動するには、JSON ボディに `"replace_parent": true` を追加します。

**サブイシューを削除する:**
```bash
echo '{"sub_issue_id": 12345}' | gh api repos/{owner}/{repo}/issues/{parent_number}/sub_issue -X DELETE --input -
```

**サブイシューの優先順位を変更する:**
```bash
echo '{"sub_issue_id": 6, "after_id": 5}' | gh api repos/{owner}/{repo}/issues/{parent_number}/sub_issues/priority -X PATCH --input -
```

`after_id` または `before_id` を使って、別のサブイシューとの相対位置を指定します。

## GraphQL を使う場合

**親とサブイシューを読み取る:**
```graphql
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      parent { number title }
      subIssues(first: 50) {
        nodes { number title state }
      }
      subIssuesSummary { total completed percentCompleted }
    }
  }
}
```

**サブイシューを追加する:**
```graphql
mutation {
  addSubIssue(input: {
    issueId: "PARENT_NODE_ID"
    subIssueId: "CHILD_NODE_ID"
  }) {
    issue { id }
    subIssue { id number title }
  }
}
```

`subIssueId` の代わりに `subIssueUrl`（イシューの HTML URL を渡す）も使用できます。別の親からサブイシューを移動するには `replaceParent: true` を追加します。

**イシューをサブイシューとして直接作成する:**
```graphql
mutation {
  createIssue(input: {
    repositoryId: "REPO_NODE_ID"
    title: "Implement login validation"
    parentIssueId: "PARENT_NODE_ID"
  }) {
    issue { id number }
  }
}
```

**サブイシューを削除する:**
```graphql
mutation {
  removeSubIssue(input: {
    issueId: "PARENT_NODE_ID"
    subIssueId: "CHILD_NODE_ID"
  }) {
    issue { id }
  }
}
```

**サブイシューの優先順位を変更する:**
```graphql
mutation {
  reprioritizeSubIssue(input: {
    issueId: "PARENT_NODE_ID"
    subIssueId: "CHILD_NODE_ID"
    afterId: "OTHER_CHILD_NODE_ID"
  }) {
    issue { id }
  }
}
```

`afterId` または `beforeId` を使って、別のサブイシューとの相対位置を指定します。

