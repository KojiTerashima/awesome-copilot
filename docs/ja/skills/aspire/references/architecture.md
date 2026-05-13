# アーキテクチャ — 詳細解説

このリファレンスでは、Aspire の内部アーキテクチャ（DCP エンジン、リソースモデル、サービスディスカバリ、ネットワーキング、テレメトリ、イベントシステム）を扱います。

---

## Developer Control Plane (DCP)

DCP は、Aspire が `aspire run` モードで使用する**ランタイムエンジン**です。主なポイント:

- **Go** で実装（.NET ではない）
- **Kubernetes 互換 API サーバー**を公開（ローカル専用で、実際の K8s クラスターではない）
- リソースのライフサイクルを管理: create, start, health-check, stop, restart
- ローカルのコンテナーランタイム（Docker、Podman、Rancher）でコンテナーを実行
- 実行ファイルをネイティブ OS プロセスとして実行
- 自動ポート割り当て付きのプロキシレイヤーでネットワーク処理
- Aspire Dashboard のリアルタイムデータの基盤を提供

### DCP と Kubernetes の比較

| 項目 | DCP（ローカル開発） | Kubernetes（本番） |
|---|---|---|
| API | Kubernetes 互換 | 完全な Kubernetes API |
| スコープ | 単一マシン | クラスター |
| ネットワーキング | ローカルプロキシ、自動ポート | サービスメッシュ、ingress |
| ストレージ | ローカルボリューム | PVC、クラウドストレージ |
| 目的 | 開発者のインナーループ | 本番デプロイ |

Kubernetes 互換 API であるため、Aspire は同じリソース抽象を理解しますが、DCP は Kubernetes ディストリビューション**ではありません**。軽量なローカルランタイムです。

---

## リソースモデル

Aspire のすべては**リソース**です。リソースモデルは階層構造になっています。

### 型階層

```
IResource (interface)
└── Resource (abstract base)
    ├── ProjectResource          — .NET project reference
    ├── ContainerResource        — Docker/OCI container
    ├── ExecutableResource       — Native process (polyglot apps)
    ├── ParameterResource        — Config value or secret
    └── Infrastructure resources
        ├── RedisResource
        ├── PostgresServerResource
        ├── MongoDBServerResource
        ├── SqlServerResource
        ├── RabbitMQServerResource
        ├── KafkaServerResource
        └── ... (one per integration)
```

### リソースのプロパティ

すべてのリソースは次を持ちます:
- **Name** — AppHost 内で一意な識別子
- **State** — ライフサイクル状態（Starting, Running, FailedToStart, Stopping, Stopped など）
- **Annotations** — リソースに付与されるメタデータ
- **Endpoints** — リソースが公開するネットワークエンドポイント
- **Environment variables** — プロセス/コンテナーへ注入される環境変数

### アノテーション

アノテーションはリソースに紐づくメタデータバッグです。主な組み込みアノテーション:

| Annotation | 目的 |
|---|---|
| `EndpointAnnotation` | HTTP/HTTPS/TCP エンドポイントを定義 |
| `EnvironmentCallbackAnnotation` | 遅延環境変数解決 |
| `HealthCheckAnnotation` | ヘルスチェック設定 |
| `ContainerImageAnnotation` | Docker イメージ詳細 |
| `VolumeAnnotation` | ボリュームマウント設定 |
| `CommandLineArgsCallbackAnnotation` | 動的 CLI 引数 |
| `ManifestPublishingCallbackAnnotation` | カスタム publish 動作 |

### リソースのライフサイクル状態

```
NotStarted → Starting → Running → Stopping → Stopped
                 ↓                     ↓
          FailedToStart           RuntimeUnhealthy
                                       ↓
                                  Restarting → Running
```

### DAG（Directed Acyclic Graph）

リソースは依存グラフを形成します。Aspire はトポロジカル順序でリソースを起動します:

