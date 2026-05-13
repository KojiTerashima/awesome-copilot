# 基本的な文章と書式設定の構文

簡単な構文を使って、GitHub上の文章やコードに高度な書式設定を適用できます。

## 見出し

見出しを作成するには、見出しテキストの前に <kbd>#</kbd> を1つから6つ追加します。使用する <kbd>#</kbd> の数によって、見出しの階層レベルと文字サイズが決まります。

```markdown
# 第一レベルの見出し
## 第二レベルの見出し
### 第三レベルの見出し
```

![サンプルの h1、h2、h3 見出しが、階層レベルを示すために文字サイズと視覚的な重みを下げながら表示されている GitHub Markdown のレンダリング例のスクリーンショット。](https://docs.github.com/assets/images/help/writing/headings-rendered.png)

2つ以上の見出しを使うと、GitHub は自動的に目次を生成します。これは、ファイルヘッダー内の「Outline」メニューアイコン <svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-list-unordered" aria-label="Table of Contents" role="img"><path d="M5.75 2.5h8.5a.75.75 0 0 1 0 1.5h-8.5a.75.75 0 0 1 0-1.5Zm0 5h8.5a.75.75 0 0 1 0 1.5h-8.5a.75.75 0 0 1 0-1.5Zm0 5h8.5a.75.75 0 0 1 0 1.5h-8.5a.75.75 0 0 1 0-1.5ZM2 14a1 1 0 1 1 0-2 1 1 0 0 1 0 2Zm1-6a1 1 0 0 1-2 0 1 1 0 0 1 2 0ZM2 4a1 1 0 1 1 0-2 1 1 0 0 1 0 2Z"></path></svg> をクリックすると利用できます。各見出しタイトルが目次に一覧表示され、タイトルをクリックすると該当セクションへ移動できます。

![README ファイルで目次のドロップダウンメニューが開かれているスクリーンショット。目次アイコンが濃いオレンジ色で囲まれています。](https://docs.github.com/assets/images/help/repository/headings-toc.png)

## テキストの装飾

コメント欄や `.md` ファイルでは、太字、斜体、取り消し線、下付き文字、上付き文字で強調を表現できます。

| スタイル | 構文 | キーボードショートカット | 例 | 出力 | |
| ---------------------- | ------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------- | -------------------------------------- | ------------------------------------------------- |
| 太字 | `** **` または `__ __` | <kbd>Command</kbd>+<kbd>B</kbd> (Mac) または <kbd>Ctrl</kbd>+<kbd>B</kbd> (Windows/Linux) | `**これは太字のテキストです**` | **これは太字のテキストです** | |
| 斜体 | `* *` または `_ _` | <kbd>Command</kbd>+<kbd>I</kbd> (Mac) または <kbd>Ctrl</kbd>+<kbd>I</kbd> (Windows/Linux) | `_このテキストは斜体です_` | *このテキストは斜体です* | |
| 取り消し線 | `~~ ~~` または `~ ~` | なし | `~~これは誤ったテキストでした~~` | ~~これは誤ったテキストでした~~ | |
| 太字と入れ子の斜体 | `** **` と `_ _` | なし | `**このテキストは _とても_ 重要です**` | **このテキストは *とても* 重要です** | |
| 全体を太字かつ斜体 | `*** ***` | なし | `***このテキストはすべて重要です***` | ***このテキストはすべて重要です*** | <!-- markdownlint-disable-line emphasis-style --> |
| 下付き文字 | `<sub> </sub>` | なし | `これは<sub>下付き文字</sub>のテキストです` | これは <sub>下付き文字</sub> のテキストです | |
| 上付き文字 | `<sup> </sup>` | なし | `これは<sup>上付き文字</sup>のテキストです` | これは <sup>上付き文字</sup> のテキストです | |
| 下線 | `<ins> </ins>` | なし | `これは<ins>下線付き</ins>のテキストです` | これは <ins>下線付き</ins> のテキストです | |

## テキストの引用

<kbd>></kbd> を使うとテキストを引用できます。

```markdown
引用ではないテキスト

> 引用であるテキスト
```

引用されたテキストは左側に縦線付きでインデントされ、灰色の文字で表示されます。

![通常のテキストと引用テキストの違いを示す GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/quoted-text-rendered.png)

> \[!NOTE]
> 会話を表示しているときは、テキストを選択してから <kbd>R</kbd> を押すと、コメント内で自動的に引用できます。コメント全体を引用したい場合は、<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-kebab-horizontal" aria-label="The horizontal kebab icon" role="img"><path d="M8 9a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3ZM1.5 9a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3Zm13 0a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3Z"></path></svg> をクリックしてから **Quote reply** を選びます。キーボードショートカットの詳細は、[Keyboard shortcuts](https://docs.github.com/en/get-started/accessibility/keyboard-shortcuts) を参照してください。

## コードの引用

文中のコードやコマンドは、単一のバッククォートで囲んで示せます。バッククォート内の文字は書式設定されません。また、<kbd>Command</kbd>+<kbd>E</kbd> (Mac) または <kbd>Ctrl</kbd>+<kbd>E</kbd> (Windows/Linux) のキーボードショートカットを押すと、Markdown の1行内コード用のバッククォートを挿入できます。

```markdown
`git status` を使うと、まだコミットしていない新規または変更済みのファイルを一覧表示できます。
```

![バッククォートで囲まれた文字が、固定幅フォントかつ薄いグレーでハイライト表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/inline-code-rendered.png)

コードやテキストを独立したブロックとして整形するには、3つのバッククォートを使います。

````markdown
基本的な Git コマンドには次のようなものがあります。
```
git status
git add
git commit
```
````

![シンタックスハイライトのないシンプルなコードブロックが表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/code-block-rendered.png)

詳細は、[Creating and highlighting code blocks](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks) を参照してください。

コードスニペットや表を頻繁に編集する場合は、GitHub のすべてのコメント欄で固定幅フォントを有効にすると便利です。詳細は、[About writing and formatting on GitHub](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/about-writing-and-formatting-on-github#enabling-fixed-width-fonts-in-the-editor) を参照してください。

## サポートされているカラーモデル

issue、pull request、discussion では、文中で色をバッククォートで囲んで示せます。サポートされているカラーモデルをバッククォートで囲むと、色のプレビューが表示されます。

```markdown
背景色はライトモードでは `#ffffff`、ダークモードでは `#000000` です。
```

![バッククォート内の HEX 値により、ここでは白と黒の小さな色見本が表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/supported-color-models-rendered.png)

現在サポートされているカラーモデルは次のとおりです。

| 色 | 構文 | 例 | 出力 |
| ----- | --------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| HEX | <code>\`#RRGGBB\`</code> | <code>\`#0969DA\`</code> | ![HEX 値 #0969DA が青い円として表示される GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/supported-color-models-hex-rendered.png) |
| RGB | <code>\`rgb(R,G,B)\`</code> | <code>\`rgb(9, 105, 218)\`</code> | ![RGB 値 9, 105, 218 が青い円として表示される GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/supported-color-models-rgb-rendered.png) |
| HSL | <code>\`hsl(H,S,L)\`</code> | <code>\`hsl(212, 92%, 45%)\`</code> | ![HSL 値 212, 92%, 45% が青い円として表示される GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/supported-color-models-hsl-rendered.png) |

> \[!NOTE]
>
> * サポートされているカラーモデルでは、バッククォート内に先頭や末尾の空白を含めることはできません。
> * 色の可視化は、issue、pull request、discussion でのみサポートされています。

## リンク

インラインリンクを作成するには、リンクテキストを角括弧 `[ ]` で囲み、その後に URL を丸括弧 `( )` で囲みます。<kbd>Command</kbd>+<kbd>K</kbd> のキーボードショートカットでもリンクを作成できます。テキストを選択した状態なら、クリップボードから URL を貼り付けるだけで自動的に選択テキストをリンク化できます。

テキストを選択して <kbd>Command</kbd>+<kbd>V</kbd> を使えば、Markdown ハイパーリンクを作成することもできます。テキスト自体をリンクで置き換えたい場合は、<kbd>Command</kbd>+<kbd>Shift</kbd>+<kbd>V</kbd> を使います。

`このサイトは [GitHub Pages](https://pages.github.com/) を使って構築されました。`

![角括弧内のテキスト「GitHub Pages」が青いハイパーリンクとして表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/link-rendered.png)

> \[!NOTE]
> GitHub は、有効な URL がコメント内に書かれると自動的にリンクを作成します。詳細は、[Autolinked references and URLs](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/autolinked-references-and-urls) を参照してください。

## セクションリンク

見出しがある任意のセクションへ直接リンクできます。レンダリングされたファイルでは、セクション見出しにマウスを重ねると <svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-link" aria-label="the link" role="img"><path d="m7.775 3.275 1.25-1.25a3.5 3.5 0 1 1 4.95 4.95l-2.5 2.5a3.5 3.5 0 0 1-4.95 0 .751.751 0 0 1 .018-1.042.751.751 0 0 1 1.042-.018 1.998 1.998 0 0 0 2.83 0l2.5-2.5a2.002 2.002 0 0 0-2.83-2.83l-1.25 1.25a.751.751 0 0 1-1.042-.018.751.751 0 0 1-.018-1.042Zm-4.69 9.64a1.998 1.998 0 0 0 2.83 0l1.25-1.25a.751.751 0 0 1 1.042.018.751.751 0 0 1 .018 1.042l-1.25 1.25a3.5 3.5 0 1 1-4.95-4.95l2.5-2.5a3.5 3.5 0 0 1 4.95 0 .751.751 0 0 1-.018 1.042.751.751 0 0 1-1.042.018 1.998 1.998 0 0 0-2.83 0l-2.5 2.5a1.998 1.998 0 0 0 0 2.83Z"></path></svg> アイコンが表示されます。そのアイコンをクリックすると、ブラウザーにアンカーが表示されます。

![リポジトリの README で、セクション見出しの左側にあるリンクアイコンが濃いオレンジ色で囲まれているスクリーンショット。](https://docs.github.com/assets/images/help/repository/readme-links.png)

編集中のファイルで見出しのアンカーを確認する必要がある場合は、次の基本ルールを使えます。

* 英字は小文字に変換されます。
* 空白はハイフン (`-`) に置き換えられます。それ以外の空白文字や句読点は削除されます。
* 行頭と行末の空白は削除されます。
* マークアップの書式は取り除かれ、内容だけが残ります（たとえば `_italics_` は `italics` になります）。
* 自動生成されたアンカーが同じファイル内の以前のアンカーと重複する場合は、ハイフンと自動増分の整数を付けて一意な識別子が生成されます。

URI フラグメント要件の詳細は、[RFC 3986: Uniform Resource Identifier (URI): Generic Syntax, Section 3.5](https://www.rfc-editor.org/rfc/rfc3986#section-3.5) を参照してください。

以下のコードブロックは、レンダリングされたコンテンツで見出しからアンカーを生成する際の基本ルールを示しています。

```markdown
# Example headings

## Sample Section

## This'll be a _Helpful_ Section About the Greek Letter Θ!
A heading containing characters not allowed in fragments, UTF-8 characters, two consecutive spaces between the first and second words, and formatting.

## This heading is not unique in the file

TEXT 1

## This heading is not unique in the file

TEXT 2

# Links to the example headings above

Link to the sample section: [Link Text](#sample-section).

Link to the helpful section: [Link Text](#thisll-be-a-helpful-section-about-the-greek-letter-Θ).

Link to the first non-unique section: [Link Text](#this-heading-is-not-unique-in-the-file).

Link to the second non-unique section: [Link Text](#this-heading-is-not-unique-in-the-file-1).
```

> \[!NOTE]
> 見出しを編集したり、同一アンカーを持つ見出しの順序を変更したりすると、アンカーも変わるため、それらの見出しへのリンクも更新する必要があります。

## 相対リンク

レンダリングされたファイルでは、相対リンクや画像パスを定義して、読者がリポジトリ内の他のファイルへ移動できるようにできます。

相対リンクとは、現在のファイルからの相対位置で指定するリンクです。たとえば、リポジトリのルートに README ファイルがあり、*docs/CONTRIBUTING.md* に別のファイルがある場合、README から *CONTRIBUTING.md* への相対リンクは次のようになります。

```text
[このプロジェクトのコントリビューションガイドライン](docs/CONTRIBUTING.md)
```

GitHub は、現在どのブランチにいるかに応じて相対リンクや画像パスを自動的に変換するため、リンクやパスは常に有効です。リンクのパスは現在のファイルからの相対パスになります。`/` で始まるリンクはリポジトリルートからの相対パスです。`./` や `../` など、すべての相対リンク演算子を利用できます。

リンクテキストは1行に収める必要があります。以下の例は機能しません。

```markdown
[Contribution
guidelines for this project](docs/CONTRIBUTING.md)
```

相対リンクは、リポジトリをクローンしたユーザーにとって扱いやすい方法です。絶対リンクはクローン環境では動かないことがあるため、リポジトリ内の他ファイルを参照するときは相対リンクの使用が推奨されます。

## カスタムアンカー

標準の HTML アンカータグ (`<a name="unique-anchor-name"></a>`) を使うと、ドキュメント内の任意の場所にナビゲーション用アンカーポイントを作成できます。曖昧な参照を避けるため、`name` 属性には接頭辞を付けるなど、一意な命名規則を使ってください。

> \[!NOTE]
> カスタムアンカーはドキュメントのアウトラインや目次には含まれません。

作成したカスタムアンカーへは、その `name` 属性の値を使ってリンクできます。構文は、見出しから自動生成されたアンカーにリンクする場合とまったく同じです。

例:

```markdown
# セクション見出し

このセクションの本文です。

<a name="my-custom-anchor-point"></a>
見出しはないものの、直接リンクしたいテキストです。

(… さらに内容 …)

[そのカスタムアンカーへのリンク](#my-custom-anchor-point)
```

> \[!TIP]
> カスタムアンカーは、自動生成される見出しリンクの命名や連番には考慮されません。

## 改行

リポジトリの issue、pull request、discussion に書き込む場合、GitHub は自動的に改行をレンダリングします。

```markdown
この例は
2行にまたがって表示されます
```

ただし `.md` ファイルでは、上の例は改行されず1行として表示されます。.md ファイルで改行を作るには、次のいずれかを含める必要があります。

* 1行目の末尾にスペースを2つ入れる。
  <pre>
  この例は  
  2行にまたがって表示されます
  </pre>

* 1行目の末尾にバックスラッシュを入れる。

  ```markdown
  この例は\
  2行にまたがって表示されます
  ```

* 1行目の末尾に HTML の単一改行タグを入れる。

  ```markdown
  この例は<br/>
  2行にまたがって表示されます
  ```

2行の間に空行を入れると、.md ファイルでも issue、pull request、discussion の Markdown でも、2行は空行を挟んで表示されます。

```markdown
この例は

空行を挟んで2行が表示されます
```

## 画像

画像を表示するには、<kbd>!</kbd> を追加し、代替テキストを `[ ]` で囲み、その後に画像へのリンクを丸括弧 `()` で囲みます。

`![GitHub issue のコメントに、Octocat が笑いながら触手を上げている画像が Markdown で追加されている様子のスクリーンショット。](https://myoctocat.com/assets/images/base-octocat.svg)`

![GitHub issue のコメントに、Markdown で追加された Octocat の画像が表示されているスクリーンショット。](https://docs.github.com/assets/images/help/writing/image-rendered.png)

GitHub では、issue、pull request、discussion、コメント、`.md` ファイルに画像を埋め込めます。リポジトリ内の画像を表示したり、オンライン画像へのリンクを追加したり、画像をアップロードしたりできます。詳細は、[Uploading assets](#uploading-assets) を参照してください。

> \[!NOTE]
> リポジトリ内の画像を表示したい場合は、絶対リンクではなく相対リンクを使ってください。

相対リンクを使って画像を表示する例をいくつか示します。

| コンテキスト | 相対リンク |
| ----------------------------------------------------------- | ---------------------------------------------------------------------- |
| 同じブランチ上の `.md` ファイル内 | `/assets/images/electrocat.png` |
| 別のブランチ上の `.md` ファイル内 | `/../main/assets/images/electrocat.png` |
| リポジトリの issue、pull request、コメント内 | `../blob/main/assets/images/electrocat.png?raw=true` |
| 別リポジトリ内の `.md` ファイル | `/../../../../github/docs/blob/main/assets/images/electrocat.png` |
| 別リポジトリの issue、pull request、コメント内 | `../../../github/docs/blob/main/assets/images/electrocat.png?raw=true` |

> \[!NOTE]
> 上の表の最後の2つの相対リンクは、閲覧者がそれらの画像を含むプライベートリポジトリに対して少なくとも読み取り権限を持っている場合にのみ機能します。

詳細は、[Relative Links](#relative-links) を参照してください。

### picture 要素

`<picture>` HTML 要素もサポートされています。

## リスト

順序なしリストは、1行または複数行のテキストの前に <kbd>-</kbd>、<kbd>\*</kbd>、または <kbd>+</kbd> を付けることで作成できます。

```markdown
- George Washington
* John Adams
+ Thomas Jefferson
```

![アメリカ合衆国の最初の3人の大統領名が箇条書きで表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/unordered-list-rendered.png)

順序付きリストにするには、各行の前に数字を付けます。

```markdown
1. James Madison
2. James Monroe
3. John Quincy Adams
```

![4代目、5代目、6代目のアメリカ大統領名が番号付きリストで表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/ordered-list-rendered.png)

### 入れ子のリスト

入れ子のリストは、ある項目の下に1つ以上のリスト項目をインデントして作成できます。

GitHub の Web エディターや、[Visual Studio Code](https://code.visualstudio.com/) のような等幅フォントを使うテキストエディターでは、リストを視覚的にそろえて入れ子のリストを作成できます。入れ子のリスト項目の前にスペースを入れ、リストマーカー文字（<kbd>-</kbd> または <kbd>\*</kbd>）が、ひとつ上の項目の本文の最初の文字の真下に来るようにします。

```markdown
1. First list item
   - First nested list item
     - Second nested list item
```

> \[!NOTE]
> Web ベースのエディターでは、まず対象の行を選択し、その後 <kbd>Tab</kbd> または <kbd>Shift</kbd>+<kbd>Tab</kbd> を使うことで、1行以上のテキストをインデントまたは逆インデントできます。

![Visual Studio Code 上の Markdown で、番号付き行と箇条書きの入れ子にインデントが付いているスクリーンショット。](https://docs.github.com/assets/images/help/writing/nested-list-alignment.png)

![番号付き項目の下に、2段階の入れ子の箇条書きが続いている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/nested-list-example-1.png)

GitHub のコメントエディターは等幅フォントを使わないため、そこで入れ子リストを作る場合は、直前のリスト項目を見て、その項目本文の前に何文字あるかを数えます。そして、その文字数ぶんのスペースを入れ子のリスト項目の前に入力します。

この例では、`100. First list item` の下に入れ子項目を追加するには、`First list item` の前に5文字（`100. `）あるので、最低5つのスペースで入れ子項目をインデントできます。

```markdown
100. First list item
     - First nested list item
```

![100 という番号の付いた項目の下に、1段階入れ子になった箇条書きが表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/nested-list-example-3.png)

同じ方法で複数レベルの入れ子リストを作成できます。たとえば、最初の入れ子項目はその本文 `First nested list item` の前に7文字（`␣␣␣␣␣-␣`）あるため、2段目の入れ子項目は最低でもさらに2文字多い、9つのスペースでインデントする必要があります。

```markdown
100. First list item
     - First nested list item
       - Second nested list item
```

![100 という番号の付いた項目の下に、2段階の入れ子箇条書きが表示されている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/nested-list-example-2.png)

その他の例は、[GitHub Flavored Markdown Spec](https://github.github.com/gfm/#example-265) を参照してください。

## タスクリスト

タスクリストを作成するには、リスト項目の先頭にハイフンとスペース、その後に `[ ]` を付けます。タスクを完了済みにするには `[x]` を使います。

```markdown
- [x] #739
- [ ] https://github.com/octo-org/octo-repo/issues/740
- [ ] すべてのタスクが完了したときに体験をもっと楽しくする :tada:
```

![Markdown のレンダリング結果のスクリーンショット。issue 参照は issue のタイトルとして表示されています。](https://docs.github.com/assets/images/help/writing/task-list-rendered-simple.png)

タスクリスト項目の説明が丸括弧で始まる場合は、<kbd>\\</kbd> でエスケープする必要があります。

`- [ ] \(任意) 後続の issue を開く`

詳細は、[About tasklists](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/about-task-lists) を参照してください。

## 人やチームへのメンション

GitHub では、<kbd>@</kbd> に続けてユーザー名やチーム名を入力すると、人や [team](https://docs.github.com/en/organizations/organizing-members-into-teams) をメンションできます。これにより通知が送られ、会話に注意を向けてもらえます。コメントを編集してユーザー名やチーム名を追加した場合も通知されます。通知の詳細は、[About notifications](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/about-notifications) を参照してください。

> \[!NOTE]
> メンションされた人に通知されるのは、その人がリポジトリへの読み取り権限を持っている場合、またリポジトリが組織所有であればその組織のメンバーである場合に限られます。

`@github/support これらの更新についてどう思いますか？`

![チームメンション「@github/support」が太字かつクリック可能なテキストとして表示される GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/mention-rendered.png)

親チームをメンションすると、その子チームのメンバーにも通知が届くため、複数のグループとのコミュニケーションが容易になります。詳細は、[About organization teams](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) を参照してください。

<kbd>@</kbd> を入力すると、プロジェクト上の人やチームの候補一覧が表示されます。入力に合わせて候補は絞り込まれるので、目的の人やチームが見つかったら、矢印キーで選択して <kbd>Tab</kbd> または <kbd>Enter</kbd> を押して補完できます。チームを指定する場合は @organization/team-name の形式で入力すると、そのチームの全メンバーが会話に購読されます。

自動補完の候補は、リポジトリのコラボレーターや、そのスレッドに参加している他のユーザーに限定されます。

## issue と pull request の参照

<kbd>#</kbd> を入力すると、そのリポジトリ内の issue や pull request の候補一覧が表示されます。issue または pull request の番号やタイトルを入力して絞り込み、ハイライトされた候補で <kbd>Tab</kbd> または <kbd>Enter</kbd> を押して補完します。

詳細は、[Autolinked references and URLs](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/autolinked-references-and-urls) を参照してください。

## 外部リソースの参照

リポジトリにカスタム自動リンク参照が設定されている場合、JIRA issue や Zendesk チケットのような外部リソースへの参照は短縮リンクに変換されます。リポジトリで利用可能な自動リンクを確認したい場合は、管理者権限を持つ人に問い合わせてください。詳細は、[Configuring autolinks to reference external resources](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/configuring-autolinks-to-reference-external-resources) を参照してください。

## アセットのアップロード

画像などのアセットは、ドラッグアンドドロップ、ファイルブラウザーからの選択、または貼り付けでアップロードできます。アセットは、issue、pull request、コメント、そしてリポジトリ内の `.md` ファイルにアップロードできます。

## 絵文字の使用

絵文字を追加するには、コロンの後に絵文字名を続けた `:EMOJICODE:` を入力します。

`@octocat :+1: この PR は良さそうです。マージ準備完了です！ :shipit:`

![+1 と shipit の絵文字コードが、視覚的な絵文字としてレンダリングされている GitHub Markdown のスクリーンショット。](https://docs.github.com/assets/images/help/writing/emoji-rendered.png)

<kbd>:</kbd> を入力すると、候補の絵文字一覧が表示されます。入力するにつれて候補は絞り込まれるので、目的の絵文字が見つかったら、**Tab** または **Enter** を押してハイライトされた候補を補完します。

利用可能な絵文字とコードの一覧は、[the Emoji-Cheat-Sheet](https://github.com/ikatyang/emoji-cheat-sheet/blob/github-actions-auto-update/README.md) を参照してください。

## 段落

段落を作成するには、行と行の間に空行を入れます。

## 脚注

脚注は、次のような角括弧構文で追加できます。

```text
Here is a simple footnote[^1].

A footnote can also have multiple lines[^2].

[^1]: My reference.
[^2]: To add line breaks within a footnote, add 2 spaces to the end of a line.  
This is a second line.
```

脚注は次のようにレンダリングされます。

![Markdown のレンダリング結果で、脚注を示す上付き番号と、注内の任意の改行が表示されているスクリーンショット。](https://docs.github.com/assets/images/help/writing/footnote-rendered.png)

> \[!NOTE]
> 脚注を Markdown のどこに書くかは、脚注がどこに表示されるかに影響しません。参照の直後に脚注を書いても、脚注は Markdown の下部にレンダリングされます。脚注は wiki ではサポートされていません。

## アラート

**Alerts** は、**callouts** または **admonitions** とも呼ばれ、重要な情報を強調するために使える、blockquote 構文ベースの Markdown 拡張です。GitHub では、内容の重要度に応じて異なる色とアイコンで表示されます。

アラートは、ユーザーの成功にとって重要な場合にのみ使い、読者を過負荷にしないよう1記事あたり1つか2つに制限してください。また、アラートを連続して配置するのは避けるべきです。アラートは他の要素の中へ入れ子にできません。

アラートを追加するには、アラート種別を指定する特別な blockquote 行を書き、その後に通常の blockquote としてアラート内容を続けます。利用できるアラートは5種類です。

```markdown
> [!NOTE]
> ユーザーが流し読みしていても知っておくべき有用な情報。

> [!TIP]
> より良く、あるいはより簡単に物事を進めるための助言。

> [!IMPORTANT]
> 目的達成のためにユーザーが知っておく必要のある重要情報。

> [!WARNING]
> 問題を避けるため、すぐに注意すべき緊急情報。

> [!CAUTION]
> 特定の操作に伴うリスクや望ましくない結果について注意を促します。
```

レンダリングされたアラートは次のようになります。

![Note、Tip、Important、Warning、Caution が、それぞれ異なる色のテキストとアイコンで表示されている Markdown アラートのレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/alerts-rendered.png)

## コメントで内容を隠す

HTML コメント内に置いた内容は、レンダリングされた Markdown では表示されません。

```text
<!-- This content will not appear in the rendered Markdown -->
```

## Markdown 書式を無視する

Markdown の文字の前に <kbd>\\</kbd> を置くと、Markdown の書式設定を無視（またはエスケープ）させることができます。

`Let's rename \*our-new-project\* to \*our-old-project\*.`

![バックスラッシュによってアスタリスクが斜体に変換されないようになっている GitHub Markdown のレンダリング結果のスクリーンショット。](https://docs.github.com/assets/images/help/writing/escaped-character-rendered.png)

バックスラッシュの詳細は、Daring Fireball の [Markdown Syntax](https://daringfireball.net/projects/markdown/syntax#backslash) を参照してください。

> \[!NOTE]
> Markdown の書式は、issue や pull request のタイトルでは無視されません。

## Markdown レンダリングの無効化

Markdown ファイルを表示しているとき、ファイル上部の **Code** をクリックすると、Markdown のレンダリングを無効にしてソース表示へ切り替えられます。

![リポジトリ内の Markdown ファイルで、ファイル操作用のオプションが表示されているスクリーンショット。Code ボタンが濃いオレンジ色で囲まれています。](https://docs.github.com/assets/images/help/writing/display-markdown-as-source-global-nav-update.png)

Markdown のレンダリングを無効にすると、行リンクのようなソース表示機能を利用できます。これらはレンダリング表示では利用できません。

## 参考資料

*[GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
*[About writing and formatting on GitHub](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/about-writing-and-formatting-on-github)
*[Working with advanced formatting](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting)
*[Quickstart for writing on GitHub](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/quickstart-for-writing-on-github)
