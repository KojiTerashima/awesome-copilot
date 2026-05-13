---
description: '.NET MAUI アプリをバージョン 9 から 10 へアップグレードするための手順です。破壊的変更、非推奨 API、ListView から CollectionView への移行戦略を含みます。'
applyTo: '**/*.csproj, **/*.cs, **/*.xaml'
---

# .NET MAUI 9 から .NET MAUI 10 へのアップグレード

このガイドは、コード更新が必要になる重要な破壊的変更と obsolete API に焦点を当てて、.NET 9 から .NET 10 へ .NET MAUI アプリをアップグレードするのを支援します。

---

## 目次

1. [クイック スタート](#quick-start)
2. [Target Framework の更新](#update-target-framework)
3. [破壊的変更 (P0 - 必須対応)](#breaking-changes-p0---must-fix)
   - [MessagingCenter が internal に変更](#messagingcenter-made-internal)
   - [ListView と TableView の非推奨化](#listview-and-tableview-deprecated)
4. [非推奨 API (P1 - 早めに修正)](#deprecated-apis-p1---fix-soon)
   - [Animation Methods](#1-animation-methods)
   - [DisplayAlert と DisplayActionSheet](#2-displayalert-and-displayactionsheet)
   - [Page.IsBusy](#3-pageisbusy)
   - [MediaPicker APIs](#4-mediapicker-apis)
5. [推奨変更 (P2)](#recommended-changes-p2)
6. [一括移行ツール](#bulk-migration-tools)
7. [アップグレード後のテスト](#testing-your-upgrade)
8. [トラブルシューティング](#troubleshooting)

---

## Quick Start

**5 ステップのアップグレード手順:**

1. `TargetFramework` を `net10.0` に更新する
2. `CommunityToolkit.Maui` を 12.3.0+ に更新する (使っている場合) - 必須
3. 破壊的変更を修正する - MessagingCenter (P0)
4. `ListView` / `TableView` を `CollectionView` に移行する (P0 - 最重要)
5. 非推奨 API を修正する - Animation methods, DisplayAlert, IsBusy, MediaPicker (P1)

> ⚠️ **主な破壊的変更**:
> - `CommunityToolkit.Maui` は **必ず** 12.3.0 以降にする必要があります
> - `ListView` と `TableView` は obsolete になりました (最も大きい移行作業です)

---

## Update Target Framework

### 単一プラットフォーム

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>
</Project>
```

### マルチプラットフォーム

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0-android;net10.0-ios;net10.0-maccatalyst;net10.0-windows10.0.19041.0</TargetFrameworks>
  </PropertyGroup>
</Project>
```

### 任意: Linux 互換 (GitHub Copilot, WSL など)

> 💡 **Linux 開発向け**: Linux 上で build する場合 (例: GitHub Codespaces, WSL, GitHub Copilot 利用時) は、iOS/Mac Catalyst target を条件付きで除外することで Linux 上でも compile 可能にできます。

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!-- Android から開始 (常にサポート) -->
    <TargetFrameworks>net10.0-android</TargetFrameworks>

    <!-- Linux 以外でのみ iOS/Mac Catalyst を追加 -->
    <TargetFrameworks Condition="!$([MSBuild]::IsOSPlatform('linux'))">$(TargetFrameworks);net10.0-ios;net10.0-maccatalyst</TargetFrameworks>

    <!-- Windows のときだけ Windows target を追加 -->
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net10.0-windows10.0.19041.0</TargetFrameworks>
  </PropertyGroup>
</Project>
```

**利点:**
- ✅ Linux でも正常に compile できる (iOS/Mac 用ツール不要)
- ✅ GitHub Codespaces と Copilot で使いやすい
- ✅ build OS に応じて正しい target が自動で含まれる
- ✅ OS を切り替えても project 変更が不要

**Reference:** [dotnet/maui#32186](https://github.com/dotnet/maui/pull/32186)

### 必須 NuGet package の更新

> ⚠️ **重要**: `CommunityToolkit.Maui` を使っている場合は、**必ず** 12.3.0 以降へ更新してください。古い version は .NET 10 と互換性がなく、compile error を引き起こします。

```bash
# CommunityToolkit.Maui を更新 (使用している場合)
dotnet add package CommunityToolkit.Maui --version 12.3.0

# 他の一般的な package も .NET 10 対応版へ更新
dotnet add package Microsoft.Maui.Controls --version 10.0.0
```

**すべての NuGet package を確認する:**
```bash
# すべての package を列挙し更新候補を確認
dotnet list package --outdated

# 互換な最新版へ一括更新
dotnet list package --outdated | grep ">" | cut -d '>' -f 1 | xargs -I {} dotnet add package {}
```

---

## Breaking Changes (P0 - Must Fix)

### MessagingCenter Made Internal

**Status:** 🚨 **破壊的変更** - `MessagingCenter` は `internal` になり、アクセスできません。

**表示されるエラー:**
```
error CS0122: 'MessagingCenter' is inaccessible due to its protection level
```

**必要な移行:**

#### Step 1: CommunityToolkit.Mvvm を install する

```bash
dotnet add package CommunityToolkit.Mvvm --version 8.3.0
```

#### Step 2: Message class を定義する

```csharp
// OLD: message class は不要
MessagingCenter.Send(this, "UserLoggedIn", userData);

// NEW: message class を作成
public class UserLoggedInMessage
{
    public UserData Data { get; set; }

    public UserLoggedInMessage(UserData data)
    {
        Data = data;
    }
}
```

#### Step 3: Send 呼び出しを更新する

```csharp
// ❌ OLD (.NET 10 では壊れる)
using Microsoft.Maui.Controls;

MessagingCenter.Send(this, "UserLoggedIn", userData);
MessagingCenter.Send<App, string>(this, "StatusChanged", "Active");

// ✅ NEW (必須)
using CommunityToolkit.Mvvm.Messaging;

WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(userData));
WeakReferenceMessenger.Default.Send(new StatusChangedMessage("Active"));
```

#### Step 4: Subscribe 呼び出しを更新する

```csharp
// ❌ OLD (.NET 10 では壊れる)
MessagingCenter.Subscribe<App, UserData>(this, "UserLoggedIn", (sender, data) =>
{
    // message を処理
    CurrentUser = data;
});

// ✅ NEW (必須)
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (recipient, message) =>
{
    // message を処理
    CurrentUser = message.Data;
});
```

#### ⚠️ 重要な挙動差: 重複 subscribe

**WeakReferenceMessenger** は、同じ recipient に対して同じ message type を複数回 register しようとすると `InvalidOperationException` を送出します (MessagingCenter では許可されていました)。

```csharp
// ❌ WeakReferenceMessenger では InvalidOperationException が発生
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler1(m));
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler2(m)); // ❌ 例外!

// ✅ Solution 1: 再登録前に解除する
WeakReferenceMessenger.Default.Unregister<UserLoggedInMessage>(this);
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler1(m));

// ✅ Solution 2: 1 つの登録で複数処理をまとめる
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) =>
{
    Handler1(m);
    Handler2(m);
});
```

**重要な理由:** 同じ message を複数箇所で subscribe している場合 (例: page constructor と `OnAppearing`) は、実行時 crash になります。

#### Step 5: 終わったら unregister する

```csharp
// ❌ OLD
MessagingCenter.Unsubscribe<App, UserData>(this, "UserLoggedIn");

// ✅ NEW (重要 - memory leak 防止)
WeakReferenceMessenger.Default.Unregister<UserLoggedInMessage>(this);

// または、この recipient 向けの全 message を解除
WeakReferenceMessenger.Default.UnregisterAll(this);
```

#### 完全な Before / After 例

**Before (.NET 9):**
```csharp
// Sender
public class LoginViewModel
{
    public async Task LoginAsync()
    {
        var user = await AuthService.LoginAsync(username, password);
        MessagingCenter.Send(this, "UserLoggedIn", user);
    }
}

// Receiver
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();

        MessagingCenter.Subscribe<LoginViewModel, User>(this, "UserLoggedIn", (sender, user) =>
        {
            WelcomeLabel.Text = $"Welcome, {user.Name}!";
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        MessagingCenter.Unsubscribe<LoginViewModel, User>(this, "UserLoggedIn");
    }
}
```

**After (.NET 10):**
```csharp
// 1. message を定義
public class UserLoggedInMessage
{
    public User User { get; }

    public UserLoggedInMessage(User user)
    {
        User = user;
    }
}

// 2. Sender
public class LoginViewModel
{
    public async Task LoginAsync()
    {
        var user = await AuthService.LoginAsync(username, password);
        WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(user));
    }
}

// 3. Receiver
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();

        WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (recipient, message) =>
        {
            WelcomeLabel.Text = $"Welcome, {message.User.Name}!";
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        WeakReferenceMessenger.Default.UnregisterAll(this);
    }
}
```

**主な違い:**
- ✅ 型安全な message class
- ✅ magic string が不要
- ✅ IntelliSense が改善される
- ✅ refactor しやすい
- ⚠️ **unregister を忘れないこと!**

---

### ListView and TableView Deprecated

**Status:** 🚨 **非推奨 (P0)** - `ListView`, `TableView`, およびすべての Cell type は obsolete です。`CollectionView` へ移行してください。

**表示される warning:**
```
warning CS0618: 'ListView' is obsolete: 'ListView is deprecated. Please use CollectionView instead.'
warning CS0618: 'TableView' is obsolete: 'Please use CollectionView instead.'
warning CS0618: 'TextCell' is obsolete: 'The controls which use TextCell (ListView and TableView) are obsolete. Please use CollectionView instead.'
```

**obsolete になる型:**
- `ListView` → `CollectionView`
- `TableView` → `CollectionView` (設定画面なら vertical StackLayout + BindableLayout も検討)
- `TextCell` → Label を使った custom DataTemplate
- `ImageCell` → Image + Label の custom DataTemplate
- `EntryCell` → Entry を含む custom DataTemplate
- `SwitchCell` → Switch を含む custom DataTemplate
- `ViewCell` → DataTemplate

**影響:** これは **大きな** 破壊的変更です。`ListView` と `TableView` は MAUI アプリで最もよく使われる control の 1 つです。

#### なぜ時間がかかるのか

`ListView` / `TableView` から `CollectionView` への変換は、単純な find-replace ではありません。

1. **event model が異なる** - `ItemSelected` → `SelectionChanged`、引数も違う
2. **grouping 方法が異なる** - `GroupDisplayBinding` は存在しない
3. **context action** - `SwipeView` に変換が必要
4. **item sizing** - `HasUnevenRows` の扱いが異なる
5. **platform-specific code** - iOS/Android の ListView 専用設定を除去する必要がある
6. **テストが必要** - `CollectionView` は virtualization の挙動が違い、性能に影響し得る

#### 移行戦略

**Step 1: ListView を棚卸しする**

```bash
# すべての ListView/TableView 利用箇所を探す
grep -r "ListView\|TableView" --include="*.xaml" --include="*.cs" .
```

**Step 2: 基本的な ListView → CollectionView**

**Before (ListView):**
```xaml
<ListView ItemsSource="{Binding Items}"
          ItemSelected="OnItemSelected"
          HasUnevenRows="True">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextCell Text="{Binding Title}"
                     Detail="{Binding Description}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**After (CollectionView):**
```xaml
<CollectionView ItemsSource="{Binding Items}"
                SelectionMode="Single"
                SelectionChanged="OnSelectionChanged">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <VerticalStackLayout Padding="10">
                <Label Text="{Binding Title}"
                       FontAttributes="Bold" />
                <Label Text="{Binding Description}"
                       FontSize="12"
                       TextColor="{StaticResource Gray600}" />
            </VerticalStackLayout>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

> ⚠️ **Note:** `CollectionView` の既定 `SelectionMode` は `None` (選択無効) です。選択を有効にするには `SelectionMode="Single"` または `SelectionMode="Multiple"` を明示してください。

**Code-behind の変更:**
```csharp
// ❌ OLD (ListView)
void OnItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    if (e.SelectedItem == null)
        return;

    var item = (MyItem)e.SelectedItem;
    // 選択処理

    // 選択解除
    ((ListView)sender).SelectedItem = null;
}

// ✅ NEW (CollectionView)
void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (e.CurrentSelection.Count == 0)
        return;

    var item = (MyItem)e.CurrentSelection.FirstOrDefault();
    // 選択処理

    // 選択解除 (任意)
    ((CollectionView)sender).SelectedItem = null;
}
```

**Step 3: Grouped ListView → Grouped CollectionView**

**Before (Grouped ListView):**
```xaml
<ListView ItemsSource="{Binding GroupedItems}"
          IsGroupingEnabled="True"
          GroupDisplayBinding="{Binding Key}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextCell Text="{Binding Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**After (Grouped CollectionView):**
```xaml
<CollectionView ItemsSource="{Binding GroupedItems}"
                IsGrouped="true">
    <CollectionView.GroupHeaderTemplate>
        <DataTemplate>
            <Label Text="{Binding Key}"
                   FontAttributes="Bold"
                   BackgroundColor="{StaticResource Gray100}"
                   Padding="10,5" />
        </DataTemplate>
    </CollectionView.GroupHeaderTemplate>

    <CollectionView.ItemTemplate>
        <DataTemplate>
            <VerticalStackLayout Padding="20,10">
                <Label Text="{Binding Name}" />
            </VerticalStackLayout>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

**Step 4: Context Actions → SwipeView**

> ⚠️ **Platform Note:** `SwipeView` は touch input 前提です。Windows desktop では、touch screen 以外 (mouse/trackpad) では機能しません。desktop 向けには button や right-click menu など代替 UI を検討してください。

**Before (ContextActions 付き ListView):**
```xaml
<ListView.ItemTemplate>
    <DataTemplate>
        <ViewCell>
            <ViewCell.ContextActions>
                <MenuItem Text="Delete"
                         IsDestructive="True"
                         Command="{Binding Source={RelativeSource AncestorType={x:Type local:MyPage}}, Path=DeleteCommand}"
                         CommandParameter="{Binding .}" />
            </ViewCell.ContextActions>

            <Label Text="{Binding Title}" Padding="10" />
        </ViewCell>
    </DataTemplate>
</ListView.ItemTemplate>
```

**After (SwipeView 付き CollectionView):**
```xaml
<CollectionView.ItemTemplate>
    <DataTemplate>
        <SwipeView>
            <SwipeView.RightItems>
                <SwipeItems>
                    <SwipeItem Text="Delete"
                              BackgroundColor="Red"
                              Command="{Binding Source={RelativeSource AncestorType={x:Type local:MyPage}}, Path=DeleteCommand}"
                              CommandParameter="{Binding .}" />
                </SwipeItems>
            </SwipeView.RightItems>

            <VerticalStackLayout Padding="10">
                <Label Text="{Binding Title}" />
            </VerticalStackLayout>
        </SwipeView>
    </DataTemplate>
</CollectionView.ItemTemplate>
```

**Step 5: 設定画面向け TableView → 代替アプローチ**

`TableView` は設定画面でよく使われます。現代的な代替案は次の通りです。

**Option 1: Grouped Data を使う CollectionView**
```xaml
<CollectionView ItemsSource="{Binding SettingGroups}"
                IsGrouped="true"
                SelectionMode="None">
    <CollectionView.GroupHeaderTemplate>
        <DataTemplate>
            <Label Text="{Binding Title}"
                   FontAttributes="Bold"
                   Margin="10,15,10,5" />
        </DataTemplate>
    </CollectionView.GroupHeaderTemplate>

    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Grid Padding="15,10" ColumnDefinitions="*,Auto">
                <Label Text="{Binding Title}"
                       VerticalOptions="Center" />
                <Switch Grid.Column="1"
                        IsToggled="{Binding IsEnabled}"
                        IsVisible="{Binding ShowSwitch}" />
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

**Option 2: Vertical StackLayout (項目数が少ない設定画面向け)**
```xaml
<ScrollView>
    <VerticalStackLayout BindableLayout.ItemsSource="{Binding Settings}"
                        Spacing="10"
                        Padding="15">
        <BindableLayout.ItemTemplate>
            <DataTemplate>
                <Border StrokeThickness="0"
                       BackgroundColor="{StaticResource Gray100}"
                       Padding="15,10">
                    <Grid ColumnDefinitions="*,Auto">
                        <Label Text="{Binding Title}"
                              VerticalOptions="Center" />
                        <Switch Grid.Column="1"
                               IsToggled="{Binding IsEnabled}" />
                    </Grid>
                </Border>
            </DataTemplate>
        </BindableLayout.ItemTemplate>
    </VerticalStackLayout>
</ScrollView>
```

**Step 6: platform-specific な ListView code を除去する**

ListView 専用の platform-specific 機能を使っていた場合は削除します。

```csharp
// ❌ OLD - これらの using は削除 ( .NET 10 では obsolete )
using Microsoft.Maui.Controls.PlatformConfiguration;
using Microsoft.Maui.Controls.PlatformConfiguration.iOSSpecific;
using Microsoft.Maui.Controls.PlatformConfiguration.AndroidSpecific;

// ❌ OLD - ListView の platform 設定を削除 (.NET 10 では obsolete)
myListView.On<iOS>().SetSeparatorStyle(SeparatorStyle.FullWidth);
myListView.On<Android>().IsFastScrollEnabled();

// ❌ OLD - Cell の platform 設定を削除 (.NET 10 では obsolete)
viewCell.On<iOS>().SetDefaultBackgroundColor(Colors.White);
viewCell.On<Android>().SetIsContextActionsLegacyModeEnabled(false);
```

**Migration:** `CollectionView` には同じ形の platform-specific 設定はありません。platform 固有の styling が必要なら:

```csharp
// ✅ NEW - 条件付きコンパイルを使う
#if IOS
var backgroundColor = Colors.White;
#elif ANDROID
var backgroundColor = Colors.Transparent;
#endif

var grid = new Grid
{
    BackgroundColor = backgroundColor,
    // ... cell content の残り
};
```

または XAML で:
```xaml
<CollectionView.ItemTemplate>
    <DataTemplate>
        <Grid>
            <Grid.BackgroundColor>
                <OnPlatform x:TypeArguments="Color">
                    <On Platform="iOS" Value="White" />
                    <On Platform="Android" Value="Transparent" />
                </OnPlatform>
            </Grid.BackgroundColor>
            <!-- Cell content -->
        </Grid>
    </DataTemplate>
</CollectionView.ItemTemplate>
```

#### よくあるパターンと落とし穴

**1. Empty View**
```xaml
<!-- CollectionView には組み込みの EmptyView がある -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.EmptyView>
        <ContentView>
            <VerticalStackLayout Padding="50" VerticalOptions="Center">
                <Label Text="No items found"
                       HorizontalTextAlignment="Center" />
            </VerticalStackLayout>
        </ContentView>
    </CollectionView.EmptyView>
    <!-- ... -->
</CollectionView>
```

**2. Pull to Refresh**
```xaml
<RefreshView IsRefreshing="{Binding IsRefreshing}"
             Command="{Binding RefreshCommand}">
    <CollectionView ItemsSource="{Binding Items}">
        <!-- ... -->
    </CollectionView>
</RefreshView>
```

**3. Item Spacing**
```xaml
<!-- spacing には ItemsLayout を使う -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.ItemsLayout>
        <LinearItemsLayout Orientation="Vertical"
                          ItemSpacing="10" />
    </CollectionView.ItemsLayout>
    <!-- ... -->
</CollectionView>
```

**4. Header と Footer**
```xaml
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.Header>
        <Label Text="My List"
               FontSize="24"
               Padding="10" />
    </CollectionView.Header>

    <CollectionView.Footer>
        <Label Text="End of list"
               Padding="10"
               HorizontalTextAlignment="Center" />
    </CollectionView.Footer>

    <!-- ItemTemplate -->
</CollectionView>
```

**5. Load More / Infinite Scroll**
```xaml
<CollectionView ItemsSource="{Binding Items}"
                RemainingItemsThreshold="5"
                RemainingItemsThresholdReachedCommand="{Binding LoadMoreCommand}">
    <!-- ItemTemplate -->
</CollectionView>
```

**6. Item Sizing の最適化**

`CollectionView` は `ItemSizingStrategy` で item の計測方法を制御します。

```xaml
<!-- 既定: 各 item を個別計測 (HasUnevenRows="True" に近い) -->
<CollectionView ItemSizingStrategy="MeasureAllItems">
    <!-- ... -->
</CollectionView>

<!-- 高速化: 最初の item だけ計測し、残りは同じ高さを使う -->
<CollectionView ItemSizingStrategy="MeasureFirstItem">
    <!-- すべての item 高さが近い場合に使う -->
</CollectionView>
```

> 💡 **Performance Tip:** list item の高さが概ね揃っているなら、大きな list では `ItemSizingStrategy="MeasureFirstItem"` を使うと性能が改善します。

#### .NET 10 Handler Changes (iOS/Mac Catalyst)

> ℹ️ **.NET 10 では iOS と Mac Catalyst で最適化済み CollectionView / CarouselView handler が既定** になり、性能と安定性が向上しています。

**.NET 9 で新 handler を opt-in していた場合**、次の code は **削除** してください。

```csharp
// ❌ .NET 10 では削除 (これらの handler は既定になった)
#if IOS || MACCATALYST
builder.ConfigureMauiHandlers(handlers =>
{
    handlers.AddHandler<CollectionView, CollectionViewHandler2>();
    handlers.AddHandler<CarouselView, CarouselViewHandler2>();
});
#endif
```

.NET 10 では最適化済み handler が自動適用されるため、追加設定は不要です。

**問題が起きた場合のみ**、legacy handler に戻せます。

```csharp
// MauiProgram.cs 内 - 必要時のみ
#if IOS || MACCATALYST
builder.ConfigureMauiHandlers(handlers =>
{
    handlers.AddHandler<Microsoft.Maui.Controls.CollectionView,
                        Microsoft.Maui.Controls.Handlers.Items.CollectionViewHandler>();
});
#endif
```

ただし Microsoft は、可能な限り新しい既定 handler を使うことを推奨しています。

#### テスト チェックリスト

移行後は次を確認してください。

- [ ] **item selection** が正しく動く
- [ ] **grouped list** が適切な header 付きで表示される
- [ ] **swipe action** (使っている場合) が iOS と Android の両方で動く
- [ ] list が空のとき **empty view** が表示される
- [ ] **pull to refresh** が動く (使っている場合)
- [ ] **scroll performance** が許容範囲にある (特に大規模 list)
- [ ] **item sizing** が正しい (`CollectionView` は既定で auto-size)
- [ ] **selection visual state** が適切に表示・解除される
- [ ] **data binding** により list が正しく更新される
- [ ] list item からの **navigation** が動く

#### 移行が複雑になる要因

`ListView` から `CollectionView` への移行が複雑なのは、次のためです。
- 各 ListView ごとに独自挙動がある
- platform-specific code を更新する必要がある
- 広範なテストが必要
- context action を SwipeView に変換する必要がある
- grouped list は template 更新が必要
- ViewModel 側の変更も必要になり得る

#### Quick Reference: ListView vs CollectionView

| Feature | ListView | CollectionView |
|---------|----------|----------------|
| **Selection Event** | `ItemSelected` | `SelectionChanged` |
| **Selection Args** | `SelectedItemChangedEventArgs` | `SelectionChangedEventArgs` |
| **Getting Selected** | `e.SelectedItem` | `e.CurrentSelection.FirstOrDefault()` |
| **Context Menus** | `ContextActions` | `SwipeView` |
| **Grouping** | `IsGroupingEnabled="True"` | `IsGrouped="true"` |
| **Group Header** | `GroupDisplayBinding` | `GroupHeaderTemplate` |
| **Even Rows** | `HasUnevenRows="False"` | Auto-sizes (既定) |
| **Empty State** | 手動実装 | `EmptyView` property |
| **Cells** | TextCell, ImageCell など | custom DataTemplate |

---

## Deprecated APIs (P1 - Fix Soon)

これらの API は .NET 10 でも動きますが、compiler warning が出ます。将来 version で削除されます。

### 1. Animation Methods

**Status:** ⚠️ **非推奨** - すべての同期 animation method は async version に置き換えられました。

**表示される warning:**
```
warning CS0618: 'ViewExtensions.FadeTo(VisualElement, double, uint, Easing)' is obsolete: 'Please use FadeToAsync instead.'
```

**Migration Table:**

| Old Method | New Method | Example |
|-----------|-----------|---------|
| `FadeTo()` | `FadeToAsync()` | `await view.FadeToAsync(0, 500);` |
| `ScaleTo()` | `ScaleToAsync()` | `await view.ScaleToAsync(1.5, 300);` |
| `TranslateTo()` | `TranslateToAsync()` | `await view.TranslateToAsync(100, 100, 250);` |
| `RotateTo()` | `RotateToAsync()` | `await view.RotateToAsync(360, 500);` |
| `RotateXTo()` | `RotateXToAsync()` | `await view.RotateXToAsync(45, 300);` |
| `RotateYTo()` | `RotateYToAsync()` | `await view.RotateYToAsync(45, 300);` |
| `ScaleXTo()` | `ScaleXToAsync()` | `await view.ScaleXToAsync(2.0, 300);` |
| `ScaleYTo()` | `ScaleYToAsync()` | `await view.ScaleYToAsync(2.0, 300);` |
| `RelRotateTo()` | `RelRotateToAsync()` | `await view.RelRotateToAsync(90, 300);` |
| `RelScaleTo()` | `RelScaleToAsync()` | `await view.RelScaleToAsync(0.5, 300);` |
| `LayoutTo()` | `LayoutToAsync()` | 下の特記事項を参照 |

#### 移行例

**単純な animation:**
```csharp
// ❌ OLD (Deprecated)
await myButton.FadeTo(0, 500);
await myButton.ScaleTo(1.5, 300);
await myButton.TranslateTo(100, 100, 250);

// ✅ NEW (Required)
await myButton.FadeToAsync(0, 500);
await myButton.ScaleToAsync(1.5, 300);
await myButton.TranslateToAsync(100, 100, 250);
```

**順次 animation:**
```csharp
// ❌ OLD
await image.FadeTo(0, 300);
await image.ScaleTo(0.5, 300);
await image.FadeTo(1, 300);

// ✅ NEW
await image.FadeToAsync(0, 300);
await image.ScaleToAsync(0.5, 300);
await image.FadeToAsync(1, 300);
```

**並列 animation:**
```csharp
// ❌ OLD
await Task.WhenAll(
    image.FadeTo(0, 300),
    image.ScaleTo(0.5, 300),
    image.RotateTo(360, 300)
);

// ✅ NEW
await Task.WhenAll(
    image.FadeToAsync(0, 300),
    image.ScaleToAsync(0.5, 300),
    image.RotateToAsync(360, 300)
);
```

**Cancellation 付き:**
```csharp
// NEW: async method は cancellation をサポート
CancellationTokenSource cts = new();

try
{
    await view.FadeToAsync(0, 2000);
}
catch (TaskCanceledException)
{
    // animation が cancel された
}

// 他所から cancel
cts.Cancel();
```

#### 特記事項: LayoutTo

`LayoutToAsync()` には特別な非推奨メッセージがあります: "Use Translation to animate layout changes."

```csharp
// ❌ OLD (Deprecated)
await view.LayoutToAsync(new Rect(100, 100, 200, 200), 250);

// ✅ NEW (代わりに TranslateToAsync を使う)
await view.TranslateToAsync(100, 100, 250);

// または Translation property を直接 animation する
var animation = new Animation(v => view.TranslationX = v, 0, 100);
animation.Commit(view, "MoveX", length: 250);
```

---

### 2. DisplayAlert and DisplayActionSheet

**Status:** ⚠️ **非推奨** - 同期 method は async version に置き換えられました。

**表示される warning:**
```
warning CS0618: 'Page.DisplayAlert(string, string, string)' is obsolete: 'Use DisplayAlertAsync instead'
```

#### 移行例

**DisplayAlert:**
```csharp
// ❌ OLD (Deprecated)
await DisplayAlert("Success", "Data saved successfully", "OK");
await DisplayAlert("Error", "Failed to save", "Cancel");
bool result = await DisplayAlert("Confirm", "Delete this item?", "Yes", "No");

// ✅ NEW (Required)
await DisplayAlertAsync("Success", "Data saved successfully", "OK");
await DisplayAlertAsync("Error", "Failed to save", "Cancel");
bool result = await DisplayAlertAsync("Confirm", "Delete this item?", "Yes", "No");
```

**DisplayActionSheet:**
```csharp
// ❌ OLD (Deprecated)
string action = await DisplayActionSheet(
    "Choose an action",
    "Cancel",
    "Delete",
    "Edit", "Share", "Duplicate"
);

// ✅ NEW (Required)
string action = await DisplayActionSheetAsync(
    "Choose an action",
    "Cancel",
    "Delete",
    "Edit", "Share", "Duplicate"
);
```

**ViewModel 内で使う場合 (IDispatcher と併用):**
```csharp
// ViewModel から呼ぶ場合は Page への access が必要
public class MyViewModel
{
    private readonly IDispatcher _dispatcher;
    private readonly Page _page;

    public MyViewModel(IDispatcher dispatcher, Page page)
    {
        _dispatcher = dispatcher;
        _page = page;
    }

    public async Task ShowAlertAsync()
    {
        await _dispatcher.DispatchAsync(async () =>
        {
            await _page.DisplayAlertAsync("Info", "Message from ViewModel", "OK");
        });
    }
}
```

---

### 3. Page.IsBusy

**Status:** ⚠️ **非推奨** - この property は .NET 11 で削除されます。

**表示される warning:**
```
warning CS0618: 'Page.IsBusy' is obsolete: 'Page.IsBusy has been deprecated and will be removed in .NET 11'
```

**非推奨になった理由:**
- platform 間で挙動が一貫しない
- カスタマイズ余地が少ない
- モダンな MVVM パターンと相性が悪い

#### 移行例

**Simple Page:**
```xaml
<!-- ❌ OLD (Deprecated) -->
<ContentPage IsBusy="{Binding IsLoading}">
    <StackLayout>
        <Label Text="Content here" />
    </StackLayout>
</ContentPage>

<!-- ✅ NEW (Recommended) -->
<ContentPage>
    <Grid>
        <!-- メイン コンテンツ -->
        <StackLayout>
            <Label Text="Content here" />
        </StackLayout>

        <!-- ローディング インジケーターの overlay -->
        <ActivityIndicator IsRunning="{Binding IsLoading}"
                          IsVisible="{Binding IsLoading}"
                          Color="{StaticResource Primary}"
                          VerticalOptions="Center"
                          HorizontalOptions="Center" />
    </Grid>
</ContentPage>
```

**With Loading Overlay:**
```xaml
<!-- ✅ Better: 独自の loading overlay -->
<ContentPage>
    <Grid>
        <!-- メイン コンテンツ -->
        <ScrollView>
            <VerticalStackLayout Padding="20">
                <Label Text="Your content here" />
            </VerticalStackLayout>
        </ScrollView>

        <!-- Loading overlay -->
        <Grid IsVisible="{Binding IsLoading}"
              BackgroundColor="#80000000">
            <VerticalStackLayout VerticalOptions="Center"
                               HorizontalOptions="Center"
                               Spacing="10">
                <ActivityIndicator IsRunning="True"
                                 Color="White" />
                <Label Text="Loading..."
                       TextColor="White" />
            </VerticalStackLayout>
        </Grid>
    </Grid>
</ContentPage>
```

**Code-Behind 内:**
```csharp
// ❌ OLD (Deprecated)
public partial class MyPage : ContentPage
{
    async Task LoadDataAsync()
    {
        IsBusy = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            IsBusy = false;
        }
    }
}

// ✅ NEW (Recommended)
public partial class MyPage : ContentPage
{
    async Task LoadDataAsync()
    {
        LoadingIndicator.IsVisible = true;
        LoadingIndicator.IsRunning = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            LoadingIndicator.IsVisible = false;
            LoadingIndicator.IsRunning = false;
        }
    }
}
```

**ViewModel 内:**
```csharp
public class MyViewModel : INotifyPropertyChanged
{
    private bool _isLoading;
    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            OnPropertyChanged();
        }
    }

    public async Task LoadDataAsync()
    {
        IsLoading = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

---

### 4. MediaPicker APIs

**Status:** ⚠️ **非推奨** - 単一選択 API は複数選択対応版へ置き換えられました。

**表示される warning:**
```
warning CS0618: 'MediaPicker.PickPhotoAsync(MediaPickerOptions)' is obsolete: 'Switch to PickPhotosAsync which also allows multiple selections.'
warning CS0618: 'MediaPicker.PickVideoAsync(MediaPickerOptions)' is obsolete: 'Switch to PickVideosAsync which also allows multiple selections.'
```

**変更点:**
- `PickPhotoAsync()` → `PickPhotosAsync()` (`List<FileResult>` を返す)
- `PickVideoAsync()` → `PickVideosAsync()` (`List<FileResult>` を返す)
- `MediaPickerOptions` に `SelectionLimit` property が追加 (既定値: 1)
- 旧 API も動作はするが obsolete 扱い

**主な挙動:**
- **既定動作は維持:** `SelectionLimit = 1` (単一選択)
- 無制限複数選択は `SelectionLimit = 0`
- 特定数まで許可する場合は `SelectionLimit > 1`

**Platform Note:**
- ✅ **iOS:** `SelectionLimit` は native picker UI で強制される
- ⚠️ **Android:** すべての custom picker が `SelectionLimit` を守るわけではない
- ⚠️ **Windows:** `SelectionLimit` 非対応なので自前 validation が必要

#### 移行例

**Simple Photo Picker (単一選択の挙動を維持):**
```csharp
// ❌ OLD (Deprecated)
var photo = await MediaPicker.PickPhotoAsync(new MediaPickerOptions
{
    Title = "Pick a photo"
});

if (photo != null)
{
    var stream = await photo.OpenReadAsync();
    MyImage.Source = ImageSource.FromStream(() => stream);
}

// ✅ NEW (同じ挙動を維持 - 1 枚だけ選択)
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    Title = "Pick a photo",
    SelectionLimit = 1  // 明示的に 1 枚だけ
});

var photo = photos.FirstOrDefault();
if (photo != null)
{
    var stream = await photo.OpenReadAsync();
    MyImage.Source = ImageSource.FromStream(() => stream);
}
```

**Simple Video Picker (単一選択の挙動を維持):**
```csharp
// ❌ OLD (Deprecated)
var video = await MediaPicker.PickVideoAsync(new MediaPickerOptions
{
    Title = "Pick a video"
});

if (video != null)
{
    VideoPlayer.Source = video.FullPath;
}

// ✅ NEW (同じ挙動を維持 - 1 本だけ選択)
var videos = await MediaPicker.PickVideosAsync(new MediaPickerOptions
{
    Title = "Pick a video",
    SelectionLimit = 1  // 明示的に 1 本だけ
});

var video = videos.FirstOrDefault();
if (video != null)
{
    VideoPlayer.Source = video.FullPath;
}
```

**Option なしの Photo Picker (既定値を使う):**
```csharp
// ❌ OLD (Deprecated)
var photo = await MediaPicker.PickPhotoAsync();

// ✅ NEW (既定の SelectionLimit = 1 なので同じ挙動)
var photos = await MediaPicker.PickPhotosAsync();
var photo = photos.FirstOrDefault();
```

**複数写真選択 (新機能):**
```csharp
// ✅ NEW: 最大 5 枚の写真を選択
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    Title = "Pick up to 5 photos",
    SelectionLimit = 5
});

