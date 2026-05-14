# よくある間違い (Python)

LLM がトレーニング データから頻繁に誤って生成するパターン。

## 従来のモデル クラス「」パイソン
# 間違っています
phoenix.evals から OpenAIModel、AnthropicModel をインポート
モデル = OpenAIModel(model="gpt-4")

# 右
phoenix.evals から LLM をインポート
llm = LLM(プロバイダー = "openai", モデル = "gpt-4o")
「」**理由**: `OpenAIModel`、`AnthropicModel` などは、`phoenix.evals.legacy` のレガシー 1.0 ラッパーです。
`LLM` クラスはプロバイダーに依存せず、現在の 2.0 API です。

## Evaluate_dataframe の代わりに run_evals を使用する「」パイソン
# 間違っています - レガシー 1.0 API
phoenix.evals から run_evals をインポート
results = run_evals(dataframe=df、evaluators=[eval1]、provide_explanation=True)
# DataFrame のリストを返す

# 右 - 現在の 2.0 API
phoenix.evalsからのインポートevaluate_dataframe
results_df = Evaluate_dataframe(dataframe=df, evaluators=[eval1])
# {name}_score dict 列を持つ単一の DataFrame を返します
「」**理由**: `run_evals` は従来の 1.0 バッチ関数です。 `evaluate_dataframe` は現在のものです
戻り形式が異なる 2.0 関数。

## 間違った結果の列名「」パイソン
# 間違っています — 列が存在しません
スコア = results_df["関連性"].mean()

# 間違っています — 列は存在しますが、数字ではなく辞書が含まれています
スコア = results_df["関連性_スコア"].mean()

