---
description: '公式推奨に従って Dart と Flutter のコードを書くための instruction。'
applyTo: '**/*.dart'
---

# Dart と Flutter

Dart および Flutter チームが推奨するベストプラクティスです。これらの instruction は [Effective Dart](https://dart.dev/effective-dart) と [Architecture Recommendations](https://docs.flutter.dev/app-architecture/recommendations) から採られています。

## Effective Dart

ここ数年で大量の Dart コードを書き、何がうまく機能し、何がそうでないかを多く学んできました。皆さんも一貫性があり、堅牢で、高速なコードを書けるよう、それを共有します。大きなテーマは 2 つあります。

1.  **一貫性を保つ。** フォーマットや casing のような事柄については、どちらがより良いかという議論は主観的であり、解決不可能です。しかし、私たちが分かっているのは、*一貫性* が客観的に有益だということです。

    2 つのコード片が違って見えるなら、それは意味のある何らかの違いがあるからであるべきです。あるコード片が目立って注意を引くなら、それは有益な理由によるべきです。

2.  **簡潔である。** Dart は親しみやすさを目指して設計されたため、C、Java、JavaScript など多くの言語と同じ文や式を受け継いでいます。しかし私たちは、それらの言語が提供するものには改良の余地が大きいと考えたため Dart を作りました。文字列補間から initializing formal まで、多くの機能を追加し、意図をより簡単かつ自然に表現できるようにしました。

    同じことを表す方法が複数あるなら、一般には最も簡潔なものを選ぶべきです。とはいえ、プログラム全体を 1 行に押し込むような `code golf` をしろという意味ではありません。目指すのは *economical* なコードであり、*dense* なコードではありません。

### トピック

ガイドラインは理解しやすいよう、いくつかのトピックに分けています。

*   **Style** – コードの配置や整理のルールを定義します。少なくとも `dart format` が面倒を見てくれない部分についてです。style トピックでは、識別子の書式 `camelCase`、`using_underscores` なども規定します。

*   **Documentation** – コメントの中に何を書くべきかを説明します。doc comment も通常のコード コメントも両方含みます。

*   **Usage** – 振る舞いを実装するために言語機能を最適に使う方法を示します。statement や expression に現れるものはここで扱います。

*   **Design** – 最も柔らかいトピックですが、最も広い範囲を扱います。ライブラリ用に一貫性があり使いやすい API を設計するうえで学んだことをまとめています。type signature や宣言に現れるものはここで扱います。

### トピックの読み方

各トピックは複数のセクションに分かれています。セクションにはガイドラインの一覧があり、各ガイドラインは次のいずれかの語で始まります。

*   **DO** ガイドラインは、常に従うべき実践を説明します。そこから外れる正当な理由があることはほとんどありません。

*   **DON'T** ガイドラインはその逆で、ほとんどの場合よい考えではないものです。他言語ほど数が多くないとよいのですが、それは歴史的なしがらみが少ないからです。

*   **PREFER** ガイドラインは、従う *べき* 実践です。ただし状況によっては別の選択のほうが理にかなうこともあります。その場合は、ガイドラインを無視することの影響を十分理解してください。

*   **AVOID** ガイドラインは "prefer" の対です。通常は行うべきでないものですが、まれに正当な理由がある場合があります。

*   **CONSIDER** ガイドラインは、状況、前例、そして自身の好みによって、従うかどうかが変わる実践です。

一部のガイドラインでは、そのルールが *適用されない* **例外** を説明します。例外として列挙されるものが網羅的とは限らないため、他のケースでは自分の判断も必要です。

これだけ聞くと、靴ひもを正しく結んでいないだけで警察がドアを蹴破ってくるように感じるかもしれません。そこまでではありません。ここにあるガイドラインの多くは常識であり、私たちは皆、分別のある人間です。目標はいつも通り、読みやすく保守しやすいコードです。

### ルール

#### Style

##### 識別子

*   型名には `UpperCamelCase` を使う。
*   extension 名には `UpperCamelCase` を使う。
*   package、directory、source file 名には `lowercase_with_underscores` を使う。
*   import prefix には `lowercase_with_underscores` を使う。
*   その他の識別子には `lowerCamelCase` を使う。
*   定数名には `lowerCamelCase` を使うことを推奨する。
*   3 文字以上の acronym や略語は、単語として大文字化する。
*   未使用の callback parameter には wildcard を使うことを推奨する。
*   private でない識別子に先頭 underscore を使わない。
*   接頭辞の文字を使わない。
*   library 名を明示しない。

##### 順序

*   `dart:` import は他の import より前に置く。
*   `package:` import は相対 import より前に置く。
*   export はすべての import のあと、別セクションにまとめる。
*   セクションはアルファベット順に並べる。

##### Formatting

*   コードは `dart format` で整形する。
*   formatter が扱いやすいようにコードを書き換えることを検討する。
*   1 行 80 文字以下を推奨する。
*   すべての flow control statement で curly brace を使う。

#### Documentation

##### Comments

*   コメントは文として整形する。
*   ドキュメントに block comment を使わない。

##### Doc comments

*   member と type のドキュメントには `///` doc comment を使う。
*   public API には doc comment を書くことを推奨する。
*   library-level の doc comment を書くことを検討する。
*   private API にも doc comment を書くことを検討する。
*   doc comment は 1 文の要約から始める。
*   doc comment の最初の文は、それだけで 1 段落に分ける。
*   周囲の文脈との冗長さは避ける。
*   主目的が副作用である関数やメソッドのコメントは、三人称の動詞で始めることを推奨する。
*   boolean でない変数や property のコメントは、名詞句で始めることを推奨する。
*   boolean の変数や property のコメントは、"Whether" に続く名詞句または動名詞句で始めることを推奨する。
*   値を返すことが主目的の関数やメソッドには、名詞句または命令形でない動詞句を使うことを推奨する。
*   property の getter と setter の両方にドキュメントを書かない。
*   library や type のコメントは名詞句で始めることを推奨する。
*   doc comment に code sample を含めることを検討する。
*   doc comment では、スコープ内識別子への参照に square bracket を使う。
*   parameter、return value、exception の説明には prose を使う。
*   doc comment は metadata annotation の前に置く。

##### Markdown

*   markdown の過剰使用を避ける。
*   整形目的で HTML を使わない。
*   code block には backtick fence を推奨する。

##### Writing

*   簡潔さを推奨する。
*   明白でない略語や acronym は避ける。
*   member の instance を指すときは "the" より "this" を使うことを推奨する。

#### Usage

##### Libraries

*   `part of` directive では string を使う。
*   他 package の `src` directory 内にある library を import しない。
*   import path が `lib` の内外へ入り込まないようにする。
*   相対 import path を推奨する。

##### Null

*   変数を明示的に `null` で初期化しない。
*   明示的な既定値として `null` を使わない。
*   等価比較で `true` や `false` を使わない。
*   初期化済みかどうかを確認する必要があるなら `late` 変数は避ける。
*   nullable type の利用には type promotion や null-check pattern を検討する。

##### Strings

*   string literal の連結には隣接文字列を使う。
*   文字列と値の組み立てには interpolation を推奨する。
*   不要な interpolation の curly brace は避ける。

##### Collections

*   可能なら collection literal を使う。
*   collection が空かどうかの判定に `.length` を使わない。
*   関数 literal を伴う `Iterable.forEach()` の使用は避ける。
*   結果の型を変えたい場合でない限り `List.from()` を使わない。
*   型による collection 絞り込みには `whereType()` を使う。
*   近くの別操作で済むなら `cast()` を使わない。
*   `cast()` の使用は避ける。

##### Functions

*   関数に名前を束縛するには function declaration を使う。
*   tear-off で済む場合は lambda を作らない。

##### Variables

*   ローカル変数での `var` と `final` は一貫したルールに従う。
*   計算できるものを保存しない。

##### Members

*   不要に field を getter/setter で包まない。
*   読み取り専用 property には `final` field を使うことを推奨する。
*   単純な member には `=>` の利用を検討する。
*   named constructor へのリダイレクトや shadowing 回避以外では `this.` を使わない。
*   可能なら field は宣言時に初期化する。

##### Constructors

*   可能な場合は initializing formal を使う。
*   constructor initializer list で済む場合は `late` を使わない。
*   空の constructor body には `{}` ではなく `;` を使う。
*   `new` を使わない。
*   冗長な `const` を使わない。

##### Error handling

*   `on` clause のない catch は避ける。
*   `on` clause のない catch で error を捨てない。
*   プログラム上の error に対してのみ `Error` を実装する object を throw する。
*   `Error` やその実装型を明示的に catch しない。
*   捕捉した exception の再送出には `rethrow` を使う。

##### Asynchrony

*   生の future より async/await を推奨する。
*   有用な効果がない場合は `async` を使わない。
*   stream の変換には higher-order method の利用を検討する。
*   `Completer` を直接使うことは避ける。
*   型引数が `Object` になり得る `FutureOr<T>` を判別するときは、`Future<T>` の判定を行う。

#### Design

##### Names

*   用語は一貫して使う。
*   略語は避ける。
*   最も説明的な名詞は最後に置くことを推奨する。
*   コードが文のように読めるようにすることを検討する。
*   boolean でない property や変数には名詞句を推奨する。
*   boolean の property や変数には命令形でない動詞句を推奨する。
*   named boolean parameter では動詞の省略を検討する。
*   boolean の property や変数には "positive" な名前を推奨する。
*   主目的が副作用の関数やメソッドには命令形の動詞句を推奨する。
*   値を返すことが主目的の関数やメソッドには、名詞句または命令形でない動詞句を推奨する。
*   実行される処理に注意を向けたい関数やメソッドには、命令形の動詞句を検討する。
*   メソッド名を `get` で始めるのは避ける。
*   object の状態を新しい object にコピーするメソッドには `to...()` という命名を推奨する。
*   元の object に裏打ちされた別表現を返すメソッドには `as...()` という命名を推奨する。
*   関数やメソッド名で parameter を説明するのは避ける。
*   type parameter の命名には既存の mnemonic convention に従う。

##### Libraries

*   宣言は private にすることを推奨する。
*   同じ library に複数 class を宣言することを検討する。

##### Classes and mixins

*   単純な関数で済むなら、1 member だけの abstract class を定義しない。
*   static member だけを含む class を定義しない。
*   継承を意図していない class を拡張しない。
*   class modifier を使い、その class が拡張可能かどうかを制御する。
*   interface として使う意図がない class を implements しない。
*   class modifier を使い、その class が interface になれるかどうかを制御する。
*   `mixin class` より、純粋な `mixin` または純粋な `class` の定義を推奨する。

##### Constructors

*   class が対応しているなら、constructor を `const` にすることを検討する。

##### Members

*   field と top-level 変数は `final` にすることを推奨する。
*   概念的に property へアクセスする処理には getter を使う。
*   概念的に property を変更する処理には setter を使う。
*   対応する getter のない setter を定義しない。
*   overloading を装うための runtime type test は避ける。
*   initializer を持たない public `late final` field は避ける。
*   nullable な `Future`、`Stream`、collection type を返すことは避ける。
*   fluent interface のためだけに method から `this` を返すことは避ける。

##### Types

*   initializer を持たない変数には型注釈を付ける。
*   型が明白でない field や top-level 変数には型注釈を付ける。
*   初期化済みローカル変数に冗長な型注釈を付けない。
*   関数宣言では戻り値型を注釈する。
*   関数宣言では parameter 型を注釈する。
*   関数式で推論される parameter 型は注釈しない。
*   initializing formal に型注釈を付けない。
*   推論されない generic invocation には型引数を書く。
*   推論される generic invocation には型引数を書かない。
*   不完全な generic type を書くことは避ける。
*   推論に失敗させるくらいなら `dynamic` を明示する。
*   関数型注釈には signature を推奨する。
*   setter に戻り値型を指定しない。
*   legacy typedef syntax を使わない。
*   typedef より inline function type を推奨する。
*   parameter には function type syntax を推奨する。
*   静的検査を無効化したいのでなければ `dynamic` は避ける。
*   値を生成しない非同期 member の戻り値型には `Future<void>` を使う。
*   戻り値型として `FutureOr<T>` を使うことは避ける。

##### Parameters

*   positional boolean parameter は避ける。
*   利用者が前方 parameter を省略したくなる可能性があるなら optional positional parameter は避ける。
*   特別な "引数なし" 値を受け取る mandatory parameter は避ける。
*   範囲を受け取るには inclusive start と exclusive end parameter を使う。

##### Equality

*   `==` を override するなら `hashCode` も override する。
*   `==` 演算子は等価性の数学的ルールに従わせる。
*   mutable class に custom equality を定義することは避ける。
*   `==` の parameter を nullable にしない。

---

## Flutter アーキテクチャ推奨事項

このページでは、アーキテクチャのベストプラクティス、その重要性、そして Flutter アプリケーションに対してそれを推奨するかどうかを示します。
これらは厳格なルールではなく推奨事項として扱い、
アプリ固有の要件に応じて調整してください。

このページのベストプラクティスには優先度があり、
Flutter チームがどれだけ強く推奨しているかを表します。

* **Strongly recommend:** 新しいアプリケーションを作り始めるなら、常にこの推奨を実装すべきです。既存アプリについても、現在のアプローチと根本的に衝突しない限り、これを実装するよう強くリファクタリングを検討すべきです。
* **Recommend**: この実践は、たいていアプリを改善します。
* **Conditional**: この実践は、特定の状況でアプリを改善します。

### 関心の分離

アプリは UI レイヤーと data レイヤーに分けるべきです。そのうえで、それぞれのレイヤー内でも責務ごとに class へロジックを分離してください。

#### 明確に定義された data レイヤーと UI レイヤーを使う。
**Strongly recommend**

関心の分離は最も重要なアーキテクチャ原則です。
data レイヤーはアプリケーション データをアプリの他部分へ公開し、アプリケーション内のビジネス ロジックの大部分を含みます。
UI レイヤーはアプリケーション データを表示し、ユーザーからのイベントを受け取ります。UI レイヤーには UI ロジック用 class と widget 用 class が分かれて含まれます。

#### data レイヤーでは repository pattern を使う。
**Strongly recommend**

repository pattern は、データ アクセス ロジックをアプリケーションの他部分から切り離すソフトウェア設計パターンです。
これは、アプリケーションのビジネス ロジックと基盤となるデータ保存機構 (database、API、file system など) の間に抽象化層を作ります。
実際には、Repository class と Service class を作ることを意味します。

#### UI レイヤーでは ViewModel と View を使う。 (MVVM)
**Strongly recommend**

関心の分離は最も重要なアーキテクチャ原則です。
この分離によって widget が "dumb" なままでいられるため、コードははるかにエラーを起こしにくくなります。

#### widget 更新の処理には `ChangeNotifiers` と `Listenables` を使う。
**Conditional**

> state-management の方法は多数あり、最終的な判断は個人の好みに帰着します。

`ChangeNotifier` API は Flutter SDK の一部であり、widget が ViewModel の変更を監視する便利な方法です。

#### widget にロジックを置かない。
**Strongly recommend**

ロジックは ViewModel の method にカプセル化すべきです。view が含んでよいロジックは次だけです。
* ViewModel 内の flag または nullable field に基づいて widget を表示・非表示する単純な if statement
* widget 側で計算する必要がある animation logic
* 画面サイズや向きなど、device 情報に基づく layout logic
* 単純な routing logic

#### domain レイヤーを使う。
**Conditional**

> 複雑なロジック要件を持つアプリで使用します。

domain レイヤーが必要なのは、アプリケーションのロジックが非常に複雑で ViewModel を圧迫している場合、
または ViewModel 間でロジックの重複が見られる場合だけです。
非常に大規模なアプリでは use-case が有用ですが、多くのアプリでは不要なオーバーヘッドになります。

### データの扱い

データを丁寧に扱うと、コードは理解しやすく、エラーを起こしにくくなり、
不正または想定外のデータが生成されるのを防げます。

#### unidirectional data flow を使う。
**Strongly recommend**

データ更新は data レイヤーから UI レイヤーへ一方向にのみ流れるべきです。
UI レイヤーでの操作は data レイヤーへ送られ、そこで処理されます。

#### ユーザー操作からのイベント処理には `Commands` を使う。
**Recommend**

Command はアプリ内の rendering error を防ぎ、UI レイヤーがイベントを data レイヤーへ送る方法を標準化します。

#### immutable data model を使う。
**Strongly recommend**

immutable data は、必要な変更が通常 data レイヤーまたは domain レイヤーの適切な場所でのみ起こることを保証するうえで重要です。
immutable object は生成後に変更できないため、変更を反映するには新しい instance を作る必要があります。
この仕組みにより UI レイヤーでの偶発的な更新を防ぎ、明確な unidirectional data flow を支えます。

#### freezed または built_value を使って immutable data model を生成する。
**Recommend**

`freezed` や `built_value` を使うと、data model に有用な機能を自動生成できます。
これらは JSON ser/des、deep equality check、copy method などの一般的な model method を生成できます。
model が多いアプリでは、こうした code generation package が build 時間を大きく増やす場合があります。

#### API model と domain model を分けて作る。
**Conditional**

> 大規模アプリで使用します。

別モデルを使うと冗長さは増えますが、ViewModel や use-case 内の複雑さを防げます。

### アプリ構造

よく整理されたコードは、アプリ自体の健全性と、そのコードを扱うチームの両方に利益をもたらします。

#### dependency injection を使う。
**Strongly recommend**

dependency injection は、アプリがグローバルにアクセス可能な object を持つことを防ぐため、コードがエラーを起こしにくくなります。
dependency injection の扱いには `provider` package の利用を推奨します。

#### navigation には `go_router` を使う。
**Recommend**

go_router は、Flutter アプリの 90% で推奨される navigation 方法です。
go_router では解決できない特定ユースケースもあるため、
その場合は `Flutter Navigator API` を直接使うか、`pub.dev` にある他 package を試してください。

#### class、file、directory には標準化された命名規則を使う。
**Recommend**

class には、その class が表すアーキテクチャ コンポーネントに応じた名前を付けることを推奨します。
たとえば次のような class が考えられます。

* HomeViewModel
* HomeScreen
* UserRepository
* ClientApiService

明確さのため、Flutter SDK の object と混同され得る名前は推奨しません。
たとえば共有 widget は `/widgets` という directory ではなく、`ui/core/` という directory に置くべきです。

#### abstract repository class を使う
**Strongly recommend**

Repository class はアプリ内のすべてのデータに対する source of truth であり、
外部 API との通信を担います。
abstract repository class を作ることで、
"development" や "staging" のような異なるアプリ環境向けに別 implementation を用意できます。

### テスト

よいテスト実践はアプリを柔軟にします。
また、新しいロジックや新しい UI を追加することを簡単かつ低リスクにします。

#### アーキテクチャ コンポーネントは個別にも一緒にもテストする。
**Strongly recommend**

* すべての service、repository、ViewModel class に unit test を書く。これらの test は各 method のロジックを個別に検証すべきです。
* view には widget test を書く。特に routing と dependency injection のテストが重要です。

#### テスト用の fake を作る (そして fake を活かせるコードを書く)。
**Strongly recommend**

fake は、各 method の内部実装よりも、入力と出力に重点を置きます。アプリケーション コードを書くときにこの考えを持っていれば、
モジュール化され、軽量で、入力と出力が明確に定義された関数や class を書くことになります。
