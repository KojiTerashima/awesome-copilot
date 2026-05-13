# Hugoリファレンス

Hugoは世界最速の静的サイトジェネレーターです。ミリ秒単位でサイトを構築し、高度なコンテンツ管理機能をサポートします。

## インストール

### Windows

```powershell
# Chocolateyを使用
choco install hugo-extended

# Scoopを使用
scoop install hugo-extended

# Wingetを使用
winget install Hugo.Hugo.Extended
```

### macOS

```bash
# Homebrewを使用
brew install hugo
```

### Linux

```bash
# Debian/Ubuntu (snap)
snap install hugo --channel=extended

# パッケージマネージャーを使用（最新でない場合あり）
sudo apt-get install hugo

# または https://gohugo.io/installation/ からダウンロード
```

## クイックスタート

### 新しいサイトの作成

```bash
# サイト作成
hugo new site mysite
cd mysite

# git初期化とテーマ追加
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> hugo.toml

# 最初の投稿作成
hugo new content posts/my-first-post.md

# 開発サーバー起動
hugo server -D
```

### ディレクトリ構成

```
mysite/
├── archetypes/      # コンテンツテンプレート
│   └── default.md
├── assets/          # 処理対象のアセット（SCSS、JS）
├── content/         # Markdownコンテンツ
│   └── posts/
├── data/            # データファイル（YAML、JSON、TOML）
├── i18n/            # 多言語対応
├── layouts/         # テンプレート
│   ├── _default/
│   ├── partials/
│   └── shortcodes/
├── static/          # 静的ファイル（そのままコピー）
├── themes/          # テーマ
└── hugo.toml        # 設定ファイル
```

## CLIコマンド

| コマンド | 説明 |
|---------|-------------|
| `hugo new site <name>` | 新しいサイトを作成 |
| `hugo new content <path>` | コンテンツファイルを作成 |
| `hugo` | `public/`にビルド |
| `hugo server` | 開発サーバーを起動 |
| `hugo mod init` | Hugoモジュールを初期化 |
| `hugo mod tidy` | モジュールを整理 |

### ビルドオプション

```bash
# 基本ビルド
hugo

# ミニファイ付きビルド
hugo --minify

# 下書きも含めてビルド
hugo -D

# 特定環境向けビルド
hugo --environment production

# カスタムディレクトリにビルド
hugo -d ./dist

# 詳細出力
hugo -v
```

### サーバーオプション

```bash
# 下書きも含めて起動
hugo server -D

# 全インターフェースにバインド
hugo server --bind 0.0.0.0

# カスタムポート指定
hugo server --port 8080

# ライブリロード無効化
hugo server --disableLiveReload

# 変更されたコンテンツにナビゲート
hugo server --navigateToChanged
```

## 設定（hugo.toml）

```toml
# 基本設定
baseURL = 'https://example.com/'
languageCode = 'en-us'
title = 'My Hugo Site'
theme = 'ananke'

# ビルド設定
[build]
  writeStats = true

# Markdown設定
[markup]
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
    [markup.goldmark.parser]
      autoHeadingID = true
      autoHeadingIDType = 'github'
    [markup.goldmark.renderer]
      unsafe = false
  [markup.highlight]
    style = 'monokai'
    lineNos = true

# タクソノミー
[taxonomies]
  category = 'categories'
  tag = 'tags'
  author = 'authors'

# メニュー
[menus]
  [[menus.main]]
    name = 'Home'
    pageRef = '/'
    weight = 10
  [[menus.main]]
    name = 'Posts'
    pageRef = '/posts'
    weight = 20

# パラメーター
[params]
  description = 'My awesome site'
  author = 'John Doe'
```

## フロントマター

HugoはTOML、YAML、JSONのフロントマターをサポートしています：

### TOML（デフォルト）

```markdown
+++
title = 'My First Post'
date = 2025-01-28T12:00:00-05:00
draft = false
tags = ['hugo', 'tutorial']
categories = ['blog']
author = 'John Doe'
+++

ここにコンテンツ...
```

