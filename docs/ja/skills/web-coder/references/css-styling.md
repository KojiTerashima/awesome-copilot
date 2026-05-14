# CSS とスタイリングのリファレンス

カスケード スタイル シート、レイアウト システム、最新のスタイル手法に関する包括的なリファレンス。

## コアコンセプト

### CSS (カスケード スタイル シート)

HTML ドキュメントのプレゼンテーションを記述するために使用されるスタイル シート言語。

**CSS を適用する 3 つの方法**:

1. **インライン**: `<div style="color: blue;">`
2. **内部**: HTML 内の `<style>` タグ
3. **外部**: 個別の `.css` ファイル (推奨)

### カスケード

複数のルールが同じ要素をターゲットとする場合に、どの CSS ルールが適用されるかを決定するアルゴリズム。

**優先順位** (最高から最低):

1. インラインスタイル
2. IDセレクター(`#id`)
3. クラスセレクター (`.class`)、属性セレクター、疑似クラス
4. 要素セレクター (`div`、`p`)
5. 継承されたプロパティ

**重要**: `!important` 宣言は通常の詳細性をオーバーライドします (慎重に使用してください)

### CSS セレクター

|セレクター |例 |説明 |
|----------|-----------|---------------|
|要素 | `p` |すべての `<p>` 要素を選択します |
|クラス | `.button` | `class="button"` を使用して要素を選択します |
| ID | `#header` | `id="header"` で要素を選択します |
|ユニバーサル | `*` |すべての要素を選択します |
|子孫 | `div p` | `<div>` 内の `<p>` (任意のレベル) |
|子供 | `div > p` | `<div>` の直接の子 `<p>` |
|隣接する兄弟 | `h1 + p` | `<h1>` の直後 | `<p>`
|一般的な兄弟 | `h1 ~ p` | `<h1>` 以降のすべての `<p>` 兄弟 |
|属性 | `[type="text"]` |特定の属性を持つ要素 |
|属性に含まれる | `[href*="example"]` |部分文字列 | が含まれています
|属性の開始 | `[href^="https"]` |文字列で始まります |
|属性の終了 | `[href$=".pdf"]` |文字列 | で終わる

### 疑似クラス

状態または位置に基づいて要素をターゲットにします。```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }
a:focus { outline: 2px solid blue; }

/* Structural */
li:first-child { font-weight: bold; }
li:last-child { border-bottom: none; }
li:nth-child(odd) { background: #f0f0f0; }
li:nth-child(3n) { color: red; }
p:not(.special) { color: gray; }

/* Form states */
input:required { border-color: red; }
input:valid { border-color: green; }
input:invalid { border-color: red; }
input:disabled { opacity: 0.5; }
input:checked + label { font-weight: bold; }
```### 擬似要素

要素の特定の部分をスタイル設定します。```css
/* First line/letter */
p::first-line { font-weight: bold; }
p::first-letter { font-size: 2em; }

/* Generated content */
.quote::before { content: '"'; }
.quote::after { content: '"'; }

/* Selection */
::selection { background: yellow; color: black; }

/* Placeholder */
input::placeholder { color: #999; }
```## ボックスモデル

すべての要素は次のような長方形のボックスです。

1. **コンテンツ**: 実際のコンテンツ (テキスト、画像)
2. **パディング**: コンテンツの周囲、境界線の内側のスペース
3. **境界線**: パディングの周囲の線
4. **マージン**: 境界線の外側のスペース```css
.box {
  /* Content size */
  width: 300px;
  height: 200px;
  
  /* Padding */
  padding: 20px; /* All sides */
  padding: 10px 20px; /* Vertical | Horizontal */
  padding: 10px 20px 15px 25px; /* Top | Right | Bottom | Left */
  
  /* Border */
  border: 2px solid #333;
  border-radius: 8px;
  
  /* Margin */
  margin: 20px auto; /* Vertical | Horizontal (auto centers) */
  
  /* Box-sizing changes how width/height work */
  box-sizing: border-box; /* Include padding/border in width/height */
}
```## レイアウト システム

### フレックスボックス

