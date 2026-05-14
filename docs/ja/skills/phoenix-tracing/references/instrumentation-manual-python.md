# 手動インストルメンテーション (Python)

デコレータまたはコンテキスト マネージャを使用してカスタム スパンを追加し、きめ細かいトレース制御を実現します。

＃＃ 設定「」バッシュ
pip インストール arise-phoenix-otel
「」

「」パイソン
phoenix.otelインポートレジスタから
Tracer_provider = register(project_name="my-app")
トレーサー = トレーサー_プロバイダー.get_tracer(__name__)
「」## クイックリファレンス

|スパンの種類 |デコレーター |使用例 |
|----------|-----------|----------|
|チェーン | `@tracer.chain` |オーケストレーション、ワークフロー、パイプライン |
|レトリバー | `@tracer.retriever` |ベクトル検索、文書検索 |
|ツール | `@tracer.tool` |外部 API 呼び出し、関数実行 |
|エージェント | `@tracer.agent` |多段階の推論、計画 |
| LLM | `@tracer.llm` | LLM API 呼び出し (手動のみ) |
|埋め込み | `@tracer.embedding` |埋め込み生成 |
|リランカー | `@tracer.reranker` |ドキュメントの再ランキング |
|ガードレール | `@tracer.guardrail` |安全性チェック、コンテンツ管理 |
|評価者 | `@tracer.evaluator` | LLM評価、品質チェック |

## デコレータアプローチ (推奨)

**用途:** 全機能計測、自動 I/O キャプチャ「」パイソン
@tracer.chain
def rag_pipeline(クエリ: str) -> str:
    docs =retrieve_documents(クエリ)
    ランク = 再ランク(ドキュメント、クエリ)
    returngenerate_response(ランク付けされた、クエリ)

@tracer.retriever
defretrieve_documents(クエリ: str) -> リスト[dict]:
    結果 = Vector_db.search(クエリ、top_k=5)
    return [{"content": doc.text, "score": doc.score} (結果のドキュメントの場合)]

@tracer.tool
def get_weather(都市: str) -> str:
    応答 = request.get(f"https://api.weather.com/{city}")
    return response.json()["天気"]
「」**カスタム スパン名:**「」パイソン
@tracer.chain(name="ラグパイプライン-v2")
def my_workflow(クエリ: str) -> str:
    返却処理(クエリ)
「」## コンテキストマネージャーのアプローチ

**用途:** 部分的な関数インストルメンテーション、カスタム属性、動的制御「」パイソン
opentelemetry.trace import Status、StatusCode から
jsonをインポートする

defretrieve_with_metadata(クエリ: str):
    Tracer.start_as_current_span( を使用)
        "ベクトル検索",
        openinference_span_kind="レトリバー"
    ) スパンとして:
        span.set_attribute("input.value", クエリ)

        結果 = Vector_db.search(クエリ、top_k=5)

        書類 = [
            {
                "ドキュメント.id": ドキュメント.id、
                "ドキュメント.コンテンツ": ドキュメント.テキスト、
                "ドキュメント.スコア": ドキュメント.スコア
            }
            結果のドキュメントの場合
        】
        span.set_attribute("retrieval.documents", json.dumps(documents))
        span.set_status(ステータス(ステータスコード.OK))

        書類を返送する
「」## 入力/出力のキャプチャ

**評価可能なスパンの I/O を常にキャプチャします。**

### 自動 I/O キャプチャ (デコレータ)

デコレータは入力引数を自動的に取得し、値を返します。```Python テーマ={null}
@tracer.chain
def handle_query(user_input: str) -> str:
    結果 = エージェント.生成(ユーザー入力)
    結果を返す.テキスト

# 自動的にキャプチャします:
# - input.value: user_input
# - 出力値: 結果テキスト
# - input.mime_type / Output.mime_type: 自動検出
「」### 手動 I/O キャプチャ (コンテキスト マネージャー)

単純な I/O キャプチャには `set_input()` と `set_output()` を使用します。```Python テーマ={null}
opentelemetry.trace import Status、StatusCode から

def handle_query(user_input: str) -> str:
    Tracer.start_as_current_span( を使用)
        "クエリ.ハンドラー",
        openinference_span_kind="チェーン"
    ) スパンとして:
        スパン.set_input(ユーザー入力)

        結果 = エージェント.生成(ユーザー入力)

        スパン.set_output(結果.テキスト)
        span.set_status(ステータス(ステータスコード.OK))

        結果を返す.テキスト
「」**何がキャプチャされるか:**```json
{
  "input.value": "2+2 とは何ですか?",
  "input.mime_type": "テキスト/プレーン",
  "output.value": "2+2 は 4 に等しい。",
  "output.mime_type": "テキスト/プレーン"
}
「」**これが重要な理由:**
- Phoenix 評価者には `input.value` と `output.value` が必要です
- Phoenix UI はデバッグ用に I/O を目立つように表示します
- データセットを微調整するためのデータのエクスポートを可能にします

### 追加のメタデータを使用したカスタム I/O

I/O と一緒にカスタム属性には `set_attribute()` を使用します。```Python テーマ={null}
def process_query(クエリ: str):
    Tracer.start_as_current_span( を使用)
        "クエリ.プロセス",
        openinference_span_kind="チェーン"
    ) スパンとして:
        # 標準 I/O
        スパン.set_input(クエリ)

        # カスタムメタデータ
        span.set_attribute("input.length", len(クエリ))

        結果 = llm.generate(クエリ)

        # 標準出力
        スパン.set_output(結果.テキスト)

        # カスタムメタデータ
        span.set_attribute("output.tokens", result.usage.total_tokens)
        span.set_status(ステータス(ステータスコード.OK))

        結果を返す
「」## 関連項目

- **スパン属性:** `span-chain.md`、`span-retriever.md`、`span-tool.md`、`span-llm.md`、`span-agent.md`、`span-embedding.md`、`span-reranker.md`、`span-guardrail.md`、`span-evaluator.md`
- **自動インスツルメンテーション:** `instrumentation-auto-python.md` (フレームワーク統合用)
- **API ドキュメント:** https://docs.arize.com/phoenix/tracing/manual-instrumentation