foreach (var photo in photos)
{
    var stream = await photo.OpenReadAsync();
    // 各 photo を処理
}

// ✅ NEW: 無制限選択
var allPhotos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    Title = "Pick photos",
    SelectionLimit = 0  // 制限なし
});
```

**複数動画選択 (新機能):**
```csharp
// ✅ NEW: 最大 3 本の動画を選択
var videos = await MediaPicker.PickVideosAsync(new MediaPickerOptions
{
    Title = "Pick up to 3 videos",
    SelectionLimit = 3
});

foreach (var video in videos)
{
    // 各動画を処理
    Console.WriteLine($"Selected: {video.FileName}");
}
```

**空結果の扱い:**
```csharp
// NEW: user が cancel した場合は null ではなく空 list を返す
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    SelectionLimit = 1
});

// ✅ 空 list を確認
if (photos.Count == 0)
{
    await DisplayAlertAsync("Cancelled", "No photo selected", "OK");
    return;
}

var photo = photos.First();
// photo を処理...
```

**Try-Catch 付き (従来同様):**
```csharp
try
{
    var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
    {
        Title = "Pick a photo",
        SelectionLimit = 1
    });

    if (photos.Count > 0)
    {
        await ProcessPhotoAsync(photos.First());
    }
}
catch (PermissionException)
{
    await DisplayAlertAsync("Permission Denied", "Camera access required", "OK");
}
catch (Exception ex)
{
    await DisplayAlertAsync("Error", $"Failed to pick photo: {ex.Message}", "OK");
}
```

#### Migration Checklist

新しい MediaPicker API に移行するときは、次を確認してください。

- [ ] `PickPhotoAsync()` を `PickPhotosAsync()` に置き換える
- [ ] `PickVideoAsync()` を `PickVideosAsync()` に置き換える
- [ ] 単一選択挙動を維持するため `SelectionLimit = 1` を設定する
- [ ] `FileResult?` を `List<FileResult>` に変更する (または `.FirstOrDefault()` を使う)
- [ ] null check を空 list check (`photos.Count == 0`) に変更する
- [ ] Android でテストし、custom picker が limit を守るか確認する (必要なら validation を追加)
- [ ] Windows でテストし、必要なら自前の limit validation を追加する
- [ ] multi-select で UX が改善するか検討する (任意)

#### Platform-Specific Validation (Windows & Android)

```csharp
// ✅ 推奨: limit を強制しない platform では手動検証する
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    Title = "Pick up to 5 photos",
    SelectionLimit = 5
});

