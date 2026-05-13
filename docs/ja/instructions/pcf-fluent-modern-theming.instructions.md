---
description: 'Fluent UI を使ったモダン テーマ設定でコンポーネントにスタイルを適用する'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# モダン テーマ設定でコンポーネントにスタイルを適用する (Preview)

[このトピックはプレリリース ドキュメントであり、変更される可能性があります。]

開発者は、自分の component を、それが組み込まれているアプリケーションの他の部分と同じ見た目になるようスタイル設定できる必要があります。これは、canvas app では [Modern controls and themes](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/modern-controls/overview-modern-controls) 機能、model-driven app では [new refreshed look](https://learn.microsoft.com/en-us/power-apps/user/modern-fluent-design) を通じて、modern theming が有効になっている場合に実現できます。

[Fluent UI React v9](https://react.fluentui.dev/) を基盤とする modern theming を使って component をスタイル設定してください。この方法は、component にとって最良のパフォーマンスと theming 体験を得るための推奨アプローチです。

## モダン テーマ設定を適用する 4 つの方法

1. **Fluent UI v9 controls**
2. **Fluent UI v8 controls**
3. **Non-Fluent UI controls**
4. **Custom theme providers**

## Fluent UI v9 Controls

Fluent UI v9 control を component としてラップするのは、modern theme が自動的にこれらの control に適用されるため、modern theming を活用する最も簡単な方法です。唯一の前提条件は、component が [React controls & platform libraries](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/react-controls-platform-libraries) への依存関係を追加していることを確認することです。

このアプローチにより、component は platform と同じ React と Fluent library を使えるため、theme token を component に渡す同じ React context を共有できます。

```xml
<resources>
  <code path="index.ts" order="1"/>
  <!-- Dependency on React controls & platform libraries -->
  <platform-library name="React" version="16.14.0" />
  <platform-library name="Fluent" version="9.46.2" />
</resources>
```

## Fluent UI v8 Controls

Fluent には、component で Fluent UI v8 control を使う場合に v9 theme 構造を適用するための migration path が用意されています。[Fluent の v8 から v9 への migration package](https://www.npmjs.com/package/@fluentui/react-migration-v8-v9) に含まれる `createV8Theme` 関数を使って、v9 theme token に基づく v8 theme を作成します。

```typescript
const theme = createV8Theme(
  context.fluentDesignLanguage.brand,
  context.fluentDesignLanguage.theme
);
return <ThemeProvider theme={theme}></ThemeProvider>;
```

## Non-Fluent UI Controls

component が Fluent UI を使わない場合は、`fluentDesignLanguage` context parameter から利用可能な v9 theme token に直接依存できます。この parameter を使うと、[theme](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/theming) token すべてにアクセスでき、theme のあらゆる側面を参照して自分自身をスタイル設定できます。

```typescript
<span style={{ fontSize: context.fluentDesignLanguage.theme.fontSizeBase300 }}>
  {"Stylizing HTML with platform provided theme."}
</span>
```

## Custom Theme Providers

component が app の現在の theme と異なるスタイルを必要とする場合は、独自の `FluentProvider` を作成し、自分の component で使う独自の theme token 群を渡します。

```typescript
<FluentProvider theme={context.fluentDesignLanguage.tokenTheme}>
  {/* your control */}
</FluentProvider>
```

## サンプル コントロール

これら各ユース ケースの例は、[Modern Theming API control](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/sample-controls/modern-theming-api-control) で確認できます。

## FAQ

### Q: 自分の control は Fluent UI v9 を使い platform library に依存していますが、modern theming は使いたくありません。component で無効化するにはどうすればよいですか?

A: これには 2 つの方法があります。

**Option 1**: 独自の component-level `FluentProvider` を作成する

```typescript
<FluentProvider theme={customFluentV9Theme}>
  {/* your control */}
</FluentProvider>
```

**Option 2**: control を `IdPrefixContext.Provider` でラップし、独自の `idPrefix` 値を設定します。これにより、component が platform から theme token を受け取らないようにできます。

```typescript
<IdPrefixProvider value="custom-control-prefix">
  <Label weight="semibold">This label is not getting Modern Theming</Label>
</IdPrefixProvider>
```

### Q: 一部の Fluent UI v9 control に style が適用されません

A: React Portal に依存する Fluent v9 control は、style が正しく適用されるよう theme provider で再度ラップする必要があります。`FluentProvider` を使用できます。

### Q: modern theming が有効かどうかはどう確認できますか?

A: token が利用可能かどうかを確認できます: `context.fluentDesignLanguage?.tokenTheme`。また、model-driven application では app setting を確認できます: `context.appSettings.getIsFluentThemingEnabled()`。

## 関連記事

- [Theming (Power Apps component framework API reference)](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/theming)
- [Modern Theming API control](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/sample-controls/modern-theming-api-control)
- [canvas apps で modern theme を使う (preview)](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/modern-controls/modern-theming)
