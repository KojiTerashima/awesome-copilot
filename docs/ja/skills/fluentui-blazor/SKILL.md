---
name: fluentui-blazor
description: >
  Blazor アプリケーションで Microsoft Fluent UI Blazor コンポーネントライブラリ
  （Microsoft.FluentUI.AspNetCore.Components NuGet パッケージ）を使用するためのガイド。
  ユーザーが Fluent UI コンポーネントを使って Blazor アプリを構築しているとき、
  ライブラリをセットアップしているとき、FluentButton、FluentDataGrid、
  FluentDialog、FluentToast、FluentNavMenu、FluentTextField、FluentSelect、
  FluentAutocomplete、FluentDesignTheme、または "Fluent" プレフィックスの任意のコンポーネントを
  使用しているときに使ってください。
  また、プロバイダー不足、JS interop の問題、テーマ設定のトラブルシューティング時にも使用してください。
---

# Fluent UI Blazor — 利用者向け使用ガイド

このスキルでは、Blazor アプリケーションで **Microsoft.FluentUI.AspNetCore.Components**（バージョン 4）NuGet パッケージを正しく使用する方法を説明します。

## 重要ルール

### 1. 手動の `<script>` または `<link>` タグは不要

このライブラリは、Blazor の static web assets と JS initializers を通じて、すべての CSS と JS を自動読み込みします。**コアライブラリ用に `<script>` や `<link>` タグを追加するようユーザーに案内しないでください。**

### 2. サービスベースのコンポーネントにはプロバイダーが必須

対応するサービスを動作させるには、これらのプロバイダーコンポーネントをルートレイアウト（例: `MainLayout.razor`）に**必ず**追加する必要があります。これがないと、サービス呼び出しは**サイレントに失敗**します（エラーなし、UI 変化なし）。

```razor
<FluentToastProvider />
<FluentDialogProvider />
<FluentMessageBarProvider />
<FluentTooltipProvider />
<FluentKeyCodeProvider />
```

### 3. Program.cs でのサービス登録

```csharp
builder.Services.AddFluentUIComponents();

// または設定付き:
builder.Services.AddFluentUIComponents(options =>
{
    options.UseTooltipServiceProvider = true;  // 既定: true
    options.ServiceLifetime = ServiceLifetime.Scoped; // 既定
});
```

**ServiceLifetime のルール:**
- `ServiceLifetime.Scoped` — Blazor Server / Interactive 向け（既定）
- `ServiceLifetime.Singleton` — Blazor WebAssembly standalone 向け
- `ServiceLifetime.Transient` — **`NotSupportedException` をスロー**

### 4. アイコンには別の NuGet パッケージが必要

```
dotnet add package Microsoft.FluentUI.AspNetCore.Components.Icons
```

`@using` エイリアスを使った使用例:

```razor
@using Icons = Microsoft.FluentUI.AspNetCore.Components.Icons

<FluentIcon Value="@(Icons.Regular.Size24.Save)" />
<FluentIcon Value="@(Icons.Filled.Size20.Delete)" Color="@Color.Error" />
```

パターン: `Icons.[Variant].[Size].[Name]`
- バリアント: `Regular`, `Filled`
- サイズ: `Size12`, `Size16`, `Size20`, `Size24`, `Size28`, `Size32`, `Size48`

カスタム画像: `Icon.FromImageUrl("/path/to/image.png")`

**文字列ベースのアイコン名は絶対に使わないでください** — アイコンは強い型付けのクラスです。

### 5. リストコンポーネントのバインディングモデル

`FluentSelect<TOption>`、`FluentCombobox<TOption>`、`FluentListbox<TOption>`、`FluentAutocomplete<TOption>` は `<InputSelect>` と同じようには動作しません。以下を使います:

- `Items` — データソース（`IEnumerable<TOption>`）
- `OptionText` — 表示テキストを取り出す `Func<TOption, string?>`
- `OptionValue` — 値文字列を取り出す `Func<TOption, string?>`
- `SelectedOption` / `SelectedOptionChanged` — 単一選択バインディング用
- `SelectedOptions` / `SelectedOptionsChanged` — 複数選択バインディング用

```razor
<FluentSelect Items="@countries"
              OptionText="@(c => c.Name)"
              OptionValue="@(c => c.Code)"
              @bind-SelectedOption="@selectedCountry"
              Label="Country" />
```

