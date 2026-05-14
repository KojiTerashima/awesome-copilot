---
name: containerize-aspnet-framework
description: 'プロジェクト向けにカスタマイズされた Dockerfile と .dockerfile ファイルを作成して、ASP.NET .NET Framework プロジェクトをコンテナ化します。'
---

# ASP.NET .NET Framework コンテナ化プロンプト

以下のコンテナ化設定で指定された ASP.NET（.NET Framework）プロジェクトをコンテナ化してください。アプリケーションを Windows Docker コンテナで実行するために必要な変更**のみ**に集中してください。コンテナ化では、ここで指定されるすべての設定を考慮する必要があります。

**重要:** これは .NET Core ではなく .NET Framework アプリケーションです。コンテナ化の手順は .NET Core アプリケーションとは異なります。

## コンテナ化設定

このプロンプトのこのセクションには、ASP.NET（.NET Framework）アプリケーションをコンテナ化するために必要な具体的な設定と構成が含まれます。このプロンプトを実行する前に、必要な情報で設定が入力されていることを確認してください。多くの場合、最初の数項目のみが必要です。後半の設定は、対象プロジェクトに該当しない場合はデフォルトのままで構いません。

指定されていない設定はデフォルト値が適用されます。デフォルト値は `[角括弧]` で示されています。

### 基本プロジェクト情報
1. コンテナ化するプロジェクト:
   - `[ProjectName (.csproj ファイルへのパスを指定)]`

2. 使用する Windows Server SKU:
   - `[Windows Server Core (Default) または Windows Server Full]`

3. 使用する Windows Server バージョン:
   - `[2022, 2019, または 2016 (Default 2022)]`

4. Docker イメージのビルドステージ用カスタムベースイメージ（標準 Microsoft ベースイメージを使う場合は "None"）:
   - `[ビルドステージで使用するベースイメージを指定 (Default None)]`

5. Docker イメージの実行ステージ用カスタムベースイメージ（標準 Microsoft ベースイメージを使う場合は "None"）:
   - `[実行ステージで使用するベースイメージを指定 (Default None)]`

### コンテナ構成
1. コンテナイメージで公開が必要なポート:
   - メイン HTTP ポート: `[例: 80]`
   - 追加ポート: `[追加ポートを列挙、または "None"]`

2. コンテナを実行するユーザーアカウント:
   - `[ユーザーアカウント、またはデフォルトの "ContainerUser"]`

3. コンテナイメージで構成が必要な IIS 設定:
   - `[特定の IIS 設定を列挙、または "None"]`

### ビルド構成
1. コンテナイメージのビルド前に実行が必要なカスタムビルド手順:
   - `[特定のビルド手順を列挙、または "None"]`

2. コンテナイメージのビルド後に実行が必要なカスタムビルド手順:
   - `[特定のビルド手順を列挙、または "None"]`

### 依存関係
1. コンテナイメージ内で GAC に登録すべき .NET アセンブリ:
   - `[アセンブリ名とバージョン、または "None"]`

2. コンテナイメージにコピーしてインストールすべき MSI:
   - `[MSI 名とバージョン、または "None"]`

3. コンテナイメージ内で登録すべき COM コンポーネント:
   - `[COM コンポーネント名、または "None"]`

### システム構成
1. コンテナイメージに追加すべきレジストリキーと値:
   - `[レジストリパスと値、または "None"]`

2. コンテナイメージに設定すべき環境変数:
   - `[変数名と値、または "Use defaults"]`

3. コンテナイメージにインストールすべき Windows Server のロールと機能:
   - `[ロール/機能名、または "None"]`

### ファイルシステム
1. コンテナイメージにコピーする必要があるファイル/ディレクトリ:
   - `[プロジェクトルートからの相対パス、または "None"]`
   - コンテナ内のコピー先: `[コンテナ内パス、または "Not applicable"]`

2. コンテナ化から除外するファイル/ディレクトリ:
   - `[除外するパス、または "None"]`

