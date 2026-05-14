# Phoenix トレーシング: カスタム メタデータ (TypeScript)

より豊かな可観測性を実現するために、カスタム属性をスパンに追加します。

## コンテキストの使用 (すべての子スパンに伝播)```タイプスクリプト
import { context } から "@arizeai/phoenix-otel";
import { setMetadata } から "@arizeai/openinference-core";

context.with(
  setMetadata(context.active(), {
    実験ID: "exp_123",
    モデルバージョン: "gpt-4-1106-プレビュー",
    環境: "実稼働"、
  })、
  非同期() => {
    // このブロック内で作成されたすべてのスパンには以下が含まれます。
    // "メタデータ" = '{"experiment_id": "exp_123", ...}'
    myApp.run(クエリ)を待ちます;
  }
);
「」## 単一スパン上```タイプスクリプト
import {traceChain} から "@arizeai/openinference-core";
import { トレース } から "@arizeai/phoenix-otel";

const myFunction =traceChain(
  非同期 (入力: 文字列) => {
    const スパン = トレース.getActiveSpan();

    スパン?.setAttribute(
      「メタデータ」、
      JSON.stringify({
        実験ID: "exp_123",
        モデルバージョン: "gpt-4-1106-プレビュー",
        環境: "実稼働"、
      })
    );

    結果を返します。
  }、
  { 名前: "私の関数" }
);

await myFunction("hello");
「」
