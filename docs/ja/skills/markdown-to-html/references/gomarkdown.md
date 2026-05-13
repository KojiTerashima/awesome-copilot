# gomarkdown/markdown リファレンス

Markdownを解析しHTMLをレンダリングするGoライブラリ。高速で拡張可能、スレッドセーフ。

## インストール

```bash
# Goプロジェクトに追加
go get github.com/gomarkdown/markdown

# CLIツールをインストール
go install github.com/gomarkdown/mdtohtml@latest
```

## 基本的な使い方

### シンプルな変換

```go
package main

import (
    "fmt"
    "github.com/gomarkdown/markdown"
)

func main() {
    md := []byte("# Hello World\n\nこれは**太字**のテキストです。")
    html := markdown.ToHTML(md, nil, nil)
    fmt.Println(string(html))
}
```

### CLIツールの使用

```bash
# ファイルをHTMLに変換
mdtohtml input.md output.html

# 標準出力に出力
mdtohtml input.md
```

## パーサーの設定

### 一般的な拡張機能

```go
import (
    "github.com/gomarkdown/markdown"
    "github.com/gomarkdown/markdown/parser"
)

// 拡張機能付きパーサーを作成
extensions := parser.CommonExtensions | parser.AutoHeadingIDs
p := parser.NewWithExtensions(extensions)

// Markdownを解析
doc := p.Parse(md)
```

### 利用可能なパーサー拡張機能