```
PostgreSQL ──→ API ──→ Frontend
Redis ────────↗
RabbitMQ ──→ Worker
```

1. PostgreSQL、Redis、RabbitMQ が最初に起動（依存なし）
2. API は PostgreSQL と Redis が healthy になってから起動
3. Frontend は API が healthy になってから起動
4. Worker は RabbitMQ が healthy になってから起動

`.WaitFor()` は依存エッジにヘルスチェックゲートを追加します。これがない場合、依存先の起動は行われますが、下流は healthy になるまで待機しません。

---

## サービスディスカバリ

Aspire は各リソースに環境変数を注入して、サービス同士が相互に発見できるようにします。サービスレジストリや DNS は不要で、純粋に環境変数注入で実現します。

### 接続文字列

データベース、キャッシュ、メッセージブローカー向け:

```
ConnectionStrings__<resource-name>=<connection-string>
```

例:
```
ConnectionStrings__cache=localhost:6379
ConnectionStrings__catalog=Host=localhost;Port=5432;Database=catalog;Username=postgres;Password=...
ConnectionStrings__messaging=amqp://guest:guest@localhost:5672
```

### サービスエンドポイント

HTTP/HTTPS サービス向け:

```
services__<resource-name>__<scheme>__0=<url>
```

例:
```
services__api__http__0=http://localhost:5234
services__api__https__0=https://localhost:7234
services__ml__http__0=http://localhost:8000
```

### `.WithReference()` の動作

```csharp
var redis = builder.AddRedis("cache");
var api = builder.AddProject<Projects.Api>("api")
    .WithReference(redis);
```

これにより次が行われます:
1. API の環境に `ConnectionStrings__cache=localhost:<auto-port>` を追加
2. DAG に依存エッジを作成（API は Redis に依存）
3. API サービス内で `builder.Configuration.GetConnectionString("cache")` が接続文字列を返す

### クロス言語サービスディスカバリ

すべての言語で同じ env var パターンを使います:

| Language | 読み取り方法 |
|---|---|
| C# | `builder.Configuration.GetConnectionString("cache")` |
| Python | `os.environ["ConnectionStrings__cache"]` |
| JavaScript | `process.env.ConnectionStrings__cache` |
| Go | `os.Getenv("ConnectionStrings__cache")` |
| Java | `System.getenv("ConnectionStrings__cache")` |
| Rust | `std::env::var("ConnectionStrings__cache")` |

---

## ネットワーキング

### プロキシアーキテクチャ

`aspire run` モードでは、DCP は公開された各エンドポイントごとにリバースプロキシを実行します:

```
Browser → Proxy (auto-assigned port) → Actual Service (target port)
```

- **port**（外部ポート）— 上書きしない限り DCP が自動割り当て
- **targetPort** — サービスが実際に待ち受けるポート
- サービス間トラフィックは可観測性のためすべてプロキシを通過

```csharp
// DCP に外部ポートを自動割り当てさせ、サービスは 8000 で待ち受け
builder.AddPythonApp("ml", "../ml", "main.py")
    .WithHttpEndpoint(targetPort: 8000);

// 外部ポートを 3000 に固定
builder.AddViteApp("web", "../frontend")
    .WithHttpEndpoint(port: 3000, targetPort: 5173);
```

### エンドポイントの種類

```csharp
// HTTP endpoint
.WithHttpEndpoint(port?, targetPort?, name?)

// HTTPS endpoint
.WithHttpsEndpoint(port?, targetPort?, name?)

// Generic endpoint (TCP, custom schemes)
.WithEndpoint(port?, targetPort?, scheme?, name?, isExternal?)

// Mark endpoints as externally accessible (for deployment)
.WithExternalHttpEndpoints()
```

---

## テレメトリ（OpenTelemetry）

Aspire は .NET サービス向けに OpenTelemetry を自動設定します。.NET 以外のサービスでは、DCP コレクターを向くように OpenTelemetry を手動設定します。