### .dockerignore 設定
1. `.dockerignore` ファイルに含めるパターン（.dockerignore には一般的なデフォルトがすでにあるため、ここでは追加パターンを指定）:
   - 追加パターン: `[追加パターンを列挙、または "None"]`

### ヘルスチェック設定
1. ヘルスチェックエンドポイント:
   - `[ヘルスチェック URL パス、または "None"]`

2. ヘルスチェック間隔とタイムアウト:
   - `[間隔とタイムアウト値、または "Use defaults"]`

### 追加指示
1. プロジェクトをコンテナ化する際に従うべきその他の指示:
   - `[具体的な要件、または "None"]`

2. 対応すべき既知の問題:
   - `[既知の問題を記述、または "None"]`

## スコープ

- ✅ 環境変数から app settings と connection strings を読み取るために config builders を使用するようにするアプリ構成の変更
- ✅ ASP.NET アプリケーション向け Dockerfile の作成と設定
- ✅ Dockerfile で複数ステージを指定し、アプリケーションの build/publish と最終イメージへの出力コピーを実施
- ✅ Windows コンテナプラットフォーム互換性の設定（Windows Server Core または Full）
- ✅ 依存関係の適切な処理（GAC アセンブリ、MSI、COM コンポーネント）
- ❌ インフラ構築は対象外（別途対応される前提）
- ❌ コンテナ化に必要な範囲を超えるコード変更は行わない

## 実行プロセス

1. 上記のコンテナ化設定を確認し、要件を把握する
2. チェックマークで変更を追跡するため `progress.md` ファイルを作成する
3. プロジェクトの .csproj ファイルで `TargetFrameworkVersion` 要素を確認し、.NET Framework バージョンを特定する
4. 以下に基づいて適切な Windows Server コンテナイメージを選択する:
   - プロジェクトから検出した .NET Framework バージョン
   - コンテナ化設定で指定された Windows Server SKU（Core または Full）
   - コンテナ化設定で指定された Windows Server バージョン（2016、2019、2022）
   - Windows Server Core タグ一覧: https://github.com/microsoft/dotnet-framework-docker/blob/main/README.aspnet.md#full-tag-listing
5. 必要な NuGet パッケージがインストールされていることを確認する。欠けていても**絶対に**インストールしないこと。未インストールの場合はユーザーが手動でインストールする必要がある。未インストールならこのプロンプトの実行を一時停止し、Visual Studio NuGet Package Manager または Visual Studio package manager console を使ってインストールするようユーザーに依頼する。必要なパッケージは次のとおり:
   - `Microsoft.Configuration.ConfigurationBuilders.Environment`
6. `web.config` を変更し、環境変数から app settings と connection strings を読み取るための configuration builders セクションと設定を追加する:
   - configSections に ConfigBuilders セクションを追加
   - ルートに configBuilders セクションを追加
   - appSettings と connectionStrings の両方に EnvironmentConfigBuilder を設定
   - 例:
     ```xml
     <configSections>
       <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
     </configSections>
     <configBuilders>
       <builders>
         <add name="Environment" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
       </builders>
     </configBuilders>
     <appSettings configBuilders="Environment">
       <!-- existing app settings -->
     </appSettings>
     <connectionStrings configBuilders="Environment">
       <!-- existing connection strings -->
     </connectionStrings>
     ```
7. Dockerfile を作成するフォルダーに `LogMonitorConfig.json` ファイルを作成し、このプロンプト末尾の参照 `LogMonitorConfig.json` をコピーする。コンテナ化設定で別途指定がない限り、このファイル内容は**変更してはならず**、参照内容と完全に一致している必要がある。
   - 特に、記録する issue レベルを変更しないこと。EventLog ソースで `Information` レベルを使うと不要なノイズが発生する。
