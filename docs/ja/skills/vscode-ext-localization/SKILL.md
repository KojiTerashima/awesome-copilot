---
name: vscode-ext-localization
description: 'Guidelines for proper localization of VS Code extensions, following VS Code extension development guidelines, libraries and good practices'
---
# VS Code 拡張機能のローカリゼーション

このスキルは、VS Code 拡張機能のあらゆる側面をローカライズするのに役立ちます

## このスキルをいつ使用するか

このスキルは、次の場合に使用します。
- 新規または既存の提供された構成 (設定)、コマンド、メニュー、ビュー、またはウォークスルーをローカライズします。
- エンド ユーザーに表示される拡張ソース コードに含まれる新規または既存のメッセージ、またはその他の文字列リソースをローカライズします。

# 指示

VS Code のローカリゼーションは、ローカライズされるリソースに応じて 3 つの異なるアプローチで構成されます。新しいローカライズ可能なリソースを作成または更新するときは、現在利用可能なすべての言語に対応するローカライズを作成または更新する必要があります。

1. `package.json` で定義された設定、コマンド、メニュー、ビュー、ビューウェルカム、ウォークスルー タイトルおよび説明などの構成
  -> ブラジルポルトガル語 (`pt-br`) ローカリゼーションの `package.nls.pt-br.json` のような、専用の `package.nls.LANGID.json` ファイル
2. ウォークスルー コンテンツ (独自の `Markdown` ファイルで定義)
  -> ブラジルポルトガル語ローカリゼーション用の `walkthrough/someStep.pt-br.md` のような専用の `Markdown` ファイル
3. 拡張機能のソース コード (JavaScript または TypeScript ファイル) にあるメッセージと文字列
  -> ブラジルポルトガル語ローカリゼーション専用の `bundle.l10n.pt-br.json`