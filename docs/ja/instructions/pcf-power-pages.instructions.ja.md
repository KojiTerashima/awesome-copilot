---
description: 'Power Pages site でコード コンポーネントを使用する'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# Power Pages でコード コンポーネントを使う

Power Pages は、Power Apps component framework を使って作成された model-driven apps 向け control をサポートするようになりました。Power Pages site の webpage でコード コンポーネントを使うには、次の流れになります。

![component framework を使ってコード コンポーネントを作成し、そのコード コンポーネントを model-driven app form に追加してから、portal の basic form 内でコード コンポーネント field を構成する手順](https://learn.microsoft.com/en-us/power-pages/configure/media/component-framework/steps.png)

これらの手順を完了すると、ユーザーは該当する [form](https://learn.microsoft.com/en-us/power-pages/getting-started/add-form) component を持つ webpage 上でコード コンポーネントを操作できるようになります。

## 前提条件

- environment でコード コンポーネント機能を有効にするには system administrator 権限が必要です
- Power Pages site version は [9.3.3.x](https://learn.microsoft.com/en-us/power-apps/maker/portals/versions/version-9.3.3.x) 以上である必要があります
- starter site package は [9.2.2103.x](https://learn.microsoft.com/en-us/power-apps/maker/portals/versions/package-version-9.2.2103) 以上である必要があります

## コード コンポーネントを作成してパッケージ化する

Power Apps component framework でコード コンポーネントを作成してパッケージ化する方法については、[Create your first component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript) を参照してください。

### サポートされる Field Type と Format

Power Pages では、コード コンポーネント利用時に使用できる field type と format が制限されています。次の表は、サポートされるすべての field data type と format を示しています。

**サポートされる型:**
- Currency
- DateAndTime.DateAndTime
- DateAndTime.DateOnly
- Decimal
- Enum
- Floating Point Number
- Multiple
- OptionSet
- SingleLine.Email
- SingleLine.Phone
- SingleLine.Text
- SingleLine.TextArea
- SingleLine.Ticker
- SingleLine.URL
- TwoOptions
- Whole

詳細は [Attributes list and descriptions](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/property#remarks) を参照してください。

### Power Pages でサポートされないコード コンポーネント

次のコード コンポーネント API はサポートされていません。
- [Device.captureAudio](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/captureaudio)
- [Device.captureImage](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/captureimage)
- [Device.captureVideo](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/capturevideo)
- [Device.getBarcodeValue](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/getbarcodevalue)
- [Device.getCurrentPosition](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/getcurrentposition)
- [Device.pickFile](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/device/pickfile)
- [Utility](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/utility)

**追加の制約:**
- [uses-feature](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/uses-feature) element は true に設定してはいけません
- Power Apps component framework で [サポートされない Value element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/property#value-elements-that-are-not-supported)
- form 内の複数 field にバインドされた Power Apps Component Framework (PCF) control はサポートされていません

## モデル駆動型アプリの Field にコード コンポーネントを追加する

model-driven app の field にコード コンポーネントを追加する方法については、[Add a code component to a field](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/add-custom-controls-to-a-field-or-entity#add-a-code-component-to-a-column) を参照してください。

> **重要**: Power Pages 向けのコード コンポーネントは、**Web** の client option を使う web browser で利用できます。

### Data Workspace を使って追加する

[Data workspace](https://learn.microsoft.com/en-us/power-pages/configure/data-workspace-forms) を使って form にコード コンポーネントを追加することもできます。

1. Data workspace form designer で Dataverse form を編集中に field を選択する
2. **+ Component** を選び、その field に適した component を選択する

   ![form にコンポーネントを追加する](https://learn.microsoft.com/en-us/power-pages/configure/media/component-framework/add-component-to-form.png)

3. **Save** と **Publish form** を選択する

## コード コンポーネント用に Power Pages Site を構成する

コード コンポーネントを model-driven app の field に追加した後、form 上でそのコード コンポーネントを使うように Power Pages を構成できます。

コード コンポーネントを有効にする方法は 2 つあります。

### Design Studio でコード コンポーネントを有効にする

design studio を使って form 上のコード コンポーネントを有効にするには、次の手順を行います。

1. [form を page に追加](https://learn.microsoft.com/en-us/power-pages/getting-started/add-form) した後、コード コンポーネントを追加した field を選択し、**Edit field** を選択する
2. **Enable custom component** field を選択する

   ![design studio で custom component を有効にする](https://learn.microsoft.com/en-us/power-pages/configure/media/component-framework/enable-code-component.png)

3. site を preview すると、custom component が有効になっていることが確認できます

### Portals Management App でコード コンポーネントを有効にする

Portals Management app を使って basic form にコード コンポーネントを追加するには、次の手順を行います。

1. [Portals Management](https://learn.microsoft.com/en-us/power-pages/configure/portal-management-app) app を開く
2. 左側の pane で **Basic Forms** を選択する
3. コード コンポーネントを追加したい form を選択する
4. **Related** を選択する
5. **Basic Form Metadata** を選択する
6. **New Basic Form Metadata** を選択する
7. **Type** に **Attribute** を選択する
8. **Attribute Logical Name** を選択する
9. **Label** を入力する
10. **Control Style** で **Code Component** を選択する
11. form を保存して閉じる

## Portal Web API を使うコード コンポーネント

コード コンポーネントは、create、retrieve、update、delete 操作を実行するために [portal Web API](https://learn.microsoft.com/en-us/power-pages/configure/web-api-overview) を使える webpage に対して構築し、追加できます。この機能により、portal solution を開発する際のカスタマイズの選択肢が広がります。詳細は [Implement a sample portal Web API component](https://learn.microsoft.com/en-us/power-pages/configure/implement-webapi-component) を参照してください。

## 次のステップ

[Tutorial: Use code components in portals](https://learn.microsoft.com/en-us/power-pages/configure/component-framework-tutorial)

## 関連項目

- [Power Apps component framework overview](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview)
- [Create your first component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript)
- [model-driven apps の列または table にコード コンポーネントを追加する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/add-custom-controls-to-a-field-or-entity)
