---
name: write-coding-standards-from-file
description: 'Write a coding standards document for a project using the coding styles from the file(s) and/or folder(s) passed as arguments in the prompt.'
---
# ファイルからコーディング標準を書き込む

Use the existing syntax of the file(s) to establish the standards and style guides for the project.複数のファイルまたはフォルダーが渡された場合は、フォルダー内の各ファイルをループして、ファイルのデータを一時メモリまたはファイルに追加し、完了したら一時データを単一のインスタンスとして使用します。 as if it were the file name to base the standards and style guideline on.

## ルールと構成

Below is a set of quasi-configuration `boolean` and `string[]` variables. Conditions for handling `true`, or other values for each variable are under the level two heading `## Variable and Parameter Configuration Conditions`.

Parameters for the prompt have a text definition. There is one required parameter **`${fileName}`**, and several optional parameters **`${folderName}`**, **`${instructions}`**, and any **`[configVariableAsParameter]`**.

### 構成変数

* addStandardsTest = false;
* addToREADME = false;
* addToREADMEInsertions = ["atBegin", "middle", "beforeEnd", "bestFitUsingContext"];
  - デフォルトは **beforeEnd** です。
* createNewFile = true;
* fetchStyleURL = true;
* findInconsistency = true;
* fixInconsistency = true;
* newFileName = ["CONTRIBUTING.md", "STYLE.md", "CODE_OF_CONDUCT.md", "CODING_STANDARDS.md", "DEVELOPING.md", "CONTRIBUTION_GUIDE.md", "GUIDELINES.md", "PROJECT_STANDARDS.md", "BEST_PRACTICES.md", "ハッキング.md"];
  - For each file in `${newFileName}`, if file does not exist, use that file name and `break`, else continue to next file name of `${newFileName}`.
* 出力仕様ToPrompt = false;
* useTemplate = "冗長"; // または「v」
  - 可能な値は `[["v", "verbose"], ["m", "minimal"], ["b", "best fit"], ["custom"]]` です。
  - プロンプト ファイルの下部、レベル 2 の見出し `## Coding Standards Templates` の下にある 2 つのサンプル テンプレートの 1 つを選択するか、より適切な別の構成を使用します。
  - **カスタム**の場合は、リクエストごとに適用します。

### プロンプトパラメータとしての構成変数

変数名がそのままプロンプトに渡される場合、または類似しているが明確に関連するテキスト値としてプロンプトに渡される場合は、デフォルトの変数値をプロンプトに渡される値でオーバーライドします。

### プロンプトパラメータ* **fileName** = インデント、変数の命名、コメント、条件付きプロシージャ、関数プロシージャ、およびファイルのコーディング言語のその他の構文関連データの観点から分析されるファイルの名前。
*folderName = 複数のファイルからデータを 1 つの集約データセットに抽出するために使用されるフォルダーの名前。インデント、変数の命名、コメント、条件付きプロシージャ、関数プロシージャ、およびファイルのコーディング言語のその他の構文関連データの観点から分析されます。
* 指示 = 特殊なケースに対して提供される追加の指示、ルール、および手順。
* [configVariableAsParameter] = 渡された場合、構成変数のデフォルト状態がオーバーライドされます。例:
  - useTemplate = 渡された場合、構成 `${useTemplate}` のデフォルトがオーバーライドされます。値は `[["v", "verbose"], ["m", "minimal"], ["b", "best fit"]]` です。

#### Required and Optional Parameters

* **fileName** - required
* folderName - *optional*
* instructions - *optional*
* [configVariableAsParameter] - *optional*

## 変数とパラメータの設定条件

### `${fileName}.length > 1 || ${folderName} != undefined`

* If true, toggle `${fixInconsistencies}` to false.

### `${addToREADME} == true`

* プロンプトに出力したり、新しいファイルを作成したりする代わりに、コーディング標準を `README.md` に挿入します。
* true の場合、`${createNewFile}` と `${outputSpecToPrompt}` の両方を false に切り替えます。

### `${addToREADMEInsertions} == "atBegin"`

* `${addToREADME}` が true の場合、コーディング標準データを `README.md` ファイルの**先頭**、タイトルの後に挿入します。