8. アプリケーションをコンテナ化するため、プロジェクトディレクトリのルートに Dockerfile を作成する
   - Dockerfile は複数ステージを使用すること:
     - Build ステージ: Windows Server Core イメージでアプリケーションをビルド
       - 設定ファイルでカスタムベースイメージが指定されていない限り、build ステージは `mcr.microsoft.com/dotnet/framework/sdk` ベースイメージを必ず使用すること
       - 最初に sln、csproj、packages.config ファイルをコピー
       - NuGet.config が存在する場合はコピーし、必要に応じてプライベートフィードを設定
       - NuGet パッケージを復元
       - その後、残りのソースコードをコピーし、MSBuild でアプリケーションを C:\publish に build/publish
     - Final ステージ: 選択した Windows Server イメージでアプリケーションを実行
       - 設定ファイルでカスタムベースイメージが指定されていない限り、final ステージは `mcr.microsoft.com/dotnet/framework/aspnet` ベースイメージを必ず使用すること
       - `LogMonitorConfig.json` をコンテナ内ディレクトリ（例: C:\LogMonitor）へコピー
       - Microsoft リポジトリから同じディレクトリに LogMonitor.exe をダウンロード
           - 正しい LogMonitor.exe URL: https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe
       - 作業ディレクトリを C:\inetpub\wwwroot に設定
       - build ステージで publish した出力（C:\publish）を final イメージへコピー
       - IIS サービス監視のため、LogMonitor.exe と ServiceMonitor.exe を実行するようコンテナのエントリポイントを設定
           - `ENTRYPOINT [ "C:\\LogMonitor\\LogMonitor.exe", "C:\\ServiceMonitor.exe", "w3svc" ]`
   - コンテナ化設定のすべての要件を必ず考慮すること:
     - Windows Server SKU とバージョン
     - 公開ポート
     - コンテナ実行ユーザー
     - IIS 設定
     - GAC アセンブリ登録
     - MSI インストール
     - COM コンポーネント登録
     - レジストリキー
     - 環境変数
     - Windows ロールと機能
     - ファイル/ディレクトリのコピー
   - このプロンプト末尾の例をベースに Dockerfile を作成するが、対象プロジェクトの要件と設定に合わせて必ずカスタマイズすること。
   - **重要:** 設定ファイルでユーザーが **明示的に要求** していない限り、Windows Server Core ベースイメージを使用すること
9. 不要なファイルを Docker イメージから除外するため、プロジェクトディレクトリのルートに `.dockerignore` ファイルを作成する。`.dockerignore` には、コンテナ化設定で指定された追加パターンに加えて、少なくとも以下を**必ず**含めること:
   - packages/
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
10. 設定で指定されている場合はヘルスチェックを構成する:
   - ヘルスチェックエンドポイントが提供されている場合は Dockerfile に HEALTHCHECK 命令を追加
11. プロジェクトファイルに次の項目を追加して dockerfile をプロジェクトに追加する: `<None Include="Dockerfile" />`
12. タスク完了時にチェックボックスを更新する: [ ] → [✓]
13. すべてのタスクが完了し Docker build が成功するまで継続する

## ビルドとランタイムの検証

Dockerfile 完成後、Docker build が成功することを確認する。次のコマンドで Docker イメージをビルドする:

```bash
docker build -t aspnet-app:latest .
```

ビルドに失敗した場合はエラーメッセージを確認し、Dockerfile またはプロジェクト構成を必要に応じて調整する。成功/失敗を報告すること。

## 進捗管理

次の構成で `progress.md` ファイルを維持する:
```markdown
# Containerization Progress

## Environment Detection
- [ ] .NET Framework version detection (version: ___)
- [ ] Windows Server SKU selection (SKU: ___)
- [ ] Windows Server version selection (Version: ___)

## Configuration Changes
- [ ] Web.config modifications for configuration builders
- [ ] NuGet package source configuration (if applicable)
- [ ] Copy LogMonitorConfig.json and adjust if required by settings

## Containerization
- [ ] Dockerfile creation
- [ ] .dockerignore file creation
- [ ] Build stage created with SDK image
- [ ] sln, csproj, packages.config, and (if applicable) NuGet.config copied for package restore
- [ ] Runtime stage created with runtime image
- [ ] Non-root user configuration
- [ ] Dependency handling (GAC, MSI, COM, registry, additional files, etc.)
- [ ] Health check configuration (if applicable)
- [ ] Special requirements implementation

## Verification
- [ ] Review containerization settings and make sure that all requirements are met
- [ ] Docker build success
```

