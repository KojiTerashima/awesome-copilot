# TypeScript SDK アノテーション パターン

TypeScript クライアントを使用して、スパン、トレース、ドキュメント、セッションにフィードバックを追加します。

## クライアントのセットアップ```タイプスクリプト
import { createClient } から "phoenix-client";
const client = createClient();  // デフォルト: http://localhost:6006
「」## スパン注釈

個々のスパンにフィードバックを追加します。```タイプスクリプト
import { addSpanAnnotation } から "phoenix-client";

await addSpanAnnotation({
  クライアント、
  スパンアノテーション: {
    スパンID: "abc123",
    名前：「品質」、
    アノテーターの種類: "人間"、
    ラベル: "高品質"、
    スコア: 0.95、
    説明: "正確で適切にフォーマットされています",
    メタデータ: { レビュアー: "アリス" }
  }、
  同期: true
});
「」## ドキュメントの注釈

RETRIEVER スパンで個々のドキュメントを評価します。```タイプスクリプト
import { addDocumentAnnotation } from "phoenix-client";

await addDocumentAnnotation({
  クライアント、
  ドキュメントの注釈: {
    スパンID: "retriever_span",
    documentPosition: 0, // 0 から始まるインデックス
    名前: "関連性"、
    アノテーターの種類: "LLM",
    ラベル: "関連"、
    スコア: 0.95
  }
});
「」## トレース注釈

トレース全体に関するフィードバック:```タイプスクリプト
import { addTraceAnnotation } から "phoenix-client";

await addTraceAnnotation({
  クライアント、
  トレースアノテーション: {
    トレースID: "trace_abc",
    名前：「正しさ」、
    アノテーターの種類: "人間"、
    ラベル: "正しい"、
    スコア: 1.0
  }
});
「」## セッションの注釈

マルチターン会話に関するフィードバック:```タイプスクリプト
import { addSessionAnnotation } から "phoenix-client";

await addSessionAnnotation({
  クライアント、
  セッションアノテーション: {
    セッションID: "session_xyz",
    名前: "user_satisfaction",
    アノテーターの種類: "人間"、
    ラベル: 「満足」、
    スコア: 0.85
  }
});
「」## RAG パイプラインの例```タイプスクリプト
import { createClient, logDocumentAnnotations, addSpanAnnotation, addTraceAnnotation } from "phoenix-client";

const client = createClient();

// ドキュメントの関連性 (バッチ)
await logDocumentAnnotations({
  クライアント、
  ドキュメントの注釈: [
    {spanId: "retriever_span"、documentPosition: 0、name: "relevance"、
      アノテーター種類: "LLM"、ラベル: "関連"、スコア: 0.95 }、
    {spanId: "retriever_span"、documentPosition: 1、name: "relevance"、
      アノテーターの種類: "LLM"、ラベル: "関連"、スコア: 0.80 }
  】
});

// LLM 応答品質
await addSpanAnnotation({
  クライアント、
  スパンアノテーション: {
    スパンID: "llm_span",
    名前：「忠実」、
    アノテーターの種類: "LLM",
    ラベル: 「忠実」、
    スコア: 0.90
  }
});

// 全体的なトレース品質
await addTraceAnnotation({
  クライアント、
  トレースアノテーション: {
    トレースID: "trace_123",
    名前：「正しさ」、
    アノテーターの種類: "人間"、
    ラベル: "正しい"、
    スコア: 1.0
  }
});
「」## API リファレンス

- [TypeScript クライアント API](https://arize-ai.github.io/phoenix/)