### `${addToREADMEInsertions} == "middle"`

* `${addToREADME}` が true の場合、コーディング標準データを `README.md` ファイルの**中央**に挿入し、標準タイトルの見出しを `README.md` 構成のタイトルと一致するように変更します。

### `${addToREADMEInsertions} == "beforeEnd"`

* `${addToREADME}` が true の場合、`README.md` ファイルの **末尾** にコーディング標準データを挿入し、最後の文字の後に新しい行を挿入してから、データを新しい行に挿入します。

### `${addToREADMEInsertions} == "bestFitUsingContext"`

* `${addToREADME}` が true の場合、`README.md` の構成とデータの流れのコンテキストに関して、`README.md` ファイルの **最も適合する行**にコーディング標準データを挿入します。

### `${addStandardsTest} == true`

* コーディング標準ファイルが完成したら、テスト ファイルを作成して、渡されたファイルがコーディング標準に準拠していることを確認します。

### `${createNewFile} == true`

* `${newFileName}` の値、または可能な値の 1 つを使用して新しいファイルを作成します。
* true の場合、`${outputSpecToPrompt}` と `${addToREADME}` の両方を false に切り替えます。### `${fetchStyleURL} == true`

* さらに、レベル 3 の見出し `### Fetch Links` の下にネストされたリンクから取得したデータを、新しいファイル、プロンプト、または `README.md` の標準、仕様、およびスタイル データを作成するためのコンテキストとして使用します。
* `### Fetch Links` の関連項目ごとに、`#fetch ${item}` を実行します。

### `${findInconsistencies} == true`

* インデント、改行、コメント、条件および関数のネスト、引用符ラッパー (文字列の `'` または `"` など) に関連する構文を評価し、分類します。
* カテゴリごとにカウントを作成し、1 つの項目がカウントの大部分と一致しない場合は、一時メモリにコミットします。
* `${fixInconsistencies}` のステータスに応じて、少数のカテゴリを編集して大部分と一致するように修正するか、一時メモリに保存された不一致をプロンプトに出力します。

### `${fixInconsistencies} == true`

* 一時メモリに保存されている不一致を使用して、対応する構文データの大部分と一致するように構文データの少数のカテゴリを編集および修正します。

### `typeof ${newFileName} == "string"`

* `string` として明示的に定義されている場合は、`${newFileName}` の値を使用して新しいファイルを作成します。

### `typeof ${newFileName} != "string"`

* **具体的に `string` として定義されていない**場合、代わりに `object` または配列として定義されている場合は、次のルールを適用して、`${newFileName}` の値を使用して新しいファイルを作成します。
  - `${newFileName}` の各ファイル名について、ファイルが存在しない場合はそのファイル名と `break` を使用し、そうでない場合は次へ進みます。

### `${outputSpecToPrompt} == true`

* ファイルを作成したり README に追加したりする代わりに、コーディング標準をプロンプトに出力します。
* true の場合、`${createNewFile}` と `${addToREADME}` の両方を false に切り替えます。

### `${useTemplate} == "v" || ${useTemplate} == "verbose"`

* コーディング標準用のデータを作成するときは、レベル 3 の見出し `### "v", "verbose"` の下のデータをガイド テンプレートとして使用します。

### `${useTemplate} == "m" || ${useTemplate} == "minimal"`

* コーディング標準用のデータを作成するときは、レベル 3 の見出し `### "m", "minimal"` の下のデータをガイド テンプレートとして使用します。

### `${useTemplate} == "b" || ${useTemplate} == "best"`

* `${fileName}` から抽出されたデータに応じて、レベル 3 の見出し `### "v", "verbose"` または `### "m", "minimal"` のいずれかを使用し、コーディング標準用のデータを構成する際のガイド テンプレートとして最適なものを使用します。

### `${useTemplate} == "custom" || ${useTemplate} == "<ANY_NAME>"`

* コーディング標準用のデータを作成するときに、ガイド テンプレートとして渡されたカスタム プロンプト、指示、テンプレート、またはその他のデータを使用します。

## **場合** `${fetchStyleURL} == true`

