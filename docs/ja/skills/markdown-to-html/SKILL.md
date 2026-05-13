---
name: markdown-to-html
description: 'marked.js、pandoc、gomarkdown/markdownなどのツールや、jekyll/jekyll、gohugoio/hugoなどのマークダウン文書をHTMLに変換するウェブテンプレートシステムでの作業に似た、MarkdownファイルをHTMLに変換する。 「markdownをhtmlに変換」「mdをhtmlに変換」「markdownをレンダリング」「markdownからhtmlを生成」などの依頼時や、.mdファイルやMarkdownをHTML出力に変換するウェブテンプレートシステムの作業時に使用。CLIとNode.jsのワークフローをGFM、CommonMark、標準Markdownフレーバーでサポート。'
---

# MarkdownからHTMLへの変換

marked.jsライブラリを使ったMarkdown文書のHTML変換や、[markedJS/marked](https://github.com/markedjs/marked)リポジトリに似た変換スクリプトの作成に関する専門スキル。カスタムスクリプトの場合、`marked.js`に限定せず、[pandoc](https://github.com/jgm/pandoc)や[gomarkdown/markdown](https://github.com/gomarkdown/markdown)などのツールのデータ変換手法を利用し、[jekyll/jekyll](https://github.com/jekyll/jekyll)や[gohugoio/hugo](https://github.com/gohugoio/hugo)のテンプレートシステムも対象。

変換スクリプトやツールは単一ファイル、バッチ変換、高度な設定に対応可能であるべき。

## このスキルを使う場面

- 「markdownをhtmlに変換」や「mdファイルを変換」などの依頼があったとき
- MarkdownをHTMLとして「レンダリング」したいとき
- .mdファイルからHTMLドキュメントを生成したいとき
- Markdownコンテンツから静的サイトを構築しているとき
- MarkdownをHTMLに変換するテンプレートシステムを構築しているとき
- 既存のテンプレートシステム用のツールやウィジェット、カスタムテンプレートを作成しているとき
- MarkdownをレンダリングされたHTMLとしてプレビューしたいとき

## MarkdownからHTMLへの変換

### 基本的な変換例

詳細は[基本的なmarkdown-to-html.md](references/basic-markdown-to-html.md)を参照

```text
    ```markdown
    # レベル1
    ## レベル2

    [リンク](https://example.com)を含む一文と、`<p>段落タグ</p>`のようなHTMLスニペット。

    - `ul`リスト項目1
    - `ul`リスト項目2

    1. `ol`リスト項目1
    2. `ol`リスト項目2

    | テーブル項目 | 説明 |
    | One | 数字の`1`の綴りです。 |
    | Two | 数字の`2`の綴りです。 |

    ```js
    var one = 1;
    var two = 2;

    function simpleMath(x, y) {
     return x + y;
    }
    console.log(simpleMath(one, two));
    ```
    ```

    ```html
    <h1>レベル1</h1>
    <h2>レベル2</h2>

    <p>[リンク](https://example.com)を含む一文と、<code>&lt;p&gt;段落タグ&lt;/p&gt;</code>のようなHTMLスニペット。</p>

    <ul>
     <li>`ul`リスト項目1</li>
     <li>`ul`リスト項目2</li>
    </ul>

    <ol>
     <li>`ol`リスト項目1</li>
     <li>`ol`リスト項目2</li>
    </ol>

    <table>
     <thead>
      <tr>
       <th>テーブル項目</th>
       <th>説明</th>
      </tr>
     </thead>
     <tbody>
      <tr>
       <td>One</td>
       <td>数字の`1`の綴りです。</td>
      </tr>
      <tr>
       <td>Two</td>
       <td>数字の`2`の綴りです。</td>
      </tr>
     </tbody>
    </table>

    <pre>
     <code>var one = 1;
     var two = 2;

     function simpleMath(x, y) {
      return x + y;
     }
     console.log(simpleMath(one, two));</code>
    </pre>
    ```
```

### コードブロックの変換

詳細は[code-blocks-to-html.md](references/code-blocks-to-html.md)を参照

```text

    ```markdown
    your code here
    ```

    ```html
    <pre><code class="language-md">
    your code here
    </code></pre>
    ```

    ```js
    console.log("Hello world");
    ```

    ```html
    <pre><code class="language-js">
    console.log("Hello world");
    </code></pre>
    ```

    ```markdown
      ```

      ```
      visible backticks
      ```

      ```
    ```

    ```html
      <pre><code>
      ```

      visible backticks

      ```
      </code></pre>
    ```
```

### 折りたたみセクションの変換

詳細は[collapsed-sections-to-html.md](references/collapsed-sections-to-html.md)を参照

```text
    ```markdown
    <details>
    <summary>詳細情報</summary>

    ### 内部の見出し

    - リスト
    - **書式設定**
    - コードブロック

        ```js
        console.log("Hello");
        ```

    </details>
    ```

    ```html
    <details>
    <summary>詳細情報</summary>

    <h3>内部の見出し</h3>

    <ul>
     <li>リスト</li>
     <li><strong>書式設定</strong></li>
     <li>コードブロック</li>
    </ul>

    <pre>
     <code class="language-js">console.log("Hello");</code>
    </pre>

    </details>
    ```
```

### 数式の変換

詳細は[writing-mathematical-expressions-to-html.md](references/writing-mathematical-expressions-to-html.md)を参照

```text
    ```markdown
    この文はインライン数式を`$`で囲んでいます: $\sqrt{3x-1}+(1+x)^2$
    ```

    ```html
    <p>この文は<code>$</code>で囲んだインライン数式を表示しています:
     <math-renderer><math xmlns="http://www.w3.org/1998/Math/MathML">
      <msqrt><mn>3</mn><mi>x</mi><mo>−</mo><mn>1</mn></msqrt>
      <mo>+</mo><mo>(</mo><mn>1</mn><mo>+</mo><mi>x</mi>
      <msup><mo>)</mo><mn>2</mn></msup>
     </math>
    </math-renderer>
    </p>
    ```

    ```markdown
    **コーシー・シュワルツの不等式**\
    $$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$
    ```

    ```html
    <p><strong>コーシー・シュワルツの不等式</strong><br>
     <math-renderer>
      <math xmlns="http://www.w3.org/1998/Math/MathML">
       <msup>
        <mrow><mo>(</mo>
         <munderover><mo data-mjx-texclass="OP">∑</mo>
          <mrow><mi>k</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi>
         </munderover>
         <msub><mi>a</mi><mi>k</mi></msub>
         <msub><mi>b</mi><mi>k</mi></msub>
         <mo>)</mo>
        </mrow>
        <mn>2</mn>
       </msup>
       <mo>≤</mo>
       <mrow><mo>(</mo>
        <munderover><mo>∑</mo>
         <mrow><mi>k</mi><mo>=</mo><mn>1</mn></mrow>
         <mi>n</mi>
        </munderover>
        <msubsup><mi>a</mi><mi>k</mi><mn>2</mn></msubsup>
        <mo>)</mo>
       </mrow>
       <mrow><mo>(</mo>
         <munderover><mo>∑</mo>
          <mrow><mi>k</mi><mo>=</mo><mn>1</mn></mrow>
          <mi>n</mi>
         </munderover>
         <msubsup><mi>b</mi><mi>k</mi><mn>2</mn></msubsup>
         <mo>)</mo>
       </mrow>
      </math>
     </math-renderer></p>
    ```
```

### テーブルの変換

詳細は[tables-to-html.md](references/tables-to-html.md)を参照

```text
    ```markdown
    | ヘッダー1  | ヘッダー2 |
    | ---------- | --------- |
    | セル内容  | セル内容  |
    | セル内容  | セル内容  |
    ```

    ```html
    <table>
     <thead><tr><th>ヘッダー1</th><th>ヘッダー2</th></tr></thead>
     <tbody>
      <tr><td>セル内容</td><td>セル内容</td></tr>
      <tr><td>セル内容</td><td>セル内容</td></tr>
     </tbody>
    </table>
    ```

    ```markdown
    | 左寄せ | 中央寄せ | 右寄せ |
    | :---   |   :---:  |    ---: |
    | git status | git status | git status |
    | git diff   | git diff   | git diff   |
    ```

    ```html
    <table>
      <thead>
       <tr>
        <th align="left">左寄せ</th>
        <th align="center">中央寄せ</th>
        <th align="right">右寄せ</th>
       </tr>
      </thead>
      <tbody>
       <tr>
        <td align="left">git status</td>
        <td align="center">git status</td>
        <td align="right">git status</td>
       </tr>
       <tr>
        <td align="left">git diff</td>
        <td align="center">git diff</td>
        <td align="right">git diff</td>
       </tr>
      </tbody>
    </table>
    ```
```

## [`markedJS/marked`](references/marked.md)の利用

### 前提条件

- Node.jsがインストールされていること（CLIまたはプログラム利用のため）
- CLI用にグローバルインストール: `npm install -g marked`
- またはローカルインストール: `npm install marked`

### クイック変換方法

[marked.md](references/marked.md)の**クイック変換方法**を参照

### ステップバイステップのワークフロー

[marked.md](references/marked.md)の**ステップバイステップワークフロー**を参照

### CLI設定

### 設定ファイルの利用

永続的なオプション用に`~/.marked.json`を作成:

```json
{
  "gfm": true,
  "breaks": true
}
```

カスタム設定を使う場合:

```bash
marked -i input.md -o output.html -c config.json
```

### CLIオプション一覧

| オプション | 説明 |
|------------|------|
| `-i, --input <file>` | 入力Markdownファイル |
| `-o, --output <file>` | 出力HTMLファイル |
| `-s, --string <string>` | ファイルではなく文字列を解析 |
| `-c, --config <file>` | カスタム設定ファイルを使用 |
| `--gfm` | GitHub Flavored Markdownを有効化 |
| `--breaks` | 改行を`<br>`に変換 |
| `--help` | 全オプションを表示 |

### セキュリティ警告

⚠️ **Markedは出力HTMLをサニタイズしません。** 信頼できない入力にはサニタイザーを使用してください:

```javascript
import { marked } from 'marked';
import DOMPurify from 'dompurify';

const unsafeHtml = marked.parse(untrustedMarkdown);
const safeHtml = DOMPurify.sanitize(unsafeHtml);
```

推奨サニタイザー:

- [DOMPurify](https://github.com/cure53/DOMPurify)（推奨）
- [sanitize-html](https://github.com/apostrophecms/sanitize-html)
- [js-xss](https://github.com/leizongmin/js-xss)

### 対応Markdownフレーバー

| フレーバー | 対応度 |
|------------|--------|
| オリジナルMarkdown | 100% |
| CommonMark 0.31 | 98% |
| GitHub Flavored Markdown | 97% |

### トラブルシューティング

| 問題 | 解決策 |
|-------|---------|
| ファイル先頭の特殊文字 | ゼロ幅文字を除去: `content.replace(/^[\u200B\u200C\u200D\uFEFF]/,"")` |
| コードブロックのハイライトなし | highlight.jsなどのシンタックスハイライターを追加 |
| テーブルがレンダリングされない | `gfm: true`オプションを設定 |
| 改行が無視される | `breaks: true`を設定 |
| XSS脆弱性の懸念 | DOMPurifyでサニタイズ |

## [`pandoc`](references/pandoc.md)の利用

### 前提条件

- Pandocがインストールされていること（<https://pandoc.org/installing.html>から入手）
- PDF出力にはLaTeX環境が必要（macOSはMacTeX、WindowsはMiKTeX、Linuxはtexlive）
- ターミナルまたはコマンドプロンプトの利用

### クイック変換方法

#### 方法1: CLI基本変換

```bash
# MarkdownをHTMLに変換
pandoc input.md -o output.html

# ヘッダー/フッターを含むスタンドアロン文書として変換
pandoc input.md -s -o output.html

# 明示的なフォーマット指定
pandoc input.md -f markdown -t html -s -o output.html
```

#### 方法2: フィルターモード（対話式）

```bash
# pandocをフィルターとして起動
pandoc

# Markdownを入力し、Ctrl-D（Linux/macOS）またはCtrl-Z+Enter（Windows）で終了
Hello *pandoc*!
# 出力: <p>Hello <em>pandoc</em>!</p>
```

#### 方法3: フォーマット変換

```bash
# HTMLからMarkdownへ
pandoc -f html -t markdown input.html -o output.md

# MarkdownからLaTeXへ
pandoc input.md -s -o output.tex

# MarkdownからPDFへ（LaTeX必須）
pandoc input.md -s -o output.pdf

# MarkdownからWordへ
pandoc input.md -s -o output.docx
```

### CLI設定

| オプション | 説明 |
|------------|------|
| `-f, --from <format>` | 入力フォーマット（markdown, html, latexなど） |
| `-t, --to <format>` | 出力フォーマット（html, latex, pdf, docxなど） |
| `-s, --standalone` | ヘッダー/フッター付きのスタンドアロン文書を生成 |
| `-o, --output <file>` | 出力ファイル（拡張子から推測） |
| `--mathml` | TeX数式をMathMLに変換 |
| `--metadata title="Title"` | ドキュメントメタデータを設定 |
| `--toc` | 目次を含める |
| `--template <file>` | カスタムテンプレートを使用 |
| `--help` | 全オプションを表示 |

### セキュリティ警告

⚠️ **Pandocは入力を忠実に処理します。** 信頼できないMarkdownを変換する場合:

- `--sandbox`モードで外部ファイルアクセスを無効化
- 入力を事前に検証
- ブラウザ表示時はHTML出力をサニタイズ

```bash
# 信頼できない入力をsandboxモードで処理
pandoc --sandbox input.md -o output.html
```

### 対応Markdownフレーバー

| フレーバー | 対応度 |
|------------|--------|
| Pandoc Markdown | 100%（ネイティブ） |
| CommonMark | 完全対応（`-f commonmark`使用） |
| GitHub Flavored Markdown | 完全対応（`-f gfm`使用） |
| MultiMarkdown | 部分対応 |

### トラブルシューティング

| 問題 | 解決策 |
|-------|---------|
| PDF生成に失敗 | LaTeX（MacTeX、MiKTeX、texlive）をインストール |
| Windowsでの文字コード問題 | `chcp 65001`を実行してからpandocを使用 |
| スタンドアロンヘッダーがない | `-s`フラグを付ける |
| 数式がレンダリングされない | `--mathml`または`--mathjax`オプションを使用 |
| テーブルがレンダリングされない | パイプとダッシュで正しいテーブル構文を使う |

## [`gomarkdown/markdown`](references/gomarkdown.md)の利用

### 前提条件

- Go 1.18以上がインストールされていること
- ライブラリのインストール: `go get github.com/gomarkdown/markdown`
- CLIツールのインストール: `go install github.com/gomarkdown/mdtohtml@latest`

### クイック変換方法

#### 方法1: シンプル変換（Goコード）

```go
package main

import (
    "fmt"
    "github.com/gomarkdown/markdown"
)

func main() {
    md := []byte("# Hello World\n\nThis is **bold** text.")
    html := markdown.ToHTML(md, nil, nil)
    fmt.Println(string(html))
}
```

#### 方法2: CLIツール

```bash
# mdtohtmlをインストール
go install github.com/gomarkdown/mdtohtml@latest

# ファイルを変換
mdtohtml input.md output.html

# ファイルを変換（標準出力へ）
mdtohtml input.md
```

#### 方法3: カスタムパーサーとレンダラー

```go
package main

import (
    "github.com/gomarkdown/markdown"
    "github.com/gomarkdown/markdown/html"
    "github.com/gomarkdown/markdown/parser"
)

func mdToHTML(md []byte) []byte {
    // 拡張機能付きパーサーを作成
    extensions := parser.CommonExtensions | parser.AutoHeadingIDs | parser.NoEmptyLineBeforeBlock
    p := parser.NewWithExtensions(extensions)
    doc := p.Parse(md)

    // 拡張機能付きHTMLレンダラーを作成
    htmlFlags := html.CommonFlags | html.HrefTargetBlank
    opts := html.RendererOptions{Flags: htmlFlags}
    renderer := html.NewRenderer(opts)

    return markdown.Render(doc, renderer)
}
```

### CLI設定

`mdtohtml` CLIツールは最小限のオプション:

```bash
mdtohtml input-file [output-file]
```

高度な設定はGoライブラリをプログラム的に利用し、パーサーやレンダラーのオプションを設定:

| パーサー拡張 | 説明 |
|--------------|------|
| `parser.CommonExtensions` | テーブル、フェンスコード、自動リンク、取り消し線など |
| `parser.AutoHeadingIDs` | 見出しにIDを自動生成 |
| `parser.NoEmptyLineBeforeBlock` | ブロック前に空行不要 |
| `parser.MathJax` | LaTeX数式のMathJax対応 |

| HTMLフラグ | 説明 |
|------------|------|
| `html.CommonFlags` | 一般的なHTML出力フラグ |
| `html.HrefTargetBlank` | リンクに`target="_blank"`を追加 |
| `html.CompletePage` | 完全なHTMLページを生成 |
| `html.UseXHTML` | XHTML出力を生成 |

### セキュリティ警告

⚠️ **gomarkdownは出力HTMLをサニタイズしません。** 信頼できない入力にはBluemondayを使用:

```go
import (
    "github.com/microcosm-cc/bluemonday"
    "github.com/gomarkdown/markdown"
)

maybeUnsafeHTML := markdown.ToHTML(md, nil, nil)
html := bluemonday.UGCPolicy().SanitizeBytes(maybeUnsafeHTML)
```

推奨サニタイザー: [Bluemonday](https://github.com/microcosm-cc/bluemonday)

### 対応Markdownフレーバー

| フレーバー | 対応度 |
|------------|--------|
| オリジナルMarkdown | 100% |
| CommonMark | 高い（拡張機能付き） |
| GitHub Flavored Markdown | 高い（テーブル、フェンスコード、取り消し線対応） |
| MathJax/LaTeX数式 | 拡張機能で対応 |
| Mmark | 対応 |

### トラブルシューティング

| 問題 | 解決策 |
|-------|---------|
| Windows/Macの改行が解析されない | `parser.NormalizeNewlines(input)`を使用 |
| テーブルがレンダリングされない | `parser.Tables`拡張を有効化 |
| コードブロックのハイライトなし | Chromaなどのシンタックスハイライターと統合 |
| 数式がレンダリングされない | `parser.MathJax`拡張を有効化 |
| XSS脆弱性 | Bluemondayでサニタイズ |

## [`jekyll`](references/jekyll.md)の利用

### 前提条件

- Ruby 2.7.0以上
- RubyGems
- GCCとMake（ネイティブ拡張用）
- JekyllとBundlerのインストール: `gem install jekyll bundler`

### クイック変換方法

#### 方法1: 新規サイト作成

```bash
# 新しいJekyllサイトを作成
jekyll new myblog

# サイトディレクトリへ移動
cd myblog

# ローカルでビルド＆サーブ
bundle exec jekyll serve

# http://localhost:4000 でアクセス
```

#### 方法2: 静的サイトのビルド

```bash
# _siteディレクトリにビルド
bundle exec jekyll build

# 本番環境でビルド
JEKYLL_ENV=production bundle exec jekyll build
```

#### 方法3: ライブリロード開発

```bash
# ライブリロード付きでサーブ
bundle exec jekyll serve --livereload

# 下書きも含めてサーブ
bundle exec jekyll serve --drafts
```

### CLI設定

| コマンド | 説明 |
|----------|------|
| `jekyll new <path>` | 新しいJekyllサイトを作成 |
| `jekyll build` | `_site`ディレクトリにビルド |
| `jekyll serve` | ローカルでビルド＆サーブ |
| `jekyll clean` | 生成ファイルを削除 |
| `jekyll doctor` | 設定の問題をチェック |

| サーブオプション | 説明 |
|------------------|------|
| `--livereload` | 変更時にブラウザをリロード |
| `--drafts` | 下書きを含める |
| `--port <port>` | サーバーポート（デフォルト4000） |
| `--host <host>` | サーバーホスト（デフォルトlocalhost） |
| `--baseurl <url>` | ベースURLを設定 |

### セキュリティ警告

⚠️ **Jekyllのセキュリティ注意点:**

- 本番環境で`safe: false`を避ける
- `_config.yml`の`exclude`で機密ファイルの公開を防止
- 外部入力を受け入れる場合はユーザー生成コンテンツをサニタイズ
- Jekyllとプラグインは常に最新に保つ

```yaml
# _config.ymlのセキュリティ設定例
exclude:
  - Gemfile
  - Gemfile.lock
  - node_modules
  - vendor
```

### 対応Markdownフレーバー

| フレーバー | 対応度 |
|------------|--------|
| Kramdown（デフォルト） | 100% |
| CommonMark | プラグイン(jekyll-commonmark)経由で対応 |
| GitHub Flavored Markdown | プラグイン(jekyll-commonmark-ghpages)経由で対応 |
| RedCarpet | プラグイン経由（非推奨） |

`_config.yml`でMarkdownプロセッサを設定:

```yaml
markdown: kramdown
kramdown:
  input: GFM
  syntax_highlighter: rouge
```

### トラブルシューティング

| 問題 | 解決策 |
|-------|---------|
| Ruby 3.0以上でサーブ失敗 | `bundle add webrick`を実行 |
| Gem依存エラー | `bundle install`を実行 |
| ビルドが遅い | `--incremental`フラグを使用 |
| Liquid構文エラー | コンテンツ内の未エスケープの`{`を確認 |
| プラグインが読み込まれない | `_config.yml`のpluginsリストに追加 |

## [`hugo`](references/hugo.md)の利用

### 前提条件

- Hugoがインストールされていること（<https://gohugo.io/installation/>から入手）
- Git（テーマやモジュール用に推奨）
- Go（Hugo Modules用に任意）

### クイック変換方法

#### 方法1: 新規サイト作成

```bash
# 新しいHugoサイトを作成
hugo new site mysite

# サイトディレクトリへ移動
cd mysite

# テーマを追加
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> hugo.toml

# コンテンツ作成
hugo new content posts/my-first-post.md

# 開発サーバー起動
hugo server -D
```

#### 方法2: 静的サイトのビルド

```bash
# publicディレクトリにビルド
hugo

# ミニファイ付きビルド
hugo --minify

# 特定環境向けビルド
hugo --environment production
```

#### 方法3: 開発サーバー

```bash
# 下書き含めてサーバー起動
hugo server -D

# ライブリロード付きで全インターフェースにバインド
hugo server --bind 0.0.0.0 --baseURL http://localhost:1313/

# ポート指定で起動
hugo server --port 8080
```

### CLI設定

| コマンド | 説明 |
|----------|------|
| `hugo new site <name>` | 新しいHugoサイトを作成 |
| `hugo new content <path>` | 新しいコンテンツファイルを作成 |
| `hugo` | `public`ディレクトリにビルド |
| `hugo server` | 開発サーバーを起動 |
| `hugo mod init` | Hugo Modulesを初期化 |

| ビルドオプション | 説明 |
|------------------|------|
| `-D, --buildDrafts` | 下書きを含める |
| `-E, --buildExpired` | 期限切れコンテンツを含める |
| `-F, --buildFuture` | 未来日付コンテンツを含める |
| `--minify` | 出力をミニファイ |
| `--gc` | ビルド後にガベージコレクション実行 |
| `-d, --destination <path>` | 出力ディレクトリ指定 |

| サーバーオプション | 説明 |
|--------------------|------|
| `--bind <ip>` | バインドするインターフェース |
| `-p, --port <port>` | ポート番号（デフォルト1313） |
| `--liveReloadPort <port>` | ライブリロード用ポート |
| `--disableLiveReload` | ライブリロードを無効化 |
| `--navigateToChanged` | 変更コンテンツへ自動遷移 |

### セキュリティ警告

⚠️ **Hugoのセキュリティ注意点:**

- 外部コマンドのセキュリティポリシーを`hugo.toml`で設定
- 公開リポジトリで`--enableGitInfo`は慎重に使用
- ユーザー生成コンテンツのショートコードパラメータを検証

```toml
# hugo.tomlのセキュリティ設定例
[security]
  enableInlineShortcodes = false
  [security.exec]
    allow = ['^go$', '^npx$', '^postcss$']
  [security.funcs]
    getenv = ['^HUGO_', '^CI$']
  [security.http]
    methods = ['(?i)GET|POST']
    urls = ['.*']
```

### 対応Markdownフレーバー

| フレーバー | 対応度 |
|------------|--------|
| Goldmark（デフォルト） | 100%（CommonMark準拠） |
| GitHub Flavored Markdown | 完全対応（テーブル、取り消し線、自動リンク） |
| CommonMark | 100% |
| Blackfriday（旧） | 非推奨 |

`hugo.toml`でMarkdownを設定:

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
    [markup.goldmark.renderer]
      unsafe = false  # trueにすると生HTMLを許可
```

### トラブルシューティング

| 問題 | 解決策 |
|-------|---------|
| パスで「ページが見つかりません」 | 設定の`baseURL`を確認 |
| テーマが読み込まれない | `themes/`またはHugo Modulesを確認 |
| ビルドが遅い | `--templateMetrics`でボトルネックを特定 |
| 生HTMLがレンダリングされない | goldmark設定の`unsafe = true`を設定 |
| 画像が読み込まれない | `static/`フォルダー構造を確認 |
| モジュールエラー | `hugo mod tidy`を実行 |

## 参考資料

### Markdownの記述とスタイリング

- [basic-markdown.md](references/basic-markdown.md)
- [code-blocks.md](references/code-blocks.md)
- [collapsed-sections.md](references/collapsed-sections.md)
- [tables.md](references/tables.md)
- [writing-mathematical-expressions.md](references/writing-mathematical-expressions.md)
- Markdownガイド: <https://www.markdownguide.org/basic-syntax/>
- Markdownのスタイリング: <https://github.com/sindresorhus/github-markdown-css>

### [`markedJS/marked`](references/marked.md)

- 公式ドキュメント: <https://marked.js.org/>
- 高度なオプション: <https://marked.js.org/using_advanced>
- 拡張性: <https://marked.js.org/using_pro>
- GitHubリポジトリ: <https://github.com/markedjs/marked>

### [`pandoc`](references/pandoc.md)

- はじめに: <https://pandoc.org/getting-started.html>
- 公式マニュアル: <https://pandoc.org/MANUAL.html>
- 拡張性: <https://pandoc.org/extras.html>
- GitHubリポジトリ: <https://github.com/jgm/pandoc>

### [`gomarkdown/markdown`](references/gomarkdown.md)

- 公式ドキュメント: <https://pkg.go.dev/github.com/gomarkdown/markdown>
- 高度な設定: <https://pkg.go.dev/github.com/gomarkdown/markdown@v0.0.0-20250810172220-2e2c11897d1a/html>
- Markdown処理: <https://blog.kowalczyk.info/article/cxn3/advanced-markdown-processing-in-go.html>
- GitHubリポジトリ: <https://github.com/gomarkdown/markdown>

### [`jekyll`](references/jekyll.md)

- 公式ドキュメント: <https://jekyllrb.com/docs/>
- 設定オプション: <https://jekyllrb.com/docs/configuration/options/>
- プラグイン: <https://jekyllrb.com/docs/plugins/>
  - [インストール](https://jekyllrb.com/docs/plugins/installation/)
  - [ジェネレーター](https://jekyllrb.com/docs/plugins/generators/)
  - [コンバーター](https://jekyllrb.com/docs/plugins/converters/)
  - [コマンド](https://jekyllrb.com/docs/plugins/commands/)
  - [タグ](https://jekyllrb.com/docs/plugins/tags/)
  - [フィルター](https://jekyllrb.com/docs/plugins/filters/)
  - [フック](https://jekyllrb.com/docs/plugins/hooks/)
- GitHubリポジトリ: <https://github.com/jekyll/jekyll>

### [`hugo`](references/hugo.md)

- 公式ドキュメント: <https://gohugo.io/documentation/>
- 全設定一覧: <https://gohugo.io/configuration/all/>
- エディタプラグイン: <https://gohugo.io/tools/editors/>
- GitHubリポジトリ: <https://github.com/gohugoio/hugo>
