---
description: 'CommunityToolkit.Mvvm (MVVM ツールキット) WPF、WinUI 3、.NET MAUI、Uno Platform、および Avalonia にわたる ViewModel、コマンド、メッセージング、検証、および DI のコーディング規約。'
applyTo: '**/*.cs, **/*.xaml, **/*.csproj'
---

# CommunityToolkit.Mvvm (MVVM ツールキット)

これらのルールは、プロジェクトが `CommunityToolkit.Mvvm` を参照するたびに適用されます。
詳細なリファレンスとエンドツーエンドの例については、`mvvm-toolkit` スキルをロードしてください。

## パッケージと言語

- `.csproj` の `CommunityToolkit.Mvvm` 8.x (またはそれ以降) を参照してください。しないでください
新しいプロジェクト用に従来の `Microsoft.Toolkit.Mvvm` (7.x) をインストールします。
- C# `LangVersion` はソース ジェネレーター (最新の SDK のデフォルト) をサポートする必要があります。

## ViewModel基本クラス

- デフォルトでは `ObservableObject` から ViewModel を継承します。
- ViewModel が必要な場合にのみ `ObservableValidator` を使用してください
`INotifyDataErrorInfo` (フォーム、設定、入力検証)。
- ViewModel が送信または受信する場合にのみ `ObservableRecipient` を使用します
`IMessenger` メッセージ。
- ツールキットのいずれかを使用する場合は、`INotifyPropertyChanged` を決して手動で実装しないでください。
基本クラスを使用できます。タイプがツールキット ベースから継承できない場合
(カスタム コントロールなど)、クラス レベルの `[ObservableObject]` を適用するか、
代わりに `[INotifyPropertyChanged]` 属性を使用します。

## プロパティ

- `[ObservableProperty]` を使用するすべての型を `partial` として宣言します (および
ネストされている場合は、すべての囲み型)。
- `[ObservableProperty]` を `name`、`_name`、またはという名前のプライベート フィールドに適用します
`m_name` — PascalCase は使用しないでください。ジェネレーターにパブリック プロパティを発行させます。
- 手動 `SetProperty(ref field, value)` ボイラープレートを作成しないでください。
フィールドは `[ObservableProperty]` に該当します。
- `[NotifyPropertyChangedFor(nameof(Derived))]` を使用して変更を発生させます
派生/計算されたプロパティの通知。
- `[NotifyCanExecuteChangedFor(nameof(XxxCommand))]` を使用してコマンドを実行します
入力が変更された場合は `CanExecute` を再評価します。
- `OnXxxChanging` / `OnXxxChanged` の部分メソッド フックを実装します。
プロパティ変更の副作用 — 自分のプロパティをサブスクライブしないでください
`PropertyChanged` イベント。
- `[property: SomeAttribute]` を使用して属性を転送します (例:
`[JsonIgnore]`、`[JsonPropertyName(...)]`) を生成されたプロパティに適用します。

## コマンド

- インスタンス メソッドでは手動で構築された `[RelayCommand]` を使用します
`RelayCommand` / `AsyncRelayCommand` インスタンス。
- `[RelayCommand]` メソッドは `void` または `Task` (または `Task<T>`) を返す必要があります。
`async void` は決して使用しないでください。例外は検出されなくなります。
- キャンセル可能な非同期作業の場合は、`CancellationToken` パラメータを宣言し、
オプションで `IncludeCancelCommand = true` を設定して、ペアになったものを公開します
@@コード0@@。
- `CanExecute = nameof(...)` と `[NotifyCanExecuteChangedFor]` を使用します。
ボタンの有効化/無効化状態を同期させるための入力。
- デフォルトの `AllowConcurrentExecutions` から `false` (デフォルト)。設定のみ
`true` 重複した呼び出しが明示的に安全で意図されている場合。
- デフォルトのエラー ポリシーは await-and-rethrow です。設定のみ
`FlowExceptionsToTaskScheduler = true` UI がバインドされるとき
`ExecutionTask` はエラー状態をレンダリングします。

## メッセージング

