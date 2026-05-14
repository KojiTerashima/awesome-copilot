# エラー分析

エバリュエーターを構築する前に、トレースをレビューして障害モードを発見します。

## プロセス

1. **サンプル** - 100 以上のトレース (エラー、負のフィードバック、ランダム)
2. **オープンコード** - トレースごとに自由形式のメモを作成します
3. **Axial コード** - ノートを故障カテゴリにグループ化する
4. **定量化** - カテゴリごとの失敗の数
5. **優先順位付け** - 頻度×重大度でランク付けします

## サンプル トレース

### スパンレベルのサンプリング (Python — DataFrame)「」パイソン
phoenix.clientインポートクライアントから

# Client() はローカル Phoenix で動作します (env vars または localhost:6006 にフォールバックします)
# リモート/クラウドの場合: Client(base_url="https://app.phoenix.arize.com", api_key="...")
client = クライアント()
spans_df = client.spans.get_spans_dataframe(project_identifier="my-app")

# 代表的なサンプルをビルドする
サンプル = pd.concat([
    spans_df[spans_df["ステータスコード"] == "エラー"].sample(30),
    spans_df[spans_df["フィードバック"] == "ネガティブ"].sample(30),
    スパン_df.サンプル(40)、
]).drop_duplicates("span_id").head(100)
「」### スパンレベルのサンプリング (TypeScript)```タイプスクリプト
import { getSpans } から "@arizeai/phoenix-client/spans";

const { スパン: エラー } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  ステータスコード: "エラー"、
  制限: 30、
});
const { スパン: allSpans } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  制限: 70、
});
const サンプル = [...エラー, ...allSpans.sort(() => Math.random() - 0.5).slice(0, 40)];
const unique = [...new Map(sample.map((s) => [s.context.span_id, s])).values()].slice(0, 100);
「」### トレースレベルのサンプリング (Python)

エラーが複数のスパンにまたがる場合 (エージェント ワークフローなど)、トレース全体をサンプリングします。「」パイソン
from datetime import datetime、timedelta

トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=24)、
    include_spans=True、
    sort="latency_ms",
    order="記述",
    制限=100、
）
# 各トレースには、trace_id、start_time、end_time、spans があります。
「」### トレースレベルのサンプリング (TypeScript)```タイプスクリプト
import { getTraces } から "@arizeai/phoenix-client/traces";

const { トレース } = await getTraces({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 24 * 60 * 60 * 1000)、
  includeSpans: true、
  制限: 100、
});
「」## メモを追加する (Python)「」パイソン
client.spans.add_span_note(
    スパン_id="abc123",
    note="タイムゾーンが間違っています - EST 午後 3 時と言っていますが、ユーザーは PST です"
）
「」## メモを追加する (TypeScript)```タイプスクリプト
import { addSpanNote } から "@arizeai/phoenix-client/spans";

await addSpanNote({
  スパン注: {
    スパンID: "abc123",
    注: 「タイムゾーンが間違っています - EST 午後 3 時と言っていますが、ユーザーは PST です」
  }
});
「」## 注意すべきこと

|タイプ |例 |
| ---- | -------- |
|事実上の誤り |間違った日付、価格、でっち上げられた機能 |
|不足している情報 |質問に答えていない、詳細が省略されている |
|トーンの問題 |文脈から見てカジュアルすぎる/フォーマルすぎる |
|ツールの問題 |間違ったツール、間違ったパラメータ |
|検索 |間違ったドキュメント、関連ドキュメントが欠落している |

## 良いメモ「」
BAD：「レスポンスが悪い」
良い: 「返答には 2 日以内に発送されるとありますが、ポリシーは 5 ～ 7 日です。」
「」## カテゴリにグループ化する「」パイソン
カテゴリ = {
    "factual_inaccuracy": ["間違った発送時間"、"間違った価格"],
    "幻覚": ["割引をでっち上げた", "発明された機能"],
    "tone_mismatch": ["企業クライアント向けの非公式"],
}
# 優先度 = 頻度 × 重大度
「」## 既存のアノテーションを取得する

### パイソン「」パイソン
# スパンデータフレームから
annotations_df = client.spans.get_span_annotations_dataframe(
    spans_dataframe=サンプル、
    project_identifier="私のアプリ",
    include_annotation_names=["品質", "正確さ"],
）
# annotations_df には、span_id (インデックス)、名前、ラベル、スコア、説明が含まれます

# または特定のスパン ID から
annotations_df = client.spans.get_span_annotations_dataframe(
    span_ids=["スパン-id-1", "スパン-id-2"],
    project_identifier="私のアプリ",
）
「」### TypeScript```タイプスクリプト
import { getSpanAnnotations } から "@arizeai/phoenix-client/spans";

const { アノテーション } = await getSpanAnnotations({
  プロジェクト: { プロジェクト名: "my-app" },
  スパン ID: ["スパン ID-1"、"スパン ID-2"]、
  includeAnnotationNames: ["品質", "正確さ"],
});

for (注釈の定数) {
  console.log(`${ann.span_id}: ${ann.name} = ${ann.result?.label} (${ann.result?.score})`);
}
「」## 彩度

新しいトレースで新しい障害モードが明らかにならなければ停止します。最小: 100 トレース。