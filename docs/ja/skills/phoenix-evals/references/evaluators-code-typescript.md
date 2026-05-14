# エバリュエーター: TypeScript のコード エバリュエーター

LLM を使用しない決定論的評価器。速く、安く、再現可能。

## 基本パターン```タイプスクリプト
import { createEvaluator } から "@arizeai/phoenix-evals";

const containsCitation = createEvaluator<{ 出力: 文字列 }>(
  ({ 出力 }) => /\[\d+\]/.test(出力) ? 1：0、
  { 名前: "引用を含む"、種類: "コード" }
);
「」## 完全な結果 (ExperimentEvaluator として)```タイプスクリプト
import { asExperimentEvaluator } から "@arizeai/phoenix-client/experiments";

const jsonValid = asExperimentEvaluator({
  名前: "json_valid",
  種類: "コード"、
  評価: async ({ 出力 }) => {
    {を試してください
      JSON.parse(文字列(出力));
      return { スコア: 1.0、ラベル: "valid_json" };
    } キャッチ (e) {
      return { スコア: 0.0、ラベル: "invalid_json"、説明: String(e) };
    }
  }、
});
「」## パラメータのタイプ```タイプスクリプト
インターフェース EvaluatorParams {
  入力: レコード<文字列、不明>;
  出力: 不明;
  予期: Record<文字列、不明>;
  メタデータ: レコード<文字列、不明>;
}
「」## 一般的なパターン

- **正規表現**: `/pattern/.test(output)`
- **JSON**: `JSON.parse()` + zod スキーマ
- **キーワード**: `output.includes(keyword)`
- **類似性**: `fastest-levenshtein`