プログラミング言語に応じて、以下のリストの各リンクに対して `#fetch (URL)` を実行します (プログラミング言語が `${fileName} == [<Language> Style Guide]` の場合)。

### リンクを取得する- [C スタイル ガイド](https://users.ece.cmu.edu/~eno/coding/CCodingStandard.html)
- [C# スタイル ガイド](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [C++ スタイルガイド](https://isocpp.github.io/CppCoreガイドライン/CppCoreガイドライン)
- [Go スタイルガイド](https://github.com/golang-standards/project-layout)
- [Java スタイルガイド](https://coderanch.com/wiki/718799/Style)
- [AngularJS アプリ スタイル ガイド](https://github.com/mgechev/angularjs-style-guide)
- [jQuery スタイルガイド](https://contribute.jquery.org/style-guide/js/)
- [JavaScript スタイル ガイド](https://www.w3schools.com/js/js_conventions.asp)
- [JSON スタイルガイド](https://google.github.io/styleguide/jsoncstyleguide.xml)
- [Kotlin スタイルガイド](https://kotlinlang.org/docs/coding-conventions.html)
- [Markdown スタイルガイド](https://cirosantilli.com/markdown-style-guide/)
- [Perl スタイルガイド](https://perldoc.perl.org/perlstyle)
- [PHP スタイルガイド](https://phptherightway.com/)
- [Python スタイルガイド](https://peps.python.org/pep-0008/)
- [Rubyスタイルガイド](https://rubystyle.guide/)
- [Rust スタイルガイド](https://github.com/rust-lang/rust/tree/HEAD/src/doc/style-guide/src)
- [Swift スタイルガイド](https://www.swift.org/documentation/api-design-guidelines/)
- [TypeScript スタイル ガイド](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [Visual Basic スタイル ガイド](https://en.wikibooks.org/wiki/Visual_Basic/Coding_Standards)
- [シェル スクリプト スタイル ガイド](https://google.github.io/styleguide/shellguide.html)
- [Git 利用スタイルガイド](https://github.com/agis/git-style-guide)
- [PowerShell スタイル ガイド](https://github.com/PoshCode/PowerShellPracticeAndStyle)
- [CSS](https://cssguidelin.es/)
- [Sass スタイルガイド](https://sass-guidelin.es/)
- [HTML スタイルガイド](https://github.com/marcobiedermann/html-style-guide)
- [Linux カーネル スタイル ガイド](https://www.kernel.org/doc/html/latest/process/coding-style.html)
- [Node.js スタイルガイド](https://github.com/felixge/node-style-guide)
- [SQLスタイルガイド](https://www.sqlstyle.guide/)
- [Angular スタイルガイド](https://angular.dev/style-guide)
- [Vue スタイルガイド](https://vuejs.org/style-guide/rules-strongly-recommended.html)
- [Django スタイル ガイド](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/)
- [SystemVerilog スタイルガイド](https://github.com/lowRISC/style-guides/blob/master/VerilogCodingStyle.md)## コーディング標準テンプレート

### `"m", "minimal"````text
    ```値下げ
    ## 1. はじめに
    * **目的:** コーディング標準が確立されている理由を簡単に説明します (例: コードの品質、保守性、チームのコラボレーションを向上させるため)。
    * **適用範囲:** この仕様が適用される言語、プロジェクト、またはモジュールを定義します。

    ## 2. 命名規則
    * **変数:** `camelCase`
    * **関数/メソッド:** `PascalCase` または `camelCase`。
    * **クラス/構造体:** `PascalCase`。
    * **定数:** `UPPER_SNAKE_CASE`。

    ## 3. 書式設定とスタイル
    * **インデント:** インデント (またはタブ) ごとに 4 つのスペースを使用します。
    * **行の長さ:** 行を最大 80 文字または 120 文字に制限します。
    * **中括弧:** 「K&R」スタイル (同じ行に開始中括弧) または「Allman」スタイル (新しい行に開始中括弧) を使用します。
    * **空白行:** コードの論理ブロックを区切るために使用する空白行の数を指定します。

    ## 4. コメントする
    * **ドキュメント文字列/関数のコメント:** 関数の目的、パラメータ、戻り値を説明します。
    * **インライン コメント:** 複雑なロジックまたは自明ではないロジックを説明します。
    * **ファイル ヘッダー:** 作成者、日付、ファイルの説明など、ファイル ヘッダーに含める情報を指定します。

    ## 5. エラー処理
    * **一般:** エラーを処理してログに記録する方法。
    * **詳細:** どの例外タイプを使用するか、およびエラー メッセージにどのような情報を含めるか。

    ## 6. ベストプラクティスとアンチパターン
    * **一般:** 回避すべき一般的なアンチパターンをリストします (グローバル変数、マジックナンバーなど)。
    * **言語固有:** プロジェクトのプログラミング言語に基づく具体的な推奨事項。

    ## 7. 例
    * ルールの正しい適用を示す小さなコード例を提供します。
    * 誤った実装の短いコード例とその修正方法を提供します。

    ## 8. 貢献と執行
    * 標準がどのように適用されるかを説明します (コードレビューなど)。
    * 標準文書自体に貢献するためのガイドを提供します。```
```### `"v", verbose"````text
    ```値下げ

    # スタイルガイド

    この文書は、このプロジェクトで使用されるスタイルと規則を定義します。
    特に明記されていない限り、すべての投稿はこれらのルールに従う必要があります。

    ## 1. 一般的なコードスタイル

    - 簡潔さよりも明確さを優先します。
    - 関数とメソッドを小さく、焦点を絞ったものにします。
    - ロジックの繰り返しを避けます。共有ヘルパー/ユーティリティを好みます。
    - 未使用の変数、インポート、コード パス、およびファイルを削除します。

    ## 2. 命名規則

    わかりやすい名前を使用してください。よく知られていない限り、略語は避けてください。

    |アイテム |大会 |例 |
    |-----------------|---------------------|----------------------|
    |変数 | `lower_snake_case` | `buffer_size` |
    |機能 | `lower_snake_case()` | `read_file()` |
    |定数 | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
    |型/構造体 | `PascalCase` | `FileHeader` |
    |ファイル名 | `lower_snake_case` | `file_reader.c` |

    ## 3. フォーマット規則

    - インデント: **スペース 4 つ**
    - 行の長さ: **最大 100 文字**
    - エンコーディング: **UTF-8**、BOM なし
    - ファイルは改行で終了します

    ### 中括弧 (C の例。言語に合わせて調整してください)```c
        if (condition) {
            do_something();
        } else {
            do_something_else();
        }
        ```### 間隔

    - キーワードの後にスペース 1 つ: `if(x)` ではなく、`if (x)`
    - トップレベル関数の間に 1 つの空白行

    ## 4. コメントとドキュメント

    - 意図が不明瞭でない限り、「何を」ではなく「なぜ」を説明します。
    - コードの変更に応じてコメントを最新の状態に保ちます。
    - パブリック関数には、目的とパラメータの短い説明を含める必要があります。

    推奨タグ:```text
        TODO: follow-up work
        FIXME: known incorrect behavior
        NOTE: non-obvious design decision
        ```## 5. エラー処理

    - エラー状態を明示的に処理します。
    - サイレント障害を回避します。エラーを返すか、適切にログに記録します。
    - 失敗時に戻る前にリソース (ファイル、メモリ、ハンドル) をクリーンアップします。

    ## 6. 実践のコミットとレビュー

    ### コミット
    - コミットごとに 1 つの論理変更。
    - 明確なコミット メッセージを書き込みます。```text
        Short summary (max ~50 chars)
        Optional longer explanation of context and rationale.
        ```### レビュー
    - プルリクエストは適度に小さくしてください。
    - レビューの議論では敬意を払い、建設的になってください。
    - 要求された変更に対処するか、同意しない場合は説明します。

    ## 7. テスト

    - 新しい機能のテストを作成します。
    - テストは決定的である必要があります (シードなしのランダム性はありません)。
    - 複雑なテストの抽象化よりも、読みやすいテスト ケースを優先します。

    ## 8. このガイドの変更

    スタイルは進化します。
    問題を報告するか、このドキュメントを更新するパッチを送信して、改善を提案してください。```
```
