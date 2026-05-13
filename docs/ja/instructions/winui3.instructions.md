---
description: 'WinUI 3 と Windows App SDK のコーディングガイドライン。デスクトップ Windows アプリ向けに、よくある UWP API の誤用を防ぎ、正しい XAML namespace、threading、windowing、MVVM パターンを徹底する。'
applyTo: '**/*.xaml, **/*.cs, **/*.csproj'
---

# WinUI 3 / Windows App SDK

## 重要ルール — 旧来の UWP API は絶対に使わない

これらの UWP パターンは、WinUI 3 デスクトップアプリでは **誤り** である。必ず Windows App SDK の同等機能を使うこと。

- **絶対に** `Windows.UI.Popups.MessageDialog` を使わない。`XamlRoot` を設定した `ContentDialog` を使う。
- **絶対に** `dialog.XamlRoot = this.Content.XamlRoot` を先に設定せずに `ContentDialog` を表示しない。
- **絶対に** `CoreDispatcher.RunAsync` や `Dispatcher.RunAsync` を使わない。`DispatcherQueue.TryEnqueue` を使う。
- **絶対に** `Window.Current` を使わない。メインウィンドウは静的な `App.MainWindow` property で追跡する。
- **絶対に** `Windows.UI.Xaml.*` namespace を使わない。`Microsoft.UI.Xaml.*` を使う。
- **絶対に** `Windows.UI.Composition` を使わない。`Microsoft.UI.Composition` を使う。
- **絶対に** `Windows.UI.Colors` を使わない。`Microsoft.UI.Colors` を使う。
- **絶対に** window 管理に `ApplicationView` や `CoreWindow` を使わない。`Microsoft.UI.Windowing.AppWindow` を使う。
- **絶対に** `CoreApplicationViewTitleBar` を使わない。`AppWindowTitleBar` を使う。
- **絶対に** `GetForCurrentView()` パターン (例: `UIViewSettings.GetForCurrentView()`) を使わない。これらはデスクトップ WinUI 3 には存在しない。代わりに `AppWindow` API を使う。
- **絶対に** UWP の `PrintManager` を直接使わない。window handle とともに `IPrintManagerInterop` を使う。
- **絶対に** 共有のために `DataTransferManager` を直接使わない。window handle とともに `IDataTransferManagerInterop` を使う。
- **絶対に** UWP の `IBackgroundTask` を使わない。`Microsoft.Windows.AppLifecycle` activation を使う。
- **絶対に** `WebAuthenticationBroker` を使わない。`OAuth2Manager` (Windows App SDK 1.7+) を使う。

## XAML パターン

- 既定の XAML namespace は `Windows.UI.Xaml` ではなく `Microsoft.UI.Xaml` に対応する。
- コンパイル済みで型安全かつ高性能な binding のため、`{Binding}` より `{x:Bind}` を優先する。
- `{x:Bind}` を使う `DataTemplate` 要素では `x:DataType` を設定する。これは template 内の compiled binding で必須である。Page/UserControl では、`x:DataType` によって compile-time binding validation が有効になるが、DataContext が変化しないなら厳密な必須ではない。
- 動的値には `Mode=OneWay`、静的値には `Mode=OneTime`、編集可能入力にだけ `Mode=TwoWay` を使う。
- 静的定数は bind せず、XAML に直接設定する。

## スレッド処理

- バックグラウンドスレッドから UI を更新するときは `DispatcherQueue.TryEnqueue(() => { ... })` を使う。
- `TryEnqueue` は `Task` ではなく `bool` を返す。つまり fire-and-forget である。
- dispatch 前に `DispatcherQueue.HasThreadAccess` でスレッドアクセスを確認する。
- WinUI 3 は ASTA ではなく標準的な STA を使う。組み込みの reentrancy protection はないため、メッセージポンプを回す async code には注意する。

## Window 管理

- WinUI 3 の `Window` から `AppWindow` を取得するには、`WindowNative.GetWindowHandle` → `Win32Interop.GetWindowIdFromWindow` → `AppWindow.GetFromWindowId` の順に使う。
- サイズ変更、移動、タイトル、presenter 操作には `AppWindow` を使う。
- カスタムタイトルバーには `CoreApplicationViewTitleBar` ではなく `AppWindow.TitleBar` property を使う。
- メインウィンドウは `App.MainWindow` (`OnLaunched` で設定する静的 property) として追跡する。

