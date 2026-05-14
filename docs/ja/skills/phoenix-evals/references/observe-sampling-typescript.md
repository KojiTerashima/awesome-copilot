# 観察: サンプリング戦略 (TypeScript)

レビューのために実稼働トレースを効率的にサンプリングする方法。

## 戦略

### 1. 失敗重視 (最優先)

サーバー側フィルターを使用して、必要なものだけを取得します。```タイプスクリプト
import { getSpans } から "@arizeai/phoenix-client/spans";

// サーバー側フィルター — エラー スパンのみが返されます
const { スパン: エラー } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  ステータスコード: "エラー"、
  制限: 100、
});

// LLM スパンのみを取得します
const { スパン: llmSpans } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  スパン種類: "LLM"、
  制限: 100、
});

// スパン名でフィルタリングします
const { スパン: chatSpans } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  名前: "チャット_コンプリート",
  制限: 100、
});
「」### 2. 外れ値```タイプスクリプト
const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  制限: 200、
});
const latency = (s: (スパンの種類)[数値]) =>
  新しい日付(s.end_time).getTime() - 新しい日付(s.start_time).getTime();
constsorted = [...spans].sort((a, b) => latency(b) - latency(a));
const lowResponses =sorted.slice(0, 50);
「」### 3. 階層化 (カバレッジ)```タイプスクリプト
// 各カテゴリから均等にサンプリングします
function stratifiedSample<T>(items: T[], groupBy: (item: T) => string, perGroup:number): T[] {
  const groups = new Map<string, T[]>();
  for (項目の定数項目) {
    const key = groupBy(項目);
    if (!groups.has(key)) groups.set(key, []);
    groups.get(key)!.push(item);
  }
  return [...groups.values()]. flatMap((g) => g.slice(0, perGroup));
}

const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  制限: 500、
});
const byQueryType = stratifiedSample(spans, (s) => s.attributes?.["metadata.query_type"] ?? "unknown", 20);
「」### 4. 指標に基づく```タイプスクリプト
import { getSpanAnnotations } から "@arizeai/phoenix-client/spans";

// スパンのアノテーションを取得し、ラベルでフィルターします
const { アノテーション } = await getSpanAnnotations({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  spanIds:spans.map((s) => s.context.span_id),
  includeAnnotationNames: ["幻覚"],
});

const flaggedSpanIds = new Set(
  annotations.filter((a) => a.result?.label === "幻覚").map((a) => a.span_id)
);
const flagged = spans.filter((s) => flaggedSpanIds.has(s.context.span_id));
「」## トレースレベルのサンプリング

リクエスト全体 (トレース内のすべてのスパン) が必要な場合は、`getTraces` を使用します。```タイプスクリプト
import { getTraces } から "@arizeai/phoenix-client/traces";

// フルスパンツリーを含む最近のトレース
const { トレース } = await getTraces({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  制限: 100、
  includeSpans: true、
});

// セッションごとにフィルタリングします (例: マルチターン会話)
const { トレース: sessionTraces } = await getTraces({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  セッションID: "ユーザーセッション-abc",
  includeSpans: true、
});

// 時間窓サンプリング
const { トレース: 最近のトレース } = await getTraces({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  startTime: new Date(Date.now() - 60 * 60 * 1000), // 過去 1 時間
  制限: 50、
  includeSpans: true、
});
「」## レビューキューの構築```タイプスクリプト
// サーバー側フィルターをレビューキューに結合します
const { スパン: errorSpans } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  ステータスコード: "エラー"、
  制限: 30、
});
const { スパン: allSpans } = await getSpans({
  プロジェクト: { プロジェクト名: "私のプロジェクト" },
  制限: 100、
});
const ランダム = allSpans.sort(() => Math.random() - 0.5).slice(0, 30);

const combed = [...errorSpans, ...random];
const unique = [...new Map(combined.map((s) => [s.context.span_id, s])).values()];
const reviewQueue = unique.slice(0, 100);
「」## サンプルサイズのガイドライン

|目的 |サイズ |
| ------- | ---- |
|初期の探索 | 50-100 |
|エラー分析 | 100+ (飽和するまで) |
|ゴールデン データセット | 100-500 |
|ジャッジキャリブレーション |クラスごとに 100 名以上 |

**飽和:** 新しいトレースが同じ失敗パターンを示した場合に停止します。