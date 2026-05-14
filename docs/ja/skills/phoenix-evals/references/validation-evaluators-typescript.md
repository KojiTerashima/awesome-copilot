# エバリュエーターの検証 (TypeScript)

LLM エバリュエーターをデプロイする前に、人間がラベルを付けたサンプルに対して検証します。
目標: **>80% TPR および >80% TNR**。

通常のタスク実験と比べて役割が逆転します。

|通常の実験 |評価者の検証 |
|---|---|
|タスク = エージェント ロジック |タスク = テスト中のエバリュエーターを実行します |
|評価者 = 出力を判断する |評価者 = 完全一致 vs 人間のグラウンド トゥルース |
|データセット = エージェントの例 |データセット = 手動でラベル付けされた黄金の例 |

## ゴールデン データセット

検証実験が Phoenix でのタスク実験と混合しないように、別のデータセット名を使用します。
人間のグラウンド トゥルースを `metadata.groundTruthLabel` に保存します。 ~50/50 のバランスを目指します:```タイプスクリプト
import type { Example } from "@arizeai/phoenix-client/types/datasets";

const gold例: 例[] = [
  { input: { q: 「フランスの首都？」 }、出力: { 回答: "パリ" }、メタデータ: { groundTruthLabel: "正しい" } }、
  { input: { q: 「フランスの首都？」 }、出力: { 回答: "リヨン" }、メタデータ: { groundTruthLabel: "不正" } }、
  { input: { q: 「フランスの首都？」 }、出力: { 回答: "主要都市..." }、メタデータ: { groundTruthLabel: "不正" } }、
];

const VALIDATOR_DATASET = "my-app-qa-evaluator-validation"; // タスク データセットから分離する
const POSITIVE_LABEL = "正しい";
const NEGATIVE_LABEL = "間違っています";
「」## 検証実験```タイプスクリプト
import { createClient } から "@arizeai/phoenix-client";
import { createOrGetDataset, getDatasetExamples } from "@arizeai/phoenix-client/datasets";
import { asExperimentEvaluator, runExperiment } from "@arizeai/phoenix-client/experiments";
import { myEvaluator } から "./myEvaluator.js";

const client = createClient();

const { datasetId } = await createOrGetDataset({ クライアント、名前: VALIDATOR_DATASET、例: goldExamples });
const { 例 } = await getDatasetExamples({ client, dataset: { datasetId } });
const groundTruth = new Map(examples.map((ex) => [ex.id, ex.metadata?.groundTruthLabel as string]));

// タスク: テスト対象のエバリュエーターを呼び出します
const task = async (例: (例の種類)[数値]) => {
  const result = await myEvaluator.evaluate({ 入力: example.input, 出力: example.output, メタデータ: example.metadata });
  result.labelを返す ?? "未知";
};

// 評価者: 人間のグラウンド トゥルースとの完全一致
const strictMatch = asExperimentEvaluator({
  名前: "完全一致"、種類: "CODE"、
  評価: ({ 出力, メタデータ }) => {
    const Expected = 文字列としてのメタデータ?.groundTruthLabel;
    const 予測 = 出力のタイプ === "文字列" ?出力: "不明";
    return { スコア: 予測 === 期待される ? 1 : 0、ラベル: 予測、説明: `Expected: ${expected}, Got: ${predicted}` };
  }、
});

const 実験 = await runExperiment({
  クライアント、実験名: `evaluator-validation-${Date.now()}`、
  データセット: { datasetId }、タスク、評価子: [exactMatch]、
});

// 混同行列を計算する
const 実行 = Object.values(experiment.runs);
const予測 = new Map((experiment.evaluationRuns ?? [])
  .filter((e) => e.name === "完全一致")
  .map((e) => [e.experimentRunId, e.result?.label ?? null]));

tp = 0、fp = 0、tn = 0、fn = 0 とします。
for (実行の定数実行) {
  if (run.error) 続行;
  const p =predicted.get(run.id), a = groundTruth.get(run.datasetExampleId);
  if (!p || !a) 続行;
  if (a === POSITIVE_LABEL && p === POSITIVE_LABEL) tp++;
  else if (a === NEGATIVE_LABEL && p === POSITIVE_LABEL) fp++;
  else if (a === NEGATIVE_LABEL && p === NEGATIVE_LABEL) tn++;
  else if (a === POSITIVE_LABEL && p === NEGATIVE_LABEL) fn++;
}
const 合計 = tp + fp + tn + fn;
const tpr = tp + fn > 0 ? (tp / (tp + fn)) * 100 : 0;
const tnr = tn + fp > 0 ? (tn / (tn + fp)) * 100 : 0;
console.log(`TPR: ${tpr.toFixed(1)}%  TNR: ${tnr.toFixed(1)}%  Accuracy: ${((tp + tn) / total * 100).toFixed(1)}%`);
「」## 結果と品質ルール

|メトリック |ターゲット |低い値は | を意味します。
|---|---|---|
| TPR（感度） | >80% |本当の失敗 (偽陰性) を見逃す |
| TNR (特異性) | >80% |正常な出力にフラグを立てます (偽陽性)。
|精度 | >80% |一般的な弱点 |

**データセットのゴールデン ルール:** ~50/50 バランス · エッジ ケースを含む · 人間によるラベルのみ · 変異しない (新しいバージョンを追加する) · 20 ～ 50 個のサンプルで十分。

**次の場合に再検証します。** テンプレートの変更を促す、モデルの変更を判断する、基準を更新する、実稼働 FP/FN のスパイク。

## 関連項目

- `validation.md` — メトリクスの定義と概念
- `experiments-running-typescript.md` — `runExperiment` API
- `experiments-datasets-typescript.md` — `createOrGetDataset` / `getDatasetExamples`