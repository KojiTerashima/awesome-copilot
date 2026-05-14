# LLM スパン

言語モデル (OpenAI、Anthropic、ローカル モデルなど) への呼び出しを表します。

## 必須の属性

|属性 |タイプ |説明 |
|----------|------|---------------|
| `openinference.span.kind` |文字列 | 「LLM」である必要があります |
| `llm.model_name` |文字列 |モデル識別子 (例: "gpt-4"、"claude-3-5-sonnet-20241022") |

## 主要な属性

|カテゴリー |属性 |例 |
|----------|-----------|----------|
| **モデル** | `llm.model_name`、`llm.provider` | "gpt-4-turbo"、"openai" |
| **トークン** | `llm.token_count.prompt`、`llm.token_count.completion`、`llm.token_count.total` | 25、8、33 |
| **コスト** | `llm.cost.prompt`、`llm.cost.completion`、`llm.cost.total` | 0.0021、0.0045、0.0066 |
| **パラメータ** | `llm.invocation_parameters` (JSON) | `{"temperature": 0.7, "max_tokens": 1024}` |
| **メッセージ** | `llm.input_messages.{i}.*`、`llm.output_messages.{i}.*` |以下の例を参照してください |
| **ツール** | `llm.tools.{i}.tool.json_schema` |関数の定義 |

## コストの追跡

**コア属性:**
- `llm.cost.prompt` - 総投入コスト (USD)
- `llm.cost.completion` - 総出力コスト (USD)
- `llm.cost.total` - 総コスト (USD)

**詳細な費用の内訳:**
- `llm.cost.prompt_details.{input,cache_read,cache_write,audio}` - 入力コストコンポーネント
- `llm.cost.completion_details.{output,reasoning,audio}` - コストコンポーネントを出力します

## メッセージ

**入力メッセージ:**
- `llm.input_messages.{i}.message.role` - 「ユーザー」、「アシスタント」、「システム」、「ツール」
- `llm.input_messages.{i}.message.content` - テキストの内容
- `llm.input_messages.{i}.message.contents.{j}` - マルチモーダル (テキスト + 画像)
- `llm.input_messages.{i}.message.tool_calls` - ツールの呼び出し

**出力メッセージ:** 入力メッセージと同じ構造。

## 例: 基本的な LLM 呼び出し```json
{
  "openinference.span.kind": "LLM",
  "llm.model_name": "claude-3-5-sonnet-20241022",
  "llm.invocation_parameters": "{\"温度\": 0.7、\"max_tokens\": 1024}",
  "llm.input_messages.0.message.role": "システム",
  "llm.input_messages.0.message.content": "あなたは役に立つアシスタントです。",
  "llm.input_messages.1.message.role": "ユーザー",
  "llm.input_messages.1.message.content": "フランスの首都はどこですか?",
  "llm.output_messages.0.message.role": "アシスタント",
  "llm.output_messages.0.message.content": "フランスの首都はパリです。",
  "llm.token_count.prompt": 25、
  "llm.token_count.completion": 8、
  "llm.token_count.total": 33
}
「」## 例: ツール呼び出しを使用した LLM```json
{
  "openinference.span.kind": "LLM",
  "llm.model_name": "gpt-4-turbo",
  "llm.input_messages.0.message.content": "サンフランシスコの天気は?",
  "llm.output_messages.0.message.tool_calls.0.tool_call.function.name": "get_weather",
  "llm.output_messages.0.message.tool_calls.0.tool_call.function.arguments": "{\"location\": \"San Francisco\"}",
  "llm.tools.0.tool.json_schema": "{\"type\": \"function\", \"function\": {\"name\": \"get_weather\"}}"
}
「」## 関連項目

- **計測:** `instrumentation-auto-python.md`、`instrumentation-manual-python.md`
- **完全な仕様:** https://github.com/Arize-ai/openinference/blob/main/spec/semantic_conventions.md