# 手動インストルメンテーション (TypeScript)

コンビニエンス ラッパーまたは withSpan を使用してカスタム スパンを追加し、きめ細かいトレース制御を実現します。

＃＃ 設定「」バッシュ
npm install @arizeai/phoenix-otel @arizeai/openinference-core
「」

```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
register({ プロジェクト名: "my-app" });
「」## クイックリファレンス

|スパンの種類 |方法 |使用例 |
|----------|----------|----------|
|チェーン | `traceChain` |ワークフロー、パイプライン、オーケストレーション |
|エージェント | `traceAgent` |多段階の推論、計画 |
|ツール | `traceTool` |外部 API、関数呼び出し |
|レトリバー | `withSpan` |ベクトル検索、文書検索 |
| LLM | `withSpan` | LLM API 呼び出し (自動インストルメンテーションを優先) |
|埋め込み | `withSpan` |埋め込み生成 |
|リランカー | `withSpan` |ドキュメントの再ランキング |
|ガードレール | `withSpan` |安全性チェック、コンテンツ管理 |
|評価者 | `withSpan` | LLM評価 |

## 便利なラッパー```タイプスクリプト
import {traceChain、traceAgent、traceTool } from "@arizeai/openinference-core";

// CHAIN - ワークフロー
const パイプライン =traceChain(
  async (クエリ: 文字列) => {
    const docs = await 取得(クエリ);
    return await generated(docs, query);
  }、
  { 名前: "ラグパイプライン" }
);

// エージェント - 推論
const エージェント = トレースエージェント(
  async (質問: 文字列) => {
    const thought = await llm.generate(`Think: ${question}`);
    return await processThought(思考);
  }、
  { 名前: "私のエージェント" }
);

// TOOL - 関数呼び出し
const getWeather = トレースツール(
  async (city: string) => fetch(`/api/weather/${city}`).then(r => r.json()),
  { 名前: "天気予報" }
);
「」## 他の種類の withSpan```タイプスクリプト
import { withSpan, getInputAttributes, getRetrieverAttributes } from "@arizeai/openinference-core";

// カスタム属性を持つ RETRIEVER
const 取得 = withSpan(
  async (クエリ: 文字列) => {
    const results = await VectorDb.search(query, { topK: 5 });
    return results.map(doc => ({ コンテンツ: doc.text, スコア: doc.score }));
  }、
  {
    種類：「レトリバー」、
    名前: "ベクトル検索"、
    processInput: (クエリ) => getInputAttributes(クエリ)、
    processOutput: (docs) => getRetrieverAttributes({ ドキュメント: docs })
  }
);
「」**オプション:**```タイプスクリプト
withSpan(fn, {
  kind: "RETRIEVER", // OpenInference スパンの種類
  name: "span-name", // スパン名 (デフォルトは関数名)
  processInput: (args) => {}, // 入力を属性に変換します
  processOutput: (result) => {}, // 出力を属性に変換します
  属性: { key: "value" } // 静的属性
});
「」## 入力/出力のキャプチャ

**評価可能なスパンの I/O を常にキャプチャします。** 自動 MIME タイプ検出には `getInputAttributes` ヘルパーと `getOutputAttributes` ヘルパーを使用します。```タイプスクリプト
インポート {
  getInputAttributes、
  getOutputAttributes、
  スパン付き、
「@arizeai/openinference-core」から;

const handleQuery = withSpan(
  async (userInput: 文字列) => {
    const result = await Agent.generate({ プロンプト: userInput });
    結果を返します。
  }、
  {
    名前: "クエリ.ハンドラー"、
    種類：「チェーン」、
    // ヘルパーを使用する - 自動 MIME タイプ検出
    processInput: (入力) => getInputAttributes(入力)、
    processOutput: (result) => getOutputAttributes(result.text),
  }
);

await handleQuery("2+2 とは何ですか?");
「」**何がキャプチャされるか:**```json
{
  "input.value": "2+2 とは何ですか?",
  "input.mime_type": "テキスト/プレーン",
  "output.value": "2+2 は 4 に等しい。",
  "output.mime_type": "テキスト/プレーン"
}
「」**ヘルパーの動作:**
- 文字列 → `text/plain`
- オブジェクト/配列 → `application/json` (自動シリアル化)
- `undefined`/`null` → 属性が設定されていません

**これが重要な理由:**
- Phoenix 評価者には `input.value` と `output.value` が必要です
- Phoenix UI はデバッグ用に I/O を目立つように表示します
- データセットを微調整するためのデータのエクスポートを可能にします

### カスタム I/O 処理

標準 I/O 属性と一緒にカスタム メタデータを追加します。```タイプスクリプト
const processWithMetadata = withSpan(
  async (クエリ: 文字列) => {
    const result = await llm.generate(query);
    結果を返します。
  }、
  {
    名前: "クエリ.プロセス"、
    種類：「チェーン」、
    processInput: (クエリ) => ({
      "input.value": クエリ、
      "input.mime_type": "テキスト/プレーン",
      "input.length": query.length, // カスタム属性
    })、
    processOutput: (結果) => ({
      "出力.値": 結果.テキスト、
      "output.mime_type": "テキスト/プレーン",
      "output.tokens": result.usage?.totalTokens, // カスタム属性
    })、
  }
);
「」## 関連項目

- **スパン属性:** `span-chain.md`、`span-retriever.md`、`span-tool.md` など。
- **属性ヘルパー:** https://docs.arize.com/phoenix/tracing/manual-instrumentation-typescript#attribute-helpers
- **自動インスツルメンテーション:** `instrumentation-auto-typescript.md` (フレームワーク統合用)