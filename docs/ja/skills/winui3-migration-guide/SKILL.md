---
name: winui3-migration-guide
description: 'UWP-to-WinUI 3 migration reference. Maps legacy UWP APIs to correct Windows App SDK equivalents with before/after code snippets. Covers namespace changes, threading (CoreDispatcher to DispatcherQueue), windowing (CoreWindow to AppWindow), dialogs, pickers, sharing, printing, background tasks, and the most common Copilot code generation mistakes.'
---
# WinUI 3 移行ガイド

このスキルは、UWP アプリを WinUI 3 / Windows App SDK に移行する場合、または生成されたコードが従来の UWP パターンではなく正しい WinUI 3 API を使用していることを確認する場合に使用します。

---

## 名前空間の変更

すべての `Windows.UI.Xaml.*` 名前空間は `Microsoft.UI.Xaml.*` に移動します。

| UWP 名前空間 | WinUI 3 名前空間 |
|--------------|-------------------|
| `Windows.UI.Xaml` | `Microsoft.UI.Xaml` |
| `Windows.UI.Xaml.Controls` | `Microsoft.UI.Xaml.Controls` |
| `Windows.UI.Xaml.Media` | `Microsoft.UI.Xaml.Media` |
| `Windows.UI.Xaml.Input` | `Microsoft.UI.Xaml.Input` |
| `Windows.UI.Xaml.Data` | `Microsoft.UI.Xaml.Data` |
| `Windows.UI.Xaml.Navigation` | `Microsoft.UI.Xaml.Navigation` |
| `Windows.UI.Xaml.Shapes` | `Microsoft.UI.Xaml.Shapes` |
| `Windows.UI.Composition` | `Microsoft.UI.Composition` |
| `Windows.UI.Input` | `Microsoft.UI.Input` |
| `Windows.UI.Colors` | `Microsoft.UI.Colors` |
| `Windows.UI.Text` | `Microsoft.UI.Text` |
| `Windows.UI.Core` | `Microsoft.UI.Dispatching` (ディスパッチャー用) |

---

## 副操縦士によくある間違いトップ 3

### 1. XamlRoot を使用しない ContentDialog```csharp
// ❌ WRONG — Throws InvalidOperationException in WinUI 3
var dialog = new ContentDialog
{
    Title = "Error",
    Content = "Something went wrong.",
    CloseButtonText = "OK"
};
await dialog.ShowAsync();
```

```csharp
// ✅ CORRECT — Set XamlRoot before showing
var dialog = new ContentDialog
{
    Title = "Error",
    Content = "Something went wrong.",
    CloseButtonText = "OK",
    XamlRoot = this.Content.XamlRoot  // Required in WinUI 3
};
await dialog.ShowAsync();
```### 2. ContentDialog の代わりに MessageDialog を使用する```csharp
// ❌ WRONG — UWP API, not available in WinUI 3 desktop
var dialog = new Windows.UI.Popups.MessageDialog("Are you sure?", "Confirm");
await dialog.ShowAsync();
```

```csharp
// ✅ CORRECT — Use ContentDialog
var dialog = new ContentDialog
{
    Title = "Confirm",
    Content = "Are you sure?",
    PrimaryButtonText = "Yes",
    CloseButtonText = "No",
    XamlRoot = this.Content.XamlRoot
};
var result = await dialog.ShowAsync();
if (result == ContentDialogResult.Primary)
{
    // User confirmed
}
```### 3. DispatcherQueue の代わりに CoreDispatcher```csharp
// ❌ WRONG — CoreDispatcher does not exist in WinUI 3
await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
{
    StatusText.Text = "Done";
});
```

```csharp
// ✅ CORRECT — Use DispatcherQueue
DispatcherQueue.TryEnqueue(() =>
{
    StatusText.Text = "Done";
});