- デフォルトは `WeakReferenceMessenger.Default` です。にのみ切り替えます
`StrongReferenceMessenger.Default` プロファイリングでメッセンジャーが
ホット、および生涯保証を文書化します。
- `(recipient, message)` ラムダ フォームを使用してハンドラーを登録します。
`static` 修飾子 — ラムダで `this` をキャプチャしないでください。
- `ObservableRecipient` 上の `IRecipient<TMessage>` インターフェイスを優先します
ViewModel なので、`RegisterAll(this)` がすべてを自動的に接続します。
@@コード0@@。
- アクティベーション時に `IsActive = true` を設定し (例: `OnNavigatedTo`)、
非アクティブ化の `IsActive = false` (例: `OnNavigatedFrom`)。
- メッセージ配信時に継承は考慮されません - それぞれを登録します
具体的なメッセージタイプを明示的に指定します。
- チャネル トークン (`int` / `string` / `Guid` オーバーロード) を使用してスコープを設定します
複数のコンシューマがサブシステムまたはウィンドウにメッセージを送信する場合
それ以外の場合は衝突します。

## 依存関係の注入

- サービスと ViewModel には `Microsoft.Extensions.DependencyInjection` を使用します
登録。 .NET 汎用ホストを優先する
(`Host.CreateDefaultBuilder()`) 構成、ロギング、スコープ
検証は自動的に接続されます。
- サービスと ViewModel を構成ルートに登録します (通常は
`App.xaml.cs`)。ページのルート ViewModel をページ内の DI から解決します。
コンストラクターまたはナビゲーション フレームワーク経由で。
- コンストラクターを通じてサービスと子 ViewModel を挿入します。電話しないでください
ViewModel、サービス、またはその他の内部からの `Ioc.Default.GetService<T>()`
DIコンテナが構築できるタイプ。
- 寿命:
  - `AddSingleton<T>()` — シェル/メインウィンドウ VM、設定、ファイル/HTTP
サービス、共有 `IMessenger`。
  - `AddTransient<T>()` — ページごとまたはドキュメントごとの VM。
  - `AddScoped<T>()` — `IServiceScope` を明示的に使用する場合のみ。めったに
クライアント アプリで必要になります。
- `IMessenger`を一度登録してください
(`services.AddSingleton<IMessenger>(WeakReferenceMessenger.Default)`)
`ObservableRecipient(messenger)` コンストラクターを介してそれを注入します。

## 検証

- `ObservableValidator` に加えて `[NotifyDataErrorInfo]` と DataAnnotation を使用します
属性 (`[Required]`、`[Range]`、`[EmailAddress]`、`[MinLength]`、
`[MaxLength]`、`[CustomValidation]`)。
- フォームを送信する前に `ValidateAllProperties()` を呼び出してください。チェック
`HasErrors`、`true` の場合は救済します。
- 送信が成功した後、`ClearAllErrors()` を使用してエラー状態をリセットするか、
フォームをリセットするとき。
- クロスプロパティ ルールについては、`ValidateProperty(value, nameof(Other))` を呼び出してください。
変更されたプロパティの `OnXxxChanged` フックから。

## XAML

- WinUI 3 / UWP の場合は、`{x:Bind}` (コンパイルされたバインディング) を優先します。
@@コード0@@。 `Mode=OneWay` または `Mode=TwoWay` を明示的に設定します — `{x:Bind}`
デフォルトは `OneTime` です。
- `Command="{x:Bind ViewModel.SaveCommand}"` を直接バインドします
生成されたコマンドプロパティ。
- バインド非同期コマンドのステータス (`IsRunning`、`ExecutionTask.Status`、
`ExecutionTask.Exception`) の代わりに進行状況/エラーを表示します
UIスレッドをブロックしています。

## 避けるべきこと

- `[ObservableProperty] private string Name;` — PascalCase フィールドが衝突します
生成されたプロパティを使用します。 lowerCamelを使用します。
- `RaisePropertyChanged(nameof(X))` を同時に手動で呼び出します
`[ObservableProperty]` — 重複した通知を生成します。
- ViewModel コンストラクター内から `Ioc.Default.GetService<T>()` —
依存関係を隠し、単体テストを中断します。
- `StrongReferenceMessenger` `OnDeactivated` / `UnregisterAll` なし —
受信者をピン留めし、漏洩します。
- メッセンジャーラムダで `this` をキャプチャする — クロージャーの割り当てと
一生の混乱。常に `(r, m) => r.OnX(m)` を `static` とともに使用してください。
- `[RelayCommand]` メソッドの `async void` — 代わりに `Task` を返します。
- `[ObservableProperty]` フィールドによって保持されている同じ参照を変更する —
等価比較子は `true` を返し、変更通知は発行されません。
代わりにインスタンスを置き換えます。
- `ObservableValidator` と `ObservableRecipient` の両方から継承 —
不可能です。コンポジションを使用する (`IMessenger` を挿入するか実装する)
手動で検証します)。