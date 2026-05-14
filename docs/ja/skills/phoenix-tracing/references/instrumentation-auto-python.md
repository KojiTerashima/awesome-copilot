# Phoenix トレーシング: 自動インストルメンテーション (Python)

**コードを変更せずに、LLM 呼び出しのスパンを自動的に作成します。**

## 概要

自動インスツルメンテーション パッチは、実行時にスパンを自動的に作成するライブラリをサポートしました。サポートされているフレームワーク (LangChain、LlamaIndex、OpenAI SDK など) に使用します。カスタム ロジックの場合は、manual-instrumentation-python.md。

## サポートされているフレームワーク

**パイソン:**

- LLM SDK: OpenAI、Anthropic、Bedrock、Mistral、Vertex AI、Groq、Ollama
- フレームワーク: LangChain、LlamaIndex、DSPy、CrewAI、Instructor、Haystack
- インストール: `pip install openinference-instrumentation-{name}`

## セットアップ

**インストールして有効にします:**「」バッシュ
pip インストール arise-phoenix-otel
pip install openinference-instrumentation-openai # 必要に応じて他を追加
「」

「」パイソン
phoenix.otelインポートレジスタから

register(project_name="my-app", auto_instrument=True) # インストールされているすべてのインストルメンタを検出します
「」**例：**「」パイソン
phoenix.otelインポートレジスタから
openaiインポートからOpenAI

register(project_name="my-app", auto_instrument=True)

クライアント = OpenAI()
応答 = client.chat.completions.create(
    モデル = "gpt-4"、
    メッセージ=[{"役割": "ユーザー", "コンテンツ": "こんにちは!"}]
）
「」トレースは、自動的にキャプチャされたモデル、入力/出力、トークン、タイミングとともに Phoenix UI に表示されます。完全な属性スキーマについては、スパン種類ファイルを参照してください。

**選択的インスツルメンテーション** (明示的制御):「」パイソン
phoenix.otelインポートレジスタから
openinference.instrumentation.openai からインポート OpenAIInstrumentor

tracer_provider = register(project_name="my-app") # auto_instrument はありません
OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)
「」## 制限事項

自動インスツルメンテーションは次のものをキャプチャしません。

- カスタム ビジネス ロジック
- 内部関数呼び出し

**例:**「」パイソン
def my_custom_workflow(クエリ: str) -> str:
    preprocessed = preprocess(query) # トレースされません
    response = client.chat.completions.create(...) # トレース済み (自動)
    postprocessed = postprocess(response) # トレースされません
    後処理して返す
「」**解決策:** 手動インストルメンテーションを追加します。「」パイソン
@tracer.chain
def my_custom_workflow(クエリ: str) -> str:
    前処理 = 前処理(クエリ)
    応答 = client.chat.completions.create(...)
    後処理 = 後処理(応答)
    後処理して返す
「」
