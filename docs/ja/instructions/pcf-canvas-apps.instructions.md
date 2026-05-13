---
description: 'Canvas Apps 向けコード コンポーネントの実装、セキュリティ、構成'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# Canvas Apps 向けコード コンポーネント

プロの開発者は、Power Apps component framework を使って canvas apps で利用できるコード コンポーネントを作成できます。app maker は [Microsoft Power Platform CLI](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/get-powerapps-cli) を使って、canvas apps にコード コンポーネントを作成、インポート、追加できます。

> **注**: 一部の API は canvas apps では利用できない場合があります。各 API を確認して、どこで利用可能かを判断することを推奨します。

## セキュリティ上の考慮事項

> **警告**: コード コンポーネントには Microsoft が生成したものではない code が含まれる可能性があり、Power Apps Studio でレンダリングされる際に security token やデータへアクセスできるおそれがあります。canvas app にコード コンポーネントを追加するときは、その code component solution が信頼できるソース由来であることを確認してください。この脆弱性は、canvas app を実行しているときには存在しません。

### Power Apps Studio でのセキュリティ警告

コード コンポーネントを含む canvas app を Power Apps Studio で開くと、潜在的に安全でない code に関する警告メッセージが表示されます。Power Apps Studio 環境内のコード コンポーネントは security token にアクセスできるため、信頼できるソースからの component だけを開く必要があります。

**ベスト プラクティス:**
- administrator と system customizer は、environment にインポートする前にすべてのコード コンポーネントを確認し、検証する
- maker には、検証後にのみ component を利用可能にする
- unmanaged solution を使ってコード コンポーネントをインポートした場合、または `pac pcf push` を使ってコード コンポーネントをインストールした場合は、`Default` publisher が表示される

![安全性の警告](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/canvas-app-safety-warning.png)

## 前提条件

- Power Apps license が必要です。詳細情報: [Power Apps component framework licensing](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview#licensing)
- environment で Power Apps component framework 機能を有効にするには、system administrator 権限が必要です

## Power Apps Component Framework 機能を有効にする

app にコード コンポーネントを追加するには、それらを使いたい各 environment で Power Apps component framework 機能を有効にする必要があります。既定では、Power Apps component feature は model-driven apps で有効です。

### Canvas Apps で有効化する手順:

1. [Power Apps](https://powerapps.microsoft.com/) にサインインする
2. **Settings** ![Settings](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/settings.png) を選択し、次に **Admin Center** を選択する

   ![Settings と Admin Center](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/select-admin-center-from-settings.png)

3. 左側の pane で **Environments** を選び、この機能を有効にしたい environment を選択してから **Settings** を選択する
4. **Product** を展開し、**Features** を選択する
5. 利用可能な機能の一覧から **Power Apps component framework for canvas apps** をオンにし、**Save** を選択する

   ![Power Apps component framework を有効化する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/enable-pcf-feature.png)

## コード コンポーネントの実装

environment で Power Apps component framework 機能を有効にしたら、コード コンポーネントのロジック実装を開始できます。段階的なチュートリアルについては、[Create your first code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript) を参照してください。

**推奨**: 実装を始める前に、canvas apps におけるコード コンポーネントの [limitations](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/limitations) を確認してください。

## Canvas App にコンポーネントを追加する

1. Power Apps Studio に移動する
2. 新しい canvas app を作成するか、コード コンポーネントを追加したい既存の app を編集する

   > **重要**: 次の手順へ進む前に、コード コンポーネントを含む solution の .zip file がすでに Microsoft Dataverse に [インポート](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/import-update-export-solutions) されていることを確認してください。

3. 左側の pane で **Add (+)** を選択し、次に **Get more components** を選択する

   ![コンポーネントを挿入する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/insert-code-components-using-get-more-components.png)

4. **Code** tab を選択し、一覧から component を選んで **Import** を選択する

   ![コンポーネントをインポートする](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/insert-component-add-sample-component.png)

5. 左側の pane で **+** を選択し、**Code components** を展開して、app に追加したい component を選択する

   ![コンポーネントを追加する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/add-sample-component-from-list.png)

> **注**: **Insert > Custom > Import component** を選択して component を追加することもできます。このオプションは将来の release で削除される予定のため、上記の流れを使うことを提案します。

### コンポーネントのプロパティ

Properties tab では、コード コンポーネントのプロパティが表示されます。

![既定のコード コンポーネント properties pane](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/property-pane-with-parameters.png)

> **注**: 既存のコード コンポーネントは、properties を既定の Properties tab で利用できるようにしたい場合、コード コンポーネントの manifest version を更新することで再インポートできます。従来どおり、properties は Advanced properties tab でも引き続き利用できます。

## Canvas App からコード コンポーネントを削除する

1. コード コンポーネントを追加した app を開く
2. 左側の pane で **Tree view** を選択し、コード コンポーネントを追加した screen を選択する
3. component の横にある **More (...)** を選択し、**Delete** を選択する

   ![コード コンポーネントを削除する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/delete-code-component.png)

4. 変更を確認するため app を保存する

## 既存のコード コンポーネントを更新する

コード コンポーネントを更新し、実行時の変更を確認したい場合は、manifest file の `version` property を変更する必要があります。変更を加えるたびに component の version を更新することを推奨します。

> **注**: 既存のコード コンポーネントが更新されるのは、Power Apps Studio で app を閉じて再度開いたときだけです。app を再度開くと、コード コンポーネントの更新を求められます。component を単に削除して再追加しても更新されません。まず更新済み solution 内のすべての customizations を publish してください。そうしないと、コード コンポーネントに行った更新は表示されません。

## 関連項目

- [Power Apps component framework overview](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview)
- [初めてのコード コンポーネントを作成する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript)
- [Power Apps component framework を学ぶ](https://learn.microsoft.com/en-us/training/paths/use-power-apps-component-framework)
