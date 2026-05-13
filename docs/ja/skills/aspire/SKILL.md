---
name: aspire
description: 'Aspire CLI、AppHost オーケストレーション、サービス ディスカバリー、統合、MCP サーバー、VS Code 拡張機能、Dev Containers、GitHub Codespaces、テンプレート、ダッシュボード、デプロイを網羅する Aspire スキル。Aspire 分散アプリケーションの作成、実行、デバッグ、構成、デプロイ、またはトラブルシューティングをユーザーが求めるときに使用します。'
---

# Aspire — ポリグロット分散アプリ オーケストレーション

Aspire は、可観測性が高く本番対応の分散アプリケーションを構築するための **コードファーストなポリグロット ツールチェーン** です。ワークロードが C#、Python、JavaScript/TypeScript、Go、Java、Rust、Bun、Deno、PowerShell のいずれであっても、単一の AppHost プロジェクトからコンテナー、実行可能ファイル、クラウド リソースをオーケストレーションします。

> **メンタルモデル:** AppHost は *指揮者* です。楽器を演奏するのではなく、各サービスに「いつ起動するか」「どう相互に見つけるか」を指示し、問題を監視します。

詳細なリファレンス資料は `references/` フォルダーにあります。必要に応じて読み込んでください。

---

## 参考資料

| Reference | 読み込むタイミング |
|---|---|
| [CLI Reference](references/cli-reference.md) | コマンド フラグ、オプション、または詳細な使い方が必要なとき |
| [MCP Server](references/mcp-server.md) | AI アシスタント向け MCP の設定、利用可能なツールを確認するとき |
| [Integrations Catalog](references/integrations-catalog.md) | MCP ツール経由で統合を探すとき、接続パターンを確認するとき |
| [Polyglot APIs](references/polyglot-apis.md) | メソッド シグネチャ、チェーン オプション、言語別パターンを確認するとき |
| [Architecture](references/architecture.md) | DCP の内部構造、リソース モデル、サービス ディスカバリー、ネットワーク、テレメトリを確認するとき |
| [Dashboard](references/dashboard.md) | ダッシュボード機能、スタンドアロン モード、GenAI Visualizer を確認するとき |
| [Deployment](references/deployment.md) | Docker、Kubernetes、Azure Container Apps、App Service へのデプロイを確認するとき |
| [Testing](references/testing.md) | AppHost に対する統合テストを行うとき |
| [Troubleshooting](references/troubleshooting.md) | 診断コード、よくあるエラーと修正方法を確認するとき |

---

## 1. Aspire ドキュメントの調査

Aspire チームは、AI アシスタント内でドキュメント ツールを直接提供する **MCP サーバー** を提供しています。設定の詳細は [MCP Server](references/mcp-server.md) を参照してください。

### Aspire CLI 13.2+（推奨 — ドキュメント検索を内蔵）

Aspire CLI **13.2 以降**（`aspire --version`）を実行している場合、MCP サーバーにはドキュメント検索ツールが含まれます。

| Tool | 説明 |
|---|---|
| `list_docs` | aspire.dev の利用可能なドキュメントをすべて一覧表示します |
| `search_docs` | インデックス化されたドキュメント全体に対して重み付き語彙検索を実行します |
| `get_doc` | slug を指定して特定のドキュメントを取得します |

