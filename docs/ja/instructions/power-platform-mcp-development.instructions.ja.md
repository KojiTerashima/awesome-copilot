---
description: 'Microsoft Copilot Studio 向けに Model Context Protocol (MCP) 統合を行う Power Platform カスタム コネクタ開発の instruction'
applyTo: '**/*.{json,csx,md}'
---

# Power Platform MCP カスタム コネクタ開発

## 指示

### MCP プロトコル統合
- MCP 通信には常に JSON-RPC 2.0 標準を実装する
- Copilot Studio との互換性のために `x-ms-agentic-protocol: mcp-streamable-1.0` header を使う
- endpoint は、標準的な REST 操作と MCP tool invocation の両方をサポートするよう構成する
- response は Copilot Studio の制約 (reference type 不可、single type のみ) に従うよう変換する

### スキーマ設計のベスト プラクティス
- Copilot Studio は扱えないため、JSON schema から `$ref` やその他の reference type を取り除く
- schema 定義では type の配列ではなく単一 type を使う
- Copilot Studio 互換性のため、`anyOf` / `oneOf` 構造は単一 schema に平坦化する
- すべての tool input schema は、外部参照なしの自己完結型にする

### 認証とセキュリティ
- Power Platform の制約内で、MCP のセキュリティ best practice に従った OAuth 2.0 を実装する
- 柔軟な認証構成のため、connection parameter set を使う
- passthrough attack を防ぐため token audience を検証する
- 検証を強化するため MCP 固有の security header を追加する
- 複数の認証方式 (標準 OAuth、拡張 OAuth、API key fallback) をサポートする

### カスタム スクリプト実装
- custom script (`script.csx`) 内で JSON-RPC 変換を処理する
- JSON-RPC error response 形式に沿った適切なエラー処理を実装する
- 認証フローに token 検証と audience チェックを追加する
- Copilot Studio 互換のために MCP server response を変換する
- 動的なセキュリティ構成には connection parameter を使う

### Swagger 定義ガイドライン
- Power Platform 互換性のため Swagger 2.0 specification を使う
- 各 endpoint に適切な `operationId` を実装する
- 適切な type と description を持つ明確な parameter schema を定義する
- すべての success / error case に対して包括的な response schema を追加する
- 適切な HTTP status code と response header を含める

### リソースとツールの管理
- MCP resource は、Copilot Studio で tool output として消費できる構造にする
- resource content には適切な MIME type 宣言を行う
- Copilot Studio 統合を改善するため audience と priority annotation を追加する
- Copilot Studio の要件を満たすよう resource 変換を実装する

### Connection Parameter の構成
- OAuth version と security level の選択には enum dropdown を使う
- 明確な parameter description と制約を提供する
- 異なるデプロイ シナリオに対応するため複数の認証 parameter set をサポートする
- 可能な場合は validation rule と default value を含める
- connection parameter value を通じて動的構成を有効にする

### エラー処理とログ
- JSON-RPC 2.0 の error format に従った包括的な error response を実装する
- 認証、検証、変換の各ステップに詳細な logging を追加する
- トラブルシューティングに役立つ明確なエラー メッセージを提供する
- エラー条件に合わせた適切な HTTP status code を含める

### テストと検証
- 実際の MCP server 実装と接続して connector をテストする
- schema 変換が Copilot Studio で正しく動作することを確認する
- サポートするすべての parameter set で認証フローを検証する
- さまざまな障害シナリオに対する適切なエラー処理を確認する
- connection parameter 構成と動的な振る舞いをテストする

## 追加ガイドライン

### Power Platform 認定要件
- 包括的なドキュメント (`readme.md`、`CUSTOMIZE.md`) を含める
- 明確なセットアップ手順と構成手順を提供する
- すべての認証オプションとセキュリティ上の考慮点を文書化する
- 適切な publisher と stack owner 情報を含める
- Power Platform connector certification standards への準拠を確認する

### MCP Server 互換性
- 標準的な MCP server 実装との互換性を考慮して設計する
- `tools/list`、`tools/call`、`resources/list` などの一般的な MCP method をサポートする
- `mcp-streamable-1.0` protocol に対して streaming response を適切に処理する
- 適切な protocol negotiation と capability detection を実装する

### Copilot Studio 統合
- tool 定義が Copilot Studio の制約内で正しく動作することを確認する
- Copilot Studio interface からの resource access と tool invocation をテストする
- 変換後 schema が会話内で期待どおりの振る舞いをすることを検証する
- Copilot Studio の agent framework との適切な統合を確認する
