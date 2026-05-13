---
name: containerize-aspnetcore
description: 'プロジェクト向けにカスタマイズした Dockerfile と .dockerfile ファイルを作成して、ASP.NET Core プロジェクトをコンテナ化します。'
---

# ASP.NET Core Docker コンテナ化プロンプト

## コンテナ化リクエスト

以下の設定で指定された ASP.NET Core (.NET) プロジェクトをコンテナ化してください。アプリケーションを Linux Docker コンテナで実行するために必要な変更**のみに**集中してください。コンテナ化では、ここで指定されたすべての設定を考慮する必要があります。

パフォーマンス、セキュリティ、保守性の観点で最適化されたコンテナになるよう、.NET Core アプリケーションのコンテナ化ベストプラクティスに従ってください。

## コンテナ化設定

このセクションには、ASP.NET Core アプリケーションをコンテナ化するために必要な具体的な設定と構成が含まれます。このプロンプトを実行する前に、必要な情報で設定を埋めてください。多くの場合、最初のいくつかの設定だけで十分です。後半の設定は、対象プロジェクトに該当しない場合はデフォルトのままで構いません。

指定されていない設定はデフォルト値に設定されます。デフォルト値は `[角括弧]` で示されています。

### 基本プロジェクト情報
1. コンテナ化するプロジェクト:
   - `[ProjectName (.csproj ファイルへのパスを指定)]`

2. 使用する .NET バージョン:
   - `[8.0 or 9.0 (Default 8.0)]`

3. 使用する Linux ディストリビューション:
   - `[debian, alpine, ubuntu, chiseled, or Azure Linux (mariner) (Default debian)]`

4. Docker イメージのビルドステージで使用するカスタムベースイメージ（標準の Microsoft ベースイメージを使う場合は "None"）:
   - `[Specify base image to use for build stage (Default None)]`

5. Docker イメージの実行ステージで使用するカスタムベースイメージ（標準の Microsoft ベースイメージを使う場合は "None"）:
   - `[Specify base image to use for run stage (Default None)]`

### コンテナ構成
1. コンテナイメージで公開する必要があるポート:
   - 主要 HTTP ポート: `[e.g., 8080]`
   - 追加ポート: `[List any additional ports, or "None"]`

2. コンテナを実行するユーザーアカウント:
   - `[User account, or default to "$APP_UID"]`

3. アプリケーション URL 構成:
   - `[Specify ASPNETCORE_URLS, or default to "http://+:8080"]`

### ビルド構成
1. コンテナイメージのビルド前に実行する必要があるカスタムビルド手順:
   - `[List any specific build steps, or "None"]`

2. コンテナイメージのビルド後に実行する必要があるカスタムビルド手順:
   - `[List any specific build steps, or "None"]`

3. 構成が必要な NuGet パッケージソース:
   - `[List any private NuGet feeds with authentication details, or "None"]`

### 依存関係
1. コンテナイメージにインストールする必要があるシステムパッケージ:
   - `[Package names for the chosen Linux distribution, or "None"]`

2. コンテナイメージにコピーする必要があるネイティブライブラリ:
   - `[Library names and paths, or "None"]`

3. インストールする必要がある追加の .NET ツール:
   - `[Tool names and versions, or "None"]`

### システム構成
1. コンテナイメージで設定する必要がある環境変数:
   - `[Variable names and values, or "Use defaults"]`

### ファイルシステム
1. コンテナイメージにコピーする必要があるファイル/ディレクトリ:
   - `[Paths relative to project root, or "None"]`
   - コンテナ内の配置先: `[Container paths, or "Not applicable"]`

2. コンテナ化から除外するファイル/ディレクトリ:
   - `[Paths to exclude, or "None"]`

3. 構成する必要があるボリュームマウントポイント:
   - `[Volume paths for persistent data, or "None"]`

### .dockerignore 構成
1. `.dockerignore` ファイルに含めるパターン（.dockerignore にはすでに一般的なデフォルトが含まれているため、ここでは追加分）:
   - 追加パターン: `[List any additional patterns, or "None"]`

### ヘルスチェック構成
1. ヘルスチェックエンドポイント:
   - `[Health check URL path, or "None"]`

