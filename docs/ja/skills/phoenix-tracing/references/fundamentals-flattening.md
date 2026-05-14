# フラット化規則

OpenInference は、データベース互換性、OpenTelemetry 互換性、および単純なクエリのために、ネストされたデータ構造をドット表記属性に平坦化します。

## フラット化ルール

**オブジェクト → ドット表記**```JavaScript
{ llm: { モデル名: "gpt-4"、トークン数: { プロンプト: 10、完了: 20 } } }
// になります
{ "llm.model_name": "gpt-4", "llm.token_count.prompt": 10, "llm.token_count.completion": 20 }
「」**配列 → ゼロインデックス表記**```JavaScript
{ llm: { input_messages: [{ 役割: "ユーザー"、コンテンツ: "こんにちは" }] } }
// になります
{ "llm.input_messages.0.message.role": "ユーザー"、"llm.input_messages.0.message.content": "こんにちは" }
「」**メッセージ規則: `.message.` セグメントが必要です**「」
llm.input_messages.{index}.message.{field}
llm.input_messages.0.message.tool_calls.0.tool_call.function.name
「」## 完全な例```JavaScript
// オリジナル
{
  openinference: { スパン: { 種類: "LLM" } }、
  llm: {
    モデル名: "クロード-3-5-ソネット-20241022",
    invocation_parameters: { 温度: 0.7、最大トークン: 1000 }、
    input_messages: [{ 役割: "ユーザー"、コンテンツ: "冗談を言ってください" }],
    Output_messages: [{ 役割: "アシスタント"、内容: "なぜニワトリは道路を渡ったのですか?" }]、
    token_count: { プロンプト: 5、完了: 10、合計: 15 }
  }
}

// フラット化 (Phoenixspans.attributes JSONB に保存)
{
  "openinference.span.kind": "LLM",
  "llm.model_name": "claude-3-5-sonnet-20241022",
  "llm.invocation_parameters": "{\"温度\": 0.7、\"max_tokens\": 1000}",
  "llm.input_messages.0.message.role": "ユーザー",
  "llm.input_messages.0.message.content": "冗談を言ってください",
  "llm.output_messages.0.message.role": "アシスタント",
  "llm.output_messages.0.message.content": "なぜ鶏は道路を渡ったのですか?",
  "llm.token_count.prompt": 5、
  "llm.token_count.completion": 10、
  "llm.token_count.total": 15
}
「」
