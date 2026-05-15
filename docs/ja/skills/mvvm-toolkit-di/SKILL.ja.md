---
name: mvvm-toolkit-di
description: 'CommunityToolkit.Mvvm ViewModel を Microsoft.Extensions.DependencyInjection に接続します。 .NET Generic Host 構成ルート、コンストラクター インジェクション、サービスの有効期間 (シングルトン / 一時的 / スコープ付き)、IMessenger の登録、ビュー内の ViewModel の解決、キー付きサービス、シームのテスト、および従来の Ioc.Default エスケープ ハッチについて説明します。 WPF、WinUI 3、.NET MAUI、Uno、および Avalonia 全体で使用します。'
---

# CommunityToolkit.Mvvm + `Microsoft.Extensions.DependencyInjection`

MVVM ツールキットは意図的に **DI コンテナなし** で出荷されます。
`Microsoft.Extensions.DependencyInjection`、同じコンテナー ASP.NET
コア、ワーカー サービス、および .NET 汎用ホストが使用されます。

> **TL;DR.** 起動時にサービス プロバイダーを一度ビルドします (推奨
> `Host.CreateDefaultBuilder()`)。サービスと ViewModel を登録します。
> コンストラクターを通じて注入します。 `Ioc.Default.GetService<T>()` は避けてください
> ユーザーコードで。

---

## このスキルをいつ使用するか

- 新しい XAML アプリ (WPF、WinUI 3、
マウイ島、宇野島、アバロニア島)
- サービス/VM ライフタイムの選択
- `IMessenger` を 1 回配線し、`ObservableRecipient` に注入します
モデルの表示
- サービス ロケーターに結合せずにページの ViewModel を解決する
- 診断中「タイプ X のサービスを解決できません」
Yをアクティブにする」

ソース ジェネレーターと ViewModel パターンについては、**`mvvm-toolkit`** を参照してください。
スキル。 Messenger pub/sub については、**`mvvm-toolkit-messenger`** を参照してください。

---

## 推奨構成ルート（Generic Host）
```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using CommunityToolkit.Mvvm.Messaging;

public partial class App : Application
{
    public IHost Host { get; }

    public App()
    {
        Host = Microsoft.Extensions.Hosting.Host
            .CreateDefaultBuilder()
            .ConfigureServices((_, services) =>
            {
                services.AddSingleton<IFilesService, FilesService>();
                services.AddSingleton<ISettingsService, SettingsService>();
                services.AddSingleton<IMessenger>(WeakReferenceMessenger.Default);

                services.AddSingleton<ShellViewModel>();
                services.AddTransient<ContactViewModel>();
                services.AddTransient<EditorViewModel>();
            })
            .Build();
    }

    public static T GetService<T>() where T : class =>
        ((App)Current).Host.Services.GetRequiredService<T>();
}
```

汎用ホストの利点:

- `Microsoft.Extensions.Configuration` を介した `appsettings.json` バインディング
- `Microsoft.Extensions.Logging` によるログ記録
- バックグラウンド作業用のホスト型サービス (`IHostedService`)
- 開発ビルドでのスコープ検証

> WPF と Windows フォームはホストの有効期間をアプリと統合する必要があります
> 生涯 - を参照
> [WPF アプリで .NET 汎用ホストを使用する](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/app-development/how-to-use-host-builder)。

### 汎用ホストなし

サービス コンテナのみが必要で、追加の依存関係は不要な場合:
```csharp
var services = new ServiceCollection();
services.AddSingleton<IFilesService, FilesService>();
services.AddTransient<ContactViewModel>();
ServiceProvider provider = services.BuildServiceProvider();
```

---

## コンストラクターのインジェクション

コンストラクターを通じてサービスと子 ViewModel を挿入します。
```csharp
public sealed partial class ContactViewModel(
    IFilesService files,
    IMessenger messenger,
    ILogger<ContactViewModel> logger)
    : ObservableRecipient(messenger)
{
    [ObservableProperty]
    private string? name;

    [RelayCommand]
    private async Task SaveAsync()
    {
        logger.LogInformation("Saving {Name}", Name);
        await files.SaveAsync(Name!);
    }
}
```

コンストラクター インジェクションがサービス ロケーターよりも優れている理由:

- 依存関係は明示的であり、呼び出しサイトで確認できます。
- 単体テストはフェイク/モックを直接挿入します
- DI コンテナは起動時に依存関係グラフを検証します。
- 登録がない場合は、最初の使用時ではなく、すぐにスローされます

---

## 生涯

|生涯 |方法 | XAML アプリでの一般的な使用法 |
|----------|----------|---------------|
|シングルトン | `AddSingleton<T>` |シェル/メイン ウィンドウ VM、設定、ファイル/HTTP サービス、共有 `IMessenger`、アプリ全体のキャッシュ |
|一時的な | `AddTransient<T>` |ページごとまたはドキュメントごとの ViewModel (解決ごとに新しいインスタンス) |
|スコープ付き | `AddScoped<T>` |クライアント アプリではほとんど必要ありません。明示的な `IServiceScope` (ウィンドウごとのスコープなど) を使用すると便利です。
```csharp
services.AddSingleton<ShellViewModel>();   // 1 instance for app lifetime
services.AddTransient<NoteViewModel>();    // new instance per resolve
services.AddScoped<DialogService>();       // 1 per scope (rare)
```

