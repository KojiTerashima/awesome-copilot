# Python SDK アノテーション パターン

Python クライアントを使用して、スパン、トレース、ドキュメント、セッションにフィードバックを追加します。

## クライアントのセットアップ「」パイソン
phoenix.clientインポートクライアントから
client = Client() # デフォルト: http://localhost:6006
「」## スパン注釈

個々のスパンにフィードバックを追加します。「」パイソン
client.spans.add_span_annotation(
    スパン_id="abc123",
    annotation_name="品質",
    annotator_kind="人間",
    ラベル="高品質",
    スコア=0.95、
    description="正確で適切にフォーマットされています",
    メタデータ={"レビュアー": "アリス"},
    同期=真
）
「」## ドキュメントの注釈

RETRIEVER スパンで個々のドキュメントを評価します。「」パイソン
client.spans.add_document_annotation(
    span_id="レトリバー_スパン",
    document_position=0, # 0 から始まるインデックス
    annotation_name="関連性",
    annotator_kind="LLM",
    ラベル="関連",
    スコア=0.95
）
「」## トレース注釈

トレース全体に関するフィードバック:「」パイソン
client.traces.add_trace_annotation(
    トレース_id="トレース_abc",
    annotation_name="正しさ",
    annotator_kind="人間",
    ラベル="正しい",
    スコア=1.0
）
「」## セッションの注釈

マルチターン会話に関するフィードバック:「」パイソン
client.sessions.add_session_annotation(
    session_id="セッション_xyz",
    annotation_name="ユーザー満足度",
    annotator_kind="人間",
    ラベル="満足",
    スコア=0.85
）
「」## RAG パイプラインの例「」パイソン
phoenix.clientインポートクライアントから
phoenix.client.resources.spans から SpanDocumentAnnotationData をインポート

client = クライアント()

# ドキュメントの関連性 (バッチ)
client.spans.log_document_annotations(
    document_annotations=[
        SpanDocumentAnnotationData(
            name="関連性"、span_id="retriever_span"、document_position=i、
            annotator_kind="LLM"、result={"ラベル": ラベル、"スコア": スコア}
        ）
        for i, (ラベル, スコア) in enumerate([
            (「関連性」、0.95)、(「関連性」、0.80)、(「無関係」、0.10)
        ])
    】
）

# LLM 応答品質
client.spans.add_span_annotation(
    スパン_id="llm_span",
    annotation_name="誠実さ",
    annotator_kind="LLM",
    ラベル="忠実",
    スコア=0.90
）

# 全体的なトレース品質
client.traces.add_trace_annotation(
    トレースID="トレース_123",
    annotation_name="正しさ",
    annotator_kind="人間",
    ラベル="正しい",
    スコア=1.0
）
「」## API リファレンス

- [Python クライアント API](https://arize-phoenix.readthedocs.io/projects/client/en/latest/)