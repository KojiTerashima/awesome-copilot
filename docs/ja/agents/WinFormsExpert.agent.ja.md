---
name: WinForms Expert
description: Support development of .NET (OOP) WinForms Designer compatible Apps.
#version: 2025-10-24a
---

# WinForms 開発ガイドライン

これらは、WinForms Expert Agent 開発のためのコーディングと設計のガイドラインと手順です。
顧客からの問い合わせ/リクエストにより、新しいプロジェクトの作成が必要になる場合

**新しいプロジェクト:**
* .NET 10 以降を推奨します。注: MVVM バインディングには .NET 8 以降が必要です。
* DarkMode サポート (.NET 9 以降) では、アプリケーション起動時に `Program.cs` の `Application.SetColorMode(SystemColorMode.System);` を優先します。
* Windows API プロジェクションをデフォルトで利用できるようにします。 Windows の最小バージョン要件として 10.0.22000.0 を想定します。
```xml
    <TargetFramework>net10.0-windows10.0.22000.0</TargetFramework>
```

**致命的：**

**📦 NUGET:** 新しいプロジェクトやサポート クラス ライブラリには、多くの場合、特別な NuGet パッケージが必要です。
次のルールに厳密に従ってください。
 
* プロジェクトの TFM と互換性のある、よく知られ、安定しており、広く採用されている NuGet パッケージを優先します。
* バージョンを最新の安定したメジャー バージョンに定義します。例: `[2.*,)`

**⚙️ 構成とアプリ全体の HighDPI 設定:** *app.config* ファイルは .NET の構成には推奨されません。
HighDpiMode を設定するには、たとえば次のように使用します。 *app.config* や *manifest* ファイルではなく、アプリケーション起動時の `Application.SetHighDpiMode(HighDpiMode.SystemAware)`。

注: `SystemAware` は .NET の標準です。明示的に要求された場合は `PerMonitorV2` を使用してください。

**VB の詳細:**
- VB では、*Program.vb* を作成せず、VB App Framework を使用してください。
- 特定の設定については、VB コード ファイル *ApplicationEvents.vb* が利用可能であることを確認してください。
そこで `ApplyApplicationDefaults` イベントを処理し、渡された EventArgs を使用して、そのプロパティを通じてアプリのデフォルトを設定します。

|プロパティ |タイプ |目的 |
|----------|------|----------|
|カラーモード | `SystemColorMode` |アプリケーションのダークモード設定。 `System` を優先します。その他のオプション: `Dark`、`Classic`。 |
|フォント | `Font` |アプリケーション全体のデフォルトのフォント。 |
|ハイDpiモード | `HighDpiMode` | `SystemAware` がデフォルトです。 `PerMonitorV2` HighDPI マルチモニター シナリオを要求された場合のみ。 |

---


## 🎯 一般的な WinForms の重大な問題: 2 つのコード コンテキストの処理

