# 数学表現の記述

GitHubで数学表現を表示するためにMarkdownを使用します。

## 数学表現の記述について

数学表現を明確に伝えるために、GitHubはMarkdown内でLaTeX形式の数式をサポートしています。詳細はWikibooksの[LaTeX/Mathematics](http://en.wikibooks.org/wiki/LaTeX/Mathematics)を参照してください。

GitHubの数式レンダリング機能は、オープンソースのJavaScriptベースの表示エンジンであるMathJaxを使用しています。MathJaxは幅広いLaTeXマクロといくつかの便利なアクセシビリティ拡張をサポートしています。詳細は[MathJaxドキュメント](http://docs.mathjax.org/en/latest/input/tex/index.html#tex-and-latex-support)および[MathJaxアクセシビリティ拡張ドキュメント](https://mathjax.github.io/MathJax-a11y/docs/#reader-guide)を参照してください。

数学表現のレンダリングはGitHubのIssues、Discussions、プルリクエスト、ウィキ、Markdownファイルで利用可能です。

## インライン数式の記述

テキスト内に数式をインラインで挿入するには2つの方法があります。数式をドル記号（`$`）で囲むか、数式の開始を<code>$\`</code>、終了を<code>\`$</code>で囲みます。後者の構文は、記述する数式にMarkdownの構文と重なる文字が含まれる場合に便利です。詳細は[基本的な記述と書式設定の構文](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)を参照してください。

```text
この文は`$`区切りを使ってインラインで数式を表示します: $\sqrt{3x-1}+(1+x)^2$
```

![インラインの数学表現を示すMarkdownのレンダリングスクリーンショット：3x-1の平方根に(1+x)の2乗を加えた式。](https://docs.github.com/assets/images/help/writing/inline-math-markdown-rendering.png)

```text
この文は$\` と \`$区切りを使ってインラインで数式を表示します: $`\sqrt{3x-1}+(1+x)^2`$
```

![バックティック構文を使ったインライン数学表現のMarkdownレンダリングスクリーンショット：3x-1の平方根に(1+x)の2乗を加えた式。](https://docs.github.com/assets/images/help/writing/inline-backtick-math-markdown-rendering.png)

## ブロックとして数式を記述する

数式をブロックとして追加するには、新しい行を開始し、数式を2つのドル記号`$$`で囲みます。

>  [!TIP] .mdファイルで記述する場合は、改行を作成するために特定の書式が必要です。例えば、以下の例のように行末にバックスラッシュを付けます。Markdownの改行についての詳細は[基本的な記述と書式設定の構文](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#line-breaks)を参照してください。

```text
**コーシー・シュワルツの不等式**\
$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$
```

![複雑な数式を示すMarkdownのレンダリングスクリーンショット。太字で「コーシー・シュワルツの不等式」と書かれ、その下に不等式の式が表示されている。](https://docs.github.com/assets/images/help/writing/math-expression-as-a-block-rendering.png)

また、<code>\`\`\`math</code>コードブロック構文を使って数式をブロック表示することもできます。この構文では`$$`区切りは不要です。以下は上記と同じ表示になります：

````text
**コーシー・シュワルツの不等式**

```math
\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
```
````

## 数式内外でのドル記号の記述

数式と同じ行にドル記号を文字として表示するには、区切り記号でない`$`をエスケープして行が正しくレンダリングされるようにする必要があります。

* 数式内では、明示的な`$`の前に`\`を付けます。

  ```text
  この数式は`\$`を使ってドル記号を表示します: $`\sqrt{\$4}`$
  ```

  ![ドル記号の前にバックスラッシュを付けて数式内に記号を表示する例のMarkdownレンダリングスクリーンショット。](https://docs.github.com/assets/images/help/writing/dollar-sign-within-math-expression.png)

* 数式外で同じ行にある場合は、明示的な`$`を囲むためにspanタグを使用します。

  ```text
  100<span>$</span>を半分に分けるには、$100/2$を計算します
  ```

  ![ドル記号をspanタグで囲み、数式の一部ではなくインラインテキストとして表示する例のMarkdownレンダリングスクリーンショット。](https://docs.github.com/assets/images/help/writing/dollar-sign-inline-math-expression.png)

## さらに読むために

* [MathJax公式サイト](http://mathjax.org)
* [GitHubでの記述と書式設定の開始方法](https://docs.github.com/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github)
* [GitHub Flavored Markdown仕様](https://github.github.com/gfm/)
