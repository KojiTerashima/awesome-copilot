# エバリュエーター: TypeScript の LLM エバリュエーター

LLM 評価者は、言語モデルを使用して出力を判断します。 Vercel AI SDKを使用します。

## クイックスタート```タイプスクリプト
import { createClassificationEvaluator } から "@arizeai/phoenix-evals";
import { openai } から "@ai-sdk/openai";

const helpness = await createClassificationEvaluator<{
  入力: 文字列;
  出力: 文字列;
}>({
  名前：「役に立つ」、
  モデル: openai("gpt-4o")、
  プロンプトテンプレート: `有用性を評価してください。
<質問>{{入力}}</質問>
<応答>{{出力}}</応答>
回答 (役に立った/役に立たなかった):`,
  選択肢: { 役に立たなかった: 0、役に立った: 1 }、
});
「」## テンプレート変数

XML タグを使用します: `<question>{{input}}</question>`、`<response>{{output}}</response>`、`<context>{{context}}</context>`

## asExperimentEvaluator を使用したカスタム エバリュエーター```タイプスクリプト
import { asExperimentEvaluator } から "@arizeai/phoenix-client/experiments";

const customEval = asExperimentEvaluator({
  名前: "カスタム"、
  種類: "LLM"、
  評価: async ({ 入力, 出力 }) => {
    // ここで LLM を呼び出します
    return { スコア: 1.0、ラベル: "合格"、説明: "..." };
  }、
});
「」## 事前に構築されたエバリュエーター```タイプスクリプト
import { createFaithhoodEvaluator } から "@arizeai/phoenix-evals";

constfaithfulEvaluator = createFaithhoodEvaluator({
  モデル: openai("gpt-4o")、
});
「」## ベストプラクティス

- 基準を具体的にする
- プロンプトに例を含めます
- 思考の連鎖には `<thinking>` を使用します