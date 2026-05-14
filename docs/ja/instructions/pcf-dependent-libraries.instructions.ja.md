---
description: 'PCF コンポーネントで依存ライブラリを使う'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# Dependent Libraries (Preview)

[このトピックはプレリリース ドキュメントであり、変更される可能性があります。]

model-driven apps では、複数の component に対する依存関係として読み込まれる別 component に含まれた、事前構築済み library を再利用できます。

複数の control に事前構築済み library の複製を持つのは望ましくありません。既存 library を再利用すると、特に library が大きい場合に、その library を使うすべての component の読み込み時間を減らせるため、パフォーマンスが向上します。library の再利用は、build process における保守オーバーヘッドの削減にも役立ちます。

## Before と After

**Before**: 各 PCF component に custom library file を内包
![各 pcf component に custom library file が含まれていることを示す図](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/dependent-library-before-example.png)

**After**: Library Control から共有関数を呼び出す component
![Library Control から共有関数を呼び出す component を示す図](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/dependent-library-after-example.png)

## 実装手順

dependent library を使うには、次のことを行います。

1. library を含む **Library component** を作成する。この component は何らかの機能を提供してもよく、単に library の container であってもかまいません。
2. 別の component が、その library component により読み込まれた library に依存するよう構成する。

既定では、library は dependent component の読み込み時に読み込まれますが、必要に応じて load on demand に構成することもできます。

この方式なら、Library Control 内の library を独立して保守でき、dependent control 側では library の複製をバンドルする必要がありません。

## 仕組み

build process が 원하는形で library を deploy できるように、component project に設定データを追加する必要があります。この設定データは、次の file を追加または編集して設定します。

- **featureconfig.json**
- **webpack.config.js**
- manifest schema を編集して **dependency を登録**

### featureconfig.json

`node_modules` folder 内に生成された file を変更せずに、component の既定 feature flag を上書きするためにこの file を追加します。

**Feature Flags:**

| Flag | Description |
|------|-------------|
| `pcfResourceDependency` | component が library resource を使えるようにする。 |
| `pcfAllowCustomWebpack` | component が custom webpack を使えるようにする。library resource を定義する component ではこの機能を有効にする必要があります。 |

既定では、これらの値は `off` です。既定を上書きするには `on` に設定します。

**Example 1:**
```json
{
  "pcfAllowCustomWebpack": "on"
}
```

**Example 2:**
```json
{
   "pcfResourceDependency": "on",
   "pcfAllowCustomWebpack": "off"
}
```

### webpack.config.js

component の build process では [Webpack](https://webpack.js.org/) を使って code と dependency を deploy 可能な asset に bundle します。これらの bundle から library を除外するには、library の alias を `externals` として指定する `webpack.config.js` file を project root folder に追加します。[Webpack externals 設定オプションの詳細](https://webpack.js.org/configuration/externals/)

library alias が `myLib` の場合、この file は次のようになります。

```javascript
/* eslint-disable */
"use strict";

module.exports = {
  externals: {
    "myLib": "myLib"
  },
}
```

### Dependencies を登録する

manifest schema の [resources](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/resources) 内にある [dependency element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/dependency) を使います。

```xml
<resources>
  <dependency
    type="control"
    name="samples_SampleNS.SampleStubLibraryPCF"
    order="1"
  />
  <code path="index.ts" order="2" />
</resources>
```

### Component の On-Demand Load としての Dependency

component の読み込み時に dependent library を読み込むのではなく、on demand で読み込むこともできます。on demand loading により、特に dependent library が大きい場合、より複雑な control が依存関係を必要なときだけ読み込める柔軟性が得られます。

![library を on demand で読み込む形で、library 内の関数を使う様子を示す図](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/dependent-library-on-demand-load.png)

on demand loading を有効にするには、次のことを行います。

**Step 1**: [control element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/control) に、次の [platform-action element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/platform-action)、[feature-usage element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/feature-usage)、[uses-feature element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/uses-feature) の child element を追加します。

```xml
<platform-action action-type="afterPageLoad" />
<feature-usage>
   <uses-feature name="Utility"
      required="true" />
</feature-usage>
```

**Step 2**: [dependency element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/dependency) の `load-type` attribute を `onDemand` に設定します。

```xml
<dependency type="control"
      name="samples_SampleNamespace.StubLibrary"
      load-type="onDemand" />
```

## 次のステップ

dependent library の作成を段階的に説明する tutorial を試してください。

[Tutorial: Use dependent libraries in a component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/tutorial-use-dependent-libraries)
