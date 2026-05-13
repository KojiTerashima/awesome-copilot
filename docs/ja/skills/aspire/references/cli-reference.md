# CLI リファレンス — 完全コマンドリファレンス

Aspire CLI（`aspire`）は、分散アプリケーションの作成・実行・公開のための主要なインターフェイスです。クロスプラットフォーム対応で、スタンドアロンとしてインストールされます（.NET CLI に結合されていませんが、`dotnet` コマンドも使用できます）。

**検証対象:** Aspire CLI 13.1.0

---

## インストール

```bash
# Linux / macOS
curl -sSL https://aspire.dev/install.sh | bash

# Windows PowerShell
irm https://aspire.dev/install.ps1 | iex

# 確認
aspire --version

# CLI 自体を更新
aspire update --self
```

---

## グローバルオプション

すべてのコマンドで以下のオプションをサポートしています。

| Option                | Description                                  |
| --------------------- | -------------------------------------------- |
| `-d, --debug`         | コンソールへのデバッグログ出力を有効化       |
| `--non-interactive`   | すべての対話プロンプトとスピナーを無効化     |
| `--wait-for-debugger` | 実行前にデバッガーのアタッチを待機           |
| `-?, -h, --help`      | ヘルプと使用方法の情報を表示                 |
| `--version`           | バージョン情報を表示                         |

---

## コマンドリファレンス

### `aspire new`

テンプレートから新しいプロジェクトを作成します。

```bash
aspire new [<template>] [options]

# Options:
#   -n, --name <name>        Project name
#   -o, --output <dir>       Output directory
#   -s, --source <source>    NuGet source for templates
#   -v, --version <version>  Version of templates to use
#   --channel <channel>      Channel (stable, daily)

# Examples:
aspire new aspire-starter
aspire new aspire-starter -n MyApp -o ./my-app
aspire new aspire-ts-cs-starter
aspire new aspire-py-starter
aspire new aspire-apphost-singlefile
```

利用可能なテンプレート:

- `aspire-starter` — ASP.NET Core/Blazor スターター + AppHost + テスト
- `aspire-ts-cs-starter` — ASP.NET Core/React + AppHost
- `aspire-py-starter` — FastAPI/React + AppHost
- `aspire-apphost-singlefile` — 空の単一ファイル AppHost

### `aspire init`

既存のプロジェクトまたはソリューションで Aspire を初期化します。

```bash
aspire init [options]

# Options:
#   -s, --source <source>    NuGet source for templates
#   -v, --version <version>  Version of templates to use
#   --channel <channel>      Channel (stable, daily)

# Example:
cd my-existing-solution
aspire init
```

既存のソリューションに AppHost と ServiceDefaults プロジェクトを追加します。どのプロジェクトをオーケストレーション対象にするかは、対話プロンプトで案内されます。

### `aspire run`

DCP（Developer Control Plane）を使って、すべてのリソースをローカルで起動します。

```bash
aspire run [options] [-- <additional arguments>]

# Options:
#   --project <path>       Path to AppHost project file

# Examples:
aspire run
aspire run --project ./src/MyApp.AppHost
```

動作:

1. AppHost プロジェクトをビルド
2. DCP エンジンを起動
3. 依存関係順（DAG）でリソースを作成
4. ゲート対象リソースのヘルスチェックを待機
5. 既定のブラウザーでダッシュボードを開く
6. ログをターミナルにストリーム出力

`Ctrl+C` を押すと、すべてのリソースを安全に停止します。

### `aspire add`

AppHost にホスティング統合を追加します。

```bash
aspire add [<integration>] [options]

# Options:
#   --project <path>         Target project file
#   -v, --version <version>  Version of integration to add
#   -s, --source <source>    NuGet source for integration

# Examples:
aspire add redis
aspire add postgresql
aspire add mongodb
```

### `aspire publish`（プレビュー）

AppHost のリソースモデルからデプロイマニフェストを生成します。

```bash
aspire publish [options] [-- <additional arguments>]

# Options:
#   --project <path>                   Path to AppHost project file
#   -o, --output-path <path>           Output directory (default: ./aspire-output)
#   --log-level <level>                Log level (trace, debug, information, warning, error, critical)
#   -e, --environment <env>            Environment (default: Production)
#   --include-exception-details        Include stack traces in pipeline logs

# Examples:
aspire publish
aspire publish --output-path ./deploy
aspire publish -e Staging
```

