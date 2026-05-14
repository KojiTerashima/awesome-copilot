# 実験: TypeScript のデータセット

評価データセットの作成と管理。

## データセットの作成```タイプスクリプト
import { createClient } から "@arizeai/phoenix-client";
import { createDataset } から "@arizeai/phoenix-client/datasets";

const client = createClient();

const { datasetId } = await createDataset({
  クライアント、
  名前: "qa-test-v1"、
  例: [
    {
      input: { 質問: 「2+2 とは何ですか?」 }、
      出力: { 答え: "4" }、
      メタデータ: { カテゴリ: "数学" },
    }、
  ]、
});
「」## 構造例```タイプスクリプト
インターフェース データセットの例 {
  入力: レコード<文字列、不明>;    // タスクの入力
  出力?: レコード<文字列、不明>;  // 期待される出力
  メタデータ?: レコード<文字列、不明>; // 追加のコンテキスト
}
「」## 本番環境のトレースから```タイプスクリプト
import { getSpans } から "@arizeai/phoenix-client/spans";

const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  parentId: null, // ルート スパンのみ
  制限: 100、
});

const 例 = spans.map((span) => ({
  入力: {クエリ:span.attributes?.["input.value"] },
  出力: {応答:span.attributes?.["output.value"] },
  メタデータ: {spanId:span.context.span_id}、
}));

await createDataset({ client, name: "production-sample", 例 });
「」## データセットの取得```タイプスクリプト
import { getDataset, listDatasets } from "@arizeai/phoenix-client/datasets";

const dataset = await getDataset({ client, datasetId: "..." });
const all = await listDatasets({ client });
「」## ベストプラクティス

- **バージョン管理**: 新しいデータセットを作成します。既存のデータセットは変更しないでください。
- **メタデータ**: ソース、カテゴリ、来歴を追跡します。
- **タイプ セーフティ**: 構造に TypeScript インターフェイスを使用します