### 自動設定されるもの（.NET サービス）

- **分散トレーシング** — HTTP クライアント/サーバースパン、DB スパン、メッセージングスパン
- **メトリクス** — ランタイムメトリクス、HTTP メトリクス、カスタムメトリクス
- **構造化ログ** — トレースコンテキストと相関されたログ
- **Exporter** — Aspire Dashboard を向く OTLP exporter

### .NET 以外のサービス設定

DCP は OTLP エンドポイントを公開します。.NET 以外のサービスで次の env var を設定します:

```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_SERVICE_NAME=<your-service-name>
```

Aspire は dashboard collector 向けに `.WithReference()` 経由で `OTEL_EXPORTER_OTLP_ENDPOINT` を自動注入します。

### ServiceDefaults パターン

`ServiceDefaults` プロジェクトは共有設定ライブラリで、次を標準化します:
- OpenTelemetry セットアップ（tracing、metrics、logging）
- ヘルスチェックエンドポイント（`/health`, `/alive`）
- レジリエンスポリシー（Polly による retries、circuit breakers）

```csharp
// In each .NET service's Program.cs
builder.AddServiceDefaults();   // adds OTel, health checks, resilience
// ... other service config ...
app.MapDefaultEndpoints();      // maps /health and /alive
```

---

## ヘルスチェック

### 組み込みヘルスチェック

すべての統合はクライアント側に自動でヘルスチェックを追加します:
- Redis: `PING` command
- PostgreSQL: `SELECT 1`
- MongoDB: `ping` command
- RabbitMQ: Connection check
- など

### WaitFor と WithReference

```csharp
// WithReference: 接続文字列を配線 + 依存エッジを作成
// （下流は依存先が healthy になる前に起動する場合がある）
.WithReference(db)

// WaitFor: ヘルスチェックでゲート — healthy になるまで下流は起動しない
.WaitFor(db)

// 一般的なパターン: 両方使う
.WithReference(db).WaitFor(db)
```

### カスタムヘルスチェック

```csharp
var api = builder.AddProject<Projects.Api>("api")
    .WithHealthCheck("ready", "/health/ready")
    .WithHealthCheck("live", "/health/live");
```

---

## イベントシステム

AppHost は、リソース状態の変化に反応するためのライフサイクルイベントをサポートしています:

```csharp
builder.Eventing.Subscribe<ResourceReadyEvent>("api", (evt, ct) =>
{
    // Fires when "api" resource becomes healthy
    Console.WriteLine($"API is ready at {evt.Resource.Name}");
    return Task.CompletedTask;
});

builder.Eventing.Subscribe<BeforeResourceStartedEvent>("db", async (evt, ct) =>
{
    // Run database migrations before the DB resource is marked as started
    await RunMigrations();
});
```

### 利用可能なイベント

| Event | タイミング |
|---|---|
| `BeforeResourceStartedEvent` | リソース起動前 |
| `ResourceReadyEvent` | リソースが healthy で ready |
| `ResourceStateChangedEvent` | 任意の状態遷移時 |
| `BeforeStartEvent` | アプリケーション全体の起動前 |
| `AfterEndpointsAllocatedEvent` | すべてのポート割り当て後 |

---

## 設定

### Parameters

```csharp
// Plain parameter
var apiKey = builder.AddParameter("api-key");

// Secret parameter (prompted at run, not logged)
var dbPassword = builder.AddParameter("db-password", secret: true);

// Use in resources
var api = builder.AddProject<Projects.Api>("api")
    .WithEnvironment("API_KEY", apiKey);

var db = builder.AddPostgres("db", password: dbPassword);
```

### 設定ソース

Parameters は（優先順で）次から解決されます:
1. コマンドライン引数
2. 環境変数
3. User secrets（`dotnet user-secrets`）
4. `appsettings.json` / `appsettings.{Environment}.json`
5. インタラクティブプロンプト（`aspire run` 中の secrets 用）

