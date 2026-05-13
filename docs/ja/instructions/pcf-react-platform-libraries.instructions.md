---
description: 'PCF コンポーネント向けの React controls と platform libraries'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# React Controls と Platform Libraries

React と platform libraries を使うと、Power Apps platform が使っているのと同じ infrastructure を利用することになります。これは、各 control ごとに React と Fluent library を個別にパッケージ化する必要がなくなることを意味します。すべての control は共通の library instance と version を共有し、シームレスで一貫した体験を提供します。

## 利点

既存の platform React と Fluent library を再利用することで、次のことが期待できます。

- **control bundle size の削減**
- **solution packaging の最適化**
- **実行時の転送、script 実行、control rendering の高速化**
- **Power Apps Fluent design system との design / theme の整合**

> **注**: GA release では、既存の virtual control は引き続き動作します。ただし、今後の platform React version の upgrade を容易にするため、最新 CLI version (>=1.37) を使って rebuild と deploy を行う必要があります。

## 前提条件

他の component と同様に、[Visual Studio Code](https://code.visualstudio.com/Download) と [Microsoft Power Platform CLI](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/powerapps-cli#install-microsoft-power-platform-cli) をインストールする必要があります。

> **注**: Windows 向けの Power Platform CLI をすでにインストールしている場合は、`pac install latest` command を使って最新 version を実行していることを確認してください。Power Platform Tools for Visual Studio Code は自動更新されるはずです。

## React コンポーネントを作成する

> **注**: これらの instruction は、すでにコード コンポーネントを作成したことがある前提です。まだであれば、[Create your first component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript) を参照してください。

`pac pcf init` command には新しい `--framework` (`-fw`) parameter があります。この parameter の値を `react` に設定します。

### コマンド パラメーター

| Parameter | Value |
|-----------|-------|
| --name | ReactSample |
| --namespace | SampleNamespace |
| --template | field |
| --framework | react |
| --run-npm-install | true (default) |

### PowerShell コマンド

次の PowerShell command は parameter の short cut を使い、React component project を作成して `npm-install` を実行します。

```powershell
pac pcf init -n ReactSample -ns SampleNamespace -t field -fw react -npm
```

これで、通常どおり `npm start` を使って test harness 内で control を build して確認できます。

control を build した後は、standard code component と同様に、solution にパッケージ化して model-driven apps (custom page を含む) や canvas apps で使用できます。

## Standard Components との違い

### ControlManifest.Input.xml

[control element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/control) の `control-type` attribute は、`standard` ではなく `virtual` に設定されます。

> **注**: この値を変更しても、component が一方の型から他方の型に変換されるわけではありません。

[resources element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/resources) の中に、2 つの新しい [platform-library element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/platform-library) child element があります。

```xml
<resources>
  <code path="index.ts" order="1" />
  <platform-library name="React" version="16.14.0" />
  <platform-library name="Fluent" version="9.46.2" />
</resources>
```

> **注**: 有効な platform library version の詳細は、Supported platform libraries list を参照してください。

**推奨**: Fluent 8 と 9 には platform libraries の使用を推奨します。Fluent を使わない場合は、`name` attribute の値が `Fluent` である `platform-library` element を削除してください。

### Index.ts

control 初期化用の [ReactControl.init](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/react-control/init) method には `div` parameter がありません。React control は DOM を直接 render しないためです。代わりに [ReactControl.updateView](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/react-control/updateview) は、React 形式で実際の control の詳細を持つ ReactElement を返します。

### bundle.js

React と Fluent library は共有されるため package に含まれず、その結果 bundle.js のサイズは小さくなります。

## サンプル コントロール

次の control が sample に含まれています。これらは standard version と同じように動作しますが、virtual control であるためパフォーマンスが向上します。

| Sample | Description | Link |
|--------|-------------|------|
| ChoicesPickerReact | standard の ChoicesPickerControl を React Control に変換したもの | ChoicesPickerReact Sample |
| FacepileReact | ReactStandardControl を React Control に変換したもの | FacepileReact |

## サポートされる Platform Libraries 一覧

platform libraries は、platform libraries 機能を使う control に対して、build 時と実行時の両方で提供されます。現在、platform から提供される version は次のとおりで、これらが現在サポートされる最新 version です。

| Library | Package | Build Version | Runtime Version |
|---------|---------|---------------|-----------------|
| React | react | 16.14.0 | 17.0.2 (Model), 16.14.0 (Canvas) |
| Fluent | @fluentui/react | 8.29.0 | 8.29.0 |
| Fluent | @fluentui/react | 8.121.1 | 8.121.1 |
| Fluent | @fluentui/react-components | >=9.4.0 <=9.46.2 | 9.68.0 |

> **注**: application は実行時に互換性のあるより高い version の platform library を読み込む場合がありますが、その version が利用可能な最新 version とは限りません。Fluent 8 と Fluent 9 はそれぞれサポートされていますが、同じ manifest で両方を指定することはできません。

## FAQ

### Q: 既存の standard control を、platform libraries を使う React control に変換できますか?

A: いいえ。新しい template を使って新しい control を作成し、その後 manifest と index.ts method を更新する必要があります。参考として、上で説明した standard sample と react sample を比較してください。

### Q: Power Pages で React controls と platform libraries を使えますか?

A: いいえ。React controls と platform libraries は現在、canvas apps と model-driven apps でのみサポートされています。Power Pages では、React control は他の field の変更に基づいて更新されません。

## 関連記事

- [コード コンポーネントとは何か](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/custom-controls-overview)
- [canvas apps 向けコード コンポーネント](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/component-framework-for-canvas-apps)
- [コード コンポーネントを作成してビルドする](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf)
- [Power Apps component framework を学ぶ](https://learn.microsoft.com/en-us/training/paths/use-power-apps-component-framework)
- [Power Pages でコード コンポーネントを使う](https://learn.microsoft.com/en-us/power-apps/maker/portals/component-framework)