2. ヘルスチェック間隔とタイムアウト:
   - `[Interval and timeout values, or "Use defaults"]`

### 追加指示
1. プロジェクトをコンテナ化するために従う必要があるその他の指示:
   - `[Specific requirements, or "None"]`

2. 対応すべき既知の問題:
   - `[Describe any known issues, or "None"]`

## スコープ

- ✅ アプリ設定と接続文字列を環境変数から読み取れるようにするためのアプリ構成変更
- ✅ ASP.NET Core アプリケーション向けの Dockerfile 作成と構成
- ✅ アプリケーションをビルド/公開し、その出力を最終イメージへコピーするための Dockerfile 複数ステージ指定
- ✅ Linux コンテナプラットフォーム互換性の構成（Alpine、Ubuntu、Chiseled、または Azure Linux (Mariner)）
- ✅ 依存関係の適切な処理（システムパッケージ、ネイティブライブラリ、追加ツール）
- ❌ インフラ構築は対象外（別途対応される前提）
- ❌ コンテナ化に必要な範囲を超えるコード変更は対象外

## 実行プロセス

1. 上記のコンテナ化設定を確認し、要件を把握する
2. 変更追跡のため、チェックマーク付きの `progress.md` ファイルを作成する
3. プロジェクトの .csproj ファイル内の `TargetFramework` 要素を確認し、.NET バージョンを判定する
4. 以下に基づいて適切な Linux コンテナイメージを選択する:
   - プロジェクトから検出した .NET バージョン
   - コンテナ化設定で指定された Linux ディストリビューション（Alpine、Ubuntu、Chiseled、または Azure Linux (Mariner)）
   - ユーザーがコンテナ化設定で特定のベースイメージを要求していない場合、ベースイメージは必ず、以下の Dockerfile 例またはドキュメントに示されるタグ付きの有効な mcr.microsoft.com/dotnet イメージであること
   - 公式 Microsoft .NET イメージ（ビルド/ランタイムステージ）:
      - SDK イメージタグ（ビルドステージ用）: https://github.com/dotnet/dotnet-docker/blob/main/README.sdk.md
      - ASP.NET Core ランタイムイメージタグ: https://github.com/dotnet/dotnet-docker/blob/main/README.aspnet.md
      - .NET ランタイムイメージタグ: https://github.com/dotnet/dotnet-docker/blob/main/README.runtime.md
5. プロジェクトディレクトリのルートに Dockerfile を作成してアプリケーションをコンテナ化する
   - Dockerfile は複数ステージを使用すること:
     - ビルドステージ: .NET SDK イメージでアプリケーションをビルド
       - 最初に csproj ファイルをコピー
       - NuGet.config が存在する場合はコピーし、必要なプライベートフィードを構成
       - NuGet パッケージを復元
       - その後、残りのソースコードをコピーしてアプリケーションを /app/publish にビルド・公開
     - 最終ステージ: 選択した .NET ランタイムイメージでアプリケーションを実行
       - 作業ディレクトリを /app に設定
       - 指示されたユーザーを設定（デフォルトでは非 root ユーザー（例: `$APP_UID`））
         - コンテナ化設定で別途指示がない限り、新規ユーザーを作成する必要は*ありません*。ユーザーアカウント指定には `$APP_UID` 変数を使用してください。
       - ビルドステージから公開済み出力を最終イメージへコピー
   - 必ずコンテナ化設定の全要件を考慮すること:
     - .NET バージョンと Linux ディストリビューション
     - 公開ポート
     - コンテナのユーザーアカウント
     - ASPNETCORE_URLS 構成
     - システムパッケージのインストール
     - ネイティブライブラリ依存関係
     - 追加の .NET ツール
     - 環境変数
     - ファイル/ディレクトリのコピー
     - ボリュームマウントポイント
     - ヘルスチェック構成
6. 不要ファイルを Docker イメージから除外するため、プロジェクトディレクトリのルートに `.dockerignore` ファイルを作成する。`.dockerignore` には、コンテナ化設定で指定された追加パターンに加えて、**必ず**少なくとも次を含めること:
   - bin/
   - obj/
   - .dockerignore
   - Dockerfile
   - .git/
   - .github/
   - .vs/
   - .vscode/
   - **/node_modules/
   - *.user
   - *.suo
   - **/.DS_Store
   - **/Thumbs.db
   - コンテナ化設定で指定された追加パターン
