# Jekyllリファレンス

JekyllはMarkdownコンテンツを完全なウェブサイトに変換する静的サイトジェネレーターです。ブログ対応で、GitHub Pagesの基盤となっています。

## インストール

### 前提条件

- Ruby 2.7.0以上
- RubyGems
- GCCとMake

### Jekyllのインストール

```bash
# JekyllとBundlerをインストール
gem install jekyll bundler
```

### プラットフォーム別インストール

```bash
# macOS（まずXcode CLIツールをインストール）
xcode-select --install
gem install jekyll bundler

# Ubuntu/Debian
sudo apt-get install ruby-full build-essential zlib1g-dev
gem install jekyll bundler

# Windows（RubyInstallerを使用）
# https://rubyinstaller.org/ からダウンロード
gem install jekyll bundler
```

## クイックスタート

### 新しいサイトの作成

```bash
# 新しいJekyllサイトを作成
jekyll new myblog

# サイトディレクトリへ移動
cd myblog

# ビルドしてサーブ
bundle exec jekyll serve

# http://localhost:4000 を開く
```

### ディレクトリ構成

```
myblog/
├── _config.yml      # サイト設定
├── _posts/          # ブログ投稿
│   └── 2025-01-28-welcome.md
├── _layouts/        # ページテンプレート
├── _includes/       # 再利用可能なコンポーネント
├── _data/           # データファイル（YAML、JSON、CSV）
├── _sass/           # Sassパーシャル
├── assets/          # CSS、JS、画像
├── index.md         # ホームページ
└── Gemfile          # Ruby依存関係
```

## CLIコマンド

| コマンド | 説明 |
|---------|-------------|
| `jekyll new <name>` | 新しいサイトを作成 |
| `jekyll build` | `_site/`にビルド |
| `jekyll serve` | ローカルでビルド＆サーブ |
| `jekyll clean` | 生成ファイルを削除 |
| `jekyll doctor` | 問題をチェック |

### ビルドオプション

```bash
# サイトをビルド
bundle exec jekyll build

# 本番環境でビルド
JEKYLL_ENV=production bundle exec jekyll build

# カスタムディレクトリにビルド
bundle exec jekyll build --destination ./public

# インクリメンタル再生成でビルド
bundle exec jekyll build --incremental
```

### サーブオプション

```bash
# ライブリロード付きでサーブ
bundle exec jekyll serve --livereload

# 下書き投稿を含める
bundle exec jekyll serve --drafts

# ポート指定
bundle exec jekyll serve --port 8080

# 全インターフェースにバインド
bundle exec jekyll serve --host 0.0.0.0
```

## 設定ファイル (_config.yml)

```yaml
# サイト設定
title: My Blog
description: A great blog
baseurl: ""
url: "https://example.com"

# ビルド設定
markdown: kramdown
theme: minima
plugins:
  - jekyll-feed
  - jekyll-seo-tag

# Kramdown設定
kramdown:
  input: GFM
  syntax_highlighter: rouge
  hard_wrap: false

# コレクション
collections:
  docs:
    output: true
    permalink: /docs/:name/

# デフォルト設定
defaults:
  - scope:
      path: ""
      type: "posts"
    values:
      layout: "post"

# 処理対象外
exclude:
  - Gemfile
  - Gemfile.lock
  - node_modules
  - vendor
```

## フロントマター

すべてのコンテンツファイルにはYAMLフロントマターが必要です：

```markdown
---
layout: post
title: "My First Post"
date: 2025-01-28 12:00:00 -0500
categories: blog tutorial
tags: [jekyll, markdown]
author: John Doe
excerpt: "簡単な紹介..."
published: true
---

ここにコンテンツを記述...
```

## Markdownプロセッサ

### Kramdown（デフォルト）

```yaml
# _config.yml
markdown: kramdown
kramdown:
  input: GFM                    # GitHub Flavored Markdown
  syntax_highlighter: rouge
  syntax_highlighter_opts:
    block:
      line_numbers: true
```

### CommonMark

```ruby
# Gemfile
gem 'jekyll-commonmark-ghpages'
```

```yaml
# _config.yml
markdown: CommonMarkGhPages
commonmark:
  options: ["SMART", "FOOTNOTES"]
  extensions: ["strikethrough", "autolink", "table"]
```

## Liquidテンプレート

### 変数

```liquid
{{ page.title }}
{{ site.title }}
{{ content }}
{{ page.date | date: "%B %d, %Y" }}
```

### ループ

```liquid
{% for post in site.posts %}
  <article>
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <p>{{ post.excerpt }}</p>
  </article>
{% endfor %}
```

### 条件分岐

```liquid
{% if page.title %}
  <h1>{{ page.title }}</h1>
{% endif %}

{% unless page.draft %}
  {{ content }}
{% endunless %}
```

### インクルード

```liquid
{% include header.html %}
{% include footer.html param="value" %}
```

## レイアウト

### 基本レイアウト (_layouts/default.html)

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{ page.title }} | {{ site.title }}</title>
  <link rel="stylesheet" href="{{ '/assets/css/style.css' | relative_url }}">
</head>
<body>
  {% include header.html %}
  <main>
    {{ content }}
  </main>
  {% include footer.html %}
</body>
</html>
```

### 投稿レイアウト (_layouts/post.html)

```html
---
layout: default
---
<article>
  <h1>{{ page.title }}</h1>
  <time>{{ page.date | date: "%B %d, %Y" }}</time>
  {{ content }}
</article>
```

## プラグイン

### よく使うプラグイン

```ruby
# Gemfile
group :jekyll_plugins do
  gem 'jekyll-feed'        # RSSフィード
  gem 'jekyll-seo-tag'     # SEOメタタグ
  gem 'jekyll-sitemap'     # XMLサイトマップ
  gem 'jekyll-paginate'    # ページネーション
  gem 'jekyll-archives'    # アーカイブページ
end
```

### プラグインの使用

```yaml
# _config.yml
plugins:
  - jekyll-feed
  - jekyll-seo-tag
  - jekyll-sitemap
```

## トラブルシューティング

| 問題 | 解決策 |
|-------|----------|
| Ruby 3.0+でwebrickエラー | `bundle add webrick` |
| 権限拒否 | `--user-install`を使うかrbenvを利用 |
| ビルドが遅い | `--incremental`を使う |
| Liquidエラー | エスケープされていない `{` `}` を確認 |
| エンコーディング問題 | 設定に `encoding: utf-8` を追加 |
| プラグインが読み込まれない | Gemfileと_config.yml両方に追加 |

## リソース

- [Jekyllドキュメント](https://jekyllrb.com/docs/)
- [Liquidテンプレート言語](https://shopify.github.io/liquid/)
- [Kramdownドキュメント](https://kramdown.gettalong.org/)
- [GitHubリポジトリ](https://github.com/jekyll/jekyll)
- [Jekyllテーマ](https://jekyllthemes.io/)
