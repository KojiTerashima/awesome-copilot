# 実験: TypeScript で実験を実行する

`runExperiment` を使用して実験を実行します。

## 基本的な使い方```タイプスクリプト
import { createClient } から "@arizeai/phoenix-client";
インポート {
  実験を実行し、
  実験評価者として、
"@arizeai/phoenix-client/experiments" から;

const client = createClient();

const task = async (例: { input: Record<文字列, 不明> }) => {
  return await callLLM(example.input.question as string);
};

const strictMatch = asExperimentEvaluator({
  名前: "完全一致",
  種類: "コード"、
  評価: async ({ 出力、期待値 }) => ({
    スコア: 出力 === 期待されますか?.答え ? 1.0 : 0.0、
    ラベル: 出力 === 期待されますか?.回答 ? "一致" : "一致なし",
  })、
});

const 実験 = await runExperiment({
  クライアント、
  実験名: "qa-実験-v1",
  データセット: { datasetId: "your-dataset-id" },
  タスク、
  評価者: [完全一致]、
});
「」## タスク関数```タイプスクリプト
// 基本的なタスク
const task = async (example) => await callLLM(example.input.question as string);

// コンテキストあり (RAG)
const ragTask = async (例) => {
  const プロンプト = `Context: ${example.input.context}\nQ: ${example.input.question}`;
  return await callLLM(プロンプト);
};
「」## 評価パラメータ```タイプスクリプト
インターフェース EvaluatorParams {
  入力: レコード<文字列、不明>;
  出力: 不明;
  予期: Record<文字列、不明>;
  メタデータ: レコード<文字列、不明>;
}
「」## オプション```タイプスクリプト
const 実験 = await runExperiment({
  クライアント、
  実験名: "私の実験",
  データセット: { データセット名: "qa-test-v1" },
  タスク、
  評価者、
  repetitions: 3, // 各例を 3 回実行します
  maxConcurrency: 5, // 同時実行を制限します
});
「」## 後で評価を追加```タイプスクリプト
import {evaluateExperiment} から "@arizeai/phoenix-client/experiments";

awaitestimateExperiment({ クライアント、実験、評価者: [newEvaluator] });
「」
