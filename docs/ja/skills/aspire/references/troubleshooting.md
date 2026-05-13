# トラブルシューティング — 診断とよくある問題

---

## 診断コード

Aspire は一般的な問題に対して診断コードを出力します。これらはビルドの警告/エラーや IDE の診断に表示されます。

### 標準診断

| Code          | Severity | Description                                                |
| ------------- | -------- | ---------------------------------------------------------- |
| **ASPIRE001** | Warning  | リソース名に無効な文字が含まれています                    |
| **ASPIRE002** | Warning  | 重複したリソース名が検出されました                         |
| **ASPIRE003** | Error    | 必須のパッケージ参照が不足しています                       |
| **ASPIRE004** | Warning  | 非推奨 API を使用しています                                |
| **ASPIRE005** | Error    | エンドポイント構成が無効です                               |
| **ASPIRE006** | Warning  | `.WaitFor()` を持つリソースにヘルスチェックが構成されていません |
| **ASPIRE007** | Warning  | コンテナーイメージのタグが指定されていません（`latest` を使用） |
| **ASPIRE008** | Error    | リソースグラフで循環依存が検出されました                   |

### 実験的診断 (ASPIREHOSTINGX\*)

これらのコードは、実験的/プレビュー API の使用を示します。意図的に実験的機能を使用する場合は、`#pragma warning disable` または `<NoWarn>` が必要になることがあります。

| Code                      | Area                             |
| ------------------------- | -------------------------------- |
| ASPIRE_HOSTINGX_0001–0005 | 実験的ホスティング API           |
| ASPIRE_HOSTINGX_0006–0010 | 実験的統合 API                   |
| ASPIRE_HOSTINGX_0011–0015 | 実験的デプロイ API               |
| ASPIRE_HOSTINGX_0016–0022 | 実験的リソースモデル API         |

実験的警告を抑制するには:

```xml
<!-- In .csproj -->
<PropertyGroup>
  <NoWarn>$(NoWarn);ASPIRE_HOSTINGX_0001</NoWarn>
</PropertyGroup>
```

または行単位で:

```csharp
#pragma warning disable ASPIRE_HOSTINGX_0001
var resource = builder.AddExperimentalResource("test");
#pragma warning restore ASPIRE_HOSTINGX_0001
```

---

## よくある問題と解決策

### コンテナーランタイム

| Problem                           | Solution                                                                                               |
| --------------------------------- | ------------------------------------------------------------------------------------------------------ |
| "Cannot connect to Docker daemon" | Docker Desktop / Podman / Rancher Desktop を起動してください                                           |
| Container fails to start          | `docker ps -a` で終了コードを確認し、ダッシュボードのコンソールログを確認してください                 |
| Port already in use               | 別プロセスがポートを使用しています。Aspire は自動割り当てしますが、`targetPort` はコンテナー上で空いている必要があります |
| Container image pull fails        | ネットワーク接続を確認し、イメージ名とタグを検証してください                                           |
| "Permission denied" on Linux      | ユーザーを `docker` グループに追加: `sudo usermod -aG docker $USER`                                   |

### サービスディスカバリー

| Problem                       | Solution                                                                     |
| ----------------------------- | ---------------------------------------------------------------------------- |
| Service can't find dependency | AppHost の `.WithReference()` を確認し、ダッシュボードの環境変数を確認してください |
| Connection string is null     | 参照リソース名が一致していません。`ConnectionStrings__<name>` を確認してください |
| Wrong port in service URL     | `targetPort` と実際のサービス待受ポートを確認してください                   |
| Env var not set               | AppHost を再ビルドし、リソース名が完全一致していることを確認してください    |

### Python ワークロード

| Problem                           | Solution                                                        |
| --------------------------------- | --------------------------------------------------------------- |
| "Python not found"                | Python が PATH 上にあることを確認し、`AddPythonApp()` にフルパスを指定してください |
| venv not found                    | `.WithVirtualEnvironment()` を使用するか、venv を手動で作成してください |
| pip packages fail to install      | `.WithPipPackages()` を使用するか、`aspire run` の前に venv にインストールしてください |
| ModuleNotFoundError               | venv が有効化されていません。`.WithVirtualEnvironment()` がこれを処理します |
| "Port already in use" for Uvicorn | `targetPort` を確認してください。別インスタンスが動作している可能性があります |

### JavaScript / TypeScript ワークロード

| Problem                       | Solution                                                         |
| ----------------------------- | ---------------------------------------------------------------- |
| "node_modules not found"      | `.WithNpmPackageInstallation()` を使用して自動インストールしてください |
| npm install fails             | `package.json` が有効か確認し、npm レジストリへの接続性を確認してください |
| Vite dev server won't start   | `vite` が devDependencies にあることと Vite 設定を確認してください |
| Port mismatch                 | `targetPort` が JS フレームワーク設定内のポートと一致していることを確認してください |
| TypeScript compilation errors | これらは Aspire ではなくサービス側で発生します。サービスログを確認してください |

### Go ワークロード

