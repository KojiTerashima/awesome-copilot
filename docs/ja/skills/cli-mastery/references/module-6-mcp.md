# モジュール 6: MCP 連携

## MCP とは？

- Model Context Protocol: AI を外部ツールに接続するための標準
- 「AI 用の USB ポート」のようなもの。互換ツールを自由に差し込める
- GitHub MCP サーバーは **組み込み済み**（リポジトリ、issue、PR、actions の検索など）

## 主なコマンド

| Command | 何をするか |
|---------|-------------|
| `/mcp` | 接続済みの MCP サーバーを一覧表示する |
| `/mcp add <name> <command>` | 新しい MCP サーバーを追加する |

## 人気の MCP サーバー

- `@modelcontextprotocol/server-postgres` — PostgreSQL データベースを問い合わせる
- `@modelcontextprotocol/server-sqlite` — SQLite データベースを問い合わせる
- `@modelcontextprotocol/server-filesystem` — 権限付きでローカルファイルへアクセスする
- `@modelcontextprotocol/server-memory` — 永続的なナレッジグラフ
- `@modelcontextprotocol/server-puppeteer` — ブラウザー自動化

## 設定

| Level | File |
|-------|------|
| User | `~/.copilot/mcp-config.json` |
| Project | `.github/mcp-config.json` |

## 設定ファイル形式

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres", "{{env.DATABASE_URL}}"],
      "env": { "NODE_ENV": "development" }
    }
  }
}
```

## セキュリティのベストプラクティス

- 認証情報を設定ファイルへ直接書かない
- 環境変数参照を使う: `{{env.SECRET}}`
- 利用前に MCP サーバーのソースを確認する
- 本当に必要なサーバーだけ接続する
