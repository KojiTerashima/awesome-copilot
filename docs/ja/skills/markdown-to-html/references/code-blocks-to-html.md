# コードブロックからHTMLへ

## フェンス付きコードブロック（言語指定なし）

### Markdown

```
function test() {
  console.log("この関数の前の空行に気づきましたか？");
}
```

### 解析されたHTML

```html
<pre><code>
function test() {
  console.log("この関数の前の空行に気づきましたか？");
}
</code></pre>
```

---

## GitHubのヒントコールアウト

### Markdown

```md
> [!TIP]
> リスト内で書式を保持するには、フェンスなしコードブロックを8スペース分インデントしてください。
```

### 解析されたHTML（GitHub特有）

```html
<blockquote class="markdown-alert markdown-alert-tip">
  <p><strong>ヒント</strong></p>
  <p>リスト内で書式を保持するには、フェンスなしコードブロックを8スペース分インデントしてください。</p>
</blockquote>
```

---

## コードブロック内にバッククォートを表示する

### Markdown

`````md
    ````
    ```
    見てください！バッククォートが見えます。
    ```
    ````
`````

### 解析されたHTML

```html
    <pre><code>
    ```

    見てください！バッククォートが見えます。

    ```
    </code></pre>
```

## シンタックスハイライト（言語識別子）

### Markdown

```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```

### 解析されたHTML

```html
<pre><code class="language-ruby">
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
</code></pre>
```

> `language-ruby` クラスはGitHubのシンタックスハイライター（Linguist + grammar）によって利用されます。

### まとめ：シンタックスハイライトのルール（HTMLレベル）

| Markdownフェンス | 解析された`<code>`タグ           |
| -------------- | ------------------------------ |
| ```js          | `<code class="language-js">`   |
| ```html        | `<code class="language-html">` |
| ```md          | `<code class="language-md">`   |
| ```（言語なし） | `<code>`                       |

---

## HTMLコメント（レンダラーによって無視される）

```md
<!-- 内部ドキュメントコメント -->
```

```html
<!-- 内部ドキュメントコメント -->
```

---

## リンク

```md
[GitHubでの執筆と書式設定について](https://docs.github.com/...)
```

```html
<a href="https://docs.github.com/...">GitHubでの執筆と書式設定について</a>
```

---

## リスト

```md
* [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
```

```html
<ul>
  <li>
    <a href="https://github.github.com/gfm/">GitHub Flavored Markdown Spec</a>
  </li>
</ul>
```

---

## 図（概念的な解析）

### Markdown

````md
```mermaid
graph TD
  A --> B
```
````

### 解析されたHTML

```html
<pre><code class="language-mermaid">
graph TD
  A --> B
</code></pre>
```

## 補足

* ここには `language-*` クラスは表示されません。なぜなら**言語識別子が指定されていない**ためです。
* 内部の三重バッククォートは `<code>` 内で**文字通りのテキストとして**保持されます。
