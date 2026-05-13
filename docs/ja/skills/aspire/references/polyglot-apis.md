# Polyglot APIs — 完全リファレンス

Aspire は 10 以上の言語/ランタイムをサポートしています。AppHost は常に .NET ですが、オーケストレーションされるワークロードは任意の言語で実行できます。各言語には、依存関係グラフに接続するためのリソースを返すホスティング手法があります。

---

## ホスティングモデルの違い

| モデル | リソース型 | 実行方法 | 例 |
|---|---|---|---|
| **Project** | `ProjectResource` | .NET プロジェクト参照（SDK によりビルド） | `AddProject<T>()` |
| **Container** | `ContainerResource` | Docker/OCI イメージ | `AddContainer()`, `AddRedis()`, `AddPostgres()` |
| **Executable** | `ExecutableResource` | ネイティブ OS プロセス | `AddExecutable()`, すべての `Add*App()` polyglot メソッド |

すべての polyglot `Add*App()` メソッドは、内部的に `ExecutableResource` インスタンスを作成します。AppHost 側で対象言語の SDK は不要で、開発マシンにそのワークロードのランタイムがインストールされていれば動作します。

---

## 公式（Microsoft メンテナンス）

### .NET / C\#

```csharp
builder.AddProject<Projects.MyApi>("api")
```

**チェーン可能なメソッド:**
- `.WithHttpEndpoint(port?, targetPort?, name?)` — HTTP エンドポイントを公開
- `.WithHttpsEndpoint(port?, targetPort?, name?)` — HTTPS エンドポイントを公開
- `.WithEndpoint(port?, targetPort?, scheme?, name?)` — 汎用エンドポイント
- `.WithReference(resource)` — 依存関係を接続（接続文字列またはサービスディスカバリー）
- `.WithReplicas(count)` — 複数インスタンスを実行
- `.WithEnvironment(key, value)` — 環境変数を設定
- `.WithEnvironment(callback)` — コールバック経由で環境変数を設定（遅延解決）
- `.WaitFor(resource)` — 依存先が健全になるまで起動しない
- `.WithExternalHttpEndpoints()` — エンドポイントを外部アクセス可能としてマーク
- `.WithOtlpExporter()` — OpenTelemetry エクスポーターを構成
- `.PublishAsDockerFile()` — 公開動作を Dockerfile に上書き

### Python

```csharp
// Standard Python script
builder.AddPythonApp("service", "../python-service", "main.py")

// Uvicorn ASGI server (FastAPI, Starlette, etc.)
builder.AddUvicornApp("fastapi", "../fastapi-app", "app:app")
```

**`AddPythonApp(name, projectDirectory, scriptPath, args?)`**

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)` — HTTP を公開
- `.WithVirtualEnvironment(path?)` — venv を使用（デフォルト: `.venv`）
- `.WithPipPackages(packages)` — 起動時に pip パッケージをインストール
- `.WithReference(resource)` — 依存関係を接続
- `.WithEnvironment(key, value)` — 環境変数を設定
- `.WaitFor(resource)` — 依存先の健全性を待機

**`AddUvicornApp(name, projectDirectory, appModule, args?)`**

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)` — HTTP を公開
- `.WithVirtualEnvironment(path?)` — venv を使用
- `.WithReference(resource)` — 依存関係を接続
- `.WithEnvironment(key, value)` — 環境変数を設定
- `.WaitFor(resource)` — 依存先の健全性を待機

**Python のサービスディスカバリー:** 環境変数は自動的に注入されます。`os.environ` で読み取ります:
```python
import os
redis_conn = os.environ["ConnectionStrings__cache"]
api_url = os.environ["services__api__http__0"]
```

### JavaScript / TypeScript

```csharp
// Generic JavaScript app (npm start)
builder.AddJavaScriptApp("frontend", "../web-app")

// Vite dev server
builder.AddViteApp("spa", "../vite-app")

// Node.js script
builder.AddNodeApp("worker", "server.js", "../node-worker")
```

**`AddJavaScriptApp(name, workingDirectory)`**

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)` — HTTP を公開
- `.WithNpmPackageInstallation()` — 起動前に `npm install` を実行
- `.WithReference(resource)` — 依存関係を接続
- `.WithEnvironment(key, value)` — 環境変数を設定
- `.WaitFor(resource)` — 依存先の健全性を待機

**`AddViteApp(name, workingDirectory)`**

チェーン可能なメソッド（`AddJavaScriptApp` と同様、加えて以下）:
- `.WithNpmPackageInstallation()` — 起動前に `npm install` を実行
- `.WithHttpEndpoint(port?, targetPort?, name?)` — Vite のデフォルトは 5173

**`AddNodeApp(name, scriptPath, workingDirectory)`**

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)` — HTTP を公開
- `.WithNpmPackageInstallation()` — 起動前に `npm install` を実行
- `.WithReference(resource)` — 依存関係を接続
- `.WithEnvironment(key, value)` — 環境変数を設定

**JS/TS のサービスディスカバリー:** 環境変数が注入されます。`process.env` を使用します:
```javascript
const redisUrl = process.env.ConnectionStrings__cache;
const apiUrl = process.env.services__api__http__0;
```

---

## コミュニティ（CommunityToolkit/Aspire）