7. コンテナ化設定で指定されている場合はヘルスチェックを構成する:
   - ヘルスチェックエンドポイントが指定されている場合、Dockerfile に HEALTHCHECK 命令を追加
   - curl または wget でヘルスエンドポイントを確認
8. タスクを完了済みにする: [ ] → [✓]
9. すべてのタスクが完了し、Docker build が成功するまで継続する

## ビルドとランタイムの検証

Dockerfile 完了後に Docker build が成功することを確認してください。Docker イメージのビルドには次のコマンドを使用します:

```bash
docker build -t aspnetcore-app:latest .
```

ビルドに失敗した場合は、エラーメッセージを確認し、Dockerfile またはプロジェクト構成を必要に応じて調整してください。成功/失敗を報告してください。

## 進捗管理

以下の構成で `progress.md` ファイルを維持してください:
```markdown
# Containerization Progress

## Environment Detection
- [ ] .NET version detection (version: ___)
- [ ] Linux distribution selection (distribution: ___)

## Configuration Changes
- [ ] Application configuration verification for environment variable support
- [ ] NuGet package source configuration (if applicable)

## Containerization
- [ ] Dockerfile creation
- [ ] .dockerignore file creation
- [ ] Build stage created with SDK image
- [ ] csproj file(s) copied for package restore
- [ ] NuGet.config copied if applicable
- [ ] Runtime stage created with runtime image
- [ ] Non-root user configuration
- [ ] Dependency handling (system packages, native libraries, tools, etc.)
- [ ] Health check configuration (if applicable)
- [ ] Special requirements implementation

## Verification
- [ ] Review containerization settings and make sure that all requirements are met
- [ ] Docker build success
```

ステップ間で確認のために停止しないでください。アプリケーションのコンテナ化が完了し、Docker build が成功するまで、体系的に継続してください。

**すべてのチェックボックスにチェックが付くまで完了ではありません！** これには、Docker イメージのビルド成功と、ビルド中に発生したすべての問題への対応が含まれます。

## Dockerfile の例

Linux ベースイメージを使用した ASP.NET Core (.NET) アプリケーション向け Dockerfile の例です。

