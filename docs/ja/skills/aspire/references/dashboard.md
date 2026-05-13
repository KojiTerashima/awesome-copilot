# Dashboard — 完全リファレンス

Aspire Dashboard は、分散アプリケーション内のすべてのリソースをリアルタイムに可観測化します。`aspire run` で自動的に起動し、単体でも実行できます。

---

## 機能

### Resources view

すべてのリソース（プロジェクト、コンテナー、実行ファイル）を次の情報とともに表示します。

- **Name** と **type**（Project、Container、Executable）
- **State**（Starting、Running、Stopped、FailedToStart など）
- **Start time** と **uptime**
- **Endpoints** — 公開されている各エンドポイントへのクリック可能な URL
- **Source** — プロジェクトパス、コンテナーイメージ、または実行ファイルパス
- **Actions** — Stop、Start、Restart ボタン

### Console logs

すべてのリソースから集約された生の stdout/stderr:

- リソース名でフィルター
- ログ内検索
- 一時停止可能な自動スクロール
- リソースごとの色分け表示

### Structured logs

アプリケーションレベルの構造化ログ（ILogger、OpenTelemetry 経由）:

- リソース、ログレベル、カテゴリ、メッセージ内容で **Filterable**
- **Expandable** — クリックすると、すべてのプロパティを含む完全なログエントリを表示
- トレースと **Correlated** — クリックで関連トレースへジャンプ
- .NET ILogger の構造化ログプロパティをサポート
- 任意言語からの OpenTelemetry ログシグナルをサポート

### Distributed traces

すべてのサービスをまたぐエンドツーエンドのリクエストトレース:

- **Waterfall view** — タイミング付きで完全なコールチェーンを表示
- **Span details** — HTTP メソッド、URL、ステータスコード、実行時間
- **Database spans** — SQL クエリ、接続の詳細
- **Messaging spans** — キュー操作、トピックへの publish
- **Error highlighting** — 失敗した span を赤で表示
- **Cross-service correlation** — .NET はトレースコンテキストを自動伝播、他言語は手動設定

### Metrics

リアルタイムおよび履歴メトリクス:

- **Runtime metrics** — CPU、メモリ、GC、スレッドプール
- **HTTP metrics** — リクエスト率、エラー率、レイテンシのパーセンタイル
- **Custom metrics** — サービスが OpenTelemetry 経由で出力する任意のメトリクス
- **Chartable** — 各メトリクスを時系列グラフで表示

### GenAI Visualizer

AI/LLM 統合を利用するアプリケーション向け:

- **Token usage** — リクエストごとの prompt tokens、completion tokens、total tokens
- **Prompt/completion pairs** — 実際に送信したプロンプトと受信した応答を表示
- **Model metadata** — 使用モデル、temperature、max tokens
- **Latency** — AI 呼び出しごとの所要時間
- OpenTelemetry 経由でサービスが [GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) を出力している必要があります

---

## Dashboard URL

デフォルトでは、ダッシュボードは自動割り当てポートで実行されます。確認方法:

- `aspire run` 起動時のターミナル出力
- MCP: `list_resources` ツール経由
- `--dashboard-port` で上書き:

```bash
aspire run --dashboard-port 18888
```

---

## Standalone Dashboard

AppHost なしでダッシュボードを実行します。すでに OpenTelemetry を出力している既存アプリケーションで有用です。

```bash
docker run --rm -d \
  -p 18888:18888 \
  -p 4317:18889 \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

| Port             | Purpose                                                      |
| ---------------- | ------------------------------------------------------------ |
| `18888`          | Dashboard web UI                                             |
| `4317` → `18889` | OTLP gRPC receiver（標準 OTel ポート → ダッシュボード内部） |

### サービス設定

OpenTelemetry exporter の送信先をダッシュボードに設定します。

```bash
# 任意言語の OpenTelemetry SDK 向け環境変数
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_SERVICE_NAME=my-service
```

### Docker Compose 例

```yaml
services:
  dashboard:
    image: mcr.microsoft.com/dotnet/aspire-dashboard:latest
    ports:
      - "18888:18888"
      - "4317:18889"

  api:
    build: ./api
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://dashboard:18889
      - OTEL_SERVICE_NAME=api

  worker:
    build: ./worker
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://dashboard:18889
      - OTEL_SERVICE_NAME=worker
```

---

## Dashboard configuration

### 認証

スタンドアロンダッシュボードは、ブラウザートークンによる認証をサポートします。

```bash
docker run --rm -d \
  -p 18888:18888 \
  -p 4317:18889 \
  -e DASHBOARD__FRONTEND__AUTHMODE=BrowserToken \
  -e DASHBOARD__FRONTEND__BROWSERTOKEN__TOKEN=my-secret-token \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

### OTLP 設定

```bash
# gRPC で OTLP を受け付ける（デフォルト）
-e DASHBOARD__OTLP__GRPC__ENDPOINT=http://0.0.0.0:18889

# HTTP で OTLP を受け付ける
-e DASHBOARD__OTLP__HTTP__ENDPOINT=http://0.0.0.0:18890

# OTLP に API キーを必須化
-e DASHBOARD__OTLP__AUTHMODE=ApiKey
-e DASHBOARD__OTLP__PRIMARYAPIKEY=my-api-key
```

### リソース制限

```bash
# 保持するログエントリ数の上限
-e DASHBOARD__TELEMETRYLIMITS__MAXLOGCOUNT=10000

# 保持するトレースエントリ数の上限
-e DASHBOARD__TELEMETRYLIMITS__MAXTRACECOUNT=10000

# メトリクスデータポイント数の上限
-e DASHBOARD__TELEMETRYLIMITS__MAXMETRICCOUNT=50000
```

---

## Copilot integration

ダッシュボードは VS Code の GitHub Copilot と統合できます。

- リソース状態について質問する
- ログやトレースを自然言語で照会する
- MCP サーバー（[MCP Server](mcp-server.md) を参照）がブリッジを提供

---

## Non-.NET service telemetry

非 .NET サービスをダッシュボードに表示するには、OpenTelemetry シグナルを出力する必要があります。Aspire は `.WithReference()` 使用時に OTLP endpoint の環境変数を自動注入します。

### Python (OpenTelemetry SDK)

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
import os

# Aspire injects OTEL_EXPORTER_OTLP_ENDPOINT automatically
endpoint = os.environ.get("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317")

provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint=endpoint)))
trace.set_tracer_provider(provider)
```

### JavaScript (OpenTelemetry SDK)

```javascript
const { NodeTracerProvider } = require("@opentelemetry/sdk-trace-node");
const { OTLPTraceExporter } = require("@opentelemetry/exporter-trace-otlp-grpc");

const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new BatchSpanProcessor(
    new OTLPTraceExporter({
      url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || "http://localhost:4317",
    })
  )
);
provider.register();
```