1 次元レイアウト システム (行または列):```css
.container {
  display: flex;
  
  /* Direction */
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  
  /* Wrapping */
  flex-wrap: wrap; /* nowrap | wrap | wrap-reverse */
  
  /* Main axis alignment */
  justify-content: center; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
  
  /* Cross axis alignment */
  align-items: center; /* flex-start | flex-end | center | stretch | baseline */
  
  /* Multi-line cross axis */
  align-content: center; /* flex-start | flex-end | center | space-between | space-around | stretch */
  
  /* Gap between items */
  gap: 1rem;
}

.item {
  /* Grow factor */
  flex-grow: 1; /* Takes available space */
  
  /* Shrink factor */
  flex-shrink: 1; /* Can shrink if needed */
  
  /* Base size */
  flex-basis: 200px; /* Initial size before growing/shrinking */
  
  /* Shorthand */
  flex: 1 1 200px; /* grow | shrink | basis */
  
  /* Individual alignment */
  align-self: flex-end; /* Overrides container's align-items */
  
  /* Order */
  order: 2; /* Change visual order (default: 0) */
}
```### CSS グリッド

2 次元レイアウト システム (行と列):```css
.container {
  display: grid;
  
  /* Define columns */
  grid-template-columns: 200px 1fr 1fr; /* Fixed | Flexible | Flexible */
  grid-template-columns: repeat(3, 1fr); /* Three equal columns */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Responsive */
  
  /* Define rows */
  grid-template-rows: 100px auto 50px; /* Fixed | Auto | Fixed */
  
  /* Named areas */
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
  
  /* Gap between cells */
  gap: 1rem; /* Row and column gap */
  row-gap: 1rem;
  column-gap: 2rem;
  
  /* Alignment */
  justify-items: start; /* Align items horizontally within cells */
  align-items: start; /* Align items vertically within cells */
  justify-content: center; /* Align grid within container horizontally */
  align-content: center; /* Align grid within container vertically */
}

.item {
  /* Span columns */
  grid-column: 1 / 3; /* Start / End */
  grid-column: span 2; /* Span 2 columns */
  
  /* Span rows */
  grid-row: 1 / 3;
  grid-row: span 2;
  
  /* Named area */
  grid-area: header;
  
  /* Individual alignment */
  justify-self: center; /* Horizontal alignment */
  align-self: center; /* Vertical alignment */
}
```### グリッドとフレックスボックス

|使用例 |ベストチョイス |
|----------|---------------|
| 1 次元レイアウト (行または列) |フレックスボックス |
| 2 次元レイアウト (行と列) |グリッド |
|項目を 1 つの軸に沿って整列させる |フレックスボックス |
|複雑なページ レイアウトを作成する |グリッド |
|項目間のスペースを分散する |フレックスボックス |
|行と列を正確に制御 |グリッド |
|コンテンツファーストのレスポンシブデザイン |フレックスボックス |
|レイアウトファーストのレスポンシブデザイン |グリッド |

## 位置決め

### ポジションの種類```css
/* Static (default) - normal flow */
.static { position: static; }

/* Relative - offset from normal position */
.relative {
  position: relative;
  top: 10px; /* Move down 10px */
  left: 20px; /* Move right 20px */
}

/* Absolute - removed from flow, positioned relative to nearest positioned ancestor */
.absolute {
  position: absolute;
  top: 0;
  right: 0;
}

/* Fixed - removed from flow, positioned relative to viewport */
.fixed {
  position: fixed;
  bottom: 20px;
  right: 20px;
}

/* Sticky - switches between relative and fixed based on scroll */
.sticky {
  position: sticky;
  top: 0; /* Sticks to top when scrolling */
}
```### インセットのプロパティ

ポジショニングの略記:```css
.element {
  position: absolute;
  inset: 0; /* All sides: top, right, bottom, left = 0 */
  inset: 10px 20px; /* Vertical | Horizontal */
  inset: 10px 20px 30px 40px; /* Top | Right | Bottom | Left */
}
```### コンテキストのスタッキング

`z-index` を使用して階層化を制御します。```css
.behind { z-index: 1; }
.ahead { z-index: 10; }
.top { z-index: 100; }
```**注意**: `z-index` は位置決めされた要素に対してのみ機能します (`static` ではありません)。

## レスポンシブデザイン

### メディアクエリ

デバイスの特性に基づいてスタイルを適用します。```css
/* Mobile-first approach */
.container {
  padding: 1rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
  }
}

