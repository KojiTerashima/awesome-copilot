# セットアップと構成

## NuGet パッケージ

| パッケージ | 目的 |
|---|---|
| `Microsoft.FluentUI.AspNetCore.Components` | コアコンポーネントライブラリ（必須） |
| `Microsoft.FluentUI.AspNetCore.Components.Icons` | アイコンパッケージ（任意、推奨） |
| `Microsoft.FluentUI.AspNetCore.Components.Emojis` | 絵文字パッケージ（任意） |
| `Microsoft.FluentUI.AspNetCore.Components.DataGrid.EntityFrameworkAdapter` | DataGrid 用 EF Core アダプター（任意） |
| `Microsoft.FluentUI.AspNetCore.Components.DataGrid.ODataAdapter` | DataGrid 用 OData アダプター（任意） |

## Program.cs での登録

```csharp
builder.Services.AddFluentUIComponents();
```

### 構成オプション（LibraryConfiguration）

| プロパティ | 型 | 既定値 | 備考 |
|---|---|---|---|
| `UseTooltipServiceProvider` | `bool` | `true` | `ITooltipService` を登録します。true の場合、レイアウトに `<FluentTooltipProvider>` を必ず追加する必要があります |
| `RequiredLabel` | `MarkupString` | 赤い `*` | 必須フィールド表示のカスタムマークアップ |
| `HideTooltipOnCursorLeave` | `bool` | `false` | カーソルがアンカーとツールチップの両方から離れたときにツールチップを閉じます |
| `ServiceLifetime` | `ServiceLifetime` | `Scoped` | `Scoped` または `Singleton` のみ。`Transient` は例外をスローします |
| `ValidateClassNames` | `bool` | `true` | CSS クラス名を `^-?[_a-zA-Z]+[_a-zA-Z0-9-]*$` に照らして検証します |
| `CollocatedJavaScriptQueryString` | `Func<string, string>?` | `v={version}` | JS ファイルのキャッシュバスティング |

### ホスティングモデル別 ServiceLifetime

| ホスティングモデル | ServiceLifetime |
|---|---|
| Blazor Server | `Scoped`（既定） |
| Blazor WebAssembly Standalone | `Singleton` |
| Blazor Web App（Interactive） | `Scoped`（既定） |
| Blazor Hybrid（MAUI） | `Singleton` |

## MainLayout.razor テンプレート

```razor
@inherits LayoutComponentBase

<FluentLayout>
    <FluentHeader Height="50">
        My App
    </FluentHeader>

    <FluentStack Orientation="Orientation.Horizontal" HorizontalGap="0" Style="height: 100%;">
        <FluentNavMenu Width="250" Collapsible="true" Title="Navigation">
            <FluentNavLink Href="/" Icon="@(Icons.Regular.Size20.Home)" Match="NavLinkMatch.All">Home</FluentNavLink>
            <FluentNavLink Href="/counter" Icon="@(Icons.Regular.Size20.NumberSymbol)">Counter</FluentNavLink>
            <FluentNavGroup Title="Settings" Icon="@(Icons.Regular.Size20.Settings)">
                <FluentNavLink Href="/settings/general">General</FluentNavLink>
                <FluentNavLink Href="/settings/profile">Profile</FluentNavLink>
            </FluentNavGroup>
        </FluentNavMenu>

        <FluentBodyContent>
            <FluentStack Orientation="Orientation.Vertical" Style="padding: 1rem;">
                @Body
            </FluentStack>
        </FluentBodyContent>
    </FluentStack>
</FluentLayout>

@* 必須プロバイダー — FluentLayout の後に配置 *@
<FluentToastProvider />
<FluentDialogProvider />
<FluentMessageBarProvider />
<FluentTooltipProvider />
<FluentKeyCodeProvider />

@* テーマ — ルートに配置 *@
<FluentDesignTheme Mode="DesignThemeModes.System"
                   OfficeColor="OfficeColor.Teams"
                   StorageName="mytheme" />
```

または、便利コンポーネントを使用します。

```razor
<FluentMainLayout Header="@header"
                  NavMenuContent="@navMenu"
                  Body="@body"
                  HeaderHeight="50"
                  NavMenuWidth="250"
                  NavMenuTitle="Navigation" />

@code {
    private RenderFragment header = @<span>My App</span>;
    private RenderFragment navMenu = @<div>
        <FluentNavLink Href="/">Home</FluentNavLink>
    </div>;
    private RenderFragment body = @<div>@Body</div>;
}
```

## _Imports.razor

以下を `_Imports.razor` に追加してください。

```razor
@using Microsoft.FluentUI.AspNetCore.Components
@using Icons = Microsoft.FluentUI.AspNetCore.Components.Icons
```

## 静的 Web アセット

手動で `<link>` や `<script>` タグを追加する必要はありません。ライブラリは次を使用します。
- **CSS**: `reboot.css`（正規化）+ コンポーネントスコープ CSS — 静的 Web アセット経由で自動読み込み
- **JS**: `lib.module.js` — Blazor の JS 初期化システム経由で自動読み込み
- コンポーネント固有 JS（例: DataGrid、Autocomplete）— 必要時に遅延読み込み

すべて `_content/Microsoft.FluentUI.AspNetCore.Components/` から配信されます。

## 登録されるサービス

`AddFluentUIComponents()` によって自動登録されるサービス:

| サービス | 実装 | 目的 |
|---|---|---|
| `GlobalState` | `GlobalState` | 共有アプリケーション状態 |
| `IToastService` | `ToastService` | トースト通知（`FluentToastProvider` が必要） |
| `IDialogService` | `DialogService` | ダイアログとパネル（`FluentDialogProvider` が必要） |
| `IMessageService` | `MessageService` | メッセージバー（`FluentMessageBarProvider` が必要） |
| `IKeyCodeService` | `KeyCodeService` | キーボードショートカット（`FluentKeyCodeProvider` が必要） |
| `IMenuService` | `MenuService` | コンテキストメニュー |
| `ITooltipService` | `TooltipService` | ツールチップ（`FluentTooltipProvider` が必要、`UseTooltipServiceProvider` で有効化） |
