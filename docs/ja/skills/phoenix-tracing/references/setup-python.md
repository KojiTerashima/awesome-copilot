# Phoenix トレース: Python のセットアップ

**`arize-phoenix-otel`.** を使用して Python で Phoenix トレースをセットアップします。

## メタデータ

|属性 |値 |
| ---------- | ----------------------------------- |
|優先順位 |クリティカル - すべてのトレースに必要 |
|セットアップ時間 | 5 分未満 |

## クイックスタート (3 行)「」パイソン
phoenix.otelインポートレジスタから
register(project_name="my-app", auto_instrument=True)
「」**`http://localhost:6006` に接続し、サポートされているすべてのライブラリを自動インストゥルメントします。**

## インストール「」バッシュ
pip インストール arise-phoenix-otel
「」**サポート対象:** Python 3.10-3.13

## 構成

### 環境変数 (推奨)「」バッシュ
import PHOENIX_API_KEY="your-api-key" # Phoenix Cloud に必要
import PHOENIX_COLLECTOR_ENDPOINT="http://localhost:6006" # またはクラウド URL
import PHOENIX_PROJECT_NAME="my-app" # オプション
「」### Python コード「」パイソン
phoenix.otelインポートレジスタから

トレーサープロバイダー = 登録(
    project_name="my-app", # プロジェクト名
    endpoint="http://localhost:6006", # Phoenix エンドポイント
    auto_instrument=True、# 自動インストゥルメントがサポートするライブラリ
    バッチ=True、# バッチ処理 (デフォルト: True)
）
「」**パラメータ:**

- `project_name`: プロジェクト名 (`PHOENIX_PROJECT_NAME` をオーバーライドします)
- `endpoint`: Phoenix URL (`PHOENIX_COLLECTOR_ENDPOINT` をオーバーライド)
- `auto_instrument`: 自動インスツルメンテーションを有効にする (デフォルト: False)
- `batch`: BatchSpanProcessor を使用します (デフォルト: True、運用環境で推奨)
- `protocol`: `"http/protobuf"` (デフォルト) または `"grpc"`

## 自動インストルメンテーション

フレームワーク用のインスツルメンタをインストールします。「」バッシュ
pip install openinference-instrumentation-openai # OpenAI SDK
pip install openinference-instrumentation-langchain # LangChain
pip install openinference-instrumentation-llama-index # LlamaIndex
# ... 必要に応じて他のものをインストールする
「」次に、自動インストルメンテーションを有効にします。「」パイソン
register(project_name="my-app", auto_instrument=True)
「」Phoenix は、インストールされているすべての OpenInference パッケージを自動的に検出し、計測します。

## バッチ処理 (本番)

デフォルトで有効になっています。環境変数を使用して構成します。「」バッシュ
export OTEL_BSP_SCHEDULE_DELAY=5000 # 5 秒ごとにバッチ
export OTEL_BSP_MAX_QUEUE_SIZE=2048 # キュー 2048 スパン
import OTEL_BSP_MAX_EXPORT_BATCH_SIZE=512 # 512 スパン/バッチを送信
「」**リンク:** https://opentelemetry.io/docs/specs/otel/configuration/sdk-environment-variables/

## 検証

1. Phoenix UI を開きます: `http://localhost:6006`
2. プロジェクトに移動します
3. アプリケーションを実行します
4. トレースを確認します (バッチ遅延内に表示されます)

## トラブルシューティング

**痕跡なし:**

- `PHOENIX_COLLECTOR_ENDPOINT` が Phoenix サーバーと一致することを確認します
- Phoenix Cloud に `PHOENIX_API_KEY` を設定します
- 設置されている計器を確認する

**属性がありません:**

- スパンの種類を確認します (ルール/ディレクトリを参照)
- 属性名を確認します (ルール/ディレクトリを参照)

## 例「」パイソン
phoenix.otelインポートレジスタから
openaiインポートからOpenAI

# 自動インストルメンテーションによるトレースを有効にする
register(project_name="my-chatbot", auto_instrument=True)

# OpenAI が自動的にインストルメント化される
クライアント = OpenAI()
応答 = client.chat.completions.create(
    モデル = "gpt-4"、
    メッセージ=[{"役割": "ユーザー", "コンテンツ": "こんにちは!"}]
）
「」## API リファレンス

- [Python OTEL API ドキュメント](https://arize-phoenix.readthedocs.io/projects/otel/en/latest/)
- [Python クライアント API ドキュメント](https://arize-phoenix.readthedocs.io/projects/client/en/latest/)