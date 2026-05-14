---
name: lsp-setup
description: 'Copilot CLI用のLSPサーバーをインストール・設定し、任意のプログラミング言語でコードインテリジェンス（定義へ移動、参照検索、ホバー、型情報）を有効化します。OSを検出し、適切なサーバーをインストールし、JSON設定（ユーザーレベルまたはリポジトリレベル）を生成します。より深いコード理解が必要な場合やLSPサーバーが未設定の場合、またはユーザーからLSPサーバーのセットアップ、インストール、設定を求められた場合に使用してください。'
---

# GitHub Copilot CLIのLSPセットアップ

**ユーティリティスキル** — Copilot CLI用のLanguage Server Protocolサーバーをインストールおよび設定します。  
使用例: 「LSPをセットアップ」「言語サーバーをインストール」「JavaのLSPを設定」「TypeScriptのLSPを追加」「コードインテリジェンスを有効化」「定義へ移動が必要」「参照検索が動作しない」「より良いコード理解が必要」  
使用禁止: 一般的なコーディング作業、IDE/エディタのLSP設定、Copilot CLI以外のセットアップ

## ワークフロー

1. **言語を尋ねる** — `ask_user`を使い、ユーザーがLSPサポートを希望するプログラミング言語を尋ねる  
2. **OSを検出する** — `uname -s`（またはWindowsの場合は`$env:OS` / `%OS%`）を実行し、macOS、Linux、Windowsを判別  
3. **LSPサーバーを調べる** — `references/lsp-servers.md`を参照し、既知のサーバー、インストールコマンド、設定スニペットを確認  
4. **スコープを尋ねる** — `ask_user`を使い、設定をユーザーレベル（`~/.copilot/lsp-config.json`）かリポジトリレベル（リポジトリルートの`lsp.json`または`.github/lsp.json`）のどちらにするか尋ねる  
5. **サーバーをインストールする** — 検出したOSに応じたインストールコマンドを実行  
6. **設定を書く** — 新しいサーバーエントリを選択した設定ファイルにマージ（ユーザーレベルは`~/.copilot/lsp-config.json`、リポジトリレベルは`lsp.json`または`.github/lsp.json`）。リポジトリレベルの設定が既に存在する場合はその場所を使い続け、存在しない場合はユーザーにどちらのリポジトリレベルの場所を希望するか尋ねる。ファイルがなければ作成し、既存のエントリは保持。  
7. **検証する** — LSPバイナリが`$PATH`にあること、設定ファイルが有効なJSONであることを確認

## 設定フォーマット

Copilot CLIはユーザーレベルまたはリポジトリレベルの場所からLSP設定を読み込み、リポジトリレベルの設定がユーザーレベルより優先されます。

- **ユーザーレベル**: `~/.copilot/lsp-config.json`  
- **リポジトリレベル**: `lsp.json`（リポジトリルート）または`.github/lsp.json`

JSON構造:

```json
{
  "lspServers": {
    "<server-key>": {
      "command": "<binary>",
      "args": ["--stdio"],
      "fileExtensions": {
        ".<ext>": "<languageId>",
        ".<ext2>": "<languageId>"
      }
    }
  }
}
```

### 重要なルール

- `command`はバイナリ名（`$PATH`上にある必要あり）または絶対パス  
- `args`にはほぼ常に標準入出力通信のための`"--stdio"`が含まれる  
- `fileExtensions`は各ファイル拡張子（先頭にドット付き）を[言語ID](https://code.visualstudio.com/docs/languages/identifiers#_known-language-identifiers)にマッピング  
- 複数のサーバーが`lspServers`内に共存可能  
- 既存ファイルにマージする際は、他のサーバーエントリを**上書きせず**、対象言語のキーのみ追加または更新

## 動作

- 言語やスコープを尋ねる際は常に`ask_user`の`choices`を使う  
- `references/lsp-servers.md`に言語がない場合は「<language> LSP server」でウェブ検索し、手動設定の案内を行う  
- パッケージマネージャーが利用できない場合（例: macOSでHomebrewなし）は、参照ファイルの代替インストール方法を提案  
- インストール後、`which <binary>`（Windowsは`where.exe`）を実行しバイナリのアクセス可能性を確認  
- 設定を書き込む前に最終的なJSON設定をユーザーに表示  
- 設定ファイルが既に存在する場合は先に読み込み、マージしてから書き込み。上書き禁止

## 検証

セットアップ後、ユーザーに以下を伝える:

1. `/exit`を入力してCopilot CLIを終了 — 新しいLSP設定は次回起動時に読み込まれるため**必須**  
2. 設定した言語のファイルがあるプロジェクトで`copilot`を再起動  
3. `/lsp`を実行してサーバーステータスを確認  
4. 定義へ移動やホバーなどのコードインテリジェンス機能を試す