/* Landscape orientation */
@media (orientation: landscape) {
  .header { height: 60px; }
}

/* High-DPI screens */
@media (min-resolution: 192dpi) {
  .logo { background-image: url('logo@2x.png'); }
}

/* Dark mode preference */
@media (prefers-color-scheme: dark) {
  body {
    background: #222;
    color: #fff;
  }
}

/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```### 対応ユニット

|単位 |説明 |例 |
|------|---------------|----------|
| `px` |ピクセル (絶対) | `16px` |
| `em` |親のフォント サイズとの相対値 | `1.5em` |
| `rem` |ルート font-size を基準とした値 | `1.5rem` |
| `%` |親との相対 | `50%` |
| `vw` |ビューポート幅 (1vw = ビューポート幅の 1%) | `50vw` |
| `vh` |ビューポートの高さ | `100vh` |
| `vmin` | vw または vh の小さい方 | `10vmin` |
| `vmax` | vw または vh の大きい方 | `10vmax` |
| `ch` | 「0」文字の幅 | `40ch` |
| `fr` |利用可能なスペースの割合 (グリッドのみ) | `1fr` |

### レスポンシブ画像```css
img {
  max-width: 100%;
  height: auto;
}

/* Art direction with picture element */
```

```html
<picture>
  <source media="(min-width: 1024px)" srcset="large.jpg">
  <source media="(min-width: 768px)" srcset="medium.jpg">
  <img src="small.jpg" alt="Responsive image">
</picture>
```## タイポグラフィー```css
.text {
  /* Font family */
  font-family: 'Helvetica Neue', Arial, sans-serif;
  
  /* Font size */
  font-size: 16px; /* Base size */
  font-size: 1rem; /* Relative to root */
  font-size: clamp(14px, 2vw, 20px); /* Responsive with min/max */
  
  /* Font weight */
  font-weight: normal; /* 400 */
  font-weight: bold; /* 700 */
  font-weight: 300; /* Light */
  
  /* Font style */
  font-style: italic;
  
  /* Line height */
  line-height: 1.5; /* 1.5 times font-size */
  line-height: 24px;
  
  /* Letter spacing */
  letter-spacing: 0.05em;
  
  /* Text alignment */
  text-align: left; /* left | right | center | justify */
  
  /* Text decoration */
  text-decoration: underline;
  text-decoration: none; /* Remove underline from links */
  
  /* Text transform */
  text-transform: uppercase; /* uppercase | lowercase | capitalize */
  
  /* Word spacing */
  word-spacing: 0.1em;
  
  /* White space handling */
  white-space: nowrap; /* Don't wrap */
  white-space: pre-wrap; /* Preserve whitespace, wrap lines */
  
  /* Text overflow */
  overflow: hidden;
  text-overflow: ellipsis; /* Show ... when text overflows */
  
  /* Word break */
  word-wrap: break-word; /* Break long words */
  overflow-wrap: break-word; /* Modern version */
}
```## 色```css
.colors {
  /* Named colors */
  color: red;
  
  /* Hex */
  color: #ff0000; /* Red */
  color: #f00; /* Shorthand */
  color: #ff0000ff; /* With alpha */
  
  /* RGB */
  color: rgb(255, 0, 0);
  color: rgba(255, 0, 0, 0.5); /* With alpha */
  color: rgb(255 0 0 / 0.5); /* Modern syntax */
  
  /* HSL (Hue, Saturation, Lightness) */
  color: hsl(0, 100%, 50%); /* Red */
  color: hsla(0, 100%, 50%, 0.5); /* With alpha */
  color: hsl(0 100% 50% / 0.5); /* Modern syntax */
  
  /* Color keywords */
  color: currentColor; /* Inherit color */
  color: transparent;
}
```### CSS カラースペース

より広い色域を実現する最新の色空間:```css
.modern-colors {
  /* Display P3 (Apple devices) */
  color: color(display-p3 1 0 0);
  
  /* Lab color space */
  color: lab(50% 125 0);
  
  /* LCH color space */
  color: lch(50% 125 0deg);
}
```## アニメーションとトランジション

### トランジション

状態間のスムーズな変化:```css
.button {
  background: blue;
  color: white;
  transition: all 0.3s ease;
  /* transition: property duration timing-function delay */
}