コミュニティ統合はすべて同じパターンに従います。AppHost に NuGet パッケージをインストールし、`Add*App()` メソッドを使用します。

### Go

**パッケージ:** `CommunityToolkit.Aspire.Hosting.Golang`

```csharp
builder.AddGolangApp("go-api", "../go-service")
    .WithHttpEndpoint(targetPort: 8080)
    .WithReference(redis)
    .WithEnvironment("LOG_LEVEL", "debug")
    .WaitFor(redis);
```

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)`
- `.WithReference(resource)`
- `.WithEnvironment(key, value)`
- `.WaitFor(resource)`

**Go のサービスディスカバリー:** `os.Getenv()` による標準の環境変数:
```go
redisAddr := os.Getenv("ConnectionStrings__cache")
```

### Java (Spring Boot)

**パッケージ:** `CommunityToolkit.Aspire.Hosting.Java`

```csharp
builder.AddSpringApp("spring-api", "../spring-service")
    .WithHttpEndpoint(targetPort: 8080)
    .WithReference(postgres)
    .WaitFor(postgres);
```

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)`
- `.WithReference(resource)`
- `.WithEnvironment(key, value)`
- `.WaitFor(resource)`
- `.WithMavenBuild()` — 起動前に Maven ビルドを実行
- `.WithGradleBuild()` — 起動前に Gradle ビルドを実行

**Java のサービスディスカバリー:** `System.getenv()` による環境変数:
```java
String dbConn = System.getenv("ConnectionStrings__db");
```

### Rust

**パッケージ:** `CommunityToolkit.Aspire.Hosting.Rust`

```csharp
builder.AddRustApp("rust-worker", "../rust-service")
    .WithHttpEndpoint(targetPort: 3000)
    .WithReference(redis)
    .WaitFor(redis);
```

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)`
- `.WithReference(resource)`
- `.WithEnvironment(key, value)`
- `.WaitFor(resource)`
- `.WithCargoBuild()` — 起動前に `cargo build` を実行

### Bun

**パッケージ:** `CommunityToolkit.Aspire.Hosting.Bun`

```csharp
builder.AddBunApp("bun-api", "../bun-service")
    .WithHttpEndpoint(targetPort: 3000)
    .WithReference(redis);
```

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)`
- `.WithReference(resource)`
- `.WithEnvironment(key, value)`
- `.WaitFor(resource)`
- `.WithBunPackageInstallation()` — 起動前に `bun install` を実行

### Deno

**パッケージ:** `CommunityToolkit.Aspire.Hosting.Deno`

```csharp
builder.AddDenoApp("deno-api", "../deno-service")
    .WithHttpEndpoint(targetPort: 8000)
    .WithReference(redis);
```

チェーン可能なメソッド:
- `.WithHttpEndpoint(port?, targetPort?, name?)`
- `.WithReference(resource)`
- `.WithEnvironment(key, value)`
- `.WaitFor(resource)`

### PowerShell

```csharp
builder.AddPowerShell("ps-script", "../scripts/process.ps1")
    .WithReference(storageAccount);
```

### Dapr

**パッケージ:** `Aspire.Hosting.Dapr`（公式）

```csharp
var dapr = builder.AddDapr();
var api = builder.AddProject<Projects.Api>("api")
    .WithDaprSidecar("api-sidecar");
```

---

## 完全なマルチ言語構成の例

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Infrastructure
var redis = builder.AddRedis("cache");
var postgres = builder.AddPostgres("pg").AddDatabase("catalog");
var mongo = builder.AddMongoDB("mongo").AddDatabase("analytics");
var rabbit = builder.AddRabbitMQ("messaging");

// .NET API (primary)
var api = builder.AddProject<Projects.CatalogApi>("api")
    .WithReference(postgres)
    .WithReference(redis)
    .WithReference(rabbit)
    .WaitFor(postgres)
    .WaitFor(redis);

// Python ML service (FastAPI)
var ml = builder.AddUvicornApp("ml", "../ml-service", "app:app")
    .WithHttpEndpoint(targetPort: 8000)
    .WithVirtualEnvironment()
    .WithReference(redis)
    .WithReference(mongo)
    .WaitFor(redis);

// TypeScript frontend (Vite + React)
var web = builder.AddViteApp("web", "../frontend")
    .WithNpmPackageInstallation()
    .WithHttpEndpoint(targetPort: 5173)
    .WithReference(api);

// Go event processor
var processor = builder.AddGolangApp("processor", "../go-processor")
    .WithReference(rabbit)
    .WithReference(mongo)
    .WaitFor(rabbit);

// Java analytics service (Spring Boot)
var analytics = builder.AddSpringApp("analytics", "../spring-analytics")
    .WithHttpEndpoint(targetPort: 8080)
    .WithReference(mongo)
    .WithReference(rabbit)
    .WaitFor(mongo);

// Rust high-perf worker
var worker = builder.AddRustApp("worker", "../rust-worker")
    .WithReference(redis)
    .WithReference(rabbit)
    .WaitFor(redis);

builder.Build().Run();
```

この 1 つの AppHost で、5 言語にまたがる 6 つのサービスと 4 つのインフラリソースを起動し、すべてを自動サービスディスカバリーで接続できます。

