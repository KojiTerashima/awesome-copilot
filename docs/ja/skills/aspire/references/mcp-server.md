# MCP Server — 完全リファレンス

Aspire は **MCP（Model Context Protocol）サーバー**を公開しており、AI コーディングアシスタントが実行中の分散アプリケーションを問い合わせ・制御し、Aspire のドキュメントを検索できるようにします。これにより AI ツールは、AI アシスタントのコンテキスト内からリソース状態の確認、ログの読み取り、トレースの表示、サービスの再起動、ドキュメント参照を行えます。

リファレンス: https://aspire.dev/get-started/configure-mcp/

---

## セットアップ: `aspire mcp init`

MCP サーバーを設定する最も簡単な方法は、Aspire CLI を使うことです。

```bash
# プロジェクトディレクトリでターミナルを開く
aspire mcp init
```

このコマンドは対話形式のセットアップを案内します。

1. **Workspace root** — ワークスペースルートのパスを尋ねます（既定は現在のディレクトリ）
2. **Environment detection** — 対応する AI 環境（VS Code、Copilot CLI、Claude Code、OpenCode）を検出し、どれを設定するか尋ねます
3. **Playwright MCP** — Aspire とあわせて Playwright MCP サーバーを設定するかどうかを任意で提示します
4. **Config creation** — 適切な設定ファイル（例: `.vscode/mcp.json`）を書き込みます
5. **AGENTS.md** — まだ存在しない場合、AI エージェント向けの Aspire 固有指示を含む `AGENTS.md` を作成します

> **Note:** `aspire mcp init` は対話プロンプト（Spectre.Console）を使用します。実行には実ターミナルが必要で、VS Code の統合ターミナルでは正しく動作しない場合があります。必要に応じて外部ターミナルを使用してください。

---

## 設定の理解

`aspire mcp init` を実行すると、CLI は検出された環境に応じた設定ファイルを作成します。

### VS Code (GitHub Copilot)

`.vscode/mcp.json` を作成または更新します。

```json
{
  "servers": {
    "aspire": {
      "type": "stdio",
      "command": "aspire",
      "args": ["mcp", "start"]
    }
  }
}
```

## MCP ツール

利用できるツールは Aspire CLI のバージョンに依存します。`aspire --version` で確認してください。

### 13.1+ で利用可能なツール（stable）

#### リソース管理ツール

これらのツールには実行中の AppHost（`aspire run`）が必要です。

| Tool                         | Description                                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------ |
| `list_resources`             | 状態、ヘルスステータス、ソース、エンドポイント、コマンドを含むすべてのリソースを一覧表示 |
| `list_console_logs`          | リソースのコンソールログを一覧表示                                                                  |
| `list_structured_logs`       | 構造化ログを一覧表示（任意でリソース名によるフィルタ可能）                                             |
| `list_traces`                | 分散トレースを一覧表示（任意のリソース名パラメータでフィルタ可能）                                      |
| `list_trace_structured_logs` | 特定のトレースに対する構造化ログを一覧表示                                                            |
| `execute_resource_command`   | リソースコマンドを実行（リソース名とコマンド名を受け取る）                                    |

#### AppHost 管理ツール

| Tool             | Description                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `list_apphosts`  | 検出されたすべての AppHost 接続を一覧表示し、作業ディレクトリ範囲内/外を示す |
| `select_apphost` | 複数の AppHost が動作中の場合に、使用する AppHost を選択                                      |

#### 連携ツール

これらは AppHost が実行中でなくても動作します。

| Tool                   | Description                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `list_integrations`    | 利用可能な Aspire ホスティング連携（データベース、メッセージブローカー、クラウドサービスなどの NuGet パッケージ）を一覧表示 |
| `get_integration_docs` | 特定の Aspire ホスティング連携パッケージのドキュメントを取得                                              |

### 13.2+ で追加されたツール（ドキュメント検索）