### `aspire config`

Aspire の設定値を管理します。

```bash
aspire config <subcommand>

# Subcommands:
#   get <key>              Get a configuration value
#   set <key> <value>      Set a configuration value
#   list                   List all configuration values
#   delete <key>           Delete a configuration value

# Examples:
aspire config list
aspire config set telemetry.enabled false
aspire config get telemetry.enabled
aspire config delete telemetry.enabled
```

### `aspire cache`

CLI 操作用のディスクキャッシュを管理します。

```bash
aspire cache <subcommand>

# Subcommands:
#   clear                  Clear all cache entries

# Example:
aspire cache clear
```

### `aspire deploy`（プレビュー）

Aspire apphost の内容を、定義されたデプロイ先へデプロイします。

```bash
aspire deploy [options] [-- <additional arguments>]

# Options:
#   --project <path>                   Path to AppHost project file
#   -o, --output-path <path>           Output path for deployment artifacts
#   --log-level <level>                Log level (trace, debug, information, warning, error, critical)
#   -e, --environment <env>            Environment (default: Production)
#   --include-exception-details        Include stack traces in pipeline logs
#   --clear-cache                      Clear deployment cache for current environment

# Example:
aspire deploy --project ./src/MyApp.AppHost
```

### `aspire do`（プレビュー）

特定のパイプラインステップとその依存関係を実行します。

```bash
aspire do <step> [options] [-- <additional arguments>]

# Options:
#   --project <path>                   Path to AppHost project file
#   -o, --output-path <path>           Output path for artifacts
#   --log-level <level>                Log level (trace, debug, information, warning, error, critical)
#   -e, --environment <env>            Environment (default: Production)
#   --include-exception-details        Include stack traces in pipeline logs

# Example:
aspire do build-images --project ./src/MyApp.AppHost
```

### `aspire update`（プレビュー）

Aspire プロジェクト内の統合を更新するか、CLI 自体を更新します。

```bash
aspire update [options]

# Options:
#   --project <path>       Path to AppHost project file
#   --self                 Update the Aspire CLI itself to the latest version
#   --channel <channel>    Channel to update to (stable, daily)

# Examples:
aspire update                          # Update project integrations
aspire update --self                   # Update the CLI itself
aspire update --self --channel daily   # Update CLI to daily build
```

### `aspire mcp`

MCP（Model Context Protocol）サーバーを管理します。

```bash
aspire mcp <subcommand>

# Subcommands:
#   init    Initialize MCP server configuration for detected agent environments
#   start   Start the MCP server
```

#### `aspire mcp init`

```bash
aspire mcp init

# Interactive — detects your AI environment and creates config files.
# Supported environments:
# - VS Code (GitHub Copilot)
# - Copilot CLI
# - Claude Code
# - OpenCode
```

検出された AI ツールに応じた設定ファイルを生成します。  
詳細は [MCP Server](mcp-server.md) を参照してください。

#### `aspire mcp start`

```bash
aspire mcp start

# Starts the MCP server using STDIO transport.
# This is typically invoked by your AI tool, not run manually.
```

---

## 存在しないコマンド

以下のコマンドは Aspire CLI 13.1 では **無効** です。代替を使用してください。

| Invalid Command | Alternative                                                          |
| --------------- | -------------------------------------------------------------------- |
| `aspire build`  | `dotnet build ./AppHost` を使用                                     |
| `aspire test`   | `dotnet test ./Tests` を使用                                        |
| `aspire dev`    | `aspire run` を使用（ファイル監視を含む）                           |
| `aspire list`   | テンプレートは `aspire new --help`、統合は `aspire add` を使用      |

---

## .NET CLI の同等コマンド

`dotnet` CLI でも、Aspire の一部タスクを実行できます。

| Aspire CLI                  | .NET CLI Equivalent              |
| --------------------------- | -------------------------------- |
| `aspire new aspire-starter` | `dotnet new aspire-starter`      |
| `aspire run`                | `dotnet run --project ./AppHost` |
| N/A                         | `dotnet build ./AppHost`         |
| N/A                         | `dotnet test ./Tests`            |

Aspire CLI は、`dotnet` に直接の同等機能がない `publish`、`deploy`、`add`、`mcp`、`config`、`cache`、`do`、`update` によって価値を提供します。