---

## ビューでの解決

コードビハインドでページのルート ViewModel を解決し、その ViewModel をプルさせます。
独自の依存関係:
```csharp
public sealed partial class ContactPage : Page
{
    public ContactViewModel ViewModel { get; }

    public ContactPage()
    {
        ViewModel = App.GetService<ContactViewModel>();
        InitializeComponent();
    }
}
```

`{x:Bind ViewModel.Xxx}` を使用して XAML でバインドする (コンパイルされたバインディング)、または
`{Binding Xxx}` 対 `DataContext`。

ナビゲーション フレームワークの場合 (WinUI 3 `Frame.Navigate`、MAUI Shell、Prism、
MVVMCross)、フレームワークにページを解決させ、ページはそのページを解決します。
DI からの ViewModel。 `new` ViewModel を手動で実行しないでください。

---

## `IMessenger` 登録

必要なメッセンジャーを一度登録し、どこにでも `IMessenger` を挿入します。
```csharp
services.AddSingleton<IMessenger>(WeakReferenceMessenger.Default);
// or
services.AddSingleton<IMessenger>(StrongReferenceMessenger.Default);
```

それから：
```csharp
public sealed partial class MyViewModel(IMessenger messenger)
    : ObservableRecipient(messenger) { }
```

ウィンドウごとのメッセンジャーの場合は、キー付きサービスまたはスコープ指定されたサービスに登録します。
インスタンスを取得し、ウィンドウごとの ViewModel に注入します。

メッセンジャーの領域については、**`mvvm-toolkit-messenger`** スキルを参照してください。

---

## キー付きサービス (.NET 8+)

同じインターフェースの異なる実装をキーによって解決します。
```csharp
services.AddKeyedSingleton<IExporter, CsvExporter>("csv");
services.AddKeyedSingleton<IExporter, JsonExporter>("json");

public sealed partial class ExportViewModel(
    [FromKeyedServices("csv")] IExporter csvExporter,
    [FromKeyedServices("json")] IExporter jsonExporter)
    : ObservableObject { /* ... */ }
```

---

## 縫い目のテスト

コンストラクターによって挿入された依存関係は、テストで簡単に交換できます。と
`Moq`:
```csharp
[Fact]
public async Task Save_calls_files_service()
{
    var files = new Mock<IFilesService>();
    var messenger = new WeakReferenceMessenger();
    var logger = NullLogger<ContactViewModel>.Instance;

    var vm = new ContactViewModel(files.Object, messenger, logger)
    {
        Name = "Ada"
    };

    await vm.SaveCommand.ExecuteAsync(null);

    files.Verify(f => f.SaveAsync("Ada"), Times.Once);
}
```

`Ioc.Default` または静的状態をモックしている場合、ViewModel は
サービス ロケーター — コンストラクター インジェクションへのリファクタリング。

---

## レガシー: `Ioc.Default`

`CommunityToolkit.Mvvm.DependencyInjection.Ioc` は、のための避難ハッチです。
コンストラクター インジェクションが不可能な場合 — XAML でインスタンス化された VM
デザイン時データ、`ValueConverter`s、コントロール テンプレートの場合。
```csharp
Ioc.Default.ConfigureServices(
    new ServiceCollection()
        .AddSingleton<IFilesService, FilesService>()
        .AddTransient<ContactViewModel>()
        .BuildServiceProvider());

var files = Ioc.Default.GetRequiredService<IFilesService>();
```

最後の手段として扱ってください。 ViewModel、サービス、および任意のクラスの内部
DI コンテナは構築できますが、コンストラクター インジェクションを優先します。

---

## よくある落とし穴

1. **`Ioc.Default.GetService<T>()` VM コンストラクター内。**
依存関係があり、単体テストが中断され、起動グラフの検証が妨げられます。
2. **すべて `Singleton`.** シングルトンとして登録された「ドキュメントごと」の VM
すべてのドキュメント間で共有状態になります - 微妙なデータ破損。
インスタンスごとの VM には `AddTransient` を使用します。
3. **複数の `BuildServiceProvider()` 呼び出し。** 各呼び出しは新しい呼び出しです
コンテナ — シングルトンは共有されません。起動時に一度ビルドします。
4. **長命オブジェクトの `IServiceProvider` をキャプチャします。**
サービスロケーターパターン。必要な特定の依存関係を注入します。
5. **開発中にスコープの検証はありません。** `Host.CreateDefaultBuilder()` を使用してください。
(開発時に `ValidateScopes` と `ValidateOnBuild` を設定します)
登録ミスは、最初の使用時ではなく、起動時に失敗します。
6. **ルート プロバイダーからスコープ指定されたサービスを解決します。**
効果的にシングルトンの有効期間に昇格します - 警告は表示されません
スコープ検証なし。有効期間を変更するか、次の解決策を実行してください。
明示的な `IServiceScope`。

---

## 参考文献

|トピック |ファイル |
|------|------|
|完全な詳細 (汎用ホストのセットアップ、ライフタイム、キー付きサービス、テスト パターン、レガシー IOC) | [`references/dependency-injection.md`](references/dependency-injection.md) |

外部の：

- DI の概要: <https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection>
- DI の使用法: <https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-usage>
- MVVM ツールキット Ioc ページ: <https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/ioc>
- 汎用ホスト: <https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host>