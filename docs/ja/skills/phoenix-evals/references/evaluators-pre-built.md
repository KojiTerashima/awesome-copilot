# エバリュエーター: 事前構築済み

探索のみに使用してください。本番前に検証します。

## パイソン「」パイソン
phoenix.evals から LLM をインポート
phoenix.evals.metrics からインポート FaithhoodEvaluator

llm = LLM(プロバイダー = "openai", モデル = "gpt-4o")
忠実度_eval = 忠実度評価者(llm=llm)
「」**注意**: `HallucinationEvaluator` は非推奨です。代わりに `FaithfulnessEvaluator` を使用してください。
スコア 1.0 = 忠実な「忠実」/「不忠実」ラベルを使用します。

## TypeScript```タイプスクリプト
import { createHallucinationEvaluator } から "@arizeai/phoenix-evals";
import { openai } から "@ai-sdk/openai";

consthallucinationEval = createHallucinationEvaluator({モデル:openai("gpt-4o") });
「」## 利用可能 (2.0)

|評価者 |タイプ |説明 |
| --------- | ---- | ----------- |
| `FaithfulnessEvaluator` | LLM |応答はコンテキストに忠実ですか? |
| `CorrectnessEvaluator` | LLM |対応は正しいでしょうか？ |
| `DocumentRelevanceEvaluator` | LLM |取得した文書は関連性がありますか? |
| `ToolSelectionEvaluator` | LLM |エージェントは正しいツールを選択しましたか? |
| `ToolInvocationEvaluator` | LLM |エージェントはツールを正しく起動しましたか? |
| `ToolResponseHandlingEvaluator` | LLM |エージェントはツールの応答に適切に対応しましたか? |
| `MatchesRegex` |コード |出力は正規表現パターンと一致しますか? |
| `PrecisionRecallFScore` |コード |精度/再現率/F スコアのメトリクス |
| `exact_match` |コード |文字列の完全一致 |

従来のエバリュエーター (`HallucinationEvaluator`、`QAEvaluator`、`RelevanceEvaluator`、
`ToxicityEvaluator`、`SummarizationEvaluator`) は `phoenix.evals.legacy` に含まれており、非推奨です。

## いつ使用するか

|状況 |推薦 |
| --------- | -------------- |
|探検 |確認するトレースを検索 |
|外れ値を見つける |スコア順に並べ替え |
|制作 |最初に検証します (人間の同意が 80% 以上) |
|ドメイン固有 |カスタムビルド |

## 探索パターン「」パイソン
phoenix.evalsからのインポートevaluate_dataframe

results_df =evaluate_dataframe(dataframe=トレース、評価者=[忠実度_eval])

# スコア列には辞書が含まれています - 数値スコアを抽出します
スコア = results_df["忠実度_スコア"].apply(
    ラムダ x: x.get("スコア", 0.0) if isinstance(x, dict) else 0.0
）
low_scores = results_df[scores < 0.5] # これらを確認してください
high_scores = results_df[scores > 0.9] # サンプルも
「」## 検証が必要です「」パイソン
sklearn.metricsインポートclassification_reportから

print(classification_report(human_labels, evaluator_results["label"]))
# 目標: >80% の同意
「」