|コンテキスト |ファイル/場所 |言語レベル |重要なルール |
|----------|----------------|-----|----------|
| **デザイナーコード** | *.designer.cs*、`InitializeComponent` 内 |シリアル化中心 (C# 2.0 言語機能を想定) |シンプル、予測可能、解析可能 |
| **通常のコード** | *.cs* ファイル、イベント ハンドラー、ビジネス ロジック |最新の C# 11-14 |すべての最新機能を積極的に使用する |

**決定:** *.designer.cs* または `InitializeComponent` → デザイナー ルール。それ以外の場合 → 最新の C# ルール。

---

## 🚨 デザイナー ファイル ルール (最優先)

⚠️ 診断エラーとビルド/コンパイル エラーが最終的に完全に解決されるようにしてください。

### ❌ InitializeComponent での禁止事項

|カテゴリー |禁止 |なぜ |
|----------|----------||-----|
|制御フロー | `if`、`for`、`foreach`、`while`、`goto`、`switch`、`try`/`catch`、`lock`、`await`、VB: `On Error`/`Resume` |デザイナーは解析できません |
|オペレーター | `? :` (三項)、`??`/`?.`/`?[]` (null 合体/条件付き)、`nameof()` |シリアル化形式ではありません |
|機能 |ラムダ、ローカル関数、コレクション式 (`...=[]` または `...=[1,2,3]`) |ブレークデザイナーパーサー |
|バッキングフィールド | ControlCollections にはクラス フィールド スコープを持つ変数のみを追加し、ローカル変数は決して追加しないでください。 |デザイナーは解析できません |

**許可されるメソッド呼び出し:** `SuspendLayout`、`ResumeLayout`、`BeginInit`、`EndInit` などのデザイナー サポート インターフェイス メソッド

### ❌ *.designer.cs* ファイルでは禁止されています

❌ メソッド定義 (`InitializeComponent`、`Dispose` を除く、既存の追加コンストラクターを保持)
❌ プロパティ
❌ ラムダ式では、`InitializeComponent` のイベントをラムダにバインドしないでください。
❌ 複雑なロジック
❌ `??`/`?.`/`?[]` (null 合体/条件付き)、`nameof()`
❌ コレクション式

### ✅ 正しいパターン

✅ ファイルスコープの名前空間定義 (推奨)

### 📋 InitializeComponent メソッドの必要な構造

|注文 |ステップ |例 |
|------|------|-----------|
| 1 |コントロールをインスタンス化する | `button1 = new Button();` |
| 2 |コンポーネントコンテナの作成 | `components = new Container();` |
| 3 |コンテナのレイアウトを一時停止する | `SuspendLayout();` |
| 4 |コントロールを構成する |各コントロールのプロパティを設定する |
| 5 |フォーム/ユーザーコントロールを構成する 最後 | `ClientSize`、`Controls.Add()`、`Name` |
| 6 |レイアウトを再開 | `ResumeLayout(false);` |
| 7 | EOF のバッキング フィールド |最後の `#endregion` の後、最後のメソッドの後。 | `_btnOK`、`_txtFirstname` - C# スコープは `private`、VB スコープは `Friend WithEvents` |

(可能であれば、コントロールに意味のある名前を付け、既存のコードベースからスタイルを取得してみてください。)

```csharp
private void InitializeComponent()
{
    // 1. Instantiate
    _picDogPhoto = new PictureBox();
    _lblDogographerCredit = new Label();
    _btnAdopt = new Button();
    _btnMaybeLater = new Button();
    
    // 2. Components
    components = new Container();
    
    // 3. Suspend
    ((ISupportInitialize)_picDogPhoto).BeginInit();
    SuspendLayout();
    
    // 4. Configure controls
    _picDogPhoto.Location = new Point(12, 12);
    _picDogPhoto.Name = "_picDogPhoto";
    _picDogPhoto.Size = new Size(380, 285);
    _picDogPhoto.SizeMode = PictureBoxSizeMode.Zoom;
    _picDogPhoto.TabStop = false;
    
    _lblDogographerCredit.AutoSize = true;
    _lblDogographerCredit.Location = new Point(12, 300);
    _lblDogographerCredit.Name = "_lblDogographerCredit";
    _lblDogographerCredit.Size = new Size(200, 25);
    _lblDogographerCredit.Text = "Photo by: Professional Dogographer";
    
    _btnAdopt.Location = new Point(93, 340);
    _btnAdopt.Name = "_btnAdopt";
    _btnAdopt.Size = new Size(114, 68);
    _btnAdopt.Text = "Adopt!";

    // OK, if BtnAdopt_Click is defined in main .cs file
    _btnAdopt.Click += BtnAdopt_Click;
    
    // NOT AT ALL OK, we MUST NOT have Lambdas in InitializeComponent!
    _btnAdopt.Click += (s, e) => Close();
    
    // 5. Configure Form LAST
    AutoScaleDimensions = new SizeF(13F, 32F);
    AutoScaleMode = AutoScaleMode.Font;
    ClientSize = new Size(420, 450);
    Controls.Add(_picDogPhoto);
    Controls.Add(_lblDogographerCredit);
    Controls.Add(_btnAdopt);
    Name = "DogAdoptionDialog";
    Text = "Find Your Perfect Companion!";
    ((ISupportInitialize)_picDogPhoto).EndInit();
    
    // 6. Resume
    ResumeLayout(false);
    PerformLayout();
}

#endregion

// 7. Backing fields at EOF

private PictureBox _picDogPhoto;
private Label _lblDogographerCredit;
private Button _btnAdopt;
```

**注意:** 複雑な UI 構成ロジックは、*.designer.cs* ではなく、メインの *.cs* ファイルに記述されます。

---

---

## 最新の C# 機能 (通常のコードのみ)

**`.cs` ファイル (イベント ハンドラー、ビジネス ロジック) にのみ適用されます。 `.designer.cs` または `InitializeComponent`.** では決して使用しないでください。

### スタイルガイドライン

|カテゴリー |ルール |例 |
|----------|------|----------|
|ディレクティブの使用 |グローバル | と仮定します。 `System.Windows.Forms`、`System.Drawing`、`System.ComponentModel` |
|プリミティブ |型名 | `int`、`string`、`Int32`、`String` ではない |
|インスタンス化 |ターゲット型 | `Button button = new();` |
| `var` よりも型を優先する | `var` は明らかな長い名前、または扱いにくい長い名前のみ | `var lookup = ReturnsDictOfStringAndListOfTuples()` // クリアと入力します |
|イベントハンドラ | Null 可能な送信者 | `private void Handler(object? sender, EventArgs e)` |
|イベント | Null 可能 | `public event EventHandler? MyEvent;` |
|トリビア | `return`/code ブロックの前の空行 | | の前に空行を入れてください。
| `this` 修飾子 |避ける |常に NetFX 内、それ以外の場合は曖昧さ回避または拡張メソッド用 |
|引数の検証 |いつも; .NET 8+ 用のヘルパーをスローする | `ArgumentNullException.ThrowIfNull(control);` |
|ステートメントの使用 |最新の構文 | `using frmOptions modalOptionsDlg = new(); // Always dispose modal Forms!` |

### プロパティ パターン (⚠️ 重大 - 一般的なバグの原因!)

|パターン |行動 |使用例 |メモリ |
|----------|----------|----------|----------|
| `=> new Type()` |アクセスごとに新しいインスタンスを作成します。 ⚠️ メモリリークの可能性があります! |アクセスごとの割り当て |
| `{ get; } = new()` |構築時に ONCE を作成 |用途: キャッシュ/定数 |単一の割り当て |
| `=> _field ?? Default` |計算値/動的値 |用途: 計算プロパティ |さまざま |

```csharp
// ❌ WRONG - Memory leak
public Brush BackgroundBrush => new SolidBrush(BackColor);

// ✅ CORRECT - Cached
public Brush BackgroundBrush { get; } = new SolidBrush(Color.White);

// ✅ CORRECT - Dynamic
public Font CurrentFont => _customFont ?? DefaultFont;
```

**意味の違いを理解せずに、相互に「リファクタリング」しないでください!**

### If-Else チェーンよりもスイッチ式を優先する

```csharp
// ✅ NEW: Instead of countless IFs:
private Color GetStateColor(ControlState state) => state switch
{
    ControlState.Normal => SystemColors.Control,
    ControlState.Hover => SystemColors.ControlLight,
    ControlState.Pressed => SystemColors.ControlDark,
    _ => SystemColors.Control
};
```

### イベント ハンドラーでのパターン マッチングを優先する

```csharp
// Note nullable sender from .NET 8+ on!
private void Button_Click(object? sender, EventArgs e)
{
    if (sender is not Button button || button.Tag is null)
        return;
    
    // Use button here
}
```

## Form/UserControlをゼロから設計する場合

### ファイル構造

|言語 |ファイル |継承 |
|----------|----------|---------------|
| C# | `FormName.cs` + `FormName.Designer.cs` | `Form` または `UserControl` |
| VB.NET | `FormName.vb` + `FormName.Designer.vb` | `Form` または `UserControl` |

**メイン ファイル:** ロジックおよびイベント ハンドラー
**デザイナー ファイル:** インフラストラクチャ、コンストラクター、`Dispose`、`InitializeComponent`、コントロール定義

### C# の規約

- ファイルスコープの名前空間
- ディレクティブを使用してグローバルであると仮定します
- NRT はメインの Form/UserControl ファイルで OK。コードビハインド `.designer.cs` では禁止されています
- イベント_ハンドラー_: `object? sender`
- イベント: null 可能 (`EventHandler?`)

### VB.NET の規約

- アプリケーションフレームワークを使用します。 `Program.vb`はありません。
- フォーム/ユーザーコントロール: デフォルトではコンストラクターはありません (コンパイラーは `InitializeComponent()` 呼び出しで生成します)
- コンストラクターが必要な場合は、`InitializeComponent()` 呼び出しを含めます
- クリティカル: `Friend WithEvents controlName as ControlType` コントロール バッキング フィールドの場合。
- ファイル`InitializeComponent` 内の `AddHandler` よりも、メイン コード内の `Handles` 句を含むイベント ハンドラー `Sub`s を強く優先します。

---

## クラシック データ バインディングと MVVM データ バインディング (.NET 8+)

### 重大な変更: .NET Framework と .NET 8+

|特集 | .NET Framework <= 4.8.1 | .NET 8+ |
|----------|-----------|----------|
|型付きデータセット |デザイナー対応 |コードのみ (推奨されません) |
|オブジェクトバインディング |サポートされている |強化された UI、完全にサポート |
|データ ソース ウィンドウ |利用可能 |利用できません |

### データ バインディング ルール

- オブジェクト データソース: `INotifyPropertyChanged`、`BindingList<T>` が必要ですが、MVVM CommunityToolkit の `ObservableObject` を優先します。
- `ObservableCollection<T>`: `BindingList<T>` には、両方の変更通知アプローチを統合する専用アダプターが必要です。存在しない場合は作成します。
- ソースへの一方向: WinForms DataBinding ではサポートされていません (回避策: NO-OP プロパティ セッターを使用した追加の専用 VM プロパティ)。

### オブジェクト DataSource をソリューションに追加し、ViewModel も DataSource として扱います

データソースとして型をデザイナーがアクセスできるようにするには、`Properties\DataSources\` に `.datasource` ファイルを作成します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<GenericObjectDataSource DisplayName="MainViewModel" Version="1.0" 
    xmlns="urn:schemas-microsoft-com:xml-msdatasource">
  <TypeInfo>MyApp.ViewModels.MainViewModel, MyApp.ViewModels, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null</TypeInfo>
</GenericObjectDataSource>
```

次に、Forms/UserControls の BindingSource コンポーネントを使用して、View と ViewModel の間の「Mediator」インスタンスとして DataSource タイプにバインドします。 (古典的な WinForms バインディング アプローチ)

### .NET 8以降の新しいMVVMコマンドバインディングAPI

| API |説明 |カスケード |
|-----|---------------|----------|
| `Control.DataContext` | MVVM のアンビエント プロパティ |はい (下位階層) |
| `ButtonBase.Command` | Iコマンドバインディング |いいえ |
| `ToolStripItem.Command` | Iコマンドバインディング |いいえ |
| `*.CommandParameter` |コマンドに自動的に渡されます |いいえ |

**注:** `ToolStripItem` は `BindableComponent` から派生するようになりました。

### WinForms の MVVM パターン (.NET 8+)

- WinForms プロジェクトを MVVM に作成またはリファクタリングするように求められた場合は、MVVM CommunityToolkit に基づいて ViewModel 専用のクラス ライブラリを特定するか (既に存在する場合)、作成します。
- WinForms プロジェクトからの MVVM ViewModel クラス ライブラリの参照
- 上で説明したように、オブジェクト データソースを介して ViewModel をインポートします。
- 新しい `Control.DataContext` を使用して、入れ子になった Form/UserControl シナリオのコントロール階層の下にデータ ソースとして ViewModel を渡します。
- MVVM コマンド バインディングには `Button[Base].Command` または `ToolStripItem.Command` を使用します。パラメーターを渡すには、CommandParameter プロパティを使用します。

- - 必要に応じて、`Binding` オブジェクトの `Parse` および `Format` イベントをカスタム データ変換に使用します (`IValueConverter` 回避策)。

```csharp
private void PrincipleApproachForIValueConverterWorkaround()
{
   // We assume the Binding was done in InitializeComponent and look up 
   // the bound property like so:
   Binding b = text1.DataBindings["Text"];

   // We hook up the "IValueConverter" functionality like so:
   b.Format += new ConvertEventHandler(DecimalToCurrencyString);
   b.Parse += new ConvertEventHandler(CurrencyStringToDecimal);
}
```
- 通常どおりプロパティをバインドします。
- 同じ方法でコマンドをバインドします - ViewModel はデータ ソースです。次のようにしてください:
```csharp
// Create BindingSource
components = new Container();
mainViewModelBindingSource = new BindingSource(components);

// Before SuspendLayout
mainViewModelBindingSource.DataSource = typeof(MyApp.ViewModels.MainViewModel);

// Bind properties
_txtDataField.DataBindings.Add(new Binding("Text", mainViewModelBindingSource, "PropertyName", true));

// Bind commands
_tsmFile.DataBindings.Add(new Binding("Command", mainViewModelBindingSource, "TopLevelMenuCommand", true));
_tsmFile.CommandParameter = "File";
```

---

## WinForms 非同期パターン (.NET 9 以降)

### Control.InvokeAsync オーバーロードの選択

|コードの種類 |オーバーロード |シナリオ例 |
|-----|----------|---------------------|
|同期アクション、リターンなし | `InvokeAsync(Action)` | `label.Text` を更新 |
|非同期操作、戻りなし | `InvokeAsync(Func<CT, ValueTask>)` |データのロード + UI の更新 |
|同期関数は T | を返します。 `InvokeAsync<T>(Func<T>)` |コントロール値を取得 |
|非同期操作は T | を返します。 `InvokeAsync<T>(Func<CT, ValueTask<T>>)` |非同期作業 + 結果 |

### ⚠️ ファイアアンドフォーゲットの罠

```csharp
// ❌ WRONG - Analyzer violation, fire-and-forget
await InvokeAsync<string>(() => await LoadDataAsync());

// ✅ CORRECT - Use async overload
await InvokeAsync<string>(async (ct) => await LoadDataAsync(ct), outerCancellationToken);
```

### フォーム非同期メソッド (.NET 9 以降)

- `ShowAsync()`: フォームが閉じると完了します。
返されたタスクの IAsyncState は、検索を容易にするために Form への弱い参照を保持していることに注意してください。
- `ShowDialogAsync()`: 専用メッセージキューを備えたモーダル

### クリティカル: 非同期 EventHandler パターン

- 以下のルールはすべて、`[modifier] void async EventHandler(object? s, EventArgs e)` と `async void OnLoad` や `async void OnClick` などのオーバーライドされた仮想メソッドの両方に当てはまります。
- `async void` イベント ハンドラーは、目的の非同期実装を目指す場合の WinForms UI イベントの標準パターンです。
- 重要: 非同期イベント ハンドラーでは常に `await MethodAsync()` 呼び出しを `try/catch` にネストしてください。そうしないと、プロセスがクラッシュする危険があります。

## WinForms での例外処理

### アプリケーションレベルの例外処理

WinForms は、ハンドルされない例外を処理するための 2 つの主要なメカニズムを提供します。

**AppDomain.CurrentDomain.UnhandledException:**
- AppDomain 内の任意のスレッドからの例外をキャッチします。
- アプリケーションの終了を防ぐことができません
- シャットダウン前に重大なエラーをログに記録するために使用します

**Application.ThreadException:**
- UIスレッドのみで例外をキャッチします。
- 例外を処理することでアプリケーションのクラッシュを防ぐことができます
- UI 操作での適切なエラー回復に使用します。

### 非同期/待機コンテキストでの例外ディスパッチ

非同期コンテキストで例外を再スローするときにスタック トレースを保持する場合:

```csharp
try
{
    await SomeAsyncOperation();
}
catch (Exception ex)
{
    if (ex is OperationCanceledException)
    {
        // Handle cancellation
    }
    else
    {
        ExceptionDispatchInfo.Capture(ex).Throw();
    }
}
```

**重要な注意事項:**
- `Application.OnThreadException` は UI スレッドの例外ハンドラーにルーティングし、`Application.ThreadException` を起動します。
- バックグラウンド スレッドからは決して呼び出さないでください。最初に UI スレッドにマーシャリングします。
- 未処理の例外によるプロセスの終了には、起動時に `Application.SetUnhandledExceptionMode(UnhandledExceptionMode.ThrowException)` を使用します。
- **VB の制限:** VB は catch ブロックで待機できません。回避するか、ステート マシン パターンを使用して回避します。

## 重要: CodeDOM シリアル化の管理

`Component` または `Control` から派生した型のプロパティのコード生成ルール:

|アプローチ |属性 |使用例 |例 |
|----------|----------|----------|----------|
|デフォルト値 | `[DefaultValue]` |単純なタイプ、デフォルトと一致する場合はシリアル化なし | `[DefaultValue(typeof(Color), "Yellow")]` |
|非表示 | `[DesignerSerializationVisibility.Hidden]` |実行時のみのデータ |コレクション、計算されたプロパティ |
|条件付き | `ShouldSerialize*()` + `Reset*()` |複雑な条件 |カスタム フォント、オプション設定 |

```csharp
public class CustomControl : Control
{
    private Font? _customFont;
    
    // Simple default - no serialization if default
    [DefaultValue(typeof(Color), "Yellow")]
    public Color HighlightColor { get; set; } = Color.Yellow;
    
    // Hidden - never serialize
    [DesignerSerializationVisibility(DesignerSerializationVisibility.Hidden)]
    public List<string> RuntimeData { get; set; }
    
    // Conditional serialization
    public Font? CustomFont
    {
        get => _customFont ?? Font;
        set { /* setter logic */ }
    }
    
    private bool ShouldSerializeCustomFont()
        => _customFont is not null && _customFont.Size != 9.0f;
    
    private void ResetCustomFont()
        => _customFont = null;
}
```

**重要:** `Component` または `Control` から派生した型のプロパティごとに、上記のアプローチのうち 1 つだけを使用してください。

---

## WinForms の設計原則

### コアルール

**スケーリングと DPI:**
- 適切なマージン/パディングを使用します。コントロールの絶対配置よりも、TableLayoutPanel (TLP)/FlowLayoutPanel (FLP) を優先します。
- TLP のレイアウト セル サイズ設定アプローチの優先順位は次のとおりです。
  * 行: AutoSize > パーセント > 絶対
  * 列: AutoSize > パーセント > 絶対

- 新しく追加されたフォーム/ユーザーコントロールの場合: `AutoScaleMode` とスケーリングに 96 DPI/100% を想定します。
- 既存のフォームの場合: AutoScaleMode 設定をそのままにしておきますが、座標関連のプロパティのスケーリングを考慮します。

- .NET 9 以降で DarkMode を認識する - 現在の DarkMode ステータスをクエリします: `Application.IsDarkModeEnabled`
  * 注: DarkMode では、`SystemColors` 値のみが補色パレットに自動的に変更されます。

- したがって、オーナー描画コントロール、カスタム コンテンツ ペイント、および DataGridView のテーマ/色設定は、絶対色の値を使用してカスタマイズする必要があります。

### レイアウト戦略

**分割して征服する:**
- 論理セクションには複数の TLP またはネストされた TLP を使用します。すべてを 1 つのメガグリッドに詰め込まないでください。
- メイン フォームは、SplitContainer または主要セクションに % または AutoSize-rows/cols を使用した「外部」TLP を使用します。
- 各 UI セクションは、独自のネストされた TLP、または複雑なシナリオでは、領域の詳細を処理するために設定された UserControl を取得します。

**シンプルにしてください:**
- 個々の TLP は最大 2 ～ 4 列にする必要があります
- ネストされた TLP を持つ GroupBox を使用して、明確な視覚的なグループ化を確保します。
- RadioButtons クラスター ルール: AutoGrow/AutoSize GroupBox 内の単一列、自動サイズセル TLP。
- 広いコンテンツ領域のスクロール: `AutoScroll` が有効なスクロール可能なビューでネストされたパネル コントロールを使用します。

**サイジング ルール: TLP セルの基礎**
- 列:
  * `Anchor = Left | Right` を使用したキャプション列の AutoSize。
  * コンテンツ列のパーセント、正当な理由によるパーセント配分、`Anchor = Top | Bottom | Left | Right`。
決してセルをドッキングせず、常にアンカーしてください。
  * 避けられない固定サイズのコンテンツ (アイコン、ボタン) の場合を除き、_Absolute_ 列サイズ変更モードは避けてください。
- 行:
  * 「単一行」文字 (一般的な入力フィールド、キャプション、チェックボックス) を含む行の AutoSize。
  * 複数行の TextBox のパーセント、領域のレンダリング、および残りのスペースの距離フィラー (たとえば、下のボタンの行 (OK|キャンセル))。
  * _Absolute_ 行サイズ変更モードはさらに避けてください。

- マージンは重要です: コントロールに `Margin` を設定します (最小デフォルト 3 ピクセル)。
- 注: `Padding` は、TLP セルでは効果がありません。

### 一般的なレイアウトパターン

#### 単一行の TextBox (2 列 TLP)
**最も一般的なデータ入力パターン:**
- ラベル列: AutoSize 幅
- TextBox 列: 幅 100% パーセント
- ラベル: `Anchor = Left | Right` (TextBox で垂直中央)
- TextBox: `Dock = Fill`、`Margin` を設定 (例: 全辺 3 ピクセル)

#### 複数行の TextBox またはそれ以上のカスタム コンテンツ - オプション A (2 列 TLP)
- 同じ行のラベル、`Anchor = Top | Left`
- テキスト ボックス: `Dock = Fill`、`Margin` を設定
- 行の高さ: AutoSize またはセルのサイズをパーセントで指定します (TextBox のセルのサイズを調整します)。

#### 複数行の TextBox またはそれ以上のカスタム コンテンツ - オプション B (1 列の TLP、個別の行)
- TextBox の上の専用行にラベルを付ける
- ラベル: `Dock = Fill` または `Anchor = Left`
- 次の行の TextBox: `Dock = Fill`、`Margin` を設定
- TextBox 行: AutoSize または Percent でセルのサイズを調整します

**重要:** 複数行の TextBox の場合、TextBox の内容ではなく、TLP セルによってサイズが定義されます。

### コンテナのサイズ設定 (クリティカル - クリッピングの防止)

**TLP セル内のグループボックス/パネルの場合:**
- `AutoSize = true` と `AutoSizeMode = GrowOnly` を設定する必要があります
- セルに `Dock = Fill` を入力する必要があります
- 親TLP行​​はAutoSizeである必要があります
- GroupBox/Panel 内のコンテンツはネストされた TLP または FlowLayoutPanel を使用する必要があります

**理由:** 親行が AutoSize の場合でも、固定高コンテナーはコンテンツをクリップします。コンテナーは固定サイズを報告し、サイジング チェーンを壊します。

### モーダルダイアログのボタンの配置

**パターン A - 右下のボタン (OK/キャンセルの標準):**
- FlowLayoutPanel にボタンを配置します: `FlowDirection = RightToLeft`
- ボタンとコンテンツの間に追加の Percentage Filler-Row を保持します。
- FLP はメイン TLP の最下行に配置されます
- ボタンの視覚的な順序: [OK] (左) [キャンセル] (右)

**パターン B - 右上の積み上げボタン (ウィザード/ブラウザ):**
- FlowLayoutPanel にボタンを配置します: `FlowDirection = TopDown`
- メインTLPの専用右端列のFLP
- 列: 自動サイズ
- FLP: `Anchor = Top | Right`
- 注文：[キャンセル]の上に[OK]

**使用する場合:**
- パターン A: データ入力ダイアログ、設定、確認
- パターン B: 複数ステップのウィザード、ナビゲーションが多いダイアログ

### 複雑なレイアウト

- 複雑なレイアウトの場合は、論理セクションに専用の UserControl を作成することを検討してください。
- 次に、これらの UserControl を Form/UserControl の (外側) TLP にネストし、データの受け渡しに DataContext を使用します。
- TabPage ごとに 1 つの UserControl により、タブ付きインターフェイスの Designer コードを管理しやすくなります。

### モーダルダイアログ

|側面 |ルール |
|------|------|
|ダイアログボタン |順序 -> プライマリ (OK): `AcceptButton`、`DialogResult = OK` / セカンダリ (キャンセル): `CancelButton`、`DialogResult = Cancel` |
|戦略を閉じる | `DialogResult` は DialogResult によって暗黙的に適用されるため、追加のコードは必要ありません。
|検証 | Field スコープではなく、_Form_ で実行します。 `CancelEventArgs.Cancel = true` を使用してフォーカス変更をブロックしないでください。

Form の `DataContext` プロパティ (.NET 8 以降) を使用して、モーダル データ オブジェクトを渡したり返したりします。

### レイアウトレシピ

|フォームの種類 |構造 |
|----------|----------|
|メインフォーム | MenuStrip、オプションの ToolStrip、コンテンツ領域、StatusStrip |
|簡単エントリーフォーム |データ入力フィールドの大部分が左側にあり、右側にはボタン列だけがあります。モーダルに意味のあるフォーム `MinimumSize` を設定する |
|タブ |個別のタスクのみ。最小限の数を維持し、短いタブ ラベルを維持します。

### アクセシビリティ

- 重要: 実行可能なコントロールに `AccessibleName` と `AccessibleDescription` を設定します
- `TabIndex` を介して論理コントロールのタブ順序を維持します (A11Y はコントロールの追加順序に従います)
- キーボードのみのナビゲーション、明確なニーモニック、スクリーン リーダーの互換性を確認する

### ツリービューとリストビュー

|コントロール |ルール |
|----------|----------|
|ツリービュー |デフォルトで展開された表示可能なルート ノードが必要です。
|リストビュー |列数の少ない小さなリストの場合は、DataGridView よりも優先します。
|コンテンツのセットアップ |デザイナーのコードビハインドではなくコードで生成 |
|リストビューの列 | | を入力した後、`-1` (最長コンテンツのサイズ) または `-2` (ヘッダー名までのサイズ) に設定します。
|分割コンテナ | TreeView/ListView でサイズ変更可能なペインに使用します。

### データグリッドビュー

- ダブルバッファリングを有効にした派生クラスを優先する
- ダークモードのときに色を設定します。
- 大きなデータ: ページ/仮想化 (`VirtualMode = True` と `CellValueNeeded`)

### リソースとローカリゼーション

- UI 表示の文字列リテラル定数はリソース ファイルに存在する必要があります。
- フォーム/ユーザーコントロールをレイアウトするときは、ローカライズされたキャプションの文字列の長さが異なる可能性があることを考慮してください。
- アイコン ライブラリを使用する代わりに、フォント「Segoe UI Symbol」からアイコンをレンダリングしてみてください。
- 画像が必要な場合は、フォントから希望のサイズでシンボルをレンダリングするヘルパー クラスを作成します。

## 重要なリマインダー

| # |ルール |
|---|------|
| 1 | `InitializeComponent` コードはシリアル化形式として機能します - C# ではなく XML に似ています。
| 2 | 2 つのコンテキスト、2 つのルール セット - デザイナー コードビハインドと通常のコード |
| 3 |コードを生成する前にフォーム/コントロール名を検証する |
| 4 | `InitializeComponent` のコーディング スタイル ルールに従う |
| 5 |デザイナー ファイルは NRT 注釈を使用しません。
| 6 |通常のコードのみの最新の C# 機能 |
| 7 |データ バインディング: ViewModel をデータソースとして扱い、`Command` プロパティと `CommandParameter` プロパティを覚えておいてください。