// Windows と一部 Android picker では limit が強制されないことがある
if (photos.Count > 5)
{
    await DisplayAlertAsync(
        "Too Many Photos",
        $"Please select up to 5 photos. You selected {photos.Count}.",
        "OK"
    );
    return;
}

// 続けて処理...
```

#### Capture Methods (変更なし)

**Note:** capture 系 method (`CapturePhotoAsync`, `CaptureVideoAsync`) は **非推奨ではなく**、変更不要です。

```csharp
// ✅ これらはそのまま使える (変更不要)
var photo = await MediaPicker.CapturePhotoAsync();
var video = await MediaPicker.CaptureVideoAsync();
```

#### Quick Migration Pattern

**既存の単一選択 code には、すべて次のパターンを使えます:**

```csharp
// ❌ OLD
var photo = await MediaPicker.PickPhotoAsync(options);
if (photo != null)
{
    // photo を処理
}

// ✅ NEW (drop-in replacement)
var photos = await MediaPicker.PickPhotosAsync(options ?? new MediaPickerOptions { SelectionLimit = 1 });
var photo = photos.FirstOrDefault();
if (photo != null)
{
    // 以前と同じ code で処理
}
```

---

## Recommended Changes (P2)

これらはすぐ必須ではありませんが、次の refactoring cycle で移行を検討してください。

### Application.MainPage

**Status:** ⚠️ **非推奨** - この property は将来 version で削除されます。

**表示される warning:**
```
warning CS0618: 'Application.MainPage' is obsolete: 'This property is deprecated. Initialize your application by overriding Application.CreateWindow...'
```

#### 移行例

```csharp
// ❌ OLD (Deprecated)
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        MainPage = new AppShell();
    }

    // 後で page を切り替える
    public void SwitchToLoginPage()
    {
        MainPage = new LoginPage();
    }
}

