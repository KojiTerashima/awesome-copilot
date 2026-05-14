# HTML とマークアップのリファレンス

HTML5、マークアップ言語、ドキュメント構造に関する包括的なリファレンス。

## コアコンセプト

### HTML (ハイパーテキスト マークアップ言語)
Web ページおよび Web アプリケーションを作成するための標準マークアップ言語。

**関連用語**: HTML5、XHTML、マークアップ、セマンティック HTML

### 要素
HTML ドキュメントの構成要素。各要素には開始タグと終了タグがあります (void 要素を除く)。

**共通要素**:
- `<div>` - 汎用コンテナ
- `<span>` - インラインコンテナ
- `<article>` - 自己完結型コンテンツ
- `<section>` - テーマ別のグループ化
- `<nav>` - ナビゲーション リンク
- `<header>` - 紹介コンテンツ
- `<footer>` - フッターの内容
- `<main>` - 主な内容
- `<aside>` - 補足的な内容

### 属性
HTML 要素に関する追加情報を提供するプロパティ。

**共通の属性**:
- `id` - 一意の識別子
- `class` - CSS クラス名
- `src` - 画像/スクリプトのソース URL
- `href` - ハイパーリンクのリファレンス
- `alt` - 代替テキスト
- `title` - アドバイザリーのタイトル
- `data-*` - カスタム データ属性
- `aria-*` - アクセシビリティ属性

### 空白の要素
コンテンツを含めることができず、終了タグを持たない要素。

**例**: `<img>`、`<br>`、`<hr>`、`<input>`、`<meta>`、`<link>`

## セマンティック HTML

### セマンティック HTML とは何ですか?
ブラウザーと開発者の両方に対してその意味を明確に説明する HTML。

**利点**:
- アクセシビリティの向上
- SEOの向上
- メンテナンスが容易
- 組み込まれた意味と構造

### 意味要素

|要素 |目的 |いつ使用するか |
|----------|----------|---------------|
| `<article>` |自己完結型の構成 |ブログ投稿、ニュース記事 |
| `<section>` |コンテンツのテーマごとのグループ化 |章、タブ付きコンテンツ |
| `<nav>` |ナビゲーションリンク |メインメニュー、パンくずリスト |
| `<aside>` |接線コンテンツ |サイドバー、関連リンク |
| `<header>` |紹介コンテンツ |ページ/セクションのヘッダー |
| `<footer>` |フッターの内容 |著作権、連絡先情報 |
| `<main>` |主な内容 |主要なページのコンテンツ |
| `<figure>` |自己完結型コンテンツ |キャプション付きの画像 |
| `<figcaption>` |図のキャプション |画像の説明 |
| `<time>` |日付/時刻 |出版日 |
| `<mark>` |強調表示されたテキスト |検索結果 |
| `<details>` |拡張可能な詳細 |アコーディオン、よくある質問 |
| `<summary>` |詳細については概要 |アコーディオンヘッダー |### 例: 意味論的な文書構造```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Semantic Page Example</title>
</head>
<body>
  <header>
    <h1>Site Title</h1>
    <nav aria-label="Main navigation">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>
  
  <main>
    <article>
      <header>
        <h2>Article Title</h2>
        <time datetime="2026-03-04">March 4, 2026</time>
      </header>
      <p>Article content goes here...</p>
      <footer>
        <p>Author: John Doe</p>
      </footer>
    </article>
  </main>
  
  <aside>
    <h3>Related Content</h3>
    <ul>
      <li><a href="/related">Related Article</a></li>
    </ul>
  </aside>
  
  <footer>
    <p>&copy; 2026 Company Name</p>
  </footer>
</body>
</html>
```## 文書構造

### ドキュメントタイプ
ドキュメントの種類と HTML のバージョンを宣言します。```html
<!DOCTYPE html>
```### ヘッドセクション
ドキュメントに関するメタデータが含まれます。

**共通要素**:
- `<meta>` - メタデータ (文字セット、ビューポート、説明)
- `<title>` - ページタイトル (ブラウザタブに表示)
- `<link>` - 外部リソース (スタイルシート、アイコン)
- `<script>` - JavaScript ファイル
- `<style>` - インラインCSS

### メタデータの例```html
<head>
  <!-- Character encoding -->
  <meta charset="UTF-8">
  
  <!-- Responsive viewport -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <!-- SEO metadata -->
  <meta name="description" content="Page description for search engines">
  <meta name="keywords" content="html, web, development">
  <meta name="author" content="Author Name">
  
  <!-- Open Graph (social media) -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="Page description">
  <meta property="og:image" content="https://example.com/image.jpg">
  
  <!-- Favicon -->
  <link rel="icon" type="image/png" href="/favicon.png">
  
  <!-- Stylesheet -->
  <link rel="stylesheet" href="styles.css">
  
  <!-- Preload critical resources -->
  <link rel="preload" href="critical.css" as="style">
  <link rel="preconnect" href="https://api.example.com">
</head>
```## フォームと入力

### フォーム要素```html
<form action="/submit" method="POST">
  <!-- Text input -->
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" required>
  
  <!-- Email input -->
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>
  
  <!-- Password input -->
  <label for="password">Password:</label>
  <input type="password" id="password" name="password" minlength="8" required>
  
  <!-- Select dropdown -->
  <label for="country">Country:</label>
  <select id="country" name="country">
    <option value="">Select...</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
  </select>
  
  <!-- Textarea -->
  <label for="message">Message:</label>
  <textarea id="message" name="message" rows="4"></textarea>
  
  <!-- Checkbox -->
  <label>
    <input type="checkbox" name="terms" required>
    I agree to the terms
  </label>
  
  <!-- Radio buttons -->
  <fieldset>
    <legend>Choose an option:</legend>
    <label>
      <input type="radio" name="option" value="a">
      Option A
    </label>
    <label>
      <input type="radio" name="option" value="b">
      Option B
    </label>
  </fieldset>
  
  <!-- Submit button -->
  <button type="submit">Submit</button>