これらのツールは [PR #14028](https://github.com/dotnet/aspire/pull/14028) で追加されました。更新するには: `aspire update --self --channel daily`。

このアプローチの詳細は David Pine の記事を参照してください: https://davidpine.dev/posts/aspire-docs-mcp-tools/

### Aspire CLI 13.1（統合ツールのみ）

13.1 では、MCP サーバーは統合の参照は提供しますが、ドキュメント検索は **提供しません**。

| Tool | 説明 |
|---|---|
| `list_integrations` | 利用可能な Aspire ホスティング統合を一覧表示します |
| `get_integration_docs` | 特定の統合パッケージのドキュメントを取得します |

13.1 で一般ドキュメントを問い合わせる場合は、主な情報源として **Context7** を使用してください（下記参照）。

### フォールバック: Context7

Aspire MCP のドキュメント ツールが利用できない場合（13.1）や MCP サーバーが起動していない場合は、**Context7**（`mcp_context7`）を使用します。

**ステップ 1 — ライブラリ ID を解決**（セッションごとに 1 回）:

`libraryName: ".NET Aspire"` で `mcp_context7_resolve-library-id` を呼び出します。

| Rank | Library ID | 使用する状況 |
|---|---|---|
| 1 | `/microsoft/aspire.dev` | 主な情報源。ガイド、統合、CLI リファレンス、デプロイ。 |
| 2 | `/dotnet/aspire` | API の内部構造、ソースレベルの実装詳細。 |
| 3 | `/communitytoolkit/aspire` | Microsoft 以外のポリグロット統合（Go、Java、Node.js、Ollama）。 |

**ステップ 2 — ドキュメントを問い合わせ:**

```
libraryId: "/microsoft/aspire.dev", query: "Python integration AddPythonApp service discovery"
libraryId: "/communitytoolkit/aspire", query: "Golang Java Node.js community integrations"
```

### フォールバック: GitHub 検索（Context7 も利用できない場合）

GitHub 上の公式ドキュメント リポジトリを検索します:
- **Docs repo:** `microsoft/aspire.dev` — path: `src/frontend/src/content/docs/`
- **Source repo:** `dotnet/aspire`
- **Samples repo:** `dotnet/aspire-samples`
- **Community integrations:** `CommunityToolkit/Aspire`

---

## 2. 前提条件とインストール

| Requirement | 詳細 |
|---|---|
| **.NET SDK** | 10.0+（非 .NET ワークロードでも必須 — AppHost は .NET） |
| **Container runtime** | Docker Desktop、Podman、Rancher Desktop |
| **IDE (optional)** | VS Code + C# Dev Kit、Visual Studio 2022、JetBrains Rider |

```bash
# Linux / macOS
curl -sSL https://aspire.dev/install.sh | bash

# Windows PowerShell
irm https://aspire.dev/install.ps1 | iex

# Verify
aspire --version

# Install templates
dotnet new install Aspire.ProjectTemplates
```

---

## 3. プロジェクト テンプレート

| Template | Command | 説明 |
|---|---|---|
| **aspire-starter** | `aspire new aspire-starter` | ASP.NET Core/Blazor スターター + AppHost + テスト |
| **aspire-ts-cs-starter** | `aspire new aspire-ts-cs-starter` | ASP.NET Core/React スターター + AppHost |
| **aspire-py-starter** | `aspire new aspire-py-starter` | FastAPI/React スターター + AppHost |
| **aspire-apphost-singlefile** | `aspire new aspire-apphost-singlefile` | 空の単一ファイル AppHost |

---

## 4. AppHost クイックスタート（ポリグロット）

AppHost はすべてのサービスをオーケストレーションします。非 .NET ワークロードはコンテナーまたは実行可能ファイルとして実行されます。

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Infrastructure
var redis = builder.AddRedis("cache");
var postgres = builder.AddPostgres("pg").AddDatabase("catalog");

// .NET API
var api = builder.AddProject<Projects.CatalogApi>("api")
    .WithReference(postgres).WithReference(redis);

// Python ML service
var ml = builder.AddPythonApp("ml-service", "../ml-service", "main.py")
    .WithHttpEndpoint(targetPort: 8000).WithReference(redis);

// React frontend (Vite)
var web = builder.AddViteApp("web", "../frontend")
    .WithHttpEndpoint(targetPort: 5173).WithReference(api);

// Go worker
var worker = builder.AddGolangApp("worker", "../go-worker")
    .WithReference(redis);

builder.Build().Run();
```

完全な API シグネチャについては [Polyglot APIs](references/polyglot-apis.md) を参照してください。

---

## 5. コア概念（要約）

| Concept | 要点 |
|---|---|
| **Run vs Publish** | `aspire run` = ローカル開発（DCP エンジン）。`aspire publish` = デプロイ マニフェストを生成。 |
| **Service discovery** | 環境変数により自動: `ConnectionStrings__<name>`, `services__<name>__http__0` |
| **Resource lifecycle** | DAG 順序 — 依存関係を先に起動。`.WaitFor()` はヘルスチェックに基づく待機ゲート。 |
| **Resource types** | `ProjectResource`, `ContainerResource`, `ExecutableResource`, `ParameterResource` |
| **Integrations** | 13 カテゴリで 144+。ホスティング パッケージ（AppHost）+ クライアント パッケージ（サービス）。 |
| **Dashboard** | リアルタイム ログ、トレース、メトリクス、GenAI ビジュアライザー。`aspire run` で自動起動。 |
| **MCP Server** | AI アシスタントは CLI（STDIO）経由で実行中アプリの照会やドキュメント検索が可能。 |
| **Testing** | `Aspire.Hosting.Testing` — xUnit/MSTest/NUnit で AppHost 全体を起動。 |
| **Deployment** | Docker、Kubernetes、Azure Container Apps、Azure App Service。 |

---

## 6. CLI クイックリファレンス

Aspire CLI 13.1 で有効なコマンド:

| Command | 説明 | Status |
|---|---|---|
| `aspire new <template>` | テンプレートから作成 | Stable |
| `aspire init` | 既存プロジェクトで初期化 | Stable |
| `aspire run` | すべてのリソースをローカルで起動 | Stable |
| `aspire add <integration>` | 統合を追加 | Stable |
| `aspire publish` | デプロイ マニフェストを生成 | Preview |
| `aspire config` | 構成設定を管理 | Stable |
| `aspire cache` | ディスク キャッシュを管理 | Stable |
| `aspire deploy` | 定義済みターゲットへデプロイ | Preview |
| `aspire do <step>` | パイプライン ステップを実行 | Preview |
| `aspire update` | 統合を更新（CLI は `--self`） | Preview |
| `aspire mcp init` | AI アシスタント向けに MCP を構成 | Stable |
| `aspire mcp start` | MCP サーバーを起動 | Stable |

フラグを含む完全なコマンド リファレンス: [CLI Reference](references/cli-reference.md)。

---

## 7. よくあるパターン

### 新しいサービスを追加する

1. サービス ディレクトリを作成する（任意の言語）
2. AppHost に追加: `Add*App()` または `AddProject<T>()`
3. 依存関係を接続: `.WithReference()`
4. 必要ならヘルス待機を設定: `.WaitFor()`
5. 実行: `aspire run`

### Docker Compose から移行する

1. `aspire new aspire-apphost-singlefile`（空の AppHost）
2. 各 `docker-compose` サービスを Aspire リソースに置き換える
3. `depends_on` → `.WithReference()` + `.WaitFor()`
4. `ports` → `.WithHttpEndpoint()`
5. `environment` → `.WithEnvironment()` または `.WithReference()`

---

## 8. 主要 URL

| Resource | URL |
|---|---|
| **Documentation** | https://aspire.dev |
| **Runtime repo** | https://github.com/dotnet/aspire |
| **Docs repo** | https://github.com/microsoft/aspire.dev |
| **Samples** | https://github.com/dotnet/aspire-samples |
| **Community Toolkit** | https://github.com/CommunityToolkit/Aspire |
| **Dashboard image** | `mcr.microsoft.com/dotnet/aspire-dashboard` |
| **Discord** | https://aka.ms/aspire/discord |
| **Reddit** | https://www.reddit.com/r/aspiredotdev/ |