```dockerfile
# ============================================================
# Stage 1: Build and publish the application
# ============================================================

# Base Image - Select the appropriate .NET SDK version and Linux distribution
# Possible tags include:
# - 8.0-bookworm-slim (Debian 12)
# - 8.0-noble (Ubuntu 24.04)
# - 8.0-alpine (Alpine Linux)
# - 9.0-bookworm-slim (Debian 12)
# - 9.0-noble (Ubuntu 24.04)
# - 9.0-alpine (Alpine Linux)
# Uses the .NET SDK image for building the application
FROM mcr.microsoft.com/dotnet/sdk:8.0-bookworm-slim AS build
ARG BUILD_CONFIGURATION=Release

WORKDIR /src

# Copy project files first for better caching
COPY ["YourProject/YourProject.csproj", "YourProject/"]
COPY ["YourOtherProject/YourOtherProject.csproj", "YourOtherProject/"]

# Copy NuGet configuration if it exists
COPY ["NuGet.config", "."]

# Restore NuGet packages
RUN dotnet restore "YourProject/YourProject.csproj"

# Copy source code
COPY . .

# Perform custom pre-build steps here, if needed
# RUN echo "Running pre-build steps..."

# Build and publish the application
WORKDIR "/src/YourProject"
RUN dotnet build "YourProject.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Publish the application
RUN dotnet publish "YourProject.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Perform custom post-build steps here, if needed
# RUN echo "Running post-build steps..."

# ============================================================
# Stage 2: Final runtime image
# ============================================================

# Base Image - Select the appropriate .NET runtime version and Linux distribution
# Possible tags include:
# - 8.0-bookworm-slim (Debian 12)
# - 8.0-noble (Ubuntu 24.04)
# - 8.0-alpine (Alpine Linux)
# - 8.0-noble-chiseled (Ubuntu 24.04 Chiseled)
# - 8.0-azurelinux3.0 (Azure Linux)
# - 9.0-bookworm-slim (Debian 12)
# - 9.0-noble (Ubuntu 24.04)
# - 9.0-alpine (Alpine Linux)
# - 9.0-noble-chiseled (Ubuntu 24.04 Chiseled)
# - 9.0-azurelinux3.0 (Azure Linux)
# Uses the .NET runtime image for running the application
FROM mcr.microsoft.com/dotnet/aspnet:8.0-bookworm-slim AS final

# Install system packages if needed (uncomment and modify as needed)
# RUN apt-get update && apt-get install -y \
#     curl \
#     wget \
#     ca-certificates \
#     libgdiplus \
#     && rm -rf /var/lib/apt/lists/*

# Install additional .NET tools if needed (uncomment and modify as needed)
# RUN dotnet tool install --global dotnet-ef --version 8.0.0
# ENV PATH="$PATH:/root/.dotnet/tools"

WORKDIR /app

# Copy published application from build stage
COPY --from=build /app/publish .

# Copy additional files if needed (uncomment and modify as needed)
# COPY ./config/appsettings.Production.json .
# COPY ./certificates/ ./certificates/

# Set environment variables
ENV ASPNETCORE_ENVIRONMENT=Production
ENV ASPNETCORE_URLS=http://+:8080

# Add custom environment variables if needed (uncomment and modify as needed)
# ENV CONNECTIONSTRINGS__DEFAULTCONNECTION="your-connection-string"
# ENV FEATURE_FLAG_ENABLED=true

# Configure SSL/TLS certificates if needed (uncomment and modify as needed)
# ENV ASPNETCORE_Kestrel__Certificates__Default__Path=/app/certificates/app.pfx
# ENV ASPNETCORE_Kestrel__Certificates__Default__Password=your_password

# Expose the port the application listens on
EXPOSE 8080
# EXPOSE 8081  # Uncomment if using HTTPS

# Install curl for health checks if not already present
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Configure health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Create volumes for persistent data if needed (uncomment and modify as needed)
# VOLUME ["/app/data", "/app/logs"]

# Switch to non-root user for security
USER $APP_UID

# Set the entry point for the application
ENTRYPOINT ["dotnet", "YourProject.dll"]
```

## この例を適用する際の調整

**注意:** コンテナ化設定の具体的要件に合わせてこのテンプレートをカスタマイズしてください。

この Dockerfile 例を調整する際は、次を行ってください:

1. `YourProject.csproj`、`YourProject.dll` などを実際のプロジェクト名に置き換える
2. 必要に応じて .NET バージョンと Linux ディストリビューションを調整する
3. 要件に応じて依存関係インストール手順を変更し、不要なものを削除する
4. アプリケーション固有の環境変数を構成する
5. ワークフローに応じてステージを追加または削除する
6. アプリケーションのヘルスチェックルートに合わせてヘルスチェックエンドポイントを更新する

## Linux ディストリビューション別バリエーション

### Alpine Linux
より小さいイメージサイズにしたい場合は Alpine Linux を使用できます:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
# ... build steps ...

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS final
# Install packages using apk
RUN apk update && apk add --no-cache curl ca-certificates
```

### Ubuntu Chiseled
攻撃対象領域を最小化するには、chiseled イメージの使用を検討してください:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0-jammy-chiseled AS final
# Note: Chiseled images have minimal packages, so you may need to use a different base for additional dependencies
```

### Azure Linux (Mariner)
Azure 最適化コンテナ向け:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0-azurelinux3.0 AS final
# Install packages using tdnf
RUN tdnf update -y && tdnf install -y curl ca-certificates && tdnf clean all
```

## ステージ命名に関する注意

- `AS stage-name` 構文で各ステージに名前を付けます
- 以前のステージからファイルをコピーするには `--from=stage-name` を使います
- 最終イメージで使わない中間ステージを複数持つこともできます
- `final` ステージが最終的なコンテナイメージになります

## セキュリティのベストプラクティス

- 本番環境では常に非 root ユーザーで実行する
- `latest` ではなく具体的なイメージタグを使う
- インストールするパッケージ数を最小限にする
- ベースイメージを最新に保つ
- マルチステージビルドを使い、ビルド依存関係を最終イメージから除外する