| Problem                    | Solution                                                   |
| -------------------------- | ---------------------------------------------------------- |
| "go not found"             | Go がインストールされ PATH にあることを確認してください   |
| Build fails                | 作業ディレクトリに `go.mod` が存在することを確認してください |
| "no Go files in directory" | `workingDir` が `main.go` のあるディレクトリを指しているか確認してください |

### Java ワークロード

| Problem                  | Solution                                                |
| ------------------------ | ------------------------------------------------------- |
| "java not found"         | JDK がインストールされ、`JAVA_HOME` が設定されていることを確認してください |
| Maven/Gradle build fails | ビルドファイルの存在とビルドツールのインストールを確認してください |
| Spring Boot won't start  | `application.properties` を確認し、メインクラスを検証してください |

### Rust ワークロード

| Problem              | Solution                                                             |
| -------------------- | -------------------------------------------------------------------- |
| "cargo not found"    | rustup で Rust をインストールしてください                            |
| Build takes too long | Rust のコンパイル時間は通常長めです。事前ビルドに `.WithCargoBuild()` を使用してください |

### ヘルスチェックと起動

| Problem                      | Solution                                                                       |
| ---------------------------- | ------------------------------------------------------------------------------ |
| Resource stuck in "Starting" | ヘルスチェックエンドポイントが応答していません。サービスログを確認してください |
| `.WaitFor()` timeout         | タイムアウトを延長するかヘルスエンドポイントを修正してください。既定は 30 秒です |
| Health check always fails    | エンドポイントパス（既定: `/health`）と、サービスが正しいポートにバインドしているか確認してください |
| Cascading startup failures   | 依存先が失敗しています。まず根本のリソースを確認してください                    |

### ダッシュボード

| Problem                               | Solution                                                                  |
| ------------------------------------- | ------------------------------------------------------------------------- |
| Dashboard doesn't open                | ターミナルの URL を確認し、固定ポートには `--dashboard-port` を使用してください |
| No logs appearing                     | サービスが stdout/stderr に出力していない可能性があります。コンソール出力を確認してください |
| No traces for non-.NET services       | サービスで OpenTelemetry SDK を構成してください。詳細は [Dashboard](dashboard.md) を参照 |
| Traces don't show cross-service calls | トレースコンテキストヘッダー（`traceparent`, `tracestate`）を伝搬してください |

### ビルドと構成

| Problem                                   | Solution                                                            |
| ----------------------------------------- | ------------------------------------------------------------------- |
| "Project not found" for `AddProject<T>()` | `.csproj` がソリューションに含まれ、AppHost から参照されていることを確認してください |
| Package version conflicts                 | すべての Aspire パッケージを同じバージョンに固定してください        |
| AppHost won't build                       | プロジェクトに `Aspire.AppHost.Sdk` があることを確認し、`dotnet restore` を実行してください |
| `aspire run` build error                  | まずビルドエラーを修正してください。`aspire run` にはビルド成功が必要です |

### デプロイ

| Problem                                  | Solution                                                             |
| ---------------------------------------- | -------------------------------------------------------------------- |
| `aspire publish` fails                   | パブリッシャーパッケージがインストールされているか確認してください（例: `Aspire.Hosting.Docker`） |
| Generated Bicep has errors               | サポートされていないリソース構成がないか確認してください            |
| Container image push fails               | レジストリの資格情報と権限を確認してください                        |
| Missing connection strings in deployment | 生成された ConfigMaps/Secrets がリソース名と一致しているか確認してください |

---

## デバッグ戦略

### 1. まずダッシュボードを確認する

ダッシュボードには、リソース状態、ログ、トレース、メトリクスが表示されます。問題があればまずここから確認してください。

### 2. 環境変数を確認する

ダッシュボードでリソースをクリックすると、注入されたすべての環境変数を確認できます。接続文字列とサービス URL が正しいか検証してください。

### 3. コンソールログを読む

Dashboard → Console Logs → 失敗しているリソースでフィルター。生の stdout/stderr に根本原因が含まれていることがよくあります。

### 4. DAG を確認する

サービスが起動しない場合は依存順序を確認してください。依存先の失敗は下流のすべてのリソースをブロックします。

### 5. AI 支援デバッグに MCP を使う

MCP が構成済みの場合（[MCP Server](mcp-server.md) を参照）、AI アシスタントに次のように尋ねてください。

- 「どのリソースが失敗していますか？」
- 「[service] のログを表示して」
- 「どのトレースでエラーが出ていますか？」

### 6. 問題を切り分ける

AppHost で他のリソースをコメントアウトし、失敗しているリソースだけを実行してください。これにより、問題がリソース自体か依存関係かを絞り込めます。

---

## ヘルプの取得

| Channel                 | URL                                            |
| ----------------------- | ---------------------------------------------- |
| GitHub Issues (runtime) | https://github.com/dotnet/aspire/issues        |
| GitHub Issues (docs)    | https://github.com/microsoft/aspire.dev/issues |
| Discord                 | https://aka.ms/aspire/discord                  |
| Stack Overflow          | タグ: `dotnet-aspire`                           |
| Reddit                  | https://www.reddit.com/r/aspiredotdev/         |