手順間で確認待ちのために停止しないこと。アプリケーションのコンテナ化が完了し、Docker build が成功するまで、体系的に継続すること。

**すべてのチェックボックスにチェックが入るまで完了ではありません！** これには、Docker イメージのビルド成功と、ビルド中に発生した問題への対処が含まれます。

## 参考資料

### Dockerfile の例

Windows Server Core ベースイメージを使用した ASP.NET（.NET Framework）アプリケーション向け Dockerfile の例です。

```dockerfile
# escape=`
# The escape directive changes the escape character from \ to `
# This is especially useful in Windows Dockerfiles where \ is the path separator

# ============================================================
# Stage 1: Build and publish the application
# ============================================================

# Base Image - Select the appropriate .NET Framework version and Windows Server Core version
# Possible tags include:
# - 4.8.1-windowsservercore-ltsc2025 (Windows Server 2025)
# - 4.8-windowsservercore-ltsc2022 (Windows Server 2022)
# - 4.8-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.8-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.2-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.7.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.1-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.6.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 3.5-windowsservercore-ltsc2025 (Windows Server 2025)
# - 3.5-windowsservercore-ltsc2022 (Windows Server 2022)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2019)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2016)
# Uses the .NET Framework SDK image for building the application
FROM mcr.microsoft.com/dotnet/framework/sdk:4.8-windowsservercore-ltsc2022 AS build
ARG BUILD_CONFIGURATION=Release

# Set the default shell to PowerShell
SHELL ["powershell", "-command"]

WORKDIR /app