// With priority:
DispatcherQueue.TryEnqueue(DispatcherQueuePriority.High, () =>
{
    ProgressBar.Value = 100;
});
```---

## ウィンドウ移行

### ウィンドウリファレンス```csharp
// ❌ WRONG — Window.Current does not exist in WinUI 3
var currentWindow = Window.Current;
```

```csharp
// ✅ CORRECT — Use a static property in App
public partial class App : Application
{
    public static Window MainWindow { get; private set; }

    protected override void OnLaunched(LaunchActivatedEventArgs args)
    {
        MainWindow = new MainWindow();
        MainWindow.Activate();
    }
}
// Access anywhere: App.MainWindow
```### ウィンドウ管理

| UWP API | WinUI 3 API |
|----------|---------------|
| `ApplicationView.TryResizeView()` | `AppWindow.Resize()` |
| `AppWindow.TryCreateAsync()` | `AppWindow.Create()` |
| `AppWindow.TryShowAsync()` | `AppWindow.Show()` |
| `AppWindow.TryConsolidateAsync()` | `AppWindow.Destroy()` |
| `AppWindow.RequestMoveXxx()` | `AppWindow.Move()` |
| `AppWindow.GetPlacement()` | `AppWindow.Position` プロパティ |
| `AppWindow.RequestPresentation()` | `AppWindow.SetPresenter()` |

### タイトルバー

| UWP API | WinUI 3 API |
|----------|---------------|
| `CoreApplicationViewTitleBar` | `AppWindowTitleBar` |
| `CoreApplicationView.TitleBar.ExtendViewIntoTitleBar` | `AppWindow.TitleBar.ExtendsContentIntoTitleBar` |

---

## ダイアログとピッカーの移行

### ファイル/フォルダー ピッカー```csharp
// ❌ WRONG — UWP style, no window handle
var picker = new FileOpenPicker();
picker.FileTypeFilter.Add(".txt");
var file = await picker.PickSingleFileAsync();
```

```csharp
// ✅ CORRECT — Initialize with window handle
var picker = new FileOpenPicker();
var hwnd = WinRT.Interop.WindowNative.GetWindowHandle(App.MainWindow);
WinRT.Interop.InitializeWithWindow.Initialize(picker, hwnd);
picker.FileTypeFilter.Add(".txt");
var file = await picker.PickSingleFileAsync();
```## スレッド移行

| UWP パターン | WinUI 3 相当 |
|-----------|-----------|
| `CoreDispatcher.RunAsync(priority, callback)` | `DispatcherQueue.TryEnqueue(priority, callback)` |
| `Dispatcher.HasThreadAccess` | `DispatcherQueue.HasThreadAccess` |
| `CoreDispatcher.ProcessEvents()` |同等のものはありません - 非同期コードを再構築します |
| `CoreWindow.GetForCurrentThread()` |利用できません — `DispatcherQueue.GetForCurrentThread()` を使用してください |

**主な違い**: UWP は、再入ブロックが組み込まれた ASTA (アプリケーション STA) を使用します。 WinUI 3 は、この保護のない標準 STA を使用します。非同期コードがメッセージをポンプするときは、再入性の問題に注意してください。

---

## バックグラウンド タスクの移行```csharp
// ❌ WRONG — UWP IBackgroundTask
public sealed class MyTask : IBackgroundTask
{
    public void Run(IBackgroundTaskInstance taskInstance) { }
}
```

```csharp
// ✅ CORRECT — Windows App SDK AppLifecycle
using Microsoft.Windows.AppLifecycle;