// ✅ NEW (Recommended)
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
    }

    protected override Window CreateWindow(IActivationState? activationState)
    {
        return new Window(new AppShell());
    }

    // 後で page を切り替える
    public void SwitchToLoginPage()
    {
        if (Windows.Count > 0)
        {
            Windows[0].Page = new LoginPage();
        }
    }
}
```

**CreateWindow の利点:**
- マルチ window support が向上する
- 初期化がより明示的になる
- 関心の分離がきれいになる
- Shell と相性がよい

---

## Bulk Migration Tools

以下の find/replace パターンを使うと、codebase を素早く更新できます。

### Visual Studio / VS Code

**Regex Mode - Find/Replace**

#### Animation Methods

```regex
Find:    \.FadeTo\(
Replace: .FadeToAsync(

Find:    \.ScaleTo\(
Replace: .ScaleToAsync(

Find:    \.TranslateTo\(
Replace: .TranslateToAsync(

Find:    \.RotateTo\(
Replace: .RotateToAsync(

Find:    \.RotateXTo\(
Replace: .RotateXToAsync(

Find:    \.RotateYTo\(
Replace: .RotateYToAsync(

Find:    \.ScaleXTo\(
Replace: .ScaleXToAsync(

Find:    \.ScaleYTo\(
Replace: .ScaleYToAsync(

Find:    \.RelRotateTo\(
Replace: .RelRotateToAsync(

Find:    \.RelScaleTo\(
Replace: .RelScaleToAsync(
```

#### Display Methods

```regex
Find:    DisplayAlert\(
Replace: DisplayAlertAsync(

Find:    DisplayActionSheet\(
Replace: DisplayActionSheetAsync(
```

#### MediaPicker Methods

**⚠️ Note:** MediaPicker の移行は return type が変わるため (`FileResult?` → `List<FileResult>`) 手作業が必要です。まず次の検索で対象を見つけてください。

```bash
# PickPhotoAsync の利用箇所を探す
grep -rn "PickPhotoAsync" --include="*.cs" .

# PickVideoAsync の利用箇所を探す
grep -rn "PickVideoAsync" --include="*.cs" .
```

**Manual Migration Pattern:**
```csharp
// Find: await MediaPicker.PickPhotoAsync(
// Replace with:
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions { SelectionLimit = 1 });
var photo = photos.FirstOrDefault();

// Find: await MediaPicker.PickVideoAsync(
// Replace with:
var videos = await MediaPicker.PickVideosAsync(new MediaPickerOptions { SelectionLimit = 1 });
var video = videos.FirstOrDefault();
```

#### ListView/TableView Detection (手動移行が必要)

**⚠️ Note:** ListView / TableView の移行は自動化できません。次の検索で対象を洗い出してください。

```bash
# XAML の ListView をすべて探す
grep -r "<ListView" --include="*.xaml" .

# XAML の TableView をすべて探す
grep -r "<TableView" --include="*.xaml" .

# C# code の ListView を探す
grep -r "new ListView\|ListView " --include="*.cs" .

# XAML 内の Cell type を探す
grep -r "TextCell\|ImageCell\|EntryCell\|SwitchCell\|ViewCell" --include="*.xaml" .

# ItemSelected handler を探す (SelectionChanged へ変更が必要)
grep -r "ItemSelected=" --include="*.xaml" .
grep -r "ItemSelected\s*\+=" --include="*.cs" .

# ContextActions を探す (SwipeView へ変更が必要)
grep -r "ContextActions" --include="*.xaml" .

# platform-specific ListView code を探す (削除対象)
grep -r "PlatformConfiguration.*ListView" --include="*.cs" .
```

**移行 inventory を作る:**
```bash
# ListView/TableView の一覧レポートを生成
echo "=== ListView/TableView Migration Inventory ===" > migration-report.txt
echo "" >> migration-report.txt
echo "XAML ListView instances:" >> migration-report.txt
grep -rn "<ListView" --include="*.xaml" . >> migration-report.txt
echo "" >> migration-report.txt
echo "XAML TableView instances:" >> migration-report.txt
grep -rn "<TableView" --include="*.xaml" . >> migration-report.txt
echo "" >> migration-report.txt
echo "ItemSelected handlers:" >> migration-report.txt
grep -rn "ItemSelected" --include="*.xaml" --include="*.cs" . >> migration-report.txt
echo "" >> migration-report.txt
cat migration-report.txt
```

### PowerShell Script

```powershell
# すべての .cs file で animation method を置換
Get-ChildItem -Path . -Recurse -Filter *.cs | ForEach-Object {
    $content = Get-Content $_.FullName -Raw

    # Animation methods
    $content = $content -replace '\.FadeTo\(', '.FadeToAsync('
    $content = $content -replace '\.ScaleTo\(', '.ScaleToAsync('
    $content = $content -replace '\.TranslateTo\(', '.TranslateToAsync('
    $content = $content -replace '\.RotateTo\(', '.RotateToAsync('
    $content = $content -replace '\.RotateXTo\(', '.RotateXToAsync('
    $content = $content -replace '\.RotateYTo\(', '.RotateYToAsync('
    $content = $content -replace '\.ScaleXTo\(', '.ScaleXToAsync('
    $content = $content -replace '\.ScaleYTo\(', '.ScaleYToAsync('
    $content = $content -replace '\.RelRotateTo\(', '.RelRotateToAsync('
    $content = $content -replace '\.RelScaleTo\(', '.RelScaleToAsync('

    # Display methods
    $content = $content -replace 'DisplayAlert\(', 'DisplayAlertAsync('
    $content = $content -replace 'DisplayActionSheet\(', 'DisplayActionSheetAsync('

    Set-Content $_.FullName $content
}

Write-Host "✅ Migration complete!"
```

---

## Testing Your Upgrade

### Build Validation

```bash
# solution を clean
dotnet clean

# package を restore
dotnet restore

# 各 platform 向けに build
dotnet build -f net10.0-android -c Release
dotnet build -f net10.0-ios -c Release
dotnet build -f net10.0-maccatalyst -c Release
dotnet build -f net10.0-windows -c Release

# warning を確認
dotnet build --no-incremental 2>&1 | grep -i "warning CS0618"
```

### Warnings as Errors を一時的に有効化する

```xml
<!-- obsolete API の利用を確実に検出するため .csproj に追加 -->
<PropertyGroup>
  <WarningsAsErrors>CS0618</WarningsAsErrors>
</PropertyGroup>
```

### Test Checklist

- [ ] すべての platform でアプリが正常起動する
- [ ] すべての animation が正しく動作する
- [ ] dialog (alert / action sheet) が正しく表示される
- [ ] loading indicator が正しく動作する (IsBusy を使っていた場合)
- [ ] component 間通信が正常に動作する (MessagingCenter 代替)
- [ ] build output に CS0618 warning が出ない
- [ ] obsolete API に関連する runtime exception が出ない

---

## Troubleshooting

### Error: 'MessagingCenter' is inaccessible due to its protection level

**Cause:** MessagingCenter は .NET 10 で internal になりました。

**Solution:**
1. `CommunityToolkit.Mvvm` package を install する
2. `WeakReferenceMessenger` に置き換える ([MessagingCenter section](#messagingcenter-made-internal) 参照)
3. 各 message type 用の message class を作る
4. unregister を忘れない

---

### Warning: Animation method is obsolete

**Cause:** 同期 animation method (`FadeTo`, `ScaleTo` など) を使っている。

**Quick Fix:**
```bash
# Bulk Migration Tools セクションの PowerShell script を使う
# または Find/Replace パターンを使う
```

**Manual Fix:**
各 animation method の末尾に `Async` を追加します。
- `FadeTo` → `FadeToAsync`
- `ScaleTo` → `ScaleToAsync`
- など

---

### Page.IsBusy doesn't work anymore

**Cause:** `IsBusy` はまだ動作しますが、非推奨です。

**Solution:** 明示的な `ActivityIndicator` に置き換えます ([IsBusy section](#3-pageisbusy) 参照)

---

### Build fails with "Target framework 'net10.0' not found"

**Cause:** .NET 10 SDK が未インストール、または version が古い。

**Solution:**
```bash
# SDK version を確認
dotnet --version  # 10.0.100 以降が必要

# .NET 10 SDK をここから install:
# https://dotnet.microsoft.com/download/dotnet/10.0

# workload を更新
dotnet workload update
```

---

### MessagingCenter migration breaks existing code

**よくある問題:**

1. **unregister を忘れている:**
   ```csharp
   // ⚠️ unregister しないと memory leak の原因
   protected override void OnDisappearing()
   {
       base.OnDisappearing();
       WeakReferenceMessenger.Default.UnregisterAll(this);
   }
   ```

2. **message type が違う:**
   ```csharp
   // ❌ Wrong
   WeakReferenceMessenger.Default.Register<UserLoggedIn>(this, handler);
   WeakReferenceMessenger.Default.Send(new UserData());  // 型が違う!

   // ✅ Correct
   WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, handler);
   WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(userData));
   ```

3. **recipient parameter の混乱:**
   ```csharp
   // recipient parameter は register した object (this)
   WeakReferenceMessenger.Default.Register<MyMessage>(this, (recipient, message) =>
   {
       // recipient == this
       // message == 送信された message
   });
   ```

---

### Warning: MediaPicker methods are obsolete

**Cause:** 非推奨の `PickPhotoAsync` または `PickVideoAsync` を使っている。

**Solution:** `PickPhotosAsync` または `PickVideosAsync` へ移行します。

```csharp
// ❌ OLD
var photo = await MediaPicker.PickPhotoAsync(options);

// ✅ NEW (単一選択を維持)
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    Title = options?.Title,
    SelectionLimit = 1
});
var photo = photos.FirstOrDefault();
```

**主な変更点:**
- return type が `FileResult?` から `List<FileResult>` に変わる
- 単一結果を取るには `.FirstOrDefault()` を使う
- 旧挙動を保つには `SelectionLimit = 1` を設定する
- `photo == null` の代わりに `photos.Count == 0` を確認する

---

### MediaPicker returns more items than SelectionLimit

**Cause:** Windows と一部 Android custom picker は `SelectionLimit` を強制しません。

**Solution:** 手動 validation を追加します。

```csharp
var photos = await MediaPicker.PickPhotosAsync(new MediaPickerOptions
{
    SelectionLimit = 5
});

if (photos.Count > 5)
{
    await DisplayAlertAsync("Error", "Too many photos selected", "OK");
    return;
}
```

---

### Animation doesn't complete after migration

**Cause:** `await` を付け忘れている。

```csharp
// ❌ Wrong - animation は走るが code がすぐ次へ進む
view.FadeToAsync(0, 500);
DoSomethingElse();

// ✅ Correct - animation 完了を待つ
await view.FadeToAsync(0, 500);
DoSomethingElse();
```

---

### Warning: ListView/TableView/TextCell is obsolete

**Cause:** 非推奨の ListView、TableView、または Cell type を使っている。

**Solution:** `CollectionView` に移行します ([ListView and TableView section](#listview-and-tableview-deprecated) 参照)

**Quick Decision Guide:**
- **Simple list** → custom DataTemplate を使う CollectionView
- **20 項目未満の設定画面** → BindableLayout を使う VerticalStackLayout
- **20 項目以上の設定画面** → grouped CollectionView
- **grouped data list** → `IsGrouped="True"` の CollectionView

---

### CollectionView doesn't have SelectedItem event

**Cause:** CollectionView は `ItemSelected` ではなく `SelectionChanged` を使います。

**Solution:**
```csharp
// ❌ OLD (ListView)
void OnItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    var item = e.SelectedItem as MyItem;
}

// ✅ NEW (CollectionView)
void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    var item = e.CurrentSelection.FirstOrDefault() as MyItem;
}
```

---

### Platform-specific ListView configuration is obsolete

**Cause:** `Microsoft.Maui.Controls.PlatformConfiguration.*Specific.ListView` extension を使っている。

**Error:**
```
warning CS0618: 'ListView' is obsolete: 'With the deprecation of ListView, this class is obsolete. Please use CollectionView instead.'
```

**Solution:**
1. platform-specific ListView の using を削除します:
   ```csharp
   // ❌ これらは削除
   using Microsoft.Maui.Controls.PlatformConfiguration;
   using Microsoft.Maui.Controls.PlatformConfiguration.iOSSpecific;
   using Microsoft.Maui.Controls.PlatformConfiguration.AndroidSpecific;
   ```

2. platform-specific ListView 呼び出しを削除します:
   ```csharp
   // ❌ これらは削除
   myListView.On<iOS>().SetSeparatorStyle(SeparatorStyle.FullWidth);
   myListView.On<Android>().IsFastScrollEnabled();
   viewCell.On<iOS>().SetDefaultBackgroundColor(Colors.White);
   ```

3. `CollectionView` の platform customization は別手段になるため、代替は CollectionView のドキュメントを参照してください。

---

### CollectionView performance issues after ListView migration

**よくある原因:**

1. **DataTemplate caching を考慮していない:**
   ```xaml
   <!-- ❌ Bad performance -->
   <CollectionView.ItemTemplate>
       <DataTemplate>
           <ComplexView />
       </DataTemplate>
   </CollectionView.ItemTemplate>

   <!-- ✅ Better - より単純な template を使う -->
   <CollectionView.ItemTemplate>
       <DataTemplate>
           <VerticalStackLayout Padding="10">
               <Label Text="{Binding Title}" />
           </VerticalStackLayout>
       </DataTemplate>
   </CollectionView.ItemTemplate>
   ```

2. **複雑にネストした layout:**
   - ItemTemplate 内で深い layout ネストを避ける
   - 可能なら `StackLayout` より `Grid` を使う
   - 複雑 layout には `FlexLayout` も検討する

3. **image が cache されていない:**
   ```xaml
   <Image Source="{Binding ImageUrl}"
          Aspect="AspectFill"
          HeightRequest="80"
          WidthRequest="80">
       <Image.Behaviors>
           <!-- 必要なら caching behavior を追加 -->
       </Image.Behaviors>
   </Image>
   ```

---

## Quick Reference Card

### Priority Checklist

**Must Fix (P0 - 破壊的 / 重大):**
- [ ] `MessagingCenter` を `WeakReferenceMessenger` に置き換える
- [ ] `ListView` を `CollectionView` に移行する
- [ ] `TableView` を `CollectionView` または `BindableLayout` に移行する
- [ ] `TextCell`, `ImageCell` などを custom DataTemplate に置き換える
- [ ] `ContextActions` を `SwipeView` に変換する
- [ ] platform-specific ListView 設定を削除する

**Should Fix (P1 - 非推奨):**
- [ ] animation method に `Async` suffix を付ける
- [ ] `DisplayAlert` → `DisplayAlertAsync` に更新する
- [ ] `DisplayActionSheet` → `DisplayActionSheetAsync` に更新する
- [ ] `Page.IsBusy` を `ActivityIndicator` に置き換える
- [ ] `PickPhotoAsync` → `PickPhotosAsync` に置き換える (`SelectionLimit = 1` を付ける)
- [ ] `PickVideoAsync` → `PickVideosAsync` に置き換える (`SelectionLimit = 1` を付ける)

**Nice to Have (P2):**
- [ ] `Application.MainPage` を `CreateWindow` に移行する

### Common Patterns

```csharp
// Animation
await view.FadeToAsync(0, 500);

// Alert
await DisplayAlertAsync("Title", "Message", "OK");

// Messaging
WeakReferenceMessenger.Default.Send(new MyMessage());
WeakReferenceMessenger.Default.Register<MyMessage>(this, (r, m) => { });
WeakReferenceMessenger.Default.UnregisterAll(this);

// Loading
IsLoading = true;
try { await LoadAsync(); }
finally { IsLoading = false; }
```

---

## Additional Resources

- **Official Docs:** https://learn.microsoft.com/dotnet/maui/
- **Migration Guide:** https://learn.microsoft.com/dotnet/maui/migration/
- **GitHub Issues:** https://github.com/dotnet/maui/issues
- **CommunityToolkit.Mvvm:** https://learn.microsoft.com/dotnet/communitytoolkit/mvvm/

---

**Document Version:** 2.0
**Last Updated:** November 2025
**Applies To:** .NET MAUI 10.0.100 and later