### YAML

```markdown
---
title: "My First Post"
date: 2025-01-28T12:00:00-05:00
draft: false
tags: ["hugo", "tutorial"]
---

ここにコンテンツ...
```

## テンプレート

### ベーステンプレート (_default/baseof.html)

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{ .Title }} | {{ .Site.Title }}</title>
  {{ partial "head.html" . }}
</head>
<body>
  {{ partial "header.html" . }}
  <main>
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
</body>
</html>
```

### シングルページ (_default/single.html)

```html
{{ define "main" }}
<article>
  <h1>{{ .Title }}</h1>
  <time>{{ .Date.Format "January 2, 2006" }}</time>
  {{ .Content }}
</article>
{{ end }}
```

### リストページ (_default/list.html)

```html
{{ define "main" }}
<h1>{{ .Title }}</h1>
{{ range .Pages }}
  <article>
    <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
    <p>{{ .Summary }}</p>
  </article>
{{ end }}
{{ end }}
```

## ショートコード

### 組み込みショートコード

```markdown
{{< figure src="/images/photo.jpg" title="My Photo" >}}

{{< youtube dQw4w9WgXcQ >}}

{{< gist user 12345 >}}

{{< highlight go >}}
fmt.Println("Hello")
{{< /highlight >}}
```

### カスタムショートコード (layouts/shortcodes/alert.html)

```html
<div class="alert alert-{{ .Get "type" | default "info" }}">
  {{ .Inner | markdownify }}
</div>
```

使用例：

```markdown
{{< alert type="warning" >}}
**警告:** これは重要です！
{{< /alert >}}
```

## コンテンツの整理

### ページバンドル

```
content/
├── posts/
│   └── my-post/           # ページバンドル
│       ├── index.md       # コンテンツ
│       └── image.jpg      # リソース
└── _index.md              # セクションページ
```

### リソースへのアクセス

```html
{{ $image := .Resources.GetMatch "image.jpg" }}
{{ with $image }}
  <img src="{{ .RelPermalink }}" alt="...">
{{ end }}
```

## Hugo Pipes（アセット処理）

### SCSSコンパイル

```html
{{ $styles := resources.Get "scss/main.scss" | toCSS | minify }}
<link rel="stylesheet" href="{{ $styles.RelPermalink }}">
```

### JavaScriptバンドル

```html
{{ $js := resources.Get "js/main.js" | js.Build | minify }}
<script src="{{ $js.RelPermalink }}"></script>
```

## タクソノミー

### 設定例

```toml
[taxonomies]
  tag = 'tags'
  category = 'categories'
```

### フロントマターでの使用例

```markdown
+++
tags = ['go', 'hugo']
categories = ['tutorials']
+++
```

### タクソノミー用語の一覧表示

```html
{{ range .Site.Taxonomies.tags }}
  <a href="{{ .Page.Permalink }}">{{ .Page.Title }} ({{ .Count }})</a>
{{ end }}
```

## 多言語サイト

```toml
defaultContentLanguage = 'en'

[languages]
  [languages.en]
    title = 'My Site'
    weight = 1
  [languages.es]
    title = 'Mi Sitio'
    weight = 2
```

## トラブルシューティング

| 問題 | 解決策 |
|-------|----------|
| ページが見つからない | `baseURL`設定を確認 |
| テーマが読み込まれない | 設定のテーマパスを確認 |
| 生のHTMLが表示されない | goldmark設定で `unsafe = true` に設定 |
| ビルドが遅い | `--templateMetrics`でデバッグ |
| モジュールエラー | `hugo mod tidy`を実行 |
| CSSが更新されない | ブラウザキャッシュをクリア、またはフィンガープリントを使用 |

## リソース

- [Hugoドキュメント](https://gohugo.io/documentation/)
- [Hugoテーマ](https://themes.gohugo.io/)
- [Hugoディスコース](https://discourse.gohugo.io/)
- [GitHubリポジトリ](https://github.com/gohugoio/hugo)
- [クイックリファレンス](https://gohugo.io/quick-reference/)
