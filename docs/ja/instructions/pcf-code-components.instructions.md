---
description: 'コード コンポーネントの構造と実装を理解する'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# コード コンポーネント

コード コンポーネントは solution component の一種で、solution file に含めて異なる environment にインポートできます。model-driven apps と canvas apps の両方に追加できます。

## 3 つの中核要素

コード コンポーネントは次の 3 要素で構成されます。

1. **Manifest**
2. **Component implementation**
3. **Resources**

> **注**: Power Apps component framework を使ったコード コンポーネントの定義と実装は、model-driven apps と canvas apps で共通です。違いは構成部分だけです。

## Manifest

manifest は component を定義する `ControlManifest.Input.xml` metadata file です。これは次を記述する XML document です。

- component の名前
- 構成できるデータの種類。`field` または `dataset`
- component 追加時にアプリケーション内で構成できる property
- component が必要とする resource file の一覧

### Manifest の目的

ユーザーがコード コンポーネントを構成するとき、manifest file 内のデータが利用可能な component を絞り込み、そのコンテキストで有効な component だけが構成対象になります。manifest file で定義された property は構成列としてレンダリングされ、ユーザーは値を指定できます。これらの property 値は、実行時に component から利用可能になります。

詳細情報: [Manifest schema reference](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/)

## Component Implementation

コード コンポーネントは TypeScript を使って実装します。各コード コンポーネントには、code component interface で説明されている method を実装する object を含める必要があります。[Power Platform CLI](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction) は `pac pcf init` command を使うと、stub 実装を含む `index.ts` file を自動生成します。

### 必須メソッド

component object は次の lifecycle method を実装します。

- **init** (必須) - page の読み込み時に呼び出される
- **updateView** (必須) - app のデータ変更時に呼び出される
- **getOutputs** (任意) - ユーザーがデータを変更したときに値を返す
- **destroy** (必須) - page を閉じるときに呼び出される

### コンポーネントのライフサイクル

#### Page Load

page の読み込み時、application は manifest のデータを使って object を作成します。

```typescript
var obj = new <"namespace on manifest">.<"constructor on manifest">();
```

例:
```typescript
var controlObj = new SampleNameSpace.LinearInputComponent();
```

次に、page は component を初期化します。

```typescript
controlObj.init(context, notifyOutputChanged, state, container);
```

**Init Parameters:**

| Parameter | Description |
|-----------|-------------|
| `context` | component の構成方法に関するすべての情報と、すべての parameter を含みます。入力 property には `context.parameters.<property name from manifest>` でアクセスします。Power Apps component framework API も含まれます。 |
| `notifyOutputChanged` | component に新しい output が用意でき、非同期に取得可能であることを framework に通知します。 |
| `state` | `setControlState` method を使って明示的に保存していた場合に、前回 page load 時の component データを含みます。 |
| `container` | UI 用の HTML 要素を開発者が追加できる HTML div element です。 |

#### ユーザーがデータを変更したとき

ユーザーが component を操作してデータを変更したら、`init` method で渡された `notifyOutputChanged` method を呼び出します。platform はこれに応答して `getOutputs` method を呼び出し、ユーザーが加えた変更を含む値を返します。`field` component の場合、通常これは新しい値です。

#### App がデータを変更したとき

platform がデータを変更すると、component の `updateView` method を呼び出し、新しい context object を parameter として渡します。この method は、component に表示される値を更新するよう実装する必要があります。

#### Page Close

ユーザーが page から離れると、コード コンポーネントはスコープを失い、object に割り当てられていたメモリはすべて解放されます。ただし、一部の method (event handler など) は browser 実装によっては残り、メモリを消費し続ける可能性があります。

**ベスト プラクティス:**
- 同じ session 内で次回利用する情報を保存するため、`setControlState` method を実装する
- page を閉じるときに event handler などの cleanup code を除去するため、`destroy` method を実装する

## Resources

manifest file の resource node は、component が visualization を実装するために必要な resource を参照します。各コード コンポーネントには、visualization を構築するための resource file が必要です。tooling が生成する `index.ts` file は `code` resource です。少なくとも 1 つの code resource が必要です。

### 追加リソース

manifest では、追加の resource file を定義できます。

- CSS file
- Image web resource
- localization 用の Resx web resource

詳細情報: [resources element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/resources)

## 関連リソース

- [コード コンポーネントを作成してビルドする](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf)
- [solution を使って拡張機能をパッケージ化および配布する方法を学ぶ](https://learn.microsoft.com/en-us/power-platform/alm/solution-concepts-alm)
