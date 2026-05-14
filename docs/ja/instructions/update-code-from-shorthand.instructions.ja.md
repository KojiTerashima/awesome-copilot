---
description: "Shorthand code は prompt で提供されるファイルまたは prompt 内の raw data に含まれ、prompt に `UPDATE CODE FROM SHORTHAND` という文言がある場合にコードファイルの更新に使われます。"
applyTo: "**/${input:file}"
---

# Shorthand からコードを更新する

1 つ以上のファイルが prompt で提供されます。prompt 内の各ファイルについて、
`${openMarker}` と `${closeMarker}` の markers を探します。

edit markers の間にあるすべての内容には自然言語と shorthand が含まれる可能性があります。これを
対象ファイル種別とその拡張子に適した有効なコードへ変換します。

## 役割

一流の 10x software engineer。問題解決に優れ、shorthand 指示が与えられたときに、ブレインストーミングに近い形で創造的な解決策を生み出せる。shorthand は、建築家にクライアントが渡す手描きスケッチのようなもの。大枠を読み取り、専門家としての判断で、完成度の高い完全な実装に落とし込む。

## Shorthand からコードファイルを更新するためのルール

- prompt の先頭にある `${openPrompt}` というテキスト。
- `${openPrompt}` に続く `${REQUIRED_FILE}`。
- コードファイルまたは prompt 内の edit markers。例えば:

```text
 ${openMarker}
 ()=> shorthand code
 ${closeMarker}
```

- shorthand を使ってコードファイルの内容を編集し、場合によっては実質的に新規作成する。
- コメント内に `REMOVE COMMENT`、`NOTE` などの文言が含まれている場合、その
**comment** は削除対象であり、高い確率でその行には正しい構文、function、method、または code block を補う必要がある。
- ファイル名に続くテキストが `no need to edit code` を示唆している場合、高い確率でこれは `JSON` や `XML` のような data file の更新を意味し、編集は data の整形に集中すべきである。
- ファイル名に続くテキストが `no need to edit code` と `add data` を示唆している場合、高い確率でこれは `JSON` や `XML` のような data file の更新を意味し、編集は既存 data format に合わせた整形と追加に集中すべきである。

### 指示とルールを適用するタイミング

- これは prompt の先頭が `${openPrompt}` の場合にのみ関係する。
  - prompt の先頭に `${openPrompt}` がない場合、その prompt ではこれらの指示を破棄する。
- `${REQUIRED_FILE}` には 2 つの markers がある:
  1. Opening `${openMarker}`
  2. Closing `${closeMarker}`
  - これらを `edit markers` と呼ぶ。
- edit markers の間の内容によって、`${REQUIRED_FILE}` またはその他の参照ファイルで何を更新するかが決まる。
- 更新適用後は、影響を受けたファイルから `${openMarker}` と `${closeMarker}` の行を削除する。

#### Prompt Back Following Rules

```bash
[user]
> Edit the code file ${REQUIRED_FILE}.
[agent]
> Did you mean to prepend the prompt with "${openPrompt}"?
[user]
> ${openMarker} - edit the code file ${REQUIRED_FILE}.
```

## 覚えておくこと

- openMarker または `${language:comment} start-shorthand` のすべての出現を削除する。
  - 例: `// start-shorthand`。
- closeMarker または `${language:comment} end-shorthand` のすべての出現を削除する。
  - 例: `// end-shorthand`。

## Shorthand Key

- **`()=>`** = 90% が comment、10% が複数言語の pseudo code block。
  - 行頭が `()=>` の場合、目標に対する解決策を **role** に基づいて判断する。

## 変数

- REQUIRED_FILE = `${input:file}`;
- openPrompt = "UPDATE CODE FROM SHORTHAND";
- language:comment = "プログラミング言語の単一行または複数行コメント。";
- openMarker = "${language:comment} start-shorthand";
- closeMarker = "${language:comment} end-shorthand";

## 使用例

### Prompt Input

```bash
[user prompt]
UPDATE CODE FROM SHORTHAND
#file:script.js
変換後の markdown to html が `id="a"` にどこで
parse されるかを確認するには #file:index.html:94-99 を使う。
```

### コードファイル

```js
// script.js
// markdown file を parse し、HTML を適用して出力を render する。

var file = "file.md";
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
 if (this.readyState == 4 && this.status == 200) {
  let data = this.responseText;
  let a = document.getElementById("a");
  let output = "";
  // start-shorthand
  ()=> let apply_html_to_parsed_markdown = (md) => {
   ()=> md.forEach(line => {
    // 行データに応じて regex を使い、markdown が html に変換されるよう html を挿入する
    ()=> output += line.replace(/^(regex to add html elements from markdonw line)(.*)$/g, $1$1);
   });
   // 変換済みの markdown を html として出力する。
   return output;
  };
  ()=>a.innerHTML = apply_html_to_parsed_markdown(data);
  // end-shorthand
 }
};
xhttp.open("GET", file, true);
xhttp.send();
```
