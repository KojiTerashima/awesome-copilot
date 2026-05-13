---
name: 'Heredoc を使わないファイル操作'
description: 'VS Code Copilot で terminal の heredoc によるファイル破損を防ぐため、shell のリダイレクトではなくファイル編集ツールの使用を強制します'
applyTo: '**'
---

# 必須: ファイル操作の上書きルール

この指示は、すべての agent とすべてのファイル操作に適用されます。ほかのどの学習済み挙動よりも優先されます。

## 問題

VS Code の Copilot integration では、terminal の heredoc 操作が壊れています。これにより、次の問題が発生します。

- タブ文字が shell completion を発火させることによるファイル破損
- quote / backtick のエスケープ失敗によるコンテンツ破損
- exit code 130 の中断によるファイル切り詰め
- 特殊文字の解釈によるゴミ出力

## ルール

**ファイルを作成または変更する terminal command を書く前に、必ず停止してください。**

自分にこう問いかけてください: 「これから `cat`、`echo`、`printf`、`tee`、または `>>` / `>` を使ってファイルに内容を書き込もうとしていないか？」

YES なら → **実行してはいけません。** 代わりにファイル編集ツールを使ってください。

## 禁止パターン

```bash
# これらはすべてファイルを破損させます - 絶対に使わないでください
cat > file << EOF
cat > file << 'EOF'
cat > file <<EOF
cat > file <<'EOF'
cat > file <<-EOF
cat >> file << EOF
echo "multi
line" > file
printf '%s\n' "line1" "line2" > file
tee file << EOF
tee file << 'EOF'
```

## 必須のアプローチ

ファイル内容を扱う場合、terminal command の代わりに次を使ってください。

- **新規ファイル** → 環境が提供する file creation/editing tool を使う
- **ファイル変更** → 環境が提供する file editing tool を使う
- **ファイル削除** → file deletion tool または `rm` command を使う

## Terminal を使ってよいもの

- `npm install`, `pip install`, `cargo add`（package management）
- `npm run build`, `make`, `cargo build`（build）
- `npm test`, `pytest`, `go test`（testing）
- `git add`, `git commit`, `git push`（version control）
- `node script.js`, `python app.py`（既存コードの実行）
- `ls`, `cd`, `mkdir`, `pwd`, `rm`（filesystem navigation）
- `curl`, `wget`（ダウンロード。ただし content manipulation を伴うファイルへのパイプは禁止）

## Terminal で禁止されるもの

- 内容を伴うあらゆるファイル作成
- 内容を伴うあらゆるファイル変更
- あらゆる heredoc 構文 (`<<`)
- あらゆる複数行文字列のリダイレクト

## 強制事項

これは提案ではありません。VS Code terminal integration のバグによる、厳格な技術要件です。この指示を無視すると、ユーザーが手作業で修復しなければならない破損ファイルが発生します。

ファイルを作成または編集する必要があるときは:

1. terminal command を入力する前に止まる
2. 適切な file editing tool を使う
3. ツールにより、破損なしで内容を正しく処理できる