# RIGHT — dict から数値スコアを抽出します
スコア = results_df["関連性_スコア"].apply(
    ラムダ x: x.get("スコア", 0.0) if isinstance(x, dict) else 0.0
）
スコア = スコア.平均()
「」**理由**: `evaluate_dataframe` は、スコア辞書を含む `{name}_score` という名前の列を返します。
`{"name": "...", "score": 1.0, "label": "...", "explanation": "..."}` のように。

## 非推奨の project_name パラメータ「」パイソン
# 間違っています
df = client.spans.get_spans_dataframe(project_name="my-project")

# 右
df = client.spans.get_spans_dataframe(project_identifier="my-project")
「」**理由**: `project_name` は非推奨となり、`project_identifier` が優先されます。
プロジェクト ID を受け入れます。

## 間違ったクライアント コンストラクター「」パイソン
# 間違っています
client = クライアント(エンドポイント="https://app.phoenix.arize.com")
client = クライアント(url="https://app.phoenix.arize.com")

# RIGHT — リモート/クラウド Phoenix 用
client = Client(base_url="https://app.phoenix.arize.com", api_key="...")

# ALSO RIGHT — ローカル Phoenix の場合 (env vars または localhost:6006 にフォールバック)
client = クライアント()
「」**理由**: パラメータは `endpoint` や `url` ではなく、`base_url` です。ローカル インスタンスの場合、
引数なしの `Client()` は正常に動作します。リモート インスタンスの場合は、`base_url` と `api_key` が必要です。

## 積極的すぎる時間フィルター「」パイソン
# 間違っています — 多くの場合ゼロ スパンを返します
from datetime import datetime、timedelta
df = client.spans.get_spans_dataframe(
    project_identifier="私のプロジェクト",
    start_time=datetime.now() - timedelta(時間=1)、
）

# RIGHT — 代わりに結果のサイズを制御するために制限を使用します
df = client.spans.get_spans_dataframe(
    project_identifier="私のプロジェクト",
    制限=50、
）
「」**理由**: トレースは任意の期間のものである可能性があります。 1 時間のウィンドウが頻繁に返されます
何もない。代わりに `limit=` を使用して結果のサイズを制御します。

## スパンを適切にフィルタリングしない「」パイソン
# WRONG — 内部 LLM 呼び出し、取得者などを含むすべてのスパンをフェッチします。
df = client.spans.get_spans_dataframe(project_identifier="my-project")

# エンドツーエンド評価の場合は RIGHT — 最上位のスパンにフィルタリングします
df = client.spans.get_spans_dataframe(
    project_identifier="私のプロジェクト",
    root_spans_only=真、
）

# RAG 評価の RIGHT — レトリーバー/LLM メトリクスの子スパンをフェッチします
all_spans = client.spans.get_spans_dataframe(
    project_identifier="私のプロジェクト",
）
retriever_spans = all_spans[all_spans["span_kind"] == "RETRIEVER"]
llm_spans = all_spans[all_spans["span_kind"] == "LLM"]
「」**理由**: エンドツーエンドの評価 (全体的な回答の品質など) には、`root_spans_only=True` を使用します。
RAG システムの場合、多くの場合、子スパンが個別に必要になります。つまり、RAG システムの取得スパンが必要になります。
忠実性の DocumentRelevance と LLM スパン。適切なスパンレベルを選択してください
あなたの評価対象に。

## スパン出力がプレーンテキストであると仮定します「」パイソン
# 誤り — 出力はプレーンテキストではなく JSON である可能性があります
df["出力"] = df["属性.出力.値"]

# RIGHT — JSON を解析し、回答フィールドを抽出します
jsonをインポートする

def extract_answer(output_value):
    そうでない場合は、ininstance(output_value, str):
        Output_value が None でない場合は str(output_value) を返します。それ以外は ""
    試してみてください:
        解析済み = json.loads(output_value)
        if isinstance(解析済み、辞書):
            キー入力の場合 (「回答」、「結果」、「出力」、「応答」):
                キー入力が解析された場合:
                    str(解析された[キー])を返します
    (json.JSONDecodeError、TypeError) を除く:
        パスする
    出力値を返す

df["output"] = df["attributes.output.value"].apply(extract_answer)
「」**理由**: LangChain やその他のフレームワークは、ルート スパンから構造化された JSON を出力することがよくあります。
`{"context": "...", "question": "...", "answer": "..."}` のように。評価者が必要とするのは
生の JSON ではなく、実際の回答テキストです。

## LLM ベースの評価に @create_evaluator を使用する「」パイソン
# 間違っています — @create_evaluator は LLM を呼び出しません
@create_evaluator(name="関連性", kind="llm")
デフォルトの関連性(入力: str、出力: str) -> str:
    pass # LLM は関係しません

# RIGHT — LLM ベースの評価には、ClassificationEvaluator を使用します
phoenix.evals より、ClassificationEvaluator、LLM をインポート

関連性 = 分類評価者(
    名前 = "関連性",
    prompt_template="これは関係ありますか?\n{{input}}\n{{output}}\n答え:",
    llm=LLM(プロバイダー="openai", モデル="gpt-4o"),
    選択肢={"関連性": 1.0, "無関係": 0.0},
）
「」**理由**: `@create_evaluator` はプレーンな Python 関数をラップしています。 `kind="llm"`の設定
これを LLM ベースとしてマークしますが、LLM 呼び出しを自分で実装する必要があります。
LLM ベースの評価の場合は、処理を行う `ClassificationEvaluator` を優先します。
LLM 呼び出し、構造化された出力解析、説明が自動的に行われます。

## ClassificationEvaluator の代わりに llm_classify を使用する「」パイソン
# 間違っています - レガシー 1.0 API
phoenix.evals からのインポート llm_classify
結果 = llm_classify(
    データフレーム=df、
    テンプレート=テンプレート_str、
    モデル=モデル、
    Rails=["関連", "無関係"],
）

# 右 - 現在の 2.0 API
phoenix.evalsからインポートClassificationEvaluator、async_evaluate_dataframe、LLM

分類子 = 分類評価者(
    名前 = "関連性",
    プロンプト_テンプレート=テンプレート_str、
    llm=LLM(プロバイダー="openai", モデル="gpt-4o"),
    選択肢={"関連性": 1.0, "無関係": 0.0},
）
results_df = await async_evaluate_dataframe(dataframe=df, evaluators=[分類子])
「」**理由**: `llm_classify` はレガシー 1.0 関数です。現在のパターンは、
`ClassificationEvaluator` でエバリュエーターを作成し、`async_evaluate_dataframe()` で実行します。

## HallucinationEvaluator の使用「」パイソン
# 間違っています — 非推奨です
phoenix.evals から HallucinationEvaluator をインポート
eval = HallucinationEvaluator(モデル)

# 右 — FaithhoodEvaluator を使用する
phoenix.evals.metrics からインポート FaithhoodEvaluator
phoenix.evals から LLM をインポート
eval = FaithhoodEvaluator(llm=LLM(provider="openai", model="gpt-4o"))
「」**理由**: `HallucinationEvaluator` は非推奨になりました。 `FaithfulnessEvaluator` はその置き換えです。
最大スコア (1.0 = 忠実) の「忠実」/「不忠実」ラベルを使用します。