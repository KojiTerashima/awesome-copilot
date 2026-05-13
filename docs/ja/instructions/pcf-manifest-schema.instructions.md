---
description: '利用可能なすべての XML 要素を含む、PCF コンポーネント向け完全 manifest schema reference'
applyTo: '**/*.xml'
---

# Manifest Schema Reference

manifest file (`ControlManifest.Input.xml`) は、コード コンポーネントを定義する metadata document です。この reference では、利用可能なすべての manifest element とその目的を一覧化します。

## ルート要素

### manifest

component 定義全体を含むルート要素です。

## コア要素

### code

component ロジックを実装する resource file を参照します。

**Attributes:**
- `path`: TypeScript / JavaScript 実装 file への path
- `order`: 読み込み順 (通常は "1")

**利用可能な platform:** Model-driven apps、canvas apps、portals

### control

namespace、version、表示情報を含む component 自体を定義します。

**主な Attributes:**
- `namespace`: component の namespace
- `constructor`: constructor 名
- `version`: semantic version (例: "1.0.0")
- `display-name-key`: 表示名用の resource key
- `description-key`: 説明用の resource key
- `control-type`: control の type ("standard" または "virtual")

**利用可能な platform:** Model-driven apps、canvas apps、portals

## Property 要素

### property

component の input または output property を定義します。

**主な Attributes:**
- `name`: property 名
- `display-name-key`: 表示名用の resource key
- `description-key`: 説明用の resource key
- `of-type`: data type (例: "SingleLine.Text"、"Whole.None"、"TwoOptions"、"DateAndTime.DateOnly")
- `usage`: property の用途 ("bound" または "input")
- `required`: property が必須かどうか (true / false)
- `of-type-group`: type-group への参照
- `default-value`: property の既定値

**利用可能な platform:** Model-driven apps、canvas apps、portals

### type-group

property が受け取れる型の group を定義します。

**用途:** 1 つの property が複数の data type を受け取れるようにする

**利用可能な platform:** Model-driven apps、canvas apps、portals

## Data Set 要素

### data-set

表形式データを扱うための dataset property を定義します。

**主な Attributes:**
- `name`: dataset 名
- `display-name-key`: 表示名用の resource key
- `description-key`: 説明用の resource key

**利用可能な platform:** Model-driven apps (canvas apps は制限付き)

## Resource 要素

### resources

すべての resource 定義 (code、CSS、image、localization) を含む container です。

**利用可能な platform:** Model-driven apps、canvas apps、portals

### css

CSS stylesheet file を参照します。

**Attributes:**
- `path`: CSS file への path
- `order`: 読み込み順

**利用可能な platform:** Model-driven apps、canvas apps、portals

### img

image resource を参照します。

**Attributes:**
- `path`: image file への path

**利用可能な platform:** Model-driven apps、canvas apps、portals

### resx

localization 用の resource file を参照します。

**Attributes:**
- `path`: .resx file への path
- `version`: version 番号

**利用可能な platform:** Model-driven apps、canvas apps、portals

## Feature Usage 要素

### uses-feature

component が特定の platform feature を使うことを宣言します。

**主な Attributes:**
- `name`: feature 名 (例: "Device.captureImage"、"Device.getCurrentPosition"、"Utility.lookupObjects"、"WebAPI")
- `required`: feature が必須かどうか (true / false)

**一般的な Feature:**
- Device.captureAudio
- Device.captureImage
- Device.captureVideo
- Device.getBarcodeValue
- Device.getCurrentPosition
- Device.pickFile
- Utility.lookupObjects
- WebAPI

**利用可否:** feature と platform により異なる

### feature-usage

feature 宣言を含む container です。

**利用可能な platform:** Model-driven apps、canvas apps

## Dependency 要素

### dependency

component に必要な外部 dependency を宣言します。

**利用可能な platform:** Model-driven apps、canvas apps

### external-service-usage

component が使う外部 service を宣言します。

**主な Attributes:**
- `enabled`: 外部 service 利用を有効にするかどうか (true / false)

**利用可能な platform:** Model-driven apps、canvas apps

## Library 要素

### platform-library

platform が提供する library (例: React、Fluent UI) を参照します。

