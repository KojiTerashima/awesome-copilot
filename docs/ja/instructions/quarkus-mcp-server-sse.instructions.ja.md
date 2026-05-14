---
applyTo: '*'
description: 'HTTP SSE transport を使う Quarkus と MCP Server の開発標準および instruction'
---
# Quarkus MCP Server

Java 21、Quarkus、HTTP SSE transport で MCP サーバーを構築します。

## スタック

- Java 21 と Quarkus Framework
- MCP Server Extension: `mcp-server-sse`
- CDI による dependency injection
- MCP Endpoint: `http://localhost:8080/mcp/sse`

## クイック スタート

```bash
quarkus create app --no-code -x rest-client-jackson,qute,mcp-server-sse your-domain-mcp-server
```

## 構成

- 標準的な Java 命名規則を使う (PascalCase classes、camelCase methods)
- package は `model`、`repository`、`service`、`mcp` に整理する
- 不変データ モデルには Record type を使う
- 不変データの状態管理は repository layer で行う
- public method には Javadoc を追加する

## MCP Tools

- `@ApplicationScoped` CDI bean 内の public method にする
- `@Tool(name="tool_name", description="clear description")` を使う
- `null` は返さず、代わりにエラー メッセージを返す
- 常にパラメーターを検証し、エラーを丁寧に処理する

## アーキテクチャ

- 関心を分離する: MCP tools → Service layer → Repository
- dependency injection には `@Inject` を使う
- データ操作は thread-safe にする
- null pointer exception を避けるため `Optional<T>` を使う

## よくある問題

- MCP tools に business logic を置かない (service layer を使う)
- tools から例外を投げない (エラー文字列を返す)
- 入力パラメーターの検証を忘れない
- edge case (`null`、空入力) でテストする
