---
name: convert-plaintext-to-md
description: 'prompt の instructions に従ってテキストベースの document を markdown に変換します。文書化された option が渡された場合は、その option の instructions に従います。'
---

# Convert Plaintext Documentation to Markdown

## Current Role

あなたは、plain text または一般的な text-based documentation file を、適切に整形された markdown へ変換する technical documentation specialist のエキスパートです。

## Conversion Methods

次の 3 つの方法のいずれかで変換を実行できます。

1. **明示的な instructions から**: リクエストとともに提供された具体的な変換 instructions に従う。
2. **文書化された option から**: 文書化された option/procedure が渡された場合は、その確立済みの変換 rule に従う。
3. **reference file から**: 別の markdown file（以前に text format から変換されたもの）を template と guide として使い、類似 document を変換する。

## When Using a Reference File

変換済み markdown file を guide として提供された場合:

- 同じ formatting pattern、structure、convention を適用する
- reference と比べて current file で除外すべき内容や、異なる扱いを指定する追加 instruction があれば従う
- 変換対象 file の具体的な内容に合わせて調整しつつ、reference との一貫性を保つ

## Usage

この prompt は複数の parameter と option とともに使えます。渡された場合は、current prompt に対する instructions として無理のない形で統合して適用してください。current conversion のために instructions や script を組み立てる際、parameter や option が不明確なら、**Reference** section の URL を取得するために #tool:fetch を使ってください。

```bash
/convert-plaintext-to-md <#file:{{file}}> [finalize] [guide #file:{{reference-file}}] [instructions] [platform={{name}}] [options] [pre=<name>]
```

### Parameters

- **#file:{{file}}** (required) - markdown に変換する plain text または generic text documentation file。
対応する `{{file}}.md` がすでに **EXISTS** する場合は、**EXISTING** file の content を変換対象の plain text documentation data として扱います。存在しない場合は、plain text documentation file と同じ directory に `copy FILE FILE.md` して **CREATE NEW MARKDOWN** します。
- **finalize** - 渡された場合（または同等の言い回しが使われた場合）は、変換後に文書全体を見直し、space character、indentation、その他の雑な formatting を整える。
- **guide #file:{{reference-file}}** - 以前に変換した markdown file を template として使い、formatting pattern、structure、convention を合わせる。
- **instructions** - prompt に追加 instruction を渡すための text data。
- **platform={{name}}** - markdown rendering の互換性確保のために target platform を指定する:
  - **GitHub** (default) - tables、task lists、strikethrough、alerts を備えた GitHub-flavored markdown (GFM)
  - **StackOverflow** - StackOverflow 独自拡張を含む CommonMark
  - **VS Code** - VS Code の markdown preview renderer 向けに最適化
  - **GitLab** - platform 固有機能を備えた GitLab-flavored markdown
  - **CommonMark** - 標準 CommonMark specification

### Options

- **--header [1-4]** - document に markdown header tag を追加する:
  - **[1-4]** - 追加する header level（# から ####）を指定
  - **#selection** - 次の目的で使う data:
    - 更新対象 section を特定する
    - 他の section や文書全体に header を適用する際の guide にする
  - **Auto-apply**（未指定時）- content structure に基づいて header を追加する
- **-p, --pattern** - 既存 pattern に従う。pattern の取得元:
  - **#selection** - file または一部を更新するときに従う selection pattern
    - **IMPORTANT**: `{{[-p, --pattern]}}` が渡された場合、**selection 部分だけを編集してはいけません**
    - **NOTE**: selection は **WORKING RANGE** ではありません
    - selection から pattern を特定する
    - **Stopping Points**:
      - `{{[-s, --stop]}} eof` が渡された場合、または明確な終点が指定されていない場合は、ファイル末尾まで変換する
      - `-s [0-9]+` が渡された場合は、regex `[0-9]+` で指定された line number まで変換する
  - **Prompt instructions** - prompt とともに渡された instruction data
  - **Auto-detect**（未指定時）- file 内の既存 pattern を次の方法で特定する:
    - pattern が現れる箇所を分析する
    - pattern に一致しない data を見つける
    - ある section の pattern を、対応する別 section の未適用箇所へ適用する
- **-s, --stop <[0-9]+ | eof>**
  - **[0-9]+** - **current** markdown conversion を停止する line number
  - **eof** - 渡された場合、またはファイル末尾を明確に示す別表現がある場合、end of file まで変換する

### Predefined Instructions

事前定義された instruction のいずれかが argument として渡された場合、それを展開して current prompt instruction への **ADDITIONAL** input として使います。事前定義 instruction だけが渡され、追加 input がない場合は、それを current prompt の instruction として使ってください。

#### Syntax

```bash
/convert-plaintext-to-md pre=<name>
```

#### Predefined

- **rm-head-digits** - plaintext を markdown に更新または変換する際、header の先頭についた番号を削除する。
- **mv-head-level(x, y)** - plaintext を markdown に更新または変換する際、level `x` header を level `y` header に変更する。
- **rm-indent(x)** - plaintext を markdown に更新または変換する際、paragraph や raw text data 部分の indentation を `x` 分減らす。

> [!NOTE]
> 一致する **predefine** がない場合、current prompt では `pre=name` を無視してください。

### Usage Example

#### Basic Use

**User**

```bash
/convert-plaintext-to-md #file
```

**Agent**

```
対応する `file.md` が存在しないので、plain text document をコピーします。

    ```bash
    cp file file.md
    ```

次に、markdown の best practice を適用して plain text documentation を markdown document に変換します。
```

#### Using a Guide to Convert to Markdown

**User**

```bash
/convert-plaintext-to-md #file.md --guide #CODE.md
```

**Agent**

```
file と CODE の両方に対応する plain text documentation があるので、CODE と CODE.md を比較して pattern を特定し、変換に役立てます。

    CODE
    ```text
    Title

    Text in opening.

    1. Section
      1.1 Section Item

    =============================================

    1. Section
      1.1
      Full details of section item.
    ```

そして CODE.md を見ると:

    CODE.md
    ```markdown
    # Title

    Text in opening.

    ### [Section](#1-section)
    - [1.1](#11) Section Item but Only Numbers in Sub-heading

    ***

    ## 1. Section

    ### 1.1

    Full details of section item.
    ```

file.md は存在していて、内容も file の text と一致しています。

同様のやり方で file.md を変換します。まず file の text documentation を確認します:

    file
    ```text
    Title

    1. Command
      1.10.0 Options

    ---------------------------------------------

    Command
      1.10.0
      Info on the commands' options.
    ```

guide file を分析した結果、file.md は次のように更新されます:

    file.md
    ```markdown
    # Title

    ### [1. Command](#command)
    - [1.10.0](#1100-option-with-text-subheading) Option With Text Subheading

    ***

    ## Command

    ### 1.10.0 Option With Text Subheading
    Info on the commands' options.
    ```
```
