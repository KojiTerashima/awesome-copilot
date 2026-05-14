# MCP ブートストラップ — クイック リファレンス

エージェントが FlowStudio MCP サーバーの呼び出しを開始するために必要なものすべて。```
Endpoint:  https://mcp.flowstudio.app/mcp
Protocol:  JSON-RPC 2.0 over HTTP POST
Transport: Streamable HTTP — single POST per request, no SSE, no WebSocket
Auth:      x-api-key header with JWT token (NOT Bearer)
```## 必須のヘッダー```
Content-Type: application/json
x-api-key: <token>
User-Agent: FlowStudio-MCP/1.0    ← required, or Cloudflare blocks you
```## ステップ 1 — ツールの発見```json
POST {"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}
```すべてのツールを名前、説明、入力スキーマとともに返します。
無料 — プランの制限にはカウントされません。

## ステップ 2 — ツールを呼び出す```json
POST {"jsonrpc":"2.0","id":1,"method":"tools/call",
      "params":{"name":"<tool_name>","arguments":{...}}}
```## 応答形状```
Success → {"result":{"content":[{"type":"text","text":"<JSON string>"}]}}
Error   → {"result":{"content":[{"type":"text","text":"{\"error\":{...}}"}]}}
```実際のデータを取得するには、常に `result.content[0].text` を JSON として解析します。

## 重要なヒント

- ツールの結果はテキスト フィールド内の JSON 文字列です — **二重解析が必要です**
- 解析された本文の `"error"` フィールド: `null` = 成功、オブジェクト = 失敗
- `environmentName` はほとんどのツールで必須ですが、**必須ではありません**:
  `list_live_environments`、`list_live_connections`、`list_store_flows`、
  `list_store_environments`、`list_store_makers`、`get_store_maker`、
  `list_store_power_apps`、`list_store_connections`
- 疑わしい場合は、`tools/list` の各ツールのスキーマの `required` 配列を確認してください。