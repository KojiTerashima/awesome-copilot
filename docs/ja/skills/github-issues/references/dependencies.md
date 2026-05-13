# Issue Dependencies（Blocked By / Blocking）

依存関係を使うと、ある issue が別の issue によってブロックされていることをマークできます。これにより、UI で確認でき、API でも追跡可能な正式な依存関係が作られます。依存関係向けの MCP ツールは存在しないため、REST または GraphQL を直接使用してください。

## REST API を使う

**この issue をブロックしている issue を一覧表示:**
```
GET /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by
```

**ブロッキング依存関係を追加:**
```
POST /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by
Body: { "issue_id": 12345 }
```

`issue_id` は数値の issue **ID**（issue number ではない）です。

**ブロッキング依存関係を削除:**
```
DELETE /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by/{issue_id}
```

## GraphQL を使う

**依存関係を読み取る:**
```graphql
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      blockedBy(first: 10) { nodes { number title state } }
      blocking(first: 10) { nodes { number title state } }
      issueDependenciesSummary { blockedBy blocking totalBlockedBy totalBlocking }
    }
  }
}
```

**依存関係を追加:**
```graphql
mutation {
  addBlockedBy(input: {
    issueId: "BLOCKED_ISSUE_NODE_ID"
    blockingIssueId: "BLOCKING_ISSUE_NODE_ID"
  }) {
    blockingIssue { number title }
  }
}
```

**依存関係を削除:**
```graphql
mutation {
  removeBlockedBy(input: {
    issueId: "BLOCKED_ISSUE_NODE_ID"
    blockingIssueId: "BLOCKING_ISSUE_NODE_ID"
  }) {
    blockingIssue { number title }
  }
}
```

## 追跡される issue（読み取り専用）

タスクリストの追跡関係は、GraphQL で読み取り専用フィールドとして利用できます。

- `trackedIssues(first: N)` - この issue のタスクリストで追跡されている issue
- `trackedInIssues(first: N)` - タスクリスト内でこの issue を参照している issue

これらは、issue がタスクリスト（`- [ ] #123`）で参照されたときに自動的に設定されます。管理用の mutation はありません。