> **Version gate:** これらのツールは [PR #14028](https://github.com/dotnet/aspire/pull/14028) で追加され、Aspire CLI **13.2** に含まれます。13.1 ではこれらのツールは表示されません。先行利用するには daily チャンネルへ更新してください: `aspire update --self --channel daily`。

| Tool          | Description                                                              |
| ------------- | ------------------------------------------------------------------------ |
| `list_docs`   | aspire.dev で利用可能なすべてのドキュメントを一覧表示                        |
| `search_docs` | インデックス化された aspire.dev ドキュメント全体に対して重み付き語彙検索を実行 |
| `get_doc`     | slug を指定して特定のドキュメントを取得                                |

これらのツールは `llms.txt` 仕様を使って aspire.dev のコンテンツをインデックス化し、重み付き語彙検索（タイトル 10x、要約 8x、見出し 6x、コード 5x、本文 1x）を提供します。AppHost が実行中でなくても利用できます。

### ドキュメントの代替手段（13.1 ユーザー向け）

Aspire CLI 13.1 を使用していて `list_docs`/`search_docs`/`get_doc` がない場合、ドキュメント問い合わせの代替として **Context7** を使ってください。詳細は [SKILL.md documentation research section](../SKILL.md#1-researching-aspire-documentation) を参照してください。

---

## MCP からリソースを除外する

リソースと関連テレメトリは、リソースにアノテーションを付けることで MCP の結果から除外できます。

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var apiService = builder.AddProject<Projects.Api>("apiservice")
    .ExcludeFromMcp();  // MCP tools から非表示

builder.AddProject<Projects.Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithReference(apiService);

builder.Build().Run();
```

---

## 対応 AI アシスタント

`aspire mcp init` コマンドは次をサポートします。

- [VS Code](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) (GitHub Copilot)
- [Copilot CLI](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli#add-an-mcp-server)
- [Claude Code](https://docs.claude.com/en/docs/claude-code/mcp)
- [OpenCode](https://opencode.ai/docs/mcp-servers/)

MCP サーバーは **STDIO transport protocol** を使用しており、このプロトコルをサポートする他のエージェント型コーディング環境でも動作する可能性があります。

---

## 使用パターン

### AI 支援によるデバッグ

MCP を設定すると、AI アシスタントで次が可能になります。

1. **実行状態を確認:**

   - 「Aspire の全リソースとその状態を一覧表示して」
   - 「データベースは健全？」
   - 「API はどのポートで動いてる？」

2. **ログを読む:**

   - 「ML サービスの最新ログを見せて」
   - 「worker ログにエラーはある？」

3. **トレースを表示:**

   - 「最後に失敗したリクエストのトレースを見せて」
   - 「API → Database 呼び出しのレイテンシは？」

4. **リソースを制御:**

   - 「API サービスを再起動して」
   - 「キューをデバッグしている間 worker を停止して」

5. **ドキュメント検索（13.2+）:**
   - 「Redis キャッシュについて Aspire ドキュメントを検索して」
   - 「サービスディスカバリはどう設定する？」
   - _（CLI 13.2+ が必要。13.1 では Context7 または `list_integrations`/`get_integration_docs` を連携固有ドキュメントに使用してください。）_

---

## セキュリティ上の考慮事項

- MCP サーバーはローカル AppHost のリソースのみ公開します
- 認証は不要です（ローカル開発専用）
- STDIO transport は、そのプロセスを起動した AI ツールでのみ動作します
- **本番環境で MCP エンドポイントをネットワークに公開しないでください**

---

## 制限事項

- AI モデルにはデータ処理の制限があります。大きなデータ項目（例: スタックトレース）は切り詰められる場合があります。
- 大量のテレメトリ集合を扱うリクエストでは、古い項目が省略されて短縮される場合があります。

---

## トラブルシューティング

問題が発生した場合は、[GitHub の open MCP issues](https://github.com/dotnet/aspire/issues?q=is%3Aissue+is%3Aopen+label%3Aarea-mcp) を確認してください。

## 関連情報

- [aspire mcp command](https://aspire.dev/reference/cli/commands/aspire-mcp/)
- [aspire mcp init command](https://aspire.dev/reference/cli/commands/aspire-mcp-init/)
- [aspire mcp start command](https://aspire.dev/reference/cli/commands/aspire-mcp-start/)
- [GitHub Copilot in the Dashboard](https://aspire.dev/dashboard/copilot/)
- [How I taught AI to read Aspire docs](https://davidpine.dev/posts/aspire-docs-mcp-tools/)

