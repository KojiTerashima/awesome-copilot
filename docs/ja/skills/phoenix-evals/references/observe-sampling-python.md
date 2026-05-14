# 観察: サンプリング戦略

レビューのために実稼働トレースを効率的にサンプリングする方法。

## 戦略

### 1. 失敗重視 (最優先)「」パイソン
エラー =spans_df[spans_df["ステータスコード"] == "エラー"]
ネガティブフィードバック =spans_df[spans_df["フィードバック"] == "ネガティブ"]
「」### 2. 外れ値「」パイソン
long_responses = spans_df.nlargest(50, "response_length")
throw_responses = spans_df.nlargest(50, "latency_ms")
「」### 3. 階層化 (カバレッジ)「」パイソン
# 各カテゴリから均等にサンプリングします
by_query_type = spans_df.groupby("metadata.query_type").apply(
    ラムダ x: x.sample(min(len(x), 20))
）
「」### 4. 指標に基づく「」パイソン
# 自動評価器によってフラグが付けられたトレースを確認する
flagged = spans_df[eval_results["label"] == "幻覚"]
borderline = spans_df[(eval_results["スコア"] > 0.3) & (eval_results["スコア"] < 0.7)]
「」## レビューキューの構築「」パイソン
def build_review_queue(spans_df, max_traces=100):
    キュー = pd.concat([
        spans_df[spans_df["ステータスコード"] == "エラー"],
        spans_df[spans_df["フィードバック"] == "ネガティブ"],
        spans_df.nlargest(10, "response_length"),
        spans_df.sample(min(30, len(spans_df))),
    ]).drop_duplicates("span_id").head(max_traces)
    返却待ち行列
「」## サンプルサイズのガイドライン

|目的 |サイズ |
| ------- | ---- |
|初期の探索 | 50-100 |
|エラー分析 | 100+ (飽和するまで) |
|ゴールデン データセット | 100-500 |
|ジャッジキャリブレーション |クラスごとに 100 名以上 |

**飽和:** 新しいトレースが同じ失敗パターンを示した場合に停止します。

## トレースレベルのサンプリング

リクエスト全体 (トレースごとのすべてのスパン) が必要な場合は、`get_traces` を使用します。「」パイソン
phoenix.clientインポートクライアントから
from datetime import datetime、timedelta

client = クライアント()

# フルスパンツリーを含む最近のトレース
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    制限=100、
    include_spans=True、
）

# 時間枠のサンプリング (例: 過去 1 時間)
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=1)、
    制限=50、
    include_spans=True、
）

# セッションごとにフィルターします (複数ターンの会話)
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    session_id="ユーザーセッション-abc",
    include_spans=True、
）

# レイテンシで並べ替えて最も遅いリクエストを見つける
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    sort="latency_ms",
    order="記述",
    制限=50、
）
「」
