# 折りたたみセクションをHTMLに変換する

## `<details>` ブロック（Markdown内の生HTML）

### Markdown

````md
<details>

<summary>折りたたみセクションのヒント</summary>

### 見出しを追加できます

折りたたみセクション内にテキストを追加できます。

画像やコードブロックも追加できます。

    ```ruby
    puts "Hello World"
    ```

</details>
````

---

### 解析されたHTML

```html
<details>
  <summary>折りたたみセクションのヒント</summary>

  <h3>見出しを追加できます</h3>

  <p>折りたたみセクション内にテキストを追加できます。</p>

  <p>画像やコードブロックも追加できます。</p>

  <pre><code class="language-ruby">
puts "Hello World"
</code></pre>
</details>
```

#### 注意事項:

* `<details>` 内のMarkdownは通常通り解析されます。
* シンタックスハイライトは `class="language-ruby"` によって保持されます。

---

## デフォルトで開く（`open`属性）

### Markdown

````md
<details open>

<summary>折りたたみセクションのヒント</summary>

### 見出しを追加できます

折りたたみセクション内にテキストを追加できます。

画像やコードブロックも追加できます。

    ```ruby
    puts "Hello World"
    ```

</details>
````

### 解析されたHTML

```html
<details open>
  <summary>折りたたみセクションのヒント</summary>

  <h3>見出しを追加できます</h3>

  <p>折りたたみセクション内にテキストを追加できます。</p>

  <p>画像やコードブロックも追加できます。</p>

  <pre><code class="language-ruby">
puts "Hello World"
</code></pre>
</details>
```

## 重要なルール

* `<details>` と `<summary>` は**生のHTML**であり、Markdown構文ではありません
* `<details>` 内のMarkdownは**通常通り解析されます**
* 折りたたみセクション内でもシンタックスハイライトは正常に機能します
* `<summary>` は**クリック可能なラベル**として使います

## インラインHTML & SVGを含む段落

### Markdown

```md
`<details>`タグを使って折りたたみセクションを作成することで、Markdownを簡潔にできます。
```

### 解析されたHTML

```html
<p>
  `<details>`タグを使って折りたたみセクションを作成することで、Markdownを簡潔にできます。
</p>
```

---

### Markdown（インラインSVGを保持）

```md
`<details>`ブロック内のMarkdownは、読者が<svg ...></svg>をクリックして詳細を展開するまで折りたたまれます。
```

### 解析されたHTML

```html
<p>
  `<details>`ブロック内のMarkdownは、読者が
  <svg version="1.1" width="16" height="16" viewBox="0 0 16 16"
       class="octicon octicon-triangle-right"
       aria-label="右向き三角形アイコン"
       role="img">
    <path d="m6.427 4.427 3.396 3.396a.25.25 0 0 1 0 .354l-3.396 3.396A.25.25 0 0 1 6 11.396V4.604a.25.25 0 0 1 .427-.177Z"></path>
  </svg>
  をクリックして詳細を展開するまで折りたたまれます。
</p>
```
