# 表で情報を整理する

コメント、イシュー、プルリクエスト、ウィキで情報を整理するために表を作成できます。

## 表の作成

パイプ `|` とハイフン `-` を使って表を作成できます。ハイフンは各列のヘッダーを作るために使い、パイプは各列を区切ります。表を正しくレンダリングするには、表の前に空行を入れる必要があります。

```markdown
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```

![2つの等しい幅の列としてレンダリングされたGitHub Markdown表のスクリーンショット。ヘッダーは太字で表示され、交互の内容行は灰色の背景になっています。](https://docs.github.com/assets/images/help/writing/table-basic-rendered.png)

表の両端のパイプは省略可能です。

セルの幅は異なっていてもよく、列内で完全に揃える必要はありません。ヘッダー行の各列には少なくとも3つのハイフンが必要です。

```markdown
| Command | Description |
| --- | --- |
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |
```

![幅の異なる2列のGitHub Markdown表のスクリーンショット。行には「git status」と「git diff」のコマンドとその説明が記載されています。](https://docs.github.com/assets/images/help/writing/table-varied-columns-rendered.png)

コードスニペットや表を頻繁に編集する場合は、GitHubのすべてのコメント欄で等幅フォントを有効にすると便利です。詳細は[GitHubでの執筆とフォーマットについて](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/about-writing-and-formatting-on-github#enabling-fixed-width-fonts-in-the-editor)を参照してください。

## 表内のコンテンツの書式設定

表内でリンク、インラインコードブロック、テキストスタイルなどの[書式設定](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)を使用できます。

```markdown
| Command | Description |
| --- | --- |
| `git status` | List all *new or modified* files |
| `git diff` | Show file differences that **haven't been** staged |
```

![コマンドがコードブロックとして書式設定され、説明に太字と斜体の書式が使われているGitHub Markdown表のスクリーンショット。](https://docs.github.com/assets/images/help/writing/table-inline-formatting-rendered.png)

ヘッダー行のハイフンの左側、右側、または両側にコロン `:` を含めることで、列内のテキストを左寄せ、中央寄せ、右寄せに設定できます。

```markdown
| Left-aligned | Center-aligned | Right-aligned |
| :---         |     :---:      |          ---: |
| git status   | git status     | git status    |
| git diff     | git diff       | git diff      |
```

![セル内のテキストを左寄せ、中央寄せ、右寄せに設定できることを示す、GitHubでレンダリングされた3列のMarkdown表のスクリーンショット。](https://docs.github.com/assets/images/help/writing/table-aligned-text-rendered.png)

セル内にパイプ `|` を含めたい場合は、パイプの前にバックスラッシュ `\` を付けます。

```markdown
| Name     | Character |
| ---      | ---       |
| Backtick | `         |
| Pipe     | \|        |
```

![通常はセルを区切るパイプがバックスラッシュでエスケープされて表示されているGitHubでレンダリングされたMarkdown表のスクリーンショット。](https://docs.github.com/assets/images/help/writing/table-escaped-character-rendered.png)

## さらに読む

* [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
* [基本的な書き方と書式設定の構文](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
