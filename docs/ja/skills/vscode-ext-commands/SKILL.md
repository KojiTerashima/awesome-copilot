---
name: vscode-ext-commands
description: 'Guidelines for contributing commands in VS Code extensions. Indicates naming convention, visibility, localization and other relevant attributes, following VS Code extension development guidelines, libraries and good practices'
---
# VS Code 拡張コマンドの貢献

このスキルは、VS Code 拡張機能でコマンドを提供するのに役立ちます

## このスキルをいつ使用するか

このスキルは、次の場合に使用します。
- VS Code 拡張機能にコマンドを追加または更新します

# 指示

VS Code コマンドは、カテゴリ、可視性、または場所に関係なく、常に `title` を定義する必要があります。コマンドの「種類」ごとに、以下に説明するいくつかの特徴を持ついくつかのパターンを使用します。

* 通常のコマンド: デフォルトでは、すべてのコマンドはコマンド パレットでアクセス可能であり、`category` を定義する必要があります。コマンドがサイド バーで使用されない限り、`icon` は必要ありません。

* サイド バー コマンド: その名前は特別なパターンに従い、アンダースコア (`_`) で始まり、`#sideBar` という接尾辞が付けられます (例: `_extensionId.someCommand#sideBar` など)。 `icon` を定義する必要があり、`enablement` のルールがある場合とない場合があります。サイド バー専用のコマンドはコマンド パレットに表示しないでください。 `view/title` または `view/item/context` に提供する場合は、それが表示されることを _order/position_ に通知する必要があります。また、使用する正しい `group` を識別するために、「他のコマンド/ボタンに関連する」という用語を使用できます。また、新しいコマンドが表示されるように条件 (`when`) を定義することをお勧めします。