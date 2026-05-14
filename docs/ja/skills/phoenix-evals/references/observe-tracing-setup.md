# 観察: トレース設定

評価用のデータをキャプチャするようにトレースを構成します。

## クイックセットアップ「」パイソン
# パイソン
phoenix.otelインポートレジスタから

register(project_name="my-app", auto_instrument=True)
「」

```タイプスクリプト
// TypeScript
import { registerPhoenix } から "@arizeai/phoenix-otel";

registerPhoenix({ projectName: "my-app", autoInstrument: true });
「」## 必須の属性

|属性 |なぜそれが重要なのか |
| --------- | -------------- |
| `input.value` |ユーザーのリクエスト |
| `output.value` |評価する応答 |
| `retrieval.documents` |忠実さの背景 |
| `tool.name`、`tool.parameters` |エージェントの評価 |
| `llm.model_name` |モデルごとに追跡 |

## Eval のカスタム属性「」パイソン
span.set_attribute("metadata.client_type", "enterprise")
span.set_attribute("metadata.query_category", "billing")
「」## 評価用のエクスポート

### スパン (Python — データフレーム)「」パイソン
phoenix.clientインポートクライアントから

# Client() はローカル Phoenix で動作します (env vars または localhost:6006 にフォールバックします)
# リモート/クラウドの場合: Client(base_url="https://app.phoenix.arize.com", api_key="...")
client = クライアント()
spans_df = client.spans.get_spans_dataframe(
    project_identifier="my-app", # not project_name= (非推奨)
    root_spans_only=真、
）

データセット = client.datasets.create_dataset(
    name="エラー分析セット",
    dataframe=spans_df[["input.value", "output.value"]],
    input_keys=["input.value"],
    Output_keys=["output.value"],
）
「」### スパン (TypeScript)```タイプスクリプト
import { getSpans } から "@arizeai/phoenix-client/spans";

const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  parentId: null, // ルート スパンのみ
  制限: 100、
});
「」### トレース (Python — 構造化)

完全なトレース ツリーが必要な場合は `get_traces` を使用します (マルチターン会話、エージェント ワークフローなど)。「」パイソン
from datetime import datetime、timedelta

トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=24)、
    include_spans=True、# トレースごとにすべてのスパンが含まれます
    制限=100、
）
# 各トレースには、trace_id、start_time、end_time、spans があります (include_spans=True の場合)
「」### トレース (TypeScript)```タイプスクリプト
import { getTraces } から "@arizeai/phoenix-client/traces";

const { トレース } = await getTraces({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 24 * 60 * 60 * 1000)、
  includeSpans: true、
  制限: 100、
});
「」## 評価を注釈としてアップロードする

### パイソン「」パイソン
phoenix.evalsからのインポートevaluate_dataframe
phoenix.evals.utils から to_annotation_dataframe にインポート

# 評価を実行する
results_df = Evaluate_dataframe(dataframe=spans_df, evaluators=[my_eval])

# Phoenix アノテーションの結果をフォーマットする
annotations_df = to_annotation_dataframe(results_df)

# フェニックスにアップロード
client.spans.log_span_annotations_dataframe(dataframe=annotations_df)
「」### TypeScript```タイプスクリプト
import { logSpanAnnotations } から "@arizeai/phoenix-client/spans";

await logSpanAnnotations({
  スパンアノテーション: [
    {
      スパンID: "abc123",
      名前：「品質」、
      ラベル: 「良い」、
      スコア: 0.95、
      アノテーターの種類: "LLM",
    }、
  ]、
});
「」注釈は、トレースと一緒に Phoenix UI に表示されます。

## 確認する

必須の属性: `input.value`、`output.value`、`status_code`
RAG の場合: `retrieval.documents`
エージェントの場合: `tool.name`、`tool.parameters`