**主な Attributes:**
- `name`: library 名 (例: "React"、"Fluent")
- `version`: library version

**利用可能な platform:** Model-driven apps、canvas apps

## Event 要素

### event

component が発火できる custom event を定義します。

**主な Attributes:**
- `name`: event 名
- `display-name-key`: 表示名用の resource key
- `description-key`: 説明用の resource key

**利用可能な platform:** Model-driven apps、canvas apps

## Action 要素

### platform-action

component が呼び出せる platform action を定義します。

**利用可能な platform:** Model-driven apps

## Manifest 構造の例

```xml
<?xml version="1.0" encoding="utf-8" ?>
<manifest>
  <control namespace="SampleNamespace"
           constructor="SampleControl"
           version="1.0.0"
           display-name-key="Sample_Display_Key"
           description-key="Sample_Desc_Key"
           control-type="standard">

    <!-- Properties -->
    <property name="sampleProperty"
              display-name-key="Property_Display_Key"
              description-key="Property_Desc_Key"
              of-type="SingleLine.Text"
              usage="bound"
              required="true" />

    <!-- Type Group Example -->
    <type-group name="numbers">
      <type>Whole.None</type>
      <type>Currency</type>
      <type>FP</type>
      <type>Decimal</type>
    </type-group>

    <property name="numericProperty"
              display-name-key="Numeric_Display_Key"
              of-type-group="numbers"
              usage="bound" />

    <!-- Data Set Example -->
    <data-set name="dataSetProperty"
              display-name-key="Dataset_Display_Key">
    </data-set>

    <!-- Events -->
    <event name="onCustomEvent"
           display-name-key="Event_Display_Key"
           description-key="Event_Desc_Key" />

    <!-- Resources -->
    <resources>
      <code path="index.ts" order="1" />
      <css path="css/SampleControl.css" order="1" />
      <img path="img/icon.png" />
      <resx path="strings/SampleControl.1033.resx" version="1.0.0" />
    </resources>

    <!-- Feature Usage -->
    <feature-usage>
      <uses-feature name="WebAPI" required="true" />
      <uses-feature name="Device.captureImage" required="false" />
    </feature-usage>

    <!-- Platform Library -->
    <platform-library name="React" version="16.8.6" />
    <platform-library name="Fluent" version="8.29.0" />

  </control>
</manifest>
```

## Manifest の検証

manifest schema は build process 中に検証されます。
- 必須要素が不足していると build error になる
- 不正な attribute 値には警告が出る
- manifest 構造の検証には `pac pcf` command を使う

## ベスト プラクティス

1. **Semantic Versioning**: component version には semantic versioning (major.minor.patch) を使う
2. **Localization Keys**: hardcode した文字列ではなく、常に resource key を使う
3. **Feature Declaration**: component が使うすべての feature を宣言する
4. **Required vs Optional**: property と feature は本当に必要な場合にだけ required としてマークする
5. **Type Groups**: 複数の numeric type を受け取る property には type-group を使う
6. **Data Types**: 要件に最も合う具体的な data type を選ぶ
7. **CSS Scoping**: host application と衝突しないよう CSS を scope する
8. **Resource Organization**: resource は separate folder (css/、img/、strings/) に整理する

## Data Type Reference

property によく使われる `of-type` 値:

- **Text**: SingleLine.Text、Multiple、SingleLine.TextArea、SingleLine.Email、SingleLine.Phone、SingleLine.Url、SingleLine.Ticker
- **Numbers**: Whole.None、Currency、FP、Decimal
- **Date/Time**: DateAndTime.DateAndTime、DateAndTime.DateOnly
- **Boolean**: TwoOptions
- **Lookup**: Lookup.Simple
- **OptionSet**: OptionSet、MultiSelectOptionSet
- **Other**: Enum

## Platform Availability の凡例

- ✅ **Model-driven apps**: 完全対応
- ✅ **Canvas apps**: 対応 (制限がある場合あり)
- ✅ **Portals**: Power Pages で対応

ほとんどの manifest element はすべての platform で利用できますが、一部の feature (特定の Device API や platform action など) は platform 固有です。必ず対象 environment でテストしてください。
