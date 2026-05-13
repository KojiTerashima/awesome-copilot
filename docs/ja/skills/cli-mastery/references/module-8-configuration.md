# モジュール 8: 設定

## 主なファイル

| File | 用途 |
|------|---------|
| `~/.copilot/config.json` | メイン設定（model、theme、logging、experimental flags） |
| `~/.copilot/mcp-config.json` | MCP サーバー |
| `~/.copilot/lsp-config.json` | 言語サーバー（ユーザーレベル） |
| `.github/lsp.json` | 言語サーバー（リポジトリレベル） |
| `~/.copilot/copilot-instructions.md` | グローバルなカスタム指示 |
| `.github/copilot-instructions.md` | リポジトリレベルのカスタム指示 |

## 環境変数

| Variable | 用途 |
|----------|---------|
| `EDITOR` | `Ctrl+G` 用のテキストエディター（外部エディターでプロンプト編集） |
| `COPILOT_LOG_LEVEL` | ログの詳細度（error/warn/info/debug/trace） |
| `GH_TOKEN` / `GITHUB_TOKEN` | GitHub 認証トークン（この順で確認される） |
| `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` | カスタム指示用の追加ディレクトリ |

## 権限モデル

- デフォルト: 編集、作成、シェルコマンドには確認が必要
- `/allow-all` または `--yolo`: セッション中のすべての確認をスキップ
- `/reset-allowed-tools`: 確認を再有効化する
- ディレクトリ許可リスト、ツール承認ゲート、MCP サーバー信頼

## ログレベル

error, warn, info, debug, trace（`COPILOT_LOG_LEVEL=debug copilot`）

debug/trace を使う場面: MCP 接続の問題、ツール失敗、予期しない挙動、バグ報告
