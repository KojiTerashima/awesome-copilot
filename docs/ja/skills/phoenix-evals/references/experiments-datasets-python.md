# 実験: Python のデータセット

評価データセットの作成と管理。

## データセットの作成「」パイソン
phoenix.clientインポートクライアントから

client = クライアント()

# 例から
データセット = client.datasets.create_dataset(
    名前 = "qa-テスト-v1",
    例=[
        {
            "input": {"question": "2+2 とは何ですか?"},
            "出力": {"答え": "4"},
            "メタデータ": {"カテゴリ": "数学"},
        }、
    ]、
）

# データフレームから
データセット = client.datasets.create_dataset(
    データフレーム=df、
    名前 = "qa-テスト-v1",
    input_keys=["質問"],
    Output_keys=["答え"],
    metadata_keys=["カテゴリ"]、
）
「」## 本番環境のトレースから「」パイソン
spans_df = client.spans.get_spans_dataframe(project_identifier="my-app")

データセット = client.datasets.create_dataset(
    dataframe=spans_df[["input.value", "output.value"]],
    name="本番サンプルv1",
    input_keys=["input.value"],
    Output_keys=["output.value"],
）
「」## データセットの取得「」パイソン
データセット = client.datasets.get_dataset(name="qa-test-v1")
df = dataset.to_dataframe()
「」## 主要なパラメータ

|パラメータ |説明 |
| --------- | ----------- |
| `input_keys` |タスク入力用の列 |
| `output_keys` |予想される出力の列 |
| `metadata_keys` |追加のコンテキスト |

## 実験でのエバリュエーターの使用

### 実験評価者としての評価者

phoenix-evals エバリュエーターを `evaluators` 引数として `run_experiment` に直接渡します。「」パイソン
functools から部分的なインポートを行う
phoenix.clientからAsyncClientをインポート
phoenix.evals からのインポート、ClassificationEvaluator、LLM、bind_evaluator

# LLM エバリュエーターを定義する
拒否 = 分類評価者(
    名前=「拒否」、
    prompt_template="これは拒否ですか?\n質問: {{query}}\n応答: {{response}}",
    llm=LLM(プロバイダー="openai", モデル="gpt-4o"),
    選択肢={"拒否": 0, "回答": 1},
）

# データセット列を評価パラメータにマップするためにバインドします
拒否_評価 = バインド_評価(拒否, {"クエリ": "入力クエリ", "応答": "出力"})

# 実験タスクを定義する
async def run_rag_task(input, rag_engine):
    return rag_engine.query(input["query"])

# エバリュエーターを使用して実験を実行する
実験 = await AsyncClient().experiments.run_experiment(
    データセット=ds、
    task=partial(run_rag_task, rag_engine=query_engine),
    実験名 = "ベースライン",
    評価者=[拒否評価者]、
    同時実行数=10、
）
「」### タスクとしての評価者 (メタ評価)

LLM エバリュエーターを実験 **タスク**として使用して、エバリュエーター自体をテストします
人間による注釈に対して:「」パイソン
phoenix.evals からインポート create_evaluator

# エバリュエーターはテストされるタスクです
def run_refusal_eval(入力、評価者):
    結果 = 評価者.評価(入力)
    結果を返す[0]

# 単純なヒューリスティック チェック ジャッジ vs 人間の合意
@create_evaluator(name="exact_match")
def strict_match(出力、期待値):
    return float(output["score"]) == float(expected["refusal_score"])

# 実行: evaluator がタスクであり、exact_match がそれを評価します
実験 = await AsyncClient().experiments.run_experiment(
    データセット=注釈付きデータセット、
    task=partial(run_refusal_eval, evaluator=refusal),
    実験名 = "審査員-v1",
    評価者=[完全一致]、
    同時実行数=10、
）
「」このパターンを使用すると、人間の判断と一致するまで評価者のプロンプトを繰り返すことができます。
完全な動作例については、`tutorials/evals/evals-2/evals_2.0_rag_demo.ipynb` を参照してください。

## ベストプラクティス

- **バージョン管理**: 新しいデータセットを作成します (例: `qa-test-v2`)。変更しないでください。
- **メタデータ**: トラックソース、カテゴリ、難易度
- **バランス**: カテゴリ全体で多様なカバレッジを確保します