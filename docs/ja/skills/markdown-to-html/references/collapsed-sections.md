# 折りたたみセクションで情報を整理する

`<details>`タグを使って折りたたみセクションを作成すると、Markdownをすっきりまとめることができます。

## 折りたたみセクションの作り方

Markdownの一部を一時的に隠して、読者が展開して内容を確認できる折りたたみセクションを作成できます。たとえば、技術的な詳細をイシューコメントに含めたいが、すべての読者にとって必ずしも関連性や興味があるとは限らない場合、その詳細を折りたたみセクションに入れることができます。

`<details>`ブロック内のMarkdownは、読者が<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-triangle-right" aria-label="The right triangle icon" role="img"><path d="m6.427 4.427 3.396 3.396a.25.25 0 0 1 0 .354l-3.396 3.396A.25.25 0 0 1 6 11.396V4.604a.25.25 0 0 1 .427-.177Z"></path></svg>をクリックして展開するまで折りたたまれたままになります。

`<details>`ブロック内では、`<summary>`タグを使って読者に中身を知らせます。ラベルは<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-triangle-right" aria-label="The right triangle icon" role="img"><path d="m6.427 4.427 3.396 3.396a.25.25 0 0 1 0 .354l-3.396 3.396A.25.25 0 0 1 6 11.396V4.604a.25.25 0 0 1 .427-.177Z"></path></svg>の右側に表示されます。

````markdown
<details>

<summary>折りたたみセクションのヒント</summary>

### 見出しを追加できます

折りたたみセクション内にテキストを追加できます。

画像やコードブロックも追加可能です。

```ruby
   puts "Hello World"
```

</details>
````

`<summary>`ラベル内のMarkdownはデフォルトで折りたたまれています：

![このページの上記MarkdownがGitHubでレンダリングされたスクリーンショット。右向きの矢印と「折りたたみセクションのヒント」という見出しが表示されています。](https://docs.github.com/assets/images/help/writing/collapsed-section-view.png)

読者が<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-triangle-right" aria-label="The right triangle icon" role="img"><path d="m6.427 4.427 3.396 3.396a.25.25 0 0 1 0 .354l-3.396 3.396A.25.25 0 0 1 6 11.396V4.604a.25.25 0 0 1 .427-.177Z"></path></svg>をクリックすると、詳細が展開されます：

![このページの上記MarkdownがGitHubでレンダリングされたスクリーンショット。折りたたみセクションには見出し、テキスト、画像、コードブロックが含まれています。](https://docs.github.com/assets/images/help/writing/open-collapsed-section.png)

オプションで、セクションをデフォルトで開いた状態にしたい場合は、`<details>`タグに`open`属性を追加します：

```html
<details open>
```

## さらに読む

* [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
* [基本的な書き方とフォーマットの構文](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
