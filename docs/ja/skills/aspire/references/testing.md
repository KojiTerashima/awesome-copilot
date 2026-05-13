# テスト — 完全リファレンス

Aspire は、完全な AppHost に対して統合テストを実行するための `Aspire.Hosting.Testing` を提供します。テストでは分散アプリケーション全体（またはその一部）を起動し、実際のサービスに対してアサーションを実行します。

---

## パッケージ

```xml
<PackageReference Include="Aspire.Hosting.Testing" Version="*" />
```

---

## 基本パターン: DistributedApplicationTestingBuilder

```csharp
// 1. AppHost からテストビルダーを作成
var builder = await DistributedApplicationTestingBuilder
    .CreateAsync<Projects.MyAppHost>();

// 2. （任意）テスト用にリソースを上書き
// ... 下のカスタマイズセクションを参照

// 3. アプリケーションをビルドして起動
await using var app = await builder.BuildAsync();
await app.StartAsync();

// 4. サービス用の HTTP クライアントを作成
var client = app.CreateHttpClient("api");

// 5. アサーションを実行
var response = await client.GetAsync("/health");
Assert.Equal(HttpStatusCode.OK, response.StatusCode);
```

---

## xUnit の例

### 基本的なヘルスチェックテスト

```csharp
public class HealthTests(ITestOutputHelper output)
{
    [Fact]
    public async Task AllServicesAreHealthy()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.AppHost>();

        await using var app = await builder.BuildAsync();
        await app.StartAsync();

        // 各サービスのヘルスエンドポイントをテスト
        var apiClient = app.CreateHttpClient("api");
        var apiHealth = await apiClient.GetAsync("/health");
        Assert.Equal(HttpStatusCode.OK, apiHealth.StatusCode);

        var workerClient = app.CreateHttpClient("worker");
        var workerHealth = await workerClient.GetAsync("/health");
        Assert.Equal(HttpStatusCode.OK, workerHealth.StatusCode);
    }
}
```

### API 統合テスト

```csharp
public class ApiTests(ITestOutputHelper output)
{
    [Fact]
    public async Task CreateOrder_ReturnsCreated()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.AppHost>();

        await using var app = await builder.BuildAsync();
        await app.StartAsync();

        var client = app.CreateHttpClient("api");

        var order = new { ProductId = 1, Quantity = 2 };
        var response = await client.PostAsJsonAsync("/orders", order);

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);

        var created = await response.Content.ReadFromJsonAsync<Order>();
        Assert.NotNull(created);
        Assert.Equal(1, created.ProductId);
    }
}
```

### 準備完了待機を含むテスト

```csharp
[Fact]
public async Task DatabaseIsSeeded()
{
    var builder = await DistributedApplicationTestingBuilder
        .CreateAsync<Projects.AppHost>();

    await using var app = await builder.BuildAsync();
    await app.StartAsync();

    // API が完全に準備完了になるまで待機（すべての依存関係が正常）
    await app.WaitForResourceReadyAsync("api");

    var client = app.CreateHttpClient("api");
    var response = await client.GetAsync("/products");

    Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    var products = await response.Content.ReadFromJsonAsync<List<Product>>();
    Assert.NotEmpty(products);
}
```

---

## MSTest の例

```csharp
[TestClass]
public class IntegrationTests
{
    [TestMethod]
    public async Task ApiReturnsProducts()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.AppHost>();

        await using var app = await builder.BuildAsync();
        await app.StartAsync();

        var client = app.CreateHttpClient("api");
        var response = await client.GetAsync("/products");

        Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);
    }
}
```

---

## NUnit の例

```csharp
[TestFixture]
public class IntegrationTests
{
    [Test]
    public async Task ApiReturnsProducts()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.AppHost>();

        await using var app = await builder.BuildAsync();
        await app.StartAsync();

        var client = app.CreateHttpClient("api");
        var response = await client.GetAsync("/products");

        Assert.That(response.StatusCode, Is.EqualTo(HttpStatusCode.OK));
    }
}
```

---

## テスト用 AppHost のカスタマイズ

### リソースを上書き

```csharp
var builder = await DistributedApplicationTestingBuilder
    .CreateAsync<Projects.AppHost>();

// 実データベースをテスト用コンテナーに置き換え
builder.Services.ConfigureHttpClientDefaults(http =>
{
    http.AddStandardResilienceHandler();
});

// テスト専用の構成を追加
builder.Configuration["TestMode"] = "true";

await using var app = await builder.BuildAsync();
await app.StartAsync();
```

### リソースを除外

```csharp
var builder = await DistributedApplicationTestingBuilder
    .CreateAsync<Projects.AppHost>(args =>
    {
        // API のみのテストでは worker を起動しない
        args.Args = ["--exclude-resource", "worker"];
    });
```

### 特定の環境でテスト

```csharp
var builder = await DistributedApplicationTestingBuilder
    .CreateAsync<Projects.AppHost>(args =>
    {
        args.Args = ["--environment", "Testing"];
    });
```

---

## 接続文字列へのアクセス

```csharp
// テストでリソースの接続文字列を取得
var connectionString = await app.GetConnectionStringAsync("db");

// これを使ってテスト内でデータベースを直接クエリ
using var conn = new NpgsqlConnection(connectionString);
await conn.OpenAsync();
var count = await conn.ExecuteScalarAsync<int>("SELECT COUNT(*) FROM products");
Assert.True(count > 0);
```

---

## ベストプラクティス

1. **`WaitForResourceReadyAsync` を使う** — リクエスト前に、すべての依存関係が正常であることを保証
2. **各テストを独立させる** — 前のテストの状態に依存しない
3. **アプリには `await using` を使う** — テスト失敗時でもクリーンアップを保証
4. **実際のインフラをテストする** — Aspire は実コンテナー（Redis、PostgreSQL など）を起動するため、高忠実度の統合テストが可能
5. **テスト用 AppHost は必要最小限に保つ** — 特定シナリオで不要なリソースを除外
6. **テスト専用の構成を使う** — テスト分離のために設定を上書き
7. **タイムアウト保護** — コンテナーの起動には時間がかかるため、妥当なテストタイムアウトを設定:

```csharp
[Fact(Timeout = 120_000)]  // 2 分
public async Task SlowIntegrationTest() { ... }
```

---

## プロジェクト構成

```
MyApp/
├── src/
│   ├── MyApp.AppHost/           # AppHost プロジェクト
│   ├── MyApp.Api/               # API サービス
│   ├── MyApp.Worker/            # Worker サービス
│   └── MyApp.ServiceDefaults/   # 共有デフォルト設定
└── tests/
    └── MyApp.Tests/             # 統合テスト
        ├── MyApp.Tests.csproj   # AppHost + Testing パッケージを参照
        └── ApiTests.cs          # テストクラス
```

```xml
<!-- MyApp.Tests.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <IsAspireTestProject>true</IsAspireTestProject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Aspire.Hosting.Testing" Version="*" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="*" />
    <PackageReference Include="xunit" Version="*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="*" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\MyApp.AppHost\MyApp.AppHost.csproj" />
  </ItemGroup>
</Project>
```