# Copy the solution and project files
COPY YourSolution.sln .
COPY YourProject/*.csproj ./YourProject/
COPY YourOtherProject/*.csproj ./YourOtherProject/

# Copy packages.config files
COPY YourProject/packages.config ./YourProject/
COPY YourOtherProject/packages.config ./YourOtherProject/

# Restore NuGet packages
RUN nuget restore YourSolution.sln

# Copy source code
COPY . .

# Perform custom pre-build steps here, if needed

# Build and publish the application to C:\publish
RUN msbuild /p:Configuration=$BUILD_CONFIGURATION `
            /p:WebPublishMethod=FileSystem `
            /p:PublishUrl=C:\publish `
            /p:DeployDefaultTarget=WebPublish

# Perform custom post-build steps here, if needed

# ============================================================
# Stage 2: Final runtime image
# ============================================================

# Base Image - Select the appropriate .NET Framework version and Windows Server Core version
# Possible tags include:
# - 4.8.1-windowsservercore-ltsc2025 (Windows Server 2025)
# - 4.8-windowsservercore-ltsc2022 (Windows Server 2022)
# - 4.8-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.8-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.2-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.7.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.1-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.6.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 3.5-windowsservercore-ltsc2025 (Windows Server 2025)
# - 3.5-windowsservercore-ltsc2022 (Windows Server 2022)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2019)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2016)
# Uses the .NET Framework ASP.NET image for running the application
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2022

# Set the default shell to PowerShell
SHELL ["powershell", "-command"]

WORKDIR /inetpub/wwwroot

# Copy from build stage
COPY --from=build /publish .

# Add any additional environment variables needed for your application (uncomment and modify as needed)
# ENV KEY=VALUE

# Install MSI packages (uncomment and modify as needed)
# COPY ./msi-installers C:/Installers
# RUN Start-Process -Wait -FilePath 'msiexec.exe' -ArgumentList '/i', 'C:\Installers\your-package.msi', '/quiet', '/norestart'

# Install custom Windows Server roles and features (uncomment and modify as needed)
# RUN dism /Online /Enable-Feature /FeatureName:YOUR-FEATURE-NAME

# Add additional Windows features (uncomment and modify as needed)
# RUN Add-WindowsFeature Some-Windows-Feature; `
#    Add-WindowsFeature Another-Windows-Feature

# Install MSI packages if needed (uncomment and modify as needed)
# COPY ./msi-installers C:/Installers
# RUN Start-Process -Wait -FilePath 'msiexec.exe' -ArgumentList '/i', 'C:\Installers\your-package.msi', '/quiet', '/norestart'

# Register assemblies in GAC if needed (uncomment and modify as needed)
# COPY ./assemblies C:/Assemblies
# RUN C:\Windows\Microsoft.NET\Framework64\v4.0.30319\gacutil -i C:/Assemblies/YourAssembly.dll

# Register COM components if needed (uncomment and modify as needed)
# COPY ./com-components C:/Components
# RUN regsvr32 /s C:/Components/YourComponent.dll

# Add registry keys if needed (uncomment and modify as needed)
# RUN New-Item -Path 'HKLM:\Software\YourApp' -Force; `
#     Set-ItemProperty -Path 'HKLM:\Software\YourApp' -Name 'Setting' -Value 'Value'

# Configure IIS settings if needed (uncomment and modify as needed)
# RUN Import-Module WebAdministration; `
#     Set-ItemProperty 'IIS:\AppPools\DefaultAppPool' -Name somePropertyName -Value 'SomePropertyValue'; `
#     Set-ItemProperty 'IIS:\Sites\Default Web Site' -Name anotherPropertyName -Value 'AnotherPropertyValue'

# Expose necessary ports - By default, IIS uses port 80
EXPOSE 80
# EXPOSE 443  # Uncomment if using HTTPS

# Copy LogMonitor from the microsoft/windows-container-tools repository
WORKDIR /LogMonitor
RUN curl -fSLo LogMonitor.exe https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe

# Copy LogMonitorConfig.json from local files
COPY LogMonitorConfig.json .

# Set non-administrator user
USER ContainerUser

# Override the container's default entry point to take advantage of the LogMonitor
ENTRYPOINT [ "C:\\LogMonitor\\LogMonitor.exe", "C:\\ServiceMonitor.exe", "w3svc" ]
```

## この例の適用方法

**注:** このテンプレートは、コンテナ化設定にある具体的な要件に基づいてカスタマイズしてください。

この Dockerfile 例を適用する際は:

1. `YourSolution.sln`、`YourProject.csproj` などを実際のファイル名に置き換える
2. 必要に応じて Windows Server と .NET Framework のバージョンを調整する
3. 要件に応じて依存関係インストール手順を修正し、不要なものは削除する
4. ワークフローに応じてステージを追加または削除する

## ステージ命名に関する注意

- `AS stage-name` 構文で各ステージに名前を付ける
- 前のステージからファイルをコピーするには `--from=stage-name` を使用する
- 最終イメージで使わない中間ステージを複数持つこともできる

### LogMonitorConfig.json

LogMonitorConfig.json ファイルはプロジェクトディレクトリのルートに作成する必要があります。このファイルはコンテナ内ログを監視する LogMonitor ツールの構成に使用されます。適切なログ機能を確保するため、このファイルの内容は次のとおり**完全一致**である必要があります:
```json
{
  "LogConfig": {
    "sources": [
      {
        "type": "EventLog",
        "startAtOldestRecord": true,
        "eventFormatMultiLine": false,
        "channels": [
          {
            "name": "system",
            "level": "Warning"
          },
          {
            "name": "application",
            "level": "Error"
          }
        ]
      },
      {
        "type": "File",
        "directory": "c:\\inetpub\\logs",
        "filter": "*.log",
        "includeSubdirectories": true,
        "includeFileNames": false
      },
      {
        "type": "ETW",
        "eventFormatMultiLine": false,
        "providers": [
          {
            "providerName": "IIS: WWW Server",
            "providerGuid": "3A2A4E84-4C21-4981-AE10-3FDA0D9B0F83",
            "level": "Information"
          },
          {
            "providerName": "Microsoft-Windows-IIS-Logging",
            "providerGuid": "7E8AD27F-B271-4EA2-A783-A47BDE29143B",
            "level": "Information"
          }
        ]
      }
    ]
  }
}
```
