---
name: microsoft-docs
description: 'Azure、.NET、Agent Framework、Aspire、VS Code、GitHubなどの公式Microsoftドキュメントから概念、チュートリアル、コード例を検索します。デフォルトはMicrosoft Learn MCPを使用し、learn.microsoft.com外のコンテンツにはContext7やAspire MCPを使用します。'
---

# Microsoft Docs

Microsoftテクノロジーエコシステム向けのリサーチスキル。learn.microsoft.comおよびそれ以外のドキュメント（VS Code、GitHub、Aspire、Agent Frameworkリポジトリ）をカバーします。

---

## デフォルト: Microsoft Learn MCP

learn.microsoft.com上の**すべてのコンテンツ**（Azure、.NET、M365、Power Platform、Agent Framework、Semantic Kernel、Windowsなど）に対してこれらのツールを使用します。Microsoftドキュメントの大多数のクエリに対する主要ツールです。

| ツール | 用途 |
|------|---------|
| `microsoft_docs_search` | learn.microsoft.comを検索 — 概念、ガイド、チュートリアル、設定 |
| `microsoft_code_sample_search` | Learnドキュメントから動作するコードスニペットを検索。最良の結果のために`language`（`python`、`csharp`など）を指定 |
| `microsoft_docs_fetch` | 特定のURLから完全なページ内容を取得（検索抜粋が不十分な場合） |

完全なチュートリアルやすべての設定オプションが必要な場合、または検索抜粋が切り詰められている場合は、検索後に`microsoft_docs_fetch`を使用してください。

### CLI代替手段

Learn MCPサーバーが利用できない場合は、ターミナルやシェル（例：Bash、PowerShell、cmd）から`mslearn` CLIを使用してください：

```bash
# 直接実行（インストール不要）
npx @microsoft/learn-cli search "BlobClient UploadAsync Azure.Storage.Blobs"

# またはグローバルインストールしてから実行
npm install -g @microsoft/learn-cli
mslearn search "BlobClient UploadAsync Azure.Storage.Blobs"
```

| MCPツール | CLIコマンド |
|----------|-------------|
| `microsoft_docs_search(query: "...")` | `mslearn search "..."` |
| `microsoft_code_sample_search(query: "...", language: "...")` | `mslearn code-search "..." --language ...` |
| `microsoft_docs_fetch(url: "...")` | `mslearn fetch "..."` |

`search`または`code-search`に`--json`を渡すと、さらなる処理用の生JSON出力を得られます。

---

## 例外: 他のツールを使う場合

以下のカテゴリは**learn.microsoft.com外**にあります。指定されたツールを使用してください。

### .NET Aspire — Aspire MCPサーバー（推奨）またはContext7を使用

Aspireドキュメントは**aspire.dev**にあり、Learnにはありません。最適なツールはAspire CLIのバージョンによります：

**CLI 13.2以降**（推奨） — Aspire MCPサーバーには組み込みのドキュメント検索ツールがあります：

| MCPツール | 説明 |
|----------|-------------|
| `list_docs` | aspire.devの利用可能なドキュメント一覧を取得 |
| `search_docs` | aspire.devコンテンツの重み付けレキシカル検索 |
| `get_doc` | スラッグで特定のドキュメントを取得 |

これらはAspire CLI 13.2で提供されています（[PR #14028](https://github.com/dotnet/aspire/pull/14028)）。更新は`aspire update --self --channel daily`で行います。参照：https://davidpine.dev/posts/aspire-docs-mcp-tools/

**CLI 13.1** — MCPサーバーは統合検索（`list_integrations`、`get_integration_docs`）を提供しますが、ドキュメント検索はありません。Context7にフォールバックしてください：

| ライブラリID | 用途 |
|---|---|
| `/microsoft/aspire.dev` | 主にガイド、統合、CLIリファレンス、デプロイメント |
| `/dotnet/aspire` | ランタイムソース — API内部、実装詳細 |
| `/communitytoolkit/aspire` | コミュニティ統合 — Go、Java、Node.js、Ollama |

### VS Code — Context7を使用

VS Codeドキュメントは**code.visualstudio.com**にあり、Learnにはありません。

| ライブラリID | 用途 |
|---|---|
| `/websites/code_visualstudio` | ユーザードキュメント — 設定、機能、デバッグ、リモート開発 |
| `/websites/code_visualstudio_api` | 拡張API — Webview、TreeView、コマンド、寄与ポイント |

### GitHub — Context7を使用

GitHubドキュメントは**docs.github.com**および**cli.github.com**にあります。

| ライブラリID | 用途 |
|---|---|
| `/websites/github_en` | Actions、API、リポジトリ、セキュリティ、管理、Copilot |
| `/websites/cli_github` | GitHub CLI（`gh`）のコマンドとフラグ |

### Agent Framework — Learn MCP + Context7を使用

Agent Frameworkのチュートリアルはlearn.microsoft.comにあります（`microsoft_docs_search`を使用）が、**GitHubリポジトリ**には公開ドキュメントより先行したAPIレベルの詳細があります。特にDevUI REST APIリファレンス、CLIオプション、.NET統合です。

| ライブラリID | 用途 |
|---|---|
| `/websites/learn_microsoft_en-us_agent-framework` | チュートリアル — DevUIガイド、トレース、ワークフローオーケストレーション |
| `/microsoft/agent-framework` | API詳細 — DevUI RESTエンドポイント、CLIフラグ、認証、.NETの`AddDevUI`/`MapDevUI` |

**DevUIのヒント:** 使い方ガイドはLearnサイトのソースで検索し、APIレベルの詳細（エンドポイントスキーマ、プロキシ設定、認証トークン）はリポジトリのソースで検索してください。

---

## Context7セットアップ

Context7のクエリでは、まずライブラリIDを解決します（一セッションにつき一度）：

1. 技術名で`mcp_context7_resolve-library-id`を呼び出す
2. 返されたライブラリIDと具体的なクエリで`mcp_context7_query-docs`を呼び出す

---

## 効果的なクエリの書き方

具体的に記述してください — バージョン、目的、言語を含めます：

```
# ❌ あいまいすぎる例
"Azure Functions"
"agent framework"

# ✅ 具体的な例
"Azure Functions Python v2 programming model"
"Cosmos DB partition key design best practices"
"GitHub Actions workflow_dispatch inputs matrix strategy"
"Aspire AddUvicornApp Python FastAPI integration"
"DevUI serve agents tracing OpenTelemetry directory discovery"
"Agent Framework workflow conditional edges branching handoff"
```

コンテキストを含める：
- 関連する場合は**バージョン**（`.NET 8`、`Aspire 13`、`VS Code 1.96`）
- **タスクの目的**（`quickstart`、`tutorial`、`overview`、`limits`、`API reference`）
- 多言語ドキュメントの場合は**言語**（`Python`、`TypeScript`、`C#`）
