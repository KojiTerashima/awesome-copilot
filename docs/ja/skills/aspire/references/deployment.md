# デプロイ — 完全リファレンス

Aspire は **オーケストレーション**（何を実行するか）と **デプロイ**（どこで実行するか）を分離します。`aspire publish` コマンドは、AppHost のリソースモデルをターゲットプラットフォーム向けのデプロイマニフェストに変換します。

---

## Publish と Deploy の違い

| 概念 | 役割 |
|---|---|
| **`aspire publish`** | デプロイ成果物（Dockerfile、Helm チャート、Bicep など）を生成 |
| **Deploy** | 生成された成果物を CI/CD パイプラインで実行 |

Aspire は直接デプロイしません。マニフェストを生成し、実際のデプロイはあなたが行います。

---

## サポート対象

### Docker

**パッケージ:** `Aspire.Hosting.Docker`

```bash
aspire publish -p docker -o ./docker-output
```

生成されるもの:
- `docker-compose.yml` — AppHost に対応するサービス定義
- 各 .NET プロジェクトの `Dockerfile`
- 環境変数設定
- ボリュームマウント
- ネットワーク設定

```csharp
// Docker 公開用の AppHost 設定
var api = builder.AddProject<Projects.Api>("api")
    .PublishAsDockerFile();  // 既定の publish 動作を上書き
```

### Kubernetes

**パッケージ:** `Aspire.Hosting.Kubernetes`

```bash
aspire publish -p kubernetes -o ./k8s-output
```

生成されるもの:
- Kubernetes YAML マニフェスト（Deployments、Services、ConfigMaps、Secrets）
- Helm チャート（任意）
- Ingress 設定
- AppHost 設定に基づくリソース制限

```csharp
// AppHost: K8s 公開のカスタマイズ
var api = builder.AddProject<Projects.Api>("api")
    .WithReplicas(3)                    // K8s の replicas に対応
    .WithExternalHttpEndpoints();       // Ingress/LoadBalancer に対応
```

### Azure Container Apps

**パッケージ:** `Aspire.Hosting.Azure.AppContainers`

```bash
aspire publish -p azure -o ./azure-output
```

生成されるもの:
- Azure Container Apps Environment 用の Bicep テンプレート
- 各サービスの Container App 定義
- Azure Container Registry 設定
- マネージド ID 設定
- Dapr コンポーネント（Dapr 連携を使用している場合）
- VNET 設定

```csharp
// AppHost: Azure 固有の設定
var api = builder.AddProject<Projects.Api>("api")
    .WithExternalHttpEndpoints()        // 外部 ingress に対応
    .WithReplicas(3);                   // 最小レプリカ数に対応

// Azure リソースは自動でプロビジョニングされる
var storage = builder.AddAzureStorage("storage");   // Storage Account を作成
var cosmos = builder.AddAzureCosmosDB("cosmos");    // Cosmos DB アカウントを作成
var sb = builder.AddAzureServiceBus("messaging");   // Service Bus 名前空間を作成
```

### Azure App Service

**パッケージ:** `Aspire.Hosting.Azure.AppService`

```bash
aspire publish -p appservice -o ./appservice-output
```

生成されるもの:
- App Service Plan と Web App 用の Bicep テンプレート
- 接続文字列設定
- アプリケーション設定

---

## リソースモデルとデプロイ先の対応

| AppHost の概念 | Docker Compose | Kubernetes | Azure Container Apps |
|---|---|---|---|
| `AddProject<T>()` | Dockerfile を持つ `service` | `Deployment` + `Service` | `Container App` |
| `AddContainer()` | `image:` を持つ `service` | `Deployment` + `Service` | `Container App` |
| `AddRedis()` | `service: redis` | `StatefulSet` | マネージド Redis |
| `AddPostgres()` | `service: postgres` | `StatefulSet` | Azure PostgreSQL |
| `.WithReference()` | `environment:` 変数 | `ConfigMap` / `Secret` | App settings |
| `.WithReplicas(n)` | `deploy: replicas: n` | `replicas: n` | `minReplicas: n` |
| `.WithVolume()` | `volumes:` | `PersistentVolumeClaim` | Azure Files |
| `.WithHttpEndpoint()` | `ports:` | `Service` port | Ingress |
| `.WithExternalHttpEndpoints()` | `ports:`（host） | `Ingress` / `LoadBalancer` | 外部 ingress |
| `AddParameter(secret: true)` | `.env` file | `Secret` | Key Vault 参照 |

---

## CI/CD 連携

### GitHub Actions の例

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Install Aspire CLI
        run: curl -sSL https://aspire.dev/install.sh | bash

      - name: Generate manifests
        run: aspire publish -p azure -o ./deploy

      - name: Deploy to Azure
        uses: azure/arm-deploy@v2
        with:
          template: ./deploy/main.bicep
          parameters: ./deploy/main.parameters.json
```

### Azure DevOps の例

```yaml
trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: UseDotNet@2
    inputs:
      version: '10.0.x'

  - script: curl -sSL https://aspire.dev/install.sh | bash
    displayName: 'Install Aspire CLI'

  - script: aspire publish -p azure -o $(Build.ArtifactStagingDirectory)/deploy
    displayName: 'Generate deployment manifests'

  - task: AzureResourceManagerTemplateDeployment@3
    inputs:
      deploymentScope: 'Resource Group'
      templateLocation: '$(Build.ArtifactStagingDirectory)/deploy/main.bicep'
```

---

## 環境別設定

### シークレットにパラメーターを使う

```csharp
// AppHost
var dbPassword = builder.AddParameter("db-password", secret: true);
var postgres = builder.AddPostgres("db", password: dbPassword);
```

デプロイ時:
- **Docker:** `.env` file から読み込み
- **Kubernetes:** `Secret` リソースから読み込み
- **Azure:** マネージド ID 経由で Key Vault から読み込み

### 条件付きリソース

```csharp
// 本番では Azure サービス、ローカルではエミュレーターを使用
if (builder.ExecutionContext.IsPublishMode)
{
    var cosmos = builder.AddAzureCosmosDB("cosmos");    // 実際の Azure リソース
}
else
{
    var cosmos = builder.AddAzureCosmosDB("cosmos")
        .RunAsEmulator();                                // ローカルエミュレーター
}
```

---

## Dev Containers と GitHub Codespaces

Aspire テンプレートには `.devcontainer/` 設定が含まれています:

```json
{
  "name": "Aspire App",
  "image": "mcr.microsoft.com/devcontainers/dotnet:10.0",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/node:1": {}
  },
  "postCreateCommand": "curl -sSL https://aspire.dev/install.sh | bash",
  "forwardPorts": [18888],
  "portsAttributes": {
    "18888": { "label": "Aspire Dashboard" }
  }
}
```

Codespaces ではポートフォワーディングが自動で機能するため、ダッシュボードとすべてのサービスエンドポイントにフォワード済み URL 経由でアクセスできます。

