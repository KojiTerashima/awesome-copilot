# コードブロックの作成とハイライト

フェンス付きコードブロックを使ってコードのサンプルを共有し、構文ハイライトを有効にします。

## フェンス付きコードブロック

コードブロックの前後にトリプルバックティック <code>\`\`\`</code> を置くことでフェンス付きコードブロックを作成できます。生のフォーマットを読みやすくするために、コードブロックの前後に空行を入れることを推奨します。

````text
```
function test() {
  console.log("notice the blank line before this function?");
}
```
````

![トリプルバックティックを使ってコードブロックを作成しているGitHub Markdownのレンダリング画面のスクリーンショット。ブロックは "function test() {" で始まっています。](https://docs.github.com/assets/images/help/writing/fenced-code-block-rendered.png)

> \[!TIP]
> リスト内でフォーマットを保持したい場合は、フェンスなしコードブロックの行を8スペース分インデントしてください。

フェンス付きコードブロック内でトリプルバックティックを表示したい場合は、クアドラプルバックティックで囲みます。

`````text
````
```
Look! You can see my backticks.
```
````
`````

![クアドラプルバックティックで囲んだ中にトリプルバックティックを書いた場合、レンダリングされた内容にトリプルバックティックが表示されるMarkdownのスクリーンショット。](https://docs.github.com/assets/images/help/writing/fenced-code-show-backticks-rendered.png)

コードスニペットや表を頻繁に編集する場合は、GitHubのすべてのコメントフィールドで等幅フォントを有効にすると便利です。詳細は[GitHubでの執筆とフォーマットについて](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/about-writing-and-formatting-on-github#enabling-fixed-width-fonts-in-the-editor)をご覧ください。

## 構文ハイライト

<!-- この機能に変更を加える場合、/get-started/learning-about-github/github-language-support に記載されている言語に影響がないか確認してください。影響がある場合は言語サポートの記事も更新してください。 -->

フェンス付きコードブロックにオプションの言語識別子を追加すると構文ハイライトが有効になります。

構文ハイライトはソースコードの色やスタイルを変えて読みやすくします。

例えば、Rubyコードを構文ハイライトするには：

````text
```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```
````

このように構文ハイライト付きでコードブロックが表示されます：

![GitHub上で表示された3行のRubyコードのスクリーンショット。コードの要素が紫、青、赤の色で表示されて読みやすくなっています。](https://docs.github.com/assets/images/help/writing/code-block-syntax-highlighting-rendered.png)

> \[!TIP]
> GitHub Pagesサイトで構文ハイライト付きのフェンス付きコードブロックを作成する場合は、言語識別子を小文字で指定してください。詳細は[GitHub PagesとJekyllについて](https://docs.github.com/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#syntax-highlighting)をご覧ください。

言語検出と構文ハイライト用の[サードパーティ文法](https://github.com/github-linguist/linguist/blob/main/vendor/README.md)の選択には[Linguist](https://github.com/github-linguist/linguist)を使用しています。使用可能なキーワードは[言語のYAMLファイル](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml)で確認できます。

## 図の作成

コードブロックを使ってMarkdown内で図を作成することもできます。GitHubはMermaid、GeoJSON、TopoJSON、ASCII STL構文をサポートしています。詳細は[図の作成](https://docs.github.com/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams)をご覧ください。

## さらに読む

* [GitHub Flavored Markdown仕様](https://github.github.com/gfm/)
* [基本的な執筆とフォーマットの構文](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
