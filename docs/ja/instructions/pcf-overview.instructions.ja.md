---
description: 'Power Apps Component Framework の概要と基礎'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# Power Apps Component Framework 概要

Power Apps component framework を使うと、プロの開発者や app maker は model-driven apps と canvas apps 向けのコード コンポーネントを作成できます。これらのコード コンポーネントは、フォーム、ビュー、ダッシュボード、canvas app の画面でデータを扱うユーザーの体験を向上させるために利用できます。

## 主な機能

PCF では次のことができます。
- フォーム上で数値テキスト値を表示する列を、`dial` または `slider` のコード コンポーネントに置き換える
- データセットに結び付いた一覧を、`Calendar` や `Map` のようなまったく異なる視覚体験に変換する

## 重要な制限

- Power Apps component framework は Unified Interface でのみ動作し、legacy web client では動作しません
- Power Apps component framework は現在、on-premises 環境ではサポートされていません

## PCF と Web Resources の違い

HTML web resources とは異なり、コード コンポーネントは次の特徴を持ちます。
- 同じコンテキストの一部としてレンダリングされる
- 他のコンポーネントと同時に読み込まれる
- ユーザーにシームレスな体験を提供する

コード コンポーネントは次のように利用できます。
- Power Apps の幅広い機能全体で使用できる
- 異なる table や form をまたいで何度でも再利用できる
- すべての HTML、CSS、TypeScript file を 1 つの solution package にまとめられる
- 環境間で移動できる
- AppSource 経由で利用可能にできる

## 主な利点

### 豊富な Framework API
- コンポーネントのライフサイクル管理
- コンテキスト データとメタデータへのアクセス
- Web API を通じたシームレスなサーバー アクセス
- ユーティリティとデータ書式設定メソッド
- デバイス機能: camera、location、microphone
- ユーザー体験要素: dialog、lookup、full-page rendering

### 開発上の利点
- モダンな web の実践をサポートする
- パフォーマンス向けに最適化されている
- 再利用性が高い
- すべての file を 1 つの solution file にまとめられる
- state を保持したまま、パフォーマンス上の理由で破棄・再読み込みされる状況に対応できる

## ライセンス要件

Power Apps component framework のライセンスは、使用するデータの種類と接続方法に基づきます。

### Premium Code Components
connector を介さず、ユーザーの browser client から外部サービスやデータに直接接続するコード コンポーネントは次の扱いになります。
- premium component と見なされる
- それらを使用する app は premium になる
- エンド ユーザーには Power Apps license が必要になる

premium として宣言するには、manifest に次を追加します。
```xml
<external-service-usage enabled="true">
  <domain>www.microsoft.com</domain>
</external-service-usage>
```

### Standard Code Components
外部サービスやデータに接続しないコード コンポーネントは次の扱いになります。
- standard feature を使う app は standard のまま維持される
- エンド ユーザーには最低でも Office 365 license が必要になる

**注**: Microsoft Dataverse に接続された model-driven apps でコード コンポーネントを使用する場合、エンド ユーザーには Power Apps license が必要です。

## 関連リソース

- [コード コンポーネントとは何か](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/custom-controls-overview)
- [canvas apps 向けコード コンポーネント](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/component-framework-for-canvas-apps)
- [コード コンポーネントを作成してビルドする](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf)
- [Power Apps component framework を学ぶ](https://learn.microsoft.com/en-us/training/paths/use-power-apps-component-framework)
- [Power Pages でコード コンポーネントを使う](https://learn.microsoft.com/en-us/power-apps/maker/portals/component-framework)

## トレーニング リソース

- [Power Apps Component Framework でコンポーネントを作成する - Training](https://learn.microsoft.com/en-us/training/paths/create-components-power-apps-component-framework/)
- [Microsoft Certified: Power Platform Developer Associate](https://learn.microsoft.com/en-us/credentials/certifications/power-platform-developer-associate/)
