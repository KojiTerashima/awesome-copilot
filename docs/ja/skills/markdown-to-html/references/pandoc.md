# Pandocリファレンス

Pandocは、多数のマークアップ形式間で変換可能なユニバーサルドキュメントコンバーターです。Markdown、HTML、LaTeX、Wordなど多くの形式に対応しています。

## インストール

### Windows

```powershell
# Chocolateyを使用
choco install pandoc

# Scoopを使用
scoop install pandoc

# または https://pandoc.org/installing.html からインストーラーをダウンロード
```

### macOS

```bash
# Homebrewを使用
brew install pandoc
```

### Linux

```bash
# Debian/Ubuntu
sudo apt-get install pandoc

# Fedora
sudo dnf install pandoc

# または https://pandoc.org/installing.html からダウンロード
```

## 基本的な使い方

### MarkdownをHTMLに変換

```bash
# 基本変換
pandoc input.md -o output.html

# ヘッダー付きのスタンドアロン文書
pandoc input.md -s -o output.html

# カスタムCSSを使用
pandoc input.md -s --css=style.css -o output.html
```

### 他の形式に変換

```bash
# PDFへ（LaTeXが必要）
pandoc input.md -s -o output.pdf

# Wordへ
pandoc input.md -s -o output.docx

# LaTeXへ
pandoc input.md -s -o output.tex

# EPUBへ
pandoc input.md -s -o output.epub
```

### 他の形式から変換

```bash
# HTMLからMarkdownへ
pandoc -f html -t markdown input.html -o output.md

# WordからMarkdownへ
pandoc input.docx -o output.md

# LaTeXからHTMLへ
pandoc -f latex -t html input.tex -o output.html
```

## よく使うオプション

| オプション | 説明 |
|--------|-------------|
| `-f, --from <format>` | 入力形式 |
| `-t, --to <format>` | 出力形式 |
| `-s, --standalone` | スタンドアロン文書を生成 |
| `-o, --output <file>` | 出力ファイル |
| `--toc` | 目次を含める |
| `--toc-depth <n>` | 目次の深さ（デフォルト: 3） |
| `-N, --number-sections` | セクション見出しに番号を付ける |
| `--css <url>` | CSSスタイルシートへのリンク |
| `--template <file>` | カスタムテンプレートを使用 |
| `--metadata <key>=<value>` | メタデータを設定 |
| `--mathml` | 数式にMathMLを使用 |
| `--mathjax` | 数式にMathJaxを使用 |
| `-V, --variable <key>=<value>` | テンプレート変数を設定 |

## Markdown拡張

Pandocは多くのMarkdown拡張をサポートしています：

```bash
# 特定の拡張を有効化
pandoc -f markdown+emoji+footnotes input.md -o output.html

# 特定の拡張を無効化
pandoc -f markdown-pipe_tables input.md -o output.html

# 厳密なMarkdownを使用
pandoc -f markdown_strict input.md -o output.html
```

### よく使う拡張

| 拡張 | 説明 |
|-----------|-------------|
| `pipe_tables` | パイプテーブル（デフォルトで有効） |
| `footnotes` | 脚注サポート |
| `emoji` | 絵文字ショートコード |
| `smart` | スマートクォートとダッシュ |
| `task_lists` | タスクリストのチェックボックス |
| `strikeout` | 打ち消し線テキスト |
| `superscript` | 上付き文字 |
| `subscript` | 下付き文字 |
| `raw_html` | 生のHTMLを通過させる |

## テンプレート

### 組み込みテンプレートの使用

```bash
# デフォルトテンプレートを表示
pandoc -D html

# カスタムテンプレートを使用
pandoc --template=mytemplate.html input.md -o output.html
```

### テンプレート変数

```html
<!DOCTYPE html>
<html>
<head>
  <title>$title$</title>
  $for(css)$
  <link rel="stylesheet" href="$css$">
  $endfor$
</head>
<body>
$body$
</body>
</html>
```

## YAMLメタデータ

Markdownファイルにメタデータを含める例：

```markdown
---
title: My Document
author: John Doe
date: 2025-01-28
abstract: |
  これは要約です。
---

# はじめに

ここに文書の内容を書きます...
```

## フィルター

### Luaフィルターの使用

```bash
pandoc --lua-filter=filter.lua input.md -o output.html
```

Luaフィルターの例（`filter.lua`）：

```lua
function Header(el)
  if el.level == 1 then
    el.classes:insert("main-title")
  end
  return el
end
```

### Pandocフィルターの使用

```bash
pandoc --filter pandoc-citeproc input.md -o output.html
```

## バッチ変換

### Bashスクリプト

```bash
#!/bin/bash
for file in *.md; do
  pandoc "$file" -s -o "${file%.md}.html"
done
```

### PowerShellスクリプト

```powershell
Get-ChildItem -Filter *.md | ForEach-Object {
  $output = $_.BaseName + ".html"
  pandoc $_.Name -s -o $output
}
```

## リソース

- [Pandocユーザーガイド](https://pandoc.org/MANUAL.html)
- [Pandocデモ](https://pandoc.org/demos.html)
- [Pandoc FAQ](https://pandoc.org/faqs.html)
- [GitHubリポジトリ](https://github.com/jgm/pandoc)
