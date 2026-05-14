# 実験: Python で実験を実行する

`run_experiment` を使用して実験を実行します。

## 基本的な使い方「」パイソン
phoenix.clientインポートクライアントから
phoenix.client.experiments から run_experiment をインポート

client = クライアント()
データセット = client.datasets.get_dataset(name="qa-test-v1")

def my_task(例):
    return call_llm(example.input["question"])

def strict_match(出力、期待値):
    Output.strip(). lower() == Expected["answer"].strip(). lower() の場合は 1.0 を返し、それ以外の場合は 0.0

実験 = run_experiment(
    データセット=データセット、
    task=my_task、
    評価者=[完全一致]、
    実験名 = "qa-実験-v1",
）
「」## タスク関数「」パイソン
# 基本的なタスク
デフォルトタスク(例):
    return call_llm(example.input["question"])

# コンテキストあり (RAG)
def rag_task(例):
    return call_llm(f"コンテキスト: {example.input['context']}\nQ: {example.input['question']}")
「」## 評価パラメータ

|パラメータ |アクセス |
| --------- | ------ |
| `output` |タスクの出力 |
| `expected` |予想される出力の例 |
| `input` |入力例 |
| `metadata` |メタデータの例 |

## オプション「」パイソン
実験 = run_experiment(
    データセット=データセット、
    task=my_task、
    評価者=評価者、
    実験名 = "私の実験",
    dry_run=3, # 3 つの例でテストする
    repetitions=3, # 各例を 3 回実行します
）
「」＃＃ 結果「」パイソン
print(実験.aggregate_scores)
# {'精度': 0.85, '忠実さ': 0.92}

Experiment.runs で実行する場合:
    print(run.output, run.scores)
「」## 後で評価を追加「」パイソン
phoenix.client.experimentsからのインポートevaluate_experiment

Evaluate_experiment(experiment=実験、evaluators=[new_evaluator])
「」
