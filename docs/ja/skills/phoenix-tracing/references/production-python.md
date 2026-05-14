# Phoenix Tracing: 制作ガイド (Python)

**重要: 本番展開用にバッチ処理、データ マスキング、およびスパン フィルタリングを構成します。**

## メタデータ

|属性 |値 |
|----------|----------|
|優先順位 |重要 - 本番環境の準備 |
|影響 |セキュリティ、パフォーマンス |
|セットアップ時間 | 5～15分 |

## バッチ処理

**バッチ処理を有効にして生産効率を高めます。** バッチ処理では、スパンを個別に送信するのではなくグループで送信することで、ネットワークのオーバーヘッドを削減します。

## データマスキング (PII 保護)

**環境変数:**「」バッシュ
import OPENINFERENCE_HIDE_INPUTS=true # input.value を非表示にする
import OPENINFERENCE_HIDE_OUTPUTS=true # 出力値を非表示にする
import OPENINFERENCE_HIDE_INPUT_MESSAGES=true # LLM 入力メッセージを非表示にする
import OPENINFERENCE_HIDE_OUTPUT_MESSAGES=true # LLM 出力メッセージを非表示にする
import OPENINFERENCE_HIDE_INPUT_IMAGES=true # 画像コンテンツを非表示にする
import OPENINFERENCE_HIDE_INPUT_TEXT=true # 埋め込みテキストを非表示にする
import OPENINFERENCE_BASE64_IMAGE_MAX_LENGTH=10000 # 画像サイズを制限する
「」**Python TraceConfig:**「」パイソン
phoenix.otelインポートレジスタから
openinference.instrumentation から TraceConfig をインポート

config = TraceConfig(
    Hide_inputs=True、
    Hide_outputs=True、
    Hide_input_messages=True
）
登録(trace_config=config)
「」**優先順位:** コード > 環境変数 > デフォルト

---

## スパンフィルタリング

**特定のコード ブロックを抑制します:**「」パイソン
phoenix.otelからインポートsuppress_tracing

Suppress_tracing() を使用:
    Internal_logging() # スパンは生成されません
「」