## ダイアログと Picker

- **ContentDialog**: `ShowAsync()` を呼ぶ前に、必ず `dialog.XamlRoot = this.Content.XamlRoot` を設定する。
- **File/Folder Pickers**: `WinRT.Interop.InitializeWithWindow.Initialize(picker, hwnd)` で初期化する。`hwnd` は `WindowNative.GetWindowHandle(App.MainWindow)` から取得する。
- **Share/Print**: COM interop interface (`IDataTransferManagerInterop`、`IPrintManagerInterop`) を window handle とともに使う。

## MVVM とデータバインディング

- MVVM 基盤には `CommunityToolkit.Mvvm` (`[ObservableProperty]`、`[RelayCommand]`) を優先する。
- service 登録と injection には `Microsoft.Extensions.DependencyInjection` を使う。
- UI (View) は layout と binding に集中させ、ロジックは ViewModel と service に置く。
- UI 応答性を保つため、I/O や長時間処理には `async` / `await` を使う。

## プロジェクト設定

- `net10.0-windows10.0.22621.0` (またはプロジェクト対象 SDK に適した TFM) を対象にする。
- project file では `<UseWinUI>true</UseWinUI>` を設定する。
- 最新安定版の `Microsoft.WindowsAppSDK` NuGet package を参照する。
- JSON serialization には source generator と併用した `System.Text.Json` を使う。

## C# コードスタイル

- file-scoped namespace を使う。
- nullable reference types を有効化する。`== null` ではなく `is null` / `is not null` を使う。
- null check を伴う `as` / `is` より pattern matching を優先する。
- 型、method、property は PascalCase。private field は camelCase。
- 波かっこは Allman style (開始波かっこを独立行に置く) を使う。
- 組み込み型には明示的な型を優先し、型が明白な場合にのみ `var` を使う。

## アクセシビリティ

- すべての対話可能 control には `AutomationProperties.Name` を設定する。
- セクション見出しには `AutomationProperties.HeadingLevel` を使う。
- 装飾要素は `AutomationProperties.AccessibilityView="Raw"` で隠す。
- キーボード操作を完全に保証する (Tab、Enter、Space、矢印キー)。
- WCAG の color contrast 要件を満たす。

## パフォーマンス

- `{Binding}` (reflection ベース) より `{x:Bind}` (compiled) を優先する。
- **NativeAOT:** Native AOT compilation では `{Binding}` (reflection ベース) は一切動作しない。サポートされるのは `{x:Bind}` (compiled binding) のみ。project が NativeAOT を使う場合は、`{x:Bind}` だけを使う。
- すぐに不要な UI 要素には `x:Load` または `x:DeferLoadStrategy` を使う。
- 大規模リストには virtualization 付きの `ItemsRepeater` を使う。
- 深い layout のネストは避け、入れ子の `StackPanel` 連鎖より `Grid` を優先する。
- すべての I/O には `async` / `await` を使い、UI thread を絶対に block しない。

## アプリ設定 (Packaged / Unpackaged)

- **Packaged app**: `ApplicationData.Current.LocalSettings` は期待どおりに動作する。
- **Unpackaged app**: カスタム settings file を使う (例: `Environment.GetFolderPath(SpecialFolder.LocalApplicationData)` 配下の JSON)。
- `ApplicationData` が常に利用可能だと仮定せず、先に packaging status を確認する。

## タイポグラフィ

- **常に** 組み込みの TextBlock style (`CaptionTextBlockStyle`、`BodyTextBlockStyle`、`BodyStrongTextBlockStyle`、`SubtitleTextBlockStyle`、`TitleTextBlockStyle`、`TitleLargeTextBlockStyle`、`DisplayTextBlockStyle`) を使う。
- `FontSize`、`FontWeight`、`FontFamily` をハードコードするより、組み込み TextBlock style を優先する。
- フォントは Segoe UI Variable が既定。変更しない。
- UI テキストは sentence casing を使う。


## テーマと色

