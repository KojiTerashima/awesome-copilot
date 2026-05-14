---
description: 'Power Apps Component Framework の制限と制約'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# 制限事項

Power Apps component framework を使うと、Power Apps と Power Pages のユーザー体験を向上させる独自のコード コンポーネントを作成できます。ただし、コード コンポーネントで一部の機能を実装する開発者を制約するいくつかの制限があります。以下はその一部です。

## 1. Canvas Apps では Dataverse 依存 API は利用不可

WebAPI を含む Microsoft Dataverse 依存 API は、現時点では Power Apps canvas applications では利用できません。個別の API の利用可否については、[Power Apps component framework API reference](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/) を参照してください。

## 2. 外部ライブラリをバンドルするか Platform Libraries を使用する

コード コンポーネントでは、[React controls & platform libraries](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/react-controls-platform-libraries) を使用するか、外部ライブラリの内容を含むすべてのコードを主要なコード バンドルにまとめてバンドルする必要があります。

Power Apps command line interface が、外部ライブラリの内容をコンポーネント固有のバンドルに取り込む際にどのように役立つかの例については、[Angular flip component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/sample-controls/angular-flip-control) のサンプルを参照してください。

## 3. HTML Web Storage Objects を使用しない

コード コンポーネントでは、`window.localStorage` や `window.sessionStorage` などの HTML web storage objects を使用してデータを保存しないでください。ユーザーの browser や mobile client にローカル保存されたデータは安全ではなく、安定して利用できる保証もありません。

## 4. Canvas Apps ではカスタム認証はサポートされない

コード コンポーネントでの custom auth は Power Apps canvas applications ではサポートされていません。代わりに、connector を使ってデータ取得やアクション実行を行ってください。

## 関連トピック

- [Power Apps component framework API reference](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/)
- [Power Apps component framework overview](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview)
