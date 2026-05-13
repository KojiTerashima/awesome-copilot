---
name: csharp-docs
description: 'C# type が XML comment で文書化され、documentation の best practice に従うようにします。'
---

# C# Documentation Best Practices

- public member は XML comment で文書化するべきです。
- internal member も、特に複雑で自明でない場合は文書化を推奨します。

## Guidance for all APIs

- `<summary>` には、type または member が何をするかを現在形の三人称動詞で始まる 1 文の簡潔な説明として記述します。
- `<remarks>` には、実装詳細、使用上の注意、その他の関連文脈を含む追加情報を記述します。
- `null`、`true`、`false`、`int`、`bool` などの言語固有 keyword には `<see langword>` を使います。
- inline code snippet には `<c>` を使います。
- member の使い方の例には `<example>` を使います。
  - code block には `<code>` を使います。`<code>` tag は `<example>` tag の中に置くべきです。コード例の言語は、たとえば `<code language="csharp">` のように `language` attribute で指定します。
- 他の type や member を文中で参照するときは `<see cref>` を使います。
- online docs の "See also" section で、文中ではなく独立した参照を示すときは `<seealso>` を使います。
- base class や interface から documentation を継承するときは `<inheritdoc/>` を使います。
  - ただし大きな behavior 変更がある場合は差分を明記してください。

## Methods

- method parameter の説明には `<param>` を使います。
  - 説明は data type を明示しない名詞句にします。
  - 導入の冠詞で始めます。
  - parameter が flag enum の場合、説明は "A bitwise combination of the enumeration values that specifies..." で始めます。
  - parameter が non-flag enum の場合、説明は "One of the enumeration values that specifies..." で始めます。
  - parameter が Boolean の場合、説明は "`<see langword="true" />` to ...; otherwise, `<see langword="false" />`." という形にします。
  - parameter が `out` parameter の場合、説明は "When this method returns, contains .... This parameter is treated as uninitialized." という形にします。
- documentation 内で parameter 名を参照するときは `<paramref>` を使います。
- generic type または method の type parameter の説明には `<typeparam>` を使います。
- documentation 内で type parameter を参照するときは `<typeparamref>` を使います。
- method の返り値の説明には `<returns>` を使います。
  - 説明は data type を明示しない名詞句にします。
  - 導入の冠詞で始めます。
  - return type が Boolean の場合、説明は "`<see langword="true" />` if ...; otherwise, `<see langword="false" />`." という形にします。

## Constructors

- summary の文言は "Initializes a new instance of the <Class> class [or struct]." とします。

## Properties

- `<summary>` は次のいずれかで始めます:
  - 読み書き可能 property なら "Gets or sets..."
  - read-only property なら "Gets..."
  - Boolean を返す property なら "Gets [or sets] a value that indicates whether..."
- property の値の説明には `<value>` を使います。
  - 説明は data type を明示しない名詞句にします。
  - property に default value がある場合は、たとえば "The default is `<see langword="false" />`" のように別文で追加します。
  - 値の型が Boolean の場合、説明は "`<see langword="true" />` if ...; otherwise, `<see langword="false" />`. The default is ..." という形にします。

## Exceptions

- constructor、property、indexer、method、operator、event が投げる exception を文書化するには `<exception cref>` を使います。
- member が直接送出するすべての exception を記録します。
- 入れ子の member が送出する exception については、ユーザーが遭遇しやすいものだけを文書化します。
- exception の説明は、それが送出される条件を記述します。
  - 文頭の "Thrown if ..." や "If ..." は省略します。たとえば "An error occurred when accessing a Message Queuing API." のように条件そのものを記述します。
