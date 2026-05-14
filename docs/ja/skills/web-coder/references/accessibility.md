# アクセシビリティリファレンス

Web アクセシビリティにより、障害のある人を含むすべての人がコンテンツを利用できるようになります。

## WCAG (Web コンテンツ アクセシビリティ ガイドライン)

### レベル
- **A**: 最低レベル
- **AA**: 標準目標 (多くの管轄区域における法的要件)
- **AAA**: アクセシビリティの強化

### 4 つの原則 (POUR)

1. **知覚可能**: ユーザーが知覚できる方法で情報が提示されます。
2. **操作可能**: UI コンポーネントとナビゲーションが操作可能です。
3. **理解可能**: 情報と UI 操作が理解できる
4. **堅牢**: コンテンツは現在および将来のテクノロジーで動作します

## ARIA (アクセス可能なリッチ インターネット アプリケーション)

### ARIA の役割```html
<!-- Landmark roles -->
<nav role="navigation">
<main role="main">
<aside role="complementary">
<footer role="contentinfo">

<!-- Widget roles -->
<div role="button" tabindex="0">Click me</div>
<div role="tab" aria-selected="true">Tab 1</div>
<div role="dialog" aria-labelledby="dialogTitle">

<!-- Document structure -->
<div role="list">
  <div role="listitem">Item 1</div>
</div>
```### ARIA 属性```html
<!-- States -->
<button aria-pressed="true">Toggle</button>
<input aria-invalid="true" aria-errormessage="error1">
<div aria-expanded="false" aria-controls="menu">Menu</div>

<!-- Properties -->
<img alt="" aria-hidden="true">
<input aria-label="Search" type="search">
<dialog aria-labelledby="title" aria-describedby="desc">
  <h2 id="title">Dialog Title</h2>
  <p id="desc">Description</p>
</dialog>

<!-- Relationships -->
<label id="label1" for="input1">Name:</label>
<input id="input1" aria-labelledby="label1">

<!-- Live regions -->
<div aria-live="polite" aria-atomic="true">
  Status updated
</div>
```## キーボード ナビゲーション

### タブオーダー```html
<!-- Natural tab order -->
<button>First</button>
<button>Second</button>

<!-- Custom tab order (avoid if possible) -->
<button tabindex="1">First</button>
<button tabindex="2">Second</button>

<!-- Programmatically focusable  (not in tab order) -->
<div tabindex="-1">Not in tab order</div>

<!-- In tab order -->
<div tabindex="0" role="button">Custom button</div>
```### キーボードイベント```javascript
element.addEventListener('keydown', (e) => {
  switch(e.key) {
    case 'Enter':
    case ' ': // Space
      // Activate
      break;
    case 'Escape':
      // Close/cancel
      break;
    case 'ArrowUp':
    case 'ArrowDown':
    case 'ArrowLeft':
    case 'ArrowRight':
      // Navigate
      break;
  }
});
```## セマンティック HTML```html
<!-- ✅ Good: semantic elements -->
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/">Home</a></li>
  </ul>
</nav>

<!-- ❌ Bad: non-semantic -->
<div class="nav">
  <div><a href="/">Home</a></div>
</div>

<!-- ✅ Good: proper headings hierarchy -->
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>

<!-- ❌ Bad: skipping levels -->
<h1>Page Title</h1>
  <h3>Skipped h2</h3>
```## フォームのアクセシビリティ```html
<form>
  <!-- Labels -->
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" required aria-required="true">
  
  <!-- Error messages -->
  <input
    type="email"
    id="email"
    aria-invalid="true"
    aria-describedby="email-error">
  <span id="email-error" role="alert">
    Please enter a valid email
  </span>
  
  <!-- Fieldset for groups -->
  <fieldset>
    <legend>Choose an option</legend>
    <label>
      <input type="radio" name="option" value="a">
      Option A
    </label>
    <label>
      <input type="radio" name="option" value="b">
      Option B
    </label>
  </fieldset>
  
  <!-- Help text -->
  <label for="password">Password:</label>
  <input
    type="password"
    id="password"
    aria-describedby="password-help">
  <span id="password-help">
    Must be at least 8 characters
  </span>