**以下のようにはしません**（誤ったパターン）:
```razor
@* WRONG — InputSelect パターンは使わない *@
<FluentSelect @bind-Value="@selectedValue">
    <option value="1">One</option>
</FluentSelect>
```

### 6. FluentAutocomplete の注意点

- 検索入力テキストには `ValueText` を使用（`Value` は非推奨）
- オプションを絞り込む必須コールバックは `OnOptionsSearch`
- 既定値は `Multiple="true"`

```razor
<FluentAutocomplete TOption="Person"
                    OnOptionsSearch="@OnSearch"
                    OptionText="@(p => p.FullName)"
                    @bind-SelectedOptions="@selectedPeople"
                    Label="Search people" />

@code {
    private void OnSearch(OptionsSearchEventArgs<Person> args)
    {
        args.Items = allPeople.Where(p =>
            p.FullName.Contains(args.Text, StringComparison.OrdinalIgnoreCase));
    }
}
```

### 7. Dialog サービスパターン

**`<FluentDialog>` タグの可視性をトグルしないでください。** サービスパターンは次のとおりです:

1. `IDialogContentComponent<TData>` を実装するコンテンツコンポーネントを作成:

```csharp
public partial class EditPersonDialog : IDialogContentComponent<Person>
{
    [Parameter] public Person Content { get; set; } = default!;

    [CascadingParameter] public FluentDialog Dialog { get; set; } = default!;

    private async Task SaveAsync()
    {
        await Dialog.CloseAsync(Content);
    }

    private async Task CancelAsync()
    {
        await Dialog.CancelAsync();
    }
}
```

2. `IDialogService` 経由でダイアログを表示:

```csharp
[Inject] private IDialogService DialogService { get; set; } = default!;

private async Task ShowEditDialog()
{
    var dialog = await DialogService.ShowDialogAsync<EditPersonDialog, Person>(
        person,
        new DialogParameters
        {
            Title = "Edit Person",
            PrimaryAction = "Save",
            SecondaryAction = "Cancel",
            Width = "500px",
            PreventDismissOnOverlayClick = true,
        });

    var result = await dialog.Result;
    if (!result.Cancelled)
    {
        var updatedPerson = result.Data as Person;
    }
}
```

簡易ダイアログの場合:
```csharp
await DialogService.ShowConfirmationAsync("Are you sure?", "Yes", "No");
await DialogService.ShowSuccessAsync("Done!");
await DialogService.ShowErrorAsync("Something went wrong.");
```

### 8. Toast 通知

```csharp
[Inject] private IToastService ToastService { get; set; } = default!;

ToastService.ShowSuccess("Item saved successfully");
ToastService.ShowError("Failed to save");
ToastService.ShowWarning("Check your input");
ToastService.ShowInfo("New update available");
```

`FluentToastProvider` のパラメーター: `Position`（既定 `TopRight`）、`Timeout`（既定 7000ms）、`MaxToastCount`（既定 4）。

### 9. デザイントークンとテーマはレンダリング後のみ動作

デザイントークンは JS interop に依存します。**`OnInitialized` では設定せず**、`OnAfterRenderAsync` を使ってください。

```razor
<FluentDesignTheme Mode="DesignThemeModes.System"
                   OfficeColor="OfficeColor.Teams"
                   StorageName="mytheme" />
```

### 10. FluentEditForm と EditForm

`FluentEditForm` が必要なのは `FluentWizard` ステップ内（ステップごとの検証）だけです。通常のフォームでは、Fluent フォームコンポーネントと標準の `EditForm` を使用します:

```razor
<EditForm Model="@model" OnValidSubmit="HandleSubmit">
    <DataAnnotationsValidator />
    <FluentTextField @bind-Value="@model.Name" Label="Name" Required />
    <FluentSelect Items="@options"
                  OptionText="@(o => o.Label)"
                  @bind-SelectedOption="@model.Category"
                  Label="Category" />
    <FluentValidationSummary />
    <FluentButton Type="ButtonType.Submit" Appearance="Appearance.Accent">Save</FluentButton>
</EditForm>
```

Fluent スタイルの検証表示には、標準 Blazor の検証コンポーネントではなく `FluentValidationMessage` と `FluentValidationSummary` を使用します。

## 参照ファイル

特定トピックの詳細ガイダンスは次を参照してください:

- [セットアップと構成](references/SETUP.md)
- [レイアウトとナビゲーション](references/LAYOUT-AND-NAVIGATION.md)
- [データグリッド](references/DATAGRID.md)
- [テーマ設定](references/THEMING.md)