| 拡張機能 | 説明 |
|-----------|-------------|
| `parser.CommonExtensions` | テーブル、フェンスコード、自動リンク、取り消し線 |
| `parser.Tables` | パイプテーブルのサポート |
| `parser.FencedCode` | 言語指定可能なフェンスコードブロック |
| `parser.Autolink` | URLの自動検出 |
| `parser.Strikethrough` | ~~取り消し線~~ テキスト |
| `parser.SpaceHeadings` | 見出しの#の後にスペースを要求 |
| `parser.HeadingIDs` | カスタム見出しID {#id} |
| `parser.AutoHeadingIDs` | 見出しIDを自動生成 |
| `parser.Footnotes` | 脚注のサポート |
| `parser.NoEmptyLineBeforeBlock` | ブロック前に空行不要 |
| `parser.HardLineBreak` | 改行を `<br>` に変換 |
| `parser.MathJax` | MathJaxのサポート |
| `parser.SuperSubscript` | 上付き^スクリプト^と下付き~スクリプト~ |
| `parser.Mmark` | Mmark構文のサポート |

## HTMLレンダラーの設定

### 一般的なフラグ

```go
import (
    "github.com/gomarkdown/markdown"
    "github.com/gomarkdown/markdown/html"
    "github.com/gomarkdown/markdown/parser"
)

// パーサー
p := parser.NewWithExtensions(parser.CommonExtensions)

// レンダラー
htmlFlags := html.CommonFlags | html.HrefTargetBlank
opts := html.RendererOptions{
    Flags: htmlFlags,
    Title: "My Document",
    CSS: "style.css",
}
renderer := html.NewRenderer(opts)

// 変換
html := markdown.ToHTML(md, p, renderer)
```

### 利用可能なHTMLフラグ

| フラグ | 説明 |
|------|-------------|
| `html.CommonFlags` | 一般的な妥当なデフォルト |
| `html.HrefTargetBlank` | リンクに `target="_blank"` を追加 |
| `html.CompletePage` | 完全なHTMLドキュメントを生成 |
| `html.UseXHTML` | XHTML出力を使用 |
| `html.FootnoteReturnLinks` | 脚注に戻るリンクを追加 |
| `html.FootnoteNoHRTag` | 脚注前に `<hr>` を出さない |
| `html.Smartypants` | スマート句読点 |
| `html.SmartypantsFractions` | スマート分数 (1/2 → ½) |
| `html.SmartypantsDashes` | スマートダッシュ (-- → –) |
| `html.SmartypantsLatexDashes` | LaTeXスタイルのダッシュ |

### レンダラーオプション

```go
opts := html.RendererOptions{
    Flags:          htmlFlags,
    Title:          "ドキュメントタイトル",
    CSS:            "path/to/style.css",
    Icon:           "favicon.ico",
    Head:           []byte("<meta name='author' content='...'>"),
    RenderNodeHook: customRenderHook,
}
```

## 完全な例

```go
package main

import (
    "os"
    "github.com/gomarkdown/markdown"
    "github.com/gomarkdown/markdown/html"
    "github.com/gomarkdown/markdown/parser"
)

func mdToHTML(md []byte) []byte {
    // 拡張機能付きパーサー
    extensions := parser.CommonExtensions | 
                  parser.AutoHeadingIDs | 
                  parser.NoEmptyLineBeforeBlock
    p := parser.NewWithExtensions(extensions)
    doc := p.Parse(md)

    // オプション付きHTMLレンダラー
    htmlFlags := html.CommonFlags | html.HrefTargetBlank
    opts := html.RendererOptions{Flags: htmlFlags}
    renderer := html.NewRenderer(opts)

    return markdown.Render(doc, renderer)
}

func main() {
    md, _ := os.ReadFile("input.md")
    html := mdToHTML(md)
    os.WriteFile("output.html", html, 0644)
}
```

## セキュリティ: 出力のサニタイズ

**重要:** gomarkdownはHTML出力をサニタイズしません。信頼できない入力にはBluemondayを使用してください：

```go
import (
    "github.com/microcosm-cc/bluemonday"
    "github.com/gomarkdown/markdown"
)

// Markdownを潜在的に安全でないHTMLに変換
unsafeHTML := markdown.ToHTML(md, nil, nil)

// Bluemondayでサニタイズ
p := bluemonday.UGCPolicy()
safeHTML := p.SanitizeBytes(unsafeHTML)
```

### Bluemondayのポリシー

| ポリシー | 説明 |
|--------|-------------|
| `UGCPolicy()` | ユーザー生成コンテンツ（最も一般的） |
| `StrictPolicy()` | すべてのHTMLを除去 |
| `StripTagsPolicy()` | タグを除去しテキストを保持 |
| `NewPolicy()` | カスタムポリシーを構築 |

## ASTの操作

### ASTへのアクセス

```go
import (
    "github.com/gomarkdown/markdown/ast"
    "github.com/gomarkdown/markdown/parser"
)

p := parser.NewWithExtensions(parser.CommonExtensions)
doc := p.Parse(md)

// ASTを歩く
ast.WalkFunc(doc, func(node ast.Node, entering bool) ast.WalkStatus {
    if heading, ok := node.(*ast.Heading); ok && entering {
        fmt.Printf("見出しレベル %d を発見\n", heading.Level)
    }
    return ast.GoToNext
})
```

### カスタムレンダラー

```go
type MyRenderer struct {
    *html.Renderer
}

func (r *MyRenderer) RenderNode(w io.Writer, node ast.Node, entering bool) ast.WalkStatus {
    // カスタムレンダリングロジック
    if heading, ok := node.(*ast.Heading); ok && entering {
        fmt.Fprintf(w, "<h%d class='custom'>", heading.Level)
        return ast.GoToNext
    }
    return r.Renderer.RenderNode(w, node, entering)
}
```

## 改行の処理

WindowsやMacの改行は正規化が必要です：

```go
// 解析前に改行を正規化
normalized := parser.NormalizeNewlines(input)
html := markdown.ToHTML(normalized, nil, nil)
```

## リソース

- [パッケージドキュメント](https://pkg.go.dev/github.com/gomarkdown/markdown)
- [高度な処理ガイド](https://blog.kowalczyk.info/article/cxn3/advanced-markdown-processing-in-go.html)
- [GitHubリポジトリ](https://github.com/gomarkdown/markdown)
- [CLIツール](https://github.com/gomarkdown/mdtohtml)
- [Bluemondayサニタイザー](https://github.com/microcosm-cc/bluemonday)
