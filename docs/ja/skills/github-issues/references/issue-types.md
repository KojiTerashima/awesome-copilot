# Issue Types（高度な GraphQL）

Issue type（Bug、Feature、Task、Epic など）は**organization** レベルで定義され、リポジトリに継承されます。ラベルを超えて Issue を分類します。

基本的な使い方では、MCP ツールが issue type をネイティブに処理します。`mcp__github__list_issue_types` を呼び出して type を確認し、`mcp__github__create_issue` または `mcp__github__update_issue` に `type: "Bug"` を渡してください。このリファレンスでは高度な GraphQL 操作を扱います。

## GraphQL Feature Header

GraphQL の issue type 操作では、すべて `GraphQL-Features: issue_types` HTTP ヘッダーが必要です。

## type を一覧表示（org または repo レベル）

```graphql
# Header: GraphQL-Features: issue_types
{
  organization(login: "OWNER") {
    issueTypes(first: 20) {
      nodes { id name color description isEnabled }
    }
  }
}
```

type は `repository.issueTypes` でもリポジトリ単位で一覧取得でき、`repository.issueType(name: "Bug")` で名前指定の取得もできます。

## Issue の type を取得

```graphql
# Header: GraphQL-Features: issue_types
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: 123) {
      issueType { id name color }
    }
  }
}
```

## 既存の Issue に type を設定

```graphql
# Header: GraphQL-Features: issue_types
mutation {
  updateIssueIssueType(input: {
    issueId: "ISSUE_NODE_ID"
    issueTypeId: "IT_xxx"
  }) {
    issue { id issueType { name } }
  }
}
```

## type を指定して Issue を作成

```graphql
# Header: GraphQL-Features: issue_types
mutation {
  createIssue(input: {
    repositoryId: "REPO_NODE_ID"
    title: "Fix login bug"
    issueTypeId: "IT_xxx"
  }) {
    issue { id number issueType { name } }
  }
}
```

type をクリアするには、`issueTypeId` に `null` を設定します。

## 利用可能な色

`GRAY`, `BLUE`, `GREEN`, `YELLOW`, `ORANGE`, `RED`, `PINK`, `PURPLE`

