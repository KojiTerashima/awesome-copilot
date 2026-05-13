---
description: 'model-driven apps と canvas apps での利用可否を含む完全な PCF API reference'
applyTo: '**/*.{ts,tsx,js}'
---

# Power Apps Component Framework API Reference

Power Apps component framework は、強力なコード コンポーネントを作成するための豊富な API 群を提供します。この reference では、利用可能なすべての interface と、それらが異なる app type で利用可能かどうかを一覧化します。

## API の利用可否

次の表は、Power Apps component framework で利用可能なすべての API interface と、それぞれが model-driven apps と canvas apps で利用可能かどうかを示しています。

| API | モデル駆動型アプリ | Canvas apps |
|-----|------------------|-------------|
| AttributeMetadata | はい | いいえ |
| Client | はい | はい |
| Column | はい | はい |
| ConditionExpression | はい | はい |
| Context | はい | はい |
| DataSet | はい | はい |
| Device | はい | はい |
| Entity | はい | はい |
| Events | はい | はい |
| Factory | はい | はい |
| Filtering | はい | はい |
| Formatting | はい | はい |
| ImageObject | はい | はい |
| Linking | はい | はい |
| Mode | はい | はい |
| Navigation | はい | はい |
| NumberFormattingInfo | はい | はい |
| Paging | はい | はい |
| Popup | はい | はい |
| PopupService | はい | はい |
| PropertyHelper | はい | はい |
| Resources | はい | はい |
| SortStatus | はい | はい |
| StandardControl | はい | はい |
| UserSettings | はい | はい |
| Utility | はい | はい |
| WebApi | はい | はい |

## 主な API Namespace

### Context APIs

`Context` object は、framework のすべての機能への access を提供し、component の lifecycle method に渡されます。そこには次が含まれます。

- **Client**: client に関する情報 (form factor、network status)
- **Device**: デバイス機能 (camera、location、microphone)
- **Factory**: framework object を作成する factory method
- **Formatting**: 数値と日付の書式設定
- **Mode**: component mode と tracking
- **Navigation**: navigation method
- **Resources**: resource (image、string) への access
- **UserSettings**: user setting (locale、number format、security role)
- **Utils**: utility method (`getEntityMetadata`、`hasEntityPrivilege`、`lookupObjects`)
- **WebApi**: Dataverse Web API method

### Data APIs

- **DataSet**: 表形式データを扱う
- **Column**: column metadata とデータに access する
- **Entity**: record data に access する
- **Filtering**: データ filtering を定義する
- **Linking**: relationship を定義する
- **Paging**: データ pagination を処理する
- **SortStatus**: sorting を管理する

### UI APIs

- **Popup**: popup dialog を作成する
- **PopupService**: popup lifecycle を管理する
- **Mode**: component の rendering mode を取得する

### Metadata APIs

- **AttributeMetadata**: column metadata (model-driven のみ)
- **PropertyHelper**: property metadata helper

### Standard Control

- **StandardControl**: lifecycle method を持つすべてのコード コンポーネントの基本 interface:
  - `init()`: component を初期化する
  - `updateView()`: component UI を更新する
  - `destroy()`: resource を cleanup する
  - `getOutputs()`: output 値を返す

## 利用ガイドライン

### Model-Driven と Canvas Apps

一部の API は platform の違いにより model-driven apps でのみ利用可能です。

- **AttributeMetadata**: model-driven のみ - 詳細な column metadata を提供する
- その他のほとんどの API は両 platform で利用可能

### API Version 互換性

- 対象 platform (model-driven または canvas) 向けに API の利用可否を常に確認する
- 一部の API は platform ごとに挙動が異なる場合がある
- 互換性を確保するため、対象 environment で component をテストする

### よくあるパターン

1. **Context API へアクセスする**
   ```typescript
   // In init or updateView
   const userLocale = context.userSettings.locale;
   const isOffline = context.client.isOffline();
   ```

2. **DataSet を扱う**
   ```typescript
   // Access dataset records
   const records = context.parameters.dataset.records;

   // Get sorted columns
   const sortedColumns = context.parameters.dataset.sorting;
   ```

3. **WebApi を使う**
   ```typescript
   // Retrieve records
   context.webAPI.retrieveMultipleRecords("account", "?$select=name");

   // Create record
   context.webAPI.createRecord("contact", data);
   ```

4. **デバイス機能**
   ```typescript
   // Capture image
   context.device.captureImage();

   // Get current position
   context.device.getCurrentPosition();
   ```

5. **Formatting**
   ```typescript
   // Format date
   context.formatting.formatDateLong(date);

   // Format number
   context.formatting.formatDecimal(value);
   ```

## ベスト プラクティス

1. **Type Safety**: 型チェックと IntelliSense のために TypeScript を使う
2. **Null Checks**: API object に access する前に必ず null / undefined を確認する
3. **Error Handling**: API call は try-catch block で囲む
4. **Platform Detection**: 挙動を適応させるため `context.client.getFormFactor()` を確認する
5. **API Availability**: 使う前に対象 platform での API 利用可否を検証する
6. **Performance**: 適切な場合は API 結果を cache し、繰り返し call を避ける

## 追加リソース

- 各 API の詳細 documentation は [Power Apps component framework API reference](https://learn.microsoft.com/powerapps/developer/component-framework/reference/) を参照してください
- 各 API の sample code は [PowerApps-Samples repository](https://github.com/microsoft/PowerApps-Samples/tree/master/component-framework) で入手できます
