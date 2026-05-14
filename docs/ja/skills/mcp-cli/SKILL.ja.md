---
name: mcp-cli
description: CLIを介してMCP（Model Context Protocol）サーバーと対話するためのインターフェース。外部ツール、API、データソースとMCPサーバーを通じてやり取りする必要がある場合、利用可能なMCPサーバーやツールを一覧表示する場合、またはコマンドラインからMCPツールを呼び出す場合に使用します。
---

# MCP-CLI

コマンドラインからMCPサーバーにアクセスします。MCPはGitHub、ファイルシステム、データベース、APIなどの外部システムとのやり取りを可能にします。

## コマンド

| コマンド                            | 出力内容                          |
| ---------------------------------- | --------------------------------- |
| `mcp-cli`                          | すべてのサーバーとツール名を一覧表示 |
| `mcp-cli <server>`                 | パラメータ付きのツールを表示       |
| `mcp-cli <server>/<tool>`          | ツールのJSONスキーマを取得         |
| `mcp-cli <server>/<tool> '<json>'` | 引数を指定してツールを呼び出す      |
| `mcp-cli grep "<glob>"`            | 名前でツールを検索                 |

**説明を含めるには `-d` を追加**（例：`mcp-cli filesystem -d`）

## ワークフロー

1. **発見**: `mcp-cli` → 利用可能なサーバーとツールを確認
2. **探索**: `mcp-cli <server>` → パラメータ付きのツールを確認
3. **検査**: `mcp-cli <server>/<tool>` → 完全なJSON入力スキーマを取得
4. **実行**: `mcp-cli <server>/<tool> '<json>'` → 引数付きで実行

## 例

```bash
# すべてのサーバーとツール名を一覧表示
mcp-cli

# パラメータ付きのすべてのツールを表示
mcp-cli filesystem

# 説明付き（より詳細）
mcp-cli filesystem -d

# 特定ツールのJSONスキーマを取得
mcp-cli filesystem/read_file

# ツールを呼び出す
mcp-cli filesystem/read_file '{"path": "./README.md"}'

# ツールを検索
mcp-cli grep "*file*"

# 解析用のJSON出力
mcp-cli filesystem/read_file '{"path": "./README.md"}' --json

# 引用符を含む複雑なJSON（heredocまたは標準入力を使用）
mcp-cli server/tool <<EOF
{"content": "Text with 'quotes' inside"}
EOF

# またはファイルやコマンドからパイプで渡す
cat args.json | mcp-cli server/tool

# すべてのTypeScriptファイルを検索し、最初のファイルを読み込む
mcp-cli filesystem/search_files '{"path": "src/", "pattern": "*.ts"}' --json | jq -r '.content[0].text' | head -1 | xargs -I {} sh -c 'mcp-cli filesystem/read_file "{\"path\": \"{}\"}"'
```

## オプション

| フラグ         | 用途                       |
| -------------- | -------------------------- |
| `-j, --json`   | スクリプト用のJSON出力      |
| `-r, --raw`    | 生テキストコンテンツ        |
| `-d`           | 説明を含める                |

## 終了コード

- `0`: 成功
- `1`: クライアントエラー（引数不正、設定不足）
- `2`: サーバーエラー（ツールの失敗）
- `3`: ネットワークエラー