</form>
```### 入力タイプ

|タイプ |目的 |例 |
|------|--------|----------|
| `text` |単一行のテキスト | `<input type="text">` |
| `email` |メールアドレス | `<input type="email">` |
| `password` |パスワードフィールド | `<input type="password">` |
| `number` |数値入力 | `<input type="number" min="0" max="100">` |
| `tel` |電話番号 | `<input type="tel">` |
| `url` | URL | `<input type="url">` |
| `date` |日付ピッカー | `<input type="date">` |
| `time` |タイムピッカー | `<input type="time">` |
| `file` |ファイルのアップロード | `<input type="file" accept="image/*">` |
| `checkbox` |チェックボックス | `<input type="checkbox">` |
| `radio` |ラジオボタン | `<input type="radio">` |
| `range` |スライダー | `<input type="range" min="0" max="100">` |
| `color` |カラーピッカー | `<input type="color">` |
| `search` |検索フィールド | `<input type="search">` |

## 関連するマークアップ言語

### XML (拡張マークアップ言語)
人間と機械の両方が読める形式でドキュメントをエンコードするためのマークアップ言語。

**HTML との主な違い**:
- すべてのタグは正しく閉じられている必要があります
- タグでは大文字と小文字が区別されます
- 属性は引用符で囲む必要があります
- カスタムタグ名が許可される

### XHTML (拡張可能なハイパーテキスト マークアップ言語)
HTML を XML として再形式化したもの。 HTML よりも厳密な構文規則。

### MathML (数学的マークアップ言語)
Web 上で数学表記を表示するためのマークアップ言語。```html
<math>
  <mrow>
    <msup>
      <mi>x</mi>
      <mn>2</mn>
    </msup>
    <mo>+</mo>
    <mn>1</mn>
  </mrow>
</math>
```### SVG (スケーラブル ベクター グラフィックス)
2 次元ベクトル グラフィックスを記述するための XML ベースのマークアップ言語。```html
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>
```## 文字エンコーディングと参照

### 文字エンコーディング
文字をバイトとして表現する方法を定義します。

**UTF-8**: ユニバーサル文字エンコーディング標準 (推奨)```html
<meta charset="UTF-8">
```### キャラクターリファレンス
HTML で特殊文字を表現する方法。

**名前付きエンティティ**:
- `&lt;` - 未満 (<)
- `&gt;` - より大きい (>)
- `&amp;` - アンパサンド (&)
- `&quot;` - 引用符 (")
- `&apos;` - アポストロフィ (')
- `&nbsp;` - 非改行スペース
- `&copy;` - 著作権 (©)

**数値エンティティ**:
- `&#60;` - 未満 (<)
- `&#169;` - 著作権 (©)
- `&#8364;` - ユーロ (€)

## ブロック コンテンツとインライン コンテンツ

### ブロックレベルのコンテンツ
新しい行から始まる、レイアウト内に「ブロック」を作成する要素。

**例**: `<div>`、`<p>`、`<h1>`-`<h6>`、`<article>`、`<section>`、`<header>`、`<footer>`、`<nav>`、`<aside>`、`<ul>`、`<ol>`、 `<li>`

### インラインレベルのコンテンツ
新しい行で始まらず、必要な幅だけを占める要素。

**例**: `<span>`、`<a>`、`<strong>`、`<em>`、`<img>`、`<code>`、`<abbr>`、`<cite>`

## ベストプラクティス

### やるべきこと
- ✅ セマンティック HTML 要素を使用する
- ✅ 適切なドキュメント構造 (DOCTYPE、html、head、body) を含める
- ✅ 文字エンコードをUTF-8に設定します
- ✅ 画像には説明的な `alt` 属性を使用します
- ✅ ラベルをフォーム入力に関連付けます
- ✅ 見出し階層を適切に使用する (h1 → h2 → h3)
- ✅ W3C バリデーターで HTML を検証
- ✅ 必要に応じて適切な ARIA ロールを使用する
- ✅ レスポンシブデザイン用のメタビューポートを含める

### やってはいけないこと
- ❌ 意味要素が存在する場合は `<div>` を使用します
- ❌ 見出しレベルをスキップ (h1 → h3)
- ❌ レイアウトにテーブルを使用する
- ❌ タグの閉じ忘れ (void 要素を除く)
- ❌ インライン スタイルを広範囲に使用する
- ❌ 画像の `alt` 属性を省略します
- ❌ ラベルなしでフォームを作成する
- ❌ 非推奨の要素を使用する (`<font>`、`<center>`、`<blink>`)

## MDN の用語集用語

**対象となる重要な用語**:
- 抽象化
- アクセシビリティツリー
- アクセシブルな説明
- アクセシブルな名前
- 属性
- ブロックレベルのコンテンツ
- パンくずリスト
- コンテキストの閲覧
- キャラクター
- 文字コード
- キャラクターリファレンス
- キャラクターセット
- ドキュメントタイプ
- ドキュメント環境
- 要素
- エンティティ
- 頭
- HTML
- HTML5
- ハイパーリンク
- ハイパーテキスト
- インラインレベルのコンテンツ
- マークアップ
- 数学ML
- メタデータ
- セマンティクス
- SVG
- タグ
- ボイド要素
- XHTML
- XML

## 追加のリソース- [MDN HTML リファレンス](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [W3C HTML仕様](https://html.spec.whatwg.org/)
- [HTML5 ドクター](http://html5doctor.com/)
- [W3C マークアップ検証サービス](https://validator.w3.org/)