</form>
```## 画像とメディア```html
<!-- Informative image -->
<img src="chart.png" alt="Sales increased 50% in Q1">

<!-- Decorative image -->
<img src="decorative.png" alt="" role="presentation">

<!-- Complex image -->
<figure>
  <img src="data-viz.png" alt="Sales data visualization">
  <figcaption>
    Detailed description of the data...
  </figcaption>
</figure>

<!-- Video with captions -->
<video controls>
  <source src="video.mp4" type="video/mp4">
  <track kind="captions" src="captions.vtt" srclang="en" label="English">
</video>
```## 色とコントラスト

### WCAG の要件

- **レベル AA**: 通常のテキストの場合は 4.5:1、大きなテキストの場合は 3:1
- **レベル AAA**: 通常のテキストの場合は 7:1、大きなテキストの場合は 4.5:1```css
/* ✅ Good contrast */
.text {
  color: #000; /* Black */
  background: #fff; /* White */
  /* Contrast: 21:1 */
}

/* Don't rely on color alone */
.error {
  color: red;
  /* ✅ Also use icon or text */
  &::before {
    content: '⚠ ';
  }
}
```## スクリーン リーダー

### ベストプラクティス```html
<!-- Skip links for navigation -->
<a href="#main-content" class="skip-link">
  Skip to main content
</a>

<!-- Accessible headings -->
<h1>Main heading (only one)</h1>

<!-- Descriptive links -->
<!-- ❌ Bad -->
<a href="/article">Read more</a>

<!-- ✅ Good -->
<a href="/article">Read more about accessibility</a>

<!-- Hidden content (screen reader only) -->
<span class="sr-only">
  Additional context for screen readers
</span>
```

```css
/* Screen reader only class */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```## 集中管理```css
/* Visible focus indicator */
:focus {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* Don't remove focus entirely */
/* ❌ Bad */
:focus {
  outline: none;
}

/* ✅ Good: custom focus style */
:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(0, 95, 204, 0.5);
}
```

```javascript
// Focus management in modal
function openModal() {
  modal.showModal();
  modal.querySelector('button').focus();
  
  // Trap focus
  modal.addEventListener('keydown', (e) => {
    if (e.key === 'Tab') {
      trapFocus(e, modal);
    }
  });
}
```## テストツール

- **axe DevTools**: ブラウザ拡張機能
- **WAVE**: Webアクセシビリティ評価ツール
- **NVDA**: スクリーン リーダー (Windows)
- **JAWS**: スクリーン リーダー (Windows)
- **VoiceOver**: スクリーン リーダー (macOS/iOS)
- **Lighthouse**: 自動監査

## チェックリスト

- [ ] セマンティック HTML が使用されています
- [ ] すべての画像には代替テキストが含まれます
- [ ] カラーコントラストが WCAG AA を満たす
- [ ] キーボード ナビゲーションが機能する
- [ ] フォーカスインジケーターが表示されます
- [ ] フォームにはラベルが付いています
- [ ] 見出し階層が正しい
- [ ] ARIA が適切に使用される
- [ ] スクリーン リーダーのテスト済み
- [ ] キーボードトラップなし

## 用語集の用語

**対象となる重要な用語**:
- アクセシビリティ
- アクセシビリティツリー
- アクセシブルな説明
- アクセシブルな名前
- アリア
- ATAG
- ブール属性 (ARIA)
- スクリーンリーダー
- UAAG
- ワイ
- WCAG

## 追加のリソース

- [WCAG 2.1 ガイドライン](https://www.w3.org/WAI/WCAG21/quickref/)
- [MDN アクセシビリティ](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [WebAIM](https://webaim.org/)
- 【A11yプロジェクト】(https://www.a11yproject.com/)