# テーマ設定

## FluentDesignTheme（推奨）

主要なテーマ設定コンポーネントです。アプリのルートに配置します。

```razor
<FluentDesignTheme Mode="DesignThemeModes.System"
                   OfficeColor="OfficeColor.Teams"
                   StorageName="mytheme" />
```

### パラメーター

| Parameter | Type | Default | Description |
|---|---|---|---|
| `Mode` | `DesignThemeModes` | `System` | `Light`、`Dark`、または `System`（OS に追従） |
| `CustomColor` | `string?` | null | 16進のアクセントカラー（例: `"#0078D4"`） |
| `OfficeColor` | `OfficeColor?` | null | プリセットアクセント: `Teams`、`Word`、`Excel`、`PowerPoint`、`Outlook`、`OneNote` |
| `NeutralBaseColor` | `string?` | null | ニュートラルパレットのベース16進カラー |
| `StorageName` | `string?` | null | このキーでテーマを localStorage に永続化します |
| `Direction` | `LocalizationDirection?` | null | `Ltr` または `Rtl` |
| `OnLuminanceChanged` | `EventCallback<LuminanceChangedEventArgs>` | | ダーク/ライトモードが変更されたときに発火 |
| `OnLoaded` | `EventCallback<LoadedEventArgs>` | | ストレージからテーマが読み込まれたときに発火 |

### 双方向バインディング

```razor
<FluentDesignTheme @bind-Mode="@themeMode"
                   @bind-OfficeColor="@officeColor"
                   @bind-CustomColor="@customColor"
                   StorageName="mytheme" />

<FluentSelect Items="@(Enum.GetValues<DesignThemeModes>())"
              @bind-SelectedOption="@themeMode"
              OptionText="@(m => m.ToString())" />

@code {
    private DesignThemeModes themeMode = DesignThemeModes.System;
    private OfficeColor? officeColor = OfficeColor.Teams;
    private string? customColor;
}
```

### 重要: JS interop 依存

`FluentDesignTheme` は内部で JavaScript interop を使用します。サーバーサイドのプリレンダリング中は動作しません。テーマ変更に反応する必要がある場合:

```csharp
// OnInitialized ではなく OnAfterRenderAsync を使用する
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        // ここで design token と安全にやり取りできる
    }
}
```

## FluentDesignSystemProvider（上級）

コンポーネントツリーのサブツリーに design token をスコープ適用するために使います。50 以上の CSS カスタムプロパティを提供します。

```razor
<FluentDesignSystemProvider AccentBaseColor="#0078D4"
                            NeutralBaseColor="#808080"
                            BaseLayerLuminance="0.95">
    <FluentButton Appearance="Appearance.Accent">テーマ適用ボタン</FluentButton>
</FluentDesignSystemProvider>
```

## Design Token Classes（DI ベース、上級）

依存性注入を通じて、プログラムから token を制御するために使います。各 token は生成されたサービスです。

```csharp
@inject AccentBaseColor AccentBaseColor

protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        // 特定の要素に対して token を設定
        await AccentBaseColor.SetValueFor(myElement, "#FF0000".ToSwatch());

        // token の値を読み取り
        var currentColor = await AccentBaseColor.GetValueFor(myElement);

        // 上書きを削除
        await AccentBaseColor.DeleteValueFor(myElement);
    }
}
```

## 利用可能な DesignThemeModes

- `DesignThemeModes.Light` — ライトテーマ
- `DesignThemeModes.Dark` — ダークテーマ
- `DesignThemeModes.System` — OS の設定に追従

## 利用可能な OfficeColor プリセット

`Teams`, `Word`, `Excel`, `PowerPoint`, `Outlook`, `OneNote`, `Loop`, `Planner`, `SharePoint`, `Stream`, `Sway`, `Viva`, `VivaEngage`, `VivaInsights`, `VivaLearning`, `VivaTopics`.