.button:hover {
  background: darkblue;
  transform: scale(1.05);
}

/* Individual properties */
.element {
  transition-property: opacity, transform;
  transition-duration: 0.3s, 0.5s;
  transition-timing-function: ease, ease-in-out;
  transition-delay: 0s, 0.1s;
}
```### キーフレーム アニメーション```css
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.element {
  animation: fadeIn 0.5s ease forwards;
  /* animation: name duration timing-function delay iteration-count direction fill-mode */
}

/* Multiple keyframes */
@keyframes slide {
  0% { transform: translateX(0); }
  50% { transform: translateX(100px); }
  100% { transform: translateX(0); }
}

.slider {
  animation: slide 2s infinite alternate;
}
```## 変換```css
.transform {
  /* Translate (move) */
  transform: translate(50px, 100px); /* X, Y */
  transform: translateX(50px);
  transform: translateY(100px);
  
  /* Rotate */
  transform: rotate(45deg);
  
  /* Scale */
  transform: scale(1.5); /* 150% size */
  transform: scale(2, 0.5); /* X, Y different */
  
  /* Skew */
  transform: skew(10deg, 5deg);
  
  /* Multiple transforms */
  transform: translate(50px, 0) rotate(45deg) scale(1.2);
  
  /* 3D transforms */
  transform: rotateX(45deg) rotateY(30deg);
  transform: perspective(500px) translateZ(100px);
}
```## CSS 変数 (カスタム プロパティ)```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --spacing: 1rem;
  --border-radius: 4px;
}

.element {
  color: var(--primary-color);
  padding: var(--spacing);
  border-radius: var(--border-radius);
  
  /* With fallback */
  color: var(--accent-color, red);
}

/* Dynamic changes */
.dark-theme {
  --primary-color: #0056b3;
  --background: #222;
  --text: #fff;
}
```## CSS プリプロセッサ

### 共通機能

- 変数
- ネスティング
- ミックスイン (再利用可能なスタイル)
- 機能
- 輸入品

**人気のプリプロセッサ**: Sass/SCSS、Less、Stylus

## ベストプラクティス

### やるべきこと

- ✅ 外部スタイルシートを使用する
- ✅ ID セレクターではなくクラス セレクターを使用する
- ✅ 特異性を低く保つ
- ✅ 応答単位を使用する (rem、em、%)
- ✅ モバイルファーストのアプローチ
- ✅ テーマに CSS 変数を使用する
- ✅ CSSを論理的に整理する
- ✅ 省略表現プロパティを使用する
- ✅ 本番用に CSS を縮小する

### やってはいけないこと

- ❌ `!important` を過度に使用する
- ❌ インラインスタイルを使用する
- ❌ 固定ピクセル幅を使用する
- ❌ オーバーネストセレクター
- ❌ ベンダー プレフィックスを手動で使用する (自動プレフィックスを使用する)
- ❌ クロスブラウザのテストを忘れる
- ❌ スタイル設定に ID を使用する
- ❌ CSS の特異性を無視する

## 用語集の用語

**対象となる重要な用語**:

- 整列コンテナ
- 調整対象
- アスペクト比
- ベースライン
- ブロック(CSS)
- 境界ボックス
- クロス軸
- CSS
- CSS オブジェクト モデル (CSSOM)
- CSSピクセル
- CSSプリプロセッサ
- ディスクリプタ(CSS)
- フォールバック調整
- フレックス
- フレックスコンテナ
- フレックスアイテム
- フレックスボックス
- 流量相対値
- グリッド
- グリッドエリア
- グリッド軸
- グリッドセル
- グリッド列
- グリッドコンテナ
- グリッド線
- グリッド行
- グリッドトラック
- 側溝
- インクオーバーフロー
- インセットのプロパティ
- レイアウトモード
- 論理プロパティ
- 主軸
- メディアクエリ
- 物理的性質
- ピクセル
- プロパティ(CSS)
- 疑似クラス
- 擬似要素
- セレクター(CSS)
- スタッキングコンテキスト
- スタイルの起源
- スタイルシート
- ベンダープレフィックス

## 追加のリソース

- [MDN CSS リファレンス](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [Flexbox の CSS トリック完全ガイド](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS トリック グリッド完全ガイド](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [使用できますか](https://caniuse.com/) - ブラウザ互換性表