- Light、Dark、High Contrast theme を自動サポートするため、brush と color には **常に** `{ThemeResource}` を使う。
- UI 要素に color 値 (`#FFFFFF`、`Colors.White` など) を **絶対に** ハードコードしない。`TextFillColorPrimaryBrush`、`CardBackgroundFillColorDefaultBrush`、`CardStrokeColorDefaultBrush` のような theme resource を使う。
- ユーザーの accent color palette には `SystemAccentColor` (および `Light1`–`Light3`、`Dark1`–`Dark3` variants) を使う。
- border には `CardStrokeColorDefaultBrush` または `ControlStrokeColorDefaultBrush` を使う。

## 余白とレイアウト

- **4px grid system** を使う。margin、padding、spacing の値はすべて 4px の倍数にする。
- 標準 spacing: 4 (compact)、8 (controls)、12 (small gutters)、16 (content padding)、24 (large gutters)。
- パフォーマンスのため、深くネストした `StackPanel` 連鎖より `Grid` を優先する。
- content サイズに合わせる行 / 列には `Auto`、比率指定には `*` を使う。固定ピクセルサイズは避ける。
- レスポンシブ layout には `AdaptiveTrigger` と組み合わせた `VisualStateManager` を使う (640px、1008px)。
- 小さな control には `ControlCornerRadius` (4px)、card、dialog、flyout には `OverlayCornerRadius` (8px) を使う。

## マテリアルとエレベーション

- app window backdrop には **Mica** (`MicaBackdrop`) を使う。透過レイヤーが上に必要で、Mica が見えるようにする。
- **Acrylic** は一時的な surface (flyout、menu、navigation pane) のみに使う。
- Mica の上に載る content layer には `LayerFillColorDefaultBrush` を使う。
- エレベーションには Z 軸 `Translation` とともに `ThemeShadow` を使う。card: 4–8 px、flyout: 32 px、dialog: 128 px。

## モーションとトランジション

- 組み込み theme transition (`EntranceThemeTransition`、`RepositionThemeTransition`、`ContentThemeTransition`、`AddDeleteThemeTransition`) を使う。
- 組み込み transition があるなら、独自 storyboard animation は避ける。

## Control 選択

- 主要な app navigation には `NavigationView` を使う (独自 sidebar ではなく)。
- 継続表示されるアプリ内通知には `InfoBar` を使う (独自 banner ではなく)。
- 文脈的な案内には `TeachingTip` を使う (独自 popup ではなく)。
- 数値入力には `NumberBox` を使う (手動検証の TextBox ではなく)。
- on/off 設定には `ToggleSwitch` を使う (CheckBox ではなく)。
- built-in の selection、virtualization、柔軟な layout を備えた modern な collection control として、データ表示には `ItemsView` を使う。
- 標準的な virtualized list と grid、特に built-in selection が必要な場合は `ListView` / `GridView` を使う。
- 完全にカスタムな virtualizing layout で描画を全面制御する必要があり、built-in selection や interaction handling が不要な場合にのみ `ItemsRepeater` を使う。
- 折りたたみ可能な section には `Expander` を使う (独自の visibility toggling ではなく)。

## エラーハンドリング

- `async void` event handler は未処理クラッシュを防ぐため、必ず try/catch で囲む。
- ユーザー向けエラーメッセージには `ContentDialog` ではなく `InfoBar` (`Severity = Error`) を使う。
- ログ記録と穏当な回復のため、`App.UnhandledException` を処理する。

## テスト

- WinUI 3 XAML type をインスタンス化するテストに、通常の MSTest や xUnit project を **絶対に** 使わない。Xaml runtime と UI thread を提供する **Unit Test App (WinUI in Desktop)** project を使う。
- 純粋ロジックのテストには `[TestMethod]` を使う。`Microsoft.UI.Xaml` type (control、page、user control) を生成または操作するテストには `[UITestMethod]` を使う。
- テスト可能な業務ロジックは、main app とは分離した **Class Library (WinUI in Desktop)** project に置く。
- Visual Studio の test discovery を有効にするため、テスト実行前に solution をビルドする。

## リソースとローカライズ

- ユーザー向け文字列は code や XAML literal ではなく `Resources.resw` file に保存する。
- ローカライズされた text binding には XAML で `x:Uid` を使う。
- DPI 修飾付き image asset (`logo.scale-200.png`) を使い、参照時は scale qualifier なし (`ms-appx:///Assets/logo.png`) にする。