// Register for activation
var args = AppInstance.GetCurrent().GetActivatedEventArgs();
if (args.Kind == ExtendedActivationKind.AppNotification)
{
    // Handle background activation
}
```---

## アプリ設定の移行

|シナリオ |パッケージ化されたアプリ |パッケージ化されていないアプリ |
|----------|---------------|-----|
|簡単設定 | `ApplicationData.Current.LocalSettings` | `LocalApplicationData` の JSON ファイル |
|ローカル ファイル ストレージ | `ApplicationData.Current.LocalFolder` | `Environment.GetFolderPath(SpecialFolder.LocalApplicationData)` |

---

## GetForCurrentView() の置き換え

すべての `GetForCurrentView()` パターンは、WinUI 3 デスクトップ アプリでは使用できません。

| UWP API | WinUI 3 の代替 |
|----------|--------|
| `UIViewSettings.GetForCurrentView()` | `AppWindow` プロパティを使用する |
| `ApplicationView.GetForCurrentView()` | `AppWindow.GetFromWindowId(windowId)` |
| `DisplayInformation.GetForCurrentView()` | Win32 `GetDpiForWindow()` または `XamlRoot.RasterizationScale` |
| `CoreApplication.GetCurrentView()` |利用できません — ウィンドウを手動で追跡します。
| `SystemNavigationManager.GetForCurrentView()` | `NavigationView` で戻るナビゲーションを直接処理する |

---

## 移行のテスト

UWP 単体テスト プロジェクトは WinUI 3 では動作しません。WinUI 3 テスト プロジェクト テンプレートに移行する必要があります。

| UWP | WinUI3 |
|-----|----------|
|単体テスト アプリ (ユニバーサル Windows) | **単体テスト アプリ (デスクトップの WinUI)** |
| UWP タイプの標準 MSTest プロジェクト | Xaml ランタイムには WinUI テスト アプリを使用する必要があります |
| `[TestMethod]` すべてのテスト用 | `[TestMethod]` ロジック用、`[UITestMethod]` XAML/UI テスト用 |
|クラス ライブラリ (ユニバーサル Windows) | **クラス ライブラリ (デスクトップの WinUI)** |```csharp
// ✅ WinUI 3 unit test — use [UITestMethod] for any XAML interaction
[UITestMethod]
public void TestMyControl()
{
    var control = new MyLibrary.MyUserControl();
    Assert.AreEqual(expected, control.MyProperty);
}
```**キー:** `[UITestMethod]` 属性は、`Microsoft.UI.Xaml` 型をインスタンス化するために必要な、XAML UI スレッドでテストを実行するようにテスト ランナーに指示します。

---

## 移行チェックリスト

1. [ ] ディレクティブを使用しているすべての `Windows.UI.Xaml.*` を `Microsoft.UI.Xaml.*` に置き換えます。
2. [ ] `Windows.UI.Colors` を `Microsoft.UI.Colors` に置き換えます
3. [ ] `CoreDispatcher.RunAsync` を `DispatcherQueue.TryEnqueue` に置き換えます
4. [ ] `Window.Current` を `App.MainWindow` 静的プロパティに置き換えます
5. [ ] `XamlRoot` をすべての `ContentDialog` インスタンスに追加します
6. [ ] `InitializeWithWindow.Initialize(picker, hwnd)` を使用してすべてのピッカーを初期化します。
7. [ ] `MessageDialog` を `ContentDialog` に置き換えます
8. [ ] `ApplicationView`/`CoreWindow` を `AppWindow` に置き換えます
9. [ ] `CoreApplicationViewTitleBar` を `AppWindowTitleBar` に置き換えます
10. [ ] すべての `GetForCurrentView()` 呼び出しを `AppWindow` と同等の呼び出しに置き換えます。
11. [ ] 共有マネージャーと印刷マネージャーの相互運用性の更新
12. [ ] `IBackgroundTask` を `AppLifecycle` アクティベーションに置き換えます
13. [ ] プロジェクト ファイルを更新: TFM を `net10.0-windows10.0.22621.0` に、`<UseWinUI>true</UseWinUI>` を追加
14. [ ] 単体テストを **単体テスト アプリ (デスクトップの WinUI)** プロジェクトに移行します。 XAML テストには `[UITestMethod]` を使用します
15. [ ] パッケージ化された構成とパッケージ化されていない構成の両方をテストする