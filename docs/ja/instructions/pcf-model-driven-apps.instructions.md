---
description: 'モデル駆動型アプリ向けコード コンポーネントの実装と構成'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# モデル駆動型アプリ向けコード コンポーネント

Power Apps component framework により、開発者は model-driven apps の visualizations を拡張できます。プロの開発者は [Microsoft Power Platform CLI](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/get-powerapps-cli) を使って、model-driven apps にコード コンポーネントを作成、デバッグ、インポート、追加できます。

## コンポーネントの使用

model-driven apps では、以下にコード コンポーネントを追加できます。
- 列
- グリッド
- サブグリッド

> **重要**: Power Apps component framework は model-driven apps では既定で有効です。Power Apps component framework を canvas apps で有効にする方法については、[Code components for canvas apps](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/component-framework-for-canvas-apps) を参照してください。

## コード コンポーネントの実装

コード コンポーネントの作成を始める前に、Power Apps component framework を使ったコンポーネント開発に必要な [前提条件](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf#prerequisites) がすべてインストールされていることを確認してください。

[create your first code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript) の記事では、コード コンポーネントを作成する手順が段階的に説明されています。

## モデル駆動型アプリにコード コンポーネントを追加する

model-driven apps の列や table にコード コンポーネントを追加する方法については、[Add code components to model-driven apps](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/add-custom-controls-to-a-field-or-entity) を参照してください。

### 例

**リニア スライダー コントロール:**

![リニア スライダー コントロールの追加](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/media/add-slider.png)

**データセット グリッド コンポーネント:**

![データセット グリッド コンポーネント](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/add-dataset-component.png)

## 既存のコード コンポーネントを更新する

コード コンポーネントを更新して実行時に変更を確認したい場合は、manifest file の version property を更新する必要があります。

**ベスト プラクティス**: 変更を加えるたびに、常にコンポーネントの version を上げることを推奨します。

## 関連項目

- [Power Apps component framework overview](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview)
- [初めてのコード コンポーネントを作成する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript)
- [Power Apps component framework を学ぶ](https://learn.microsoft.com/en-us/training/paths/use-power-apps-component-framework)
