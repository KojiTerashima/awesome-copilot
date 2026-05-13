# 基本的なMarkdownからHTMLへの変換

## 見出し

### Markdown

```md
# 基本的な文章と書式の構文
```

### 解析されたHTML

```html
<h1>基本的な文章と書式の構文</h1>
```

```md
## 見出し
```

```html
<h2>見出し</h2>
```

```md
### 3番目のレベルの見出し
```

```html
<h3>3番目のレベルの見出し</h3>
```

### Markdown

```md
見出し 2
---
```

### 解析されたHTML

```html
<h2>見出し 2</h2>
```

---

## 段落

### Markdown

```md
GitHub上で文章やコードの高度な書式設定を簡単な構文で作成します。
```

### 解析されたHTML

```html
<p>GitHub上で文章やコードの高度な書式設定を簡単な構文で作成します。</p>
```

---

## インライン書式

### 太字

```md
**これは太字のテキストです**
```

```html
<strong>これは太字のテキストです</strong>
```

---

### 斜体

```md
_このテキストは斜体です_
```

```html
<em>このテキストは斜体です</em>
```

---

### 太字＋斜体

```md
***このテキストはすべて重要です***
```

```html
<strong><em>このテキストはすべて重要です</em></strong>
```

---

### 打ち消し線（GFM）

```md
~~これは誤ったテキストでした~~
```

```html
<del>これは誤ったテキストでした</del>
```

---

### 下付き文字 / 上付き文字（生のHTMLパススルー）

```md
これは<sub>下付き文字</sub>のテキストです
```

```html
<p>これは<sub>下付き文字</sub>のテキストです</p>
```

```md
これは<sup>上付き文字</sup>のテキストです
```

```html
<p>これは<sup>上付き文字</sup>のテキストです</p>
```

---

## 引用

### Markdown

```md
> これは引用文です
```

### 解析されたHTML

```html
<blockquote>
  <p>これは引用文です</p>
</blockquote>
```

---

### GitHubアラート（NOTE）

```md
> [!NOTE]
> 役立つ情報です。
```

```html
<blockquote class="markdown-alert markdown-alert-note">
  <p><strong>注意</strong></p>
  <p>役立つ情報です。</p>
</blockquote>
```

> ⚠️ `markdown-alert-*` クラスはGitHub固有であり、標準Markdownではありません。

---

## インラインコード

```md
`git status`を使ってファイルを一覧表示します。
```

```html
<p><code>git status</code>を使ってファイルを一覧表示します。</p>
```

---

## コードブロック

### Markdown

````md
```markdown
git status
git add
```
````

### 解析されたHTML

```html
<pre><code class="language-markdown">
git status
git add
</code></pre>
```

---

## 表

### Markdown

```md
| スタイル | 構文 |
|------|--------|
| 太字 | ** ** |
```

### 解析されたHTML

```html
<table>
  <thead>
    <tr>
      <th>スタイル</th>
      <th>構文</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>太字</td>
      <td><strong> </strong></td>
    </tr>
  </tbody>
</table>
```

---

## リンク

### Markdown

```md
[GitHub Pages](https://pages.github.com/)
```

### 解析されたHTML

```html
<a href="https://pages.github.com/">GitHub Pages</a>
```

---

## 画像

### Markdown

```md
![代替テキスト](image.png)
```

### 解析されたHTML

```html
<img src="image.png" alt="代替テキスト">
```

---

## リスト

### 順不同リスト

```md
- ジョージ・ワシントン
- ジョン・アダムズ
```

```html
<ul>
  <li>ジョージ・ワシントン</li>
  <li>ジョン・アダムズ</li>
</ul>
```

---

### 順序付きリスト

```md
1. ジェームズ・マディソン
2. ジェームズ・モンロー
```

```html
<ol>
  <li>ジェームズ・マディソン</li>
  <li>ジェームズ・モンロー</li>
</ol>
```

---

### ネストされたリスト

```md
1. 最初の項目
   - ネストされた項目
```

```html
<ol>
  <li>
    最初の項目
    <ul>
      <li>ネストされた項目</li>
    </ul>
  </li>
</ol>
```

---

## タスクリスト（GitHub Flavored Markdown）

```md
- [x] 完了
- [ ] 保留中
```

```html
<ul>
  <li>
    <input type="checkbox" checked disabled> 完了
  </li>
  <li>
    <input type="checkbox" disabled> 保留中
  </li>
</ul>
```

---

## メンション

```md
@github/support
```

```html
<a href="https://github.com/github/support" class="user-mention">@github/support</a>
```

---

## 脚注

### Markdown

```md
ここに脚注があります[^1]。

[^1]: 私の参照です。
```

### 解析されたHTML

```html
<p>
  ここに脚注があります
  <sup id="fnref-1">
    <a href="#fn-1">1</a>
  </sup>。
</p>

<section class="footnotes">
  <ol>
    <li id="fn-1">
      <p>私の参照です。</p>
    </li>
  </ol>
</section>
```

---

## HTMLコメント（非表示コンテンツ）

```md
<!-- この内容は表示されません -->
```

```html
<!-- この内容は表示されません -->
```

---

## エスケープされたMarkdown文字

```md
\*斜体ではありません\*
```

```html
<p>*斜体ではありません*</p>
```

---

## 絵文字

```md
:+1:
```

```html
<img class="emoji" alt="👍" src="...">
```

（GitHubは絵文字を`<img>`タグに置き換えます。）

---
