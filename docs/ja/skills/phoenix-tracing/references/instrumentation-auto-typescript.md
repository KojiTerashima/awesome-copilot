# 自動インストルメンテーション (TypeScript)

コードを変更せずに、LLM 呼び出しのスパンを自動的に作成します。

## サポートされているフレームワーク

- **LLM SDK:** OpenAI
- **フレームワーク:** ラングチェーン
- **インストール:** `npm install @arizeai/openinference-instrumentation-{name}`

## セットアップ

**CommonJS (自動):**```JavaScript
const { register } = require("@arizeai/phoenix-otel");
const OpenAI = require("openai");

register({ プロジェクト名: "my-app" });

const client = new OpenAI();
「」**ESM (マニュアルが必要):**```タイプスクリプト
import { register, registerInstrumentations } from "@arizeai/phoenix-otel";
import { OpenAIInstrumentation } from "@arizeai/openinference-instrumentation-openai";
「openai」から OpenAI をインポートします。

register({ プロジェクト名: "my-app" });

const インストルメンテーション = new OpenAIInstrumentation();
インストルメンテーション.manuallyInstrument(OpenAI);
registerInstrumentations({ インストルメンテーション: [インストルメンテーション] });
「」**理由:** ESM インポートは `register()` が実行される前にホイストされます。

## 制限事項

**自動インスツルメンテーションがキャプチャしないもの:**```タイプスクリプト
非同期関数 myWorkflow(クエリ: string): Promise<string> {
  const 前処理 = 前処理 (クエリ) を待ちます。        // トレースされません
  const response = await client.chat.completions.create(...);  // トレース (自動)
  const postprocessed = 後処理 (応答) を待ちます。   // トレースされません
  後処理を返します。
}
「」**解決策:** カスタム ロジック用の手動インストルメンテーションを追加します。```タイプスクリプト
import {traceChain} から "@arizeai/openinference-core";

const myWorkflow =traceChain(
  async (クエリ: 文字列): Promise<string> => {
    const 前処理 = 前処理 (クエリ) を待ちます。
    const response = await client.chat.completions.create(...);
    const postprocessed = 後処理 (応答) を待ちます。
    後処理を返します。
  }、
  { 名前: "私のワークフロー" }
);
「」## 自動と手動の組み合わせ```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
import {traceChain} から "@arizeai/openinference-core";

register({ プロジェクト名: "my-app" });

const client = new OpenAI();

const ワークフロー =traceChain(
  async (クエリ: 文字列) => {
    const 前処理 = 前処理 (クエリ) を待ちます。
    const response = await client.chat.completions.create(...);  // 自動インストルメント化
    後処理(応答)を返します。
  }、
  { 名前: "私のワークフロー" }
);
「」
