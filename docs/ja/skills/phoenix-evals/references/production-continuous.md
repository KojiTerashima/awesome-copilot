# 生産: 継続的な評価

能力と回帰の評価および継続的なフィードバック ループ。

## 2 種類の eval

|タイプ |合格率目標 |目的 |更新 |
| ---- | ---------------- | ------- | ------ |
| **能力** | 50-80% |改善策 |より難しいケースを追加する |
| **回帰** | 95-100% |キャッチ破損 |修正されたバグを追加 |

## 彩度

能力評価が合格率 >95% に達すると、能力評価は飽和状態になります。
1.回帰スイートに合格したケースを卒業する
2. 新しい困難なケースを機能スイートに追加する

## フィードバック ループ「」
本番環境 → トラフィックのサンプル → エバリュエータの実行 → 障害の検出
    ↑ ↓
デプロイ ← CI 評価の実行 ← テスト ケースの作成 ← エラー分析
「」## 実装

継続的な監視ループを構築します。

1. **最近のトレースを定期的にサンプリング**します (例: 1 時間あたり 100 トレース)
2. サンプリングされたトレースに対して **エバリュエーターを実行**
3. **結果を追跡するために Phoenix にログ**する
4. 人間によるレビューのための **結果に関するキュー**
5. 繰り返される障害パターンから **テスト ケースを作成**

### パイソン「」パイソン
phoenix.clientインポートクライアントから
from datetime import datetime、timedelta

client = クライアント()

# 1. 最近のスパンのサンプル (評価用の完全な属性を含む)
spans_df = client.spans.get_spans_dataframe(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=1)、
    root_spans_only=真、
    制限=100、
）

#2. エバリュエーターを実行する
phoenix.evalsからのインポートevaluate_dataframe

results_df = 評価データフレーム(
    データフレーム=スパン_df、
    評価者=[品質評価、安全評価]、
）

# 3. 結果を注釈としてアップロードする
phoenix.evals.utils から to_annotation_dataframe にインポート

annotations_df = to_annotation_dataframe(results_df)
client.spans.log_span_annotations_dataframe(dataframe=annotations_df)
「」### TypeScript```タイプスクリプト
import { getSpans } から "@arizeai/phoenix-client/spans";
import { logSpanAnnotations } から "@arizeai/phoenix-client/spans";

// 1. 最近のスパンのサンプル
const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 60 * 60 * 1000)、
  parentId: null, // ルート スパンのみ
  制限: 100、
});

// 2. エバリュエーターを実行します (ユーザー定義)
const results = await Promise.all(
  spans.map(async (スパン) => ({
    スパンID:span.context.span_id、
    ...await runEvaluators(span, [qualityEval, safetyEval]),
  }))
);

// 3. 結果を注釈としてアップロードする
await logSpanAnnotations({
  spanAnnotations: results.map((r) => ({
    スパン ID: r.spanId、
    名前：「品質」、
    スコア: r.qualityScore、
    ラベル: r.qualityLabel、
    annotatorKind: "LLM" を const として、
  }))、
});
「」トレースレベルの監視 (エージェント ワークフローなど) の場合は、`get_traces`/`getTraces` を使用してトレースを識別します。「」パイソン
# Python: 遅いトレースを特定する
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=1)、
    sort="latency_ms",
    order="記述",
    制限=50、
）
「」

```タイプスクリプト
// TypeScript: 遅いトレースを識別する
import { getTraces } から "@arizeai/phoenix-client/traces";

const { トレース } = await getTraces({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 60 * 60 * 1000)、
  制限: 50、
});
「」## アラート中

|状態 |重大度 |アクション |
| --------- | -------- | ------ |
|回帰 < 98% |クリティカル |オンコールページ |
|能力の低下 |警告 | Slack 通知 |
| 7d の能力 > 95% |情報 |スケジュールの見直し |

## 重要な原則

- **2 つのスイート** - 常に機能 + 回帰
- **卒業生のケース** - 一貫したパスを回帰に移動する
- **傾向を追跡** - スナップショットだけでなく、長期にわたって監視します