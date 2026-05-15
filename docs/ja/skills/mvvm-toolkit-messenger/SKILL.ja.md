---
name: mvvm-toolkit-messenger
description: 'CommunityToolkit.Mvvm ViewModel (または任意のオブジェクト) 間の分離された通信のためのメッセンジャー パブリッシュ/サブスクライブ。 WeakReferenceMessenger と StrongReferenceMessenger、IRecipient<TMessage>、RequestMessage<T> / AsyncRequestMessage<T> / CollectionRequestMessage<T>、ValueChangedMessage<T>、チャネル (トークン)、および ObservableRecipient のアクティブ化ライフサイクルについて説明します。 WPF、WinUI 3、.NET MAUI、Uno、および Avalonia 全体で使用します。'
---

# CommunityToolkit.Mvvm メッセンジャー

共有を強制しない ViewModel (または任意のオブジェクト) のパブリッシュ/サブスクライブ メッセージング
参考グラフ。 `CommunityToolkit.Mvvm` 8.x の一部。

> **TL;DR.** デフォルトは `WeakReferenceMessenger.Default` です。ハンドラーを登録する
> `(recipient, message)` ラムダと `static` 修飾子を使用すると、
> `this` を決してキャプチャしないでください。 `ObservableRecipient` から継承して切り替えます
> `IsActive` をアクティブ化/非アクティブ化して、自動登録/登録解除を取得します。

---

## このスキルをいつ使用するか

- 2 つ以上の ViewModel がイベント (ログイン、テーマの変更、
保存、ナビゲーション)、相互参照を保持せずに
- ViewModel は別の VM に値を要求する必要があります (リクエスト/応答)
- チャネル トークンを使用してイベントのスコープをサブシステムまたはウィンドウに設定している
- 「ハンドラーが起動しない」または弱参照受信者の存続期間を診断する
問題

ソース ジェネレーター、基本クラス、コマンドについては、**`mvvm-toolkit`** を参照してください。
スキル。 DI 配線（`IMessenger` インスタンスの登録）については、を参照してください。
**`mvvm-toolkit-di`**。

---

## 実装を選択してください

|タイプ |いつ |
|------|------|
| `WeakReferenceMessenger.Default` | **デフォルト。** 受信者は弱く保持されます。登録中であっても GC の対象となります。内部トリミングはフル GC 中に実行されます。手動 `Cleanup()` は必要ありません。 |
| `StrongReferenceMessenger.Default` |プロファイラーは、メッセンジャーがホットであり、割り当てが重要であることを示しています。受信者は `Unregister` を送信するまで固定されます。登録解除を忘れると流出します。 |
|カスタム `IMessenger` インスタンス |ウィンドウごと/スコープごと (例: アプリ ウィンドウごとに 1 つのメッセンジャー)。直接構築し、DI 経由で注入します。 |

`ObservableRecipient` のパラメータなしのコンストラクターは次を使用します
@@コード0@@。別の `IMessenger` をその
オーバーライドするコンストラクター。

---

## メッセージを定義する

ツールキットには基本クラスが同梱されています。どのクラスも機能します。
```csharp
using CommunityToolkit.Mvvm.Messaging.Messages;

// Single-payload broadcast
public sealed class LoggedInUserChangedMessage(User user)
    : ValueChangedMessage<User>(user);

// Custom shape (records are great for this)
public sealed record ThemeChangedMessage(AppTheme NewTheme);

// Empty signal
public sealed record RefreshRequestedMessage;
```

---

## 宛先を登録する

### ラムダスタイル (推奨)
```csharp
WeakReferenceMessenger.Default.Register<MyViewModel, ThemeChangedMessage>(
    this,
    static (recipient, message) => recipient.OnThemeChanged(message.NewTheme));
```

`static` 修飾子は、誤ったクロージャーの割り当てを防ぎ、
ラムダの `this` — 代わりに `recipient` パラメーターを使用します。

### `IRecipient<TMessage>` インターフェイス スタイル
```csharp
public sealed class MyViewModel : ObservableRecipient,
    IRecipient<ThemeChangedMessage>,
    IRecipient<RefreshRequestedMessage>
{
    public void Receive(ThemeChangedMessage message) { /* ... */ }
    public void Receive(RefreshRequestedMessage message) { /* ... */ }
}
```

`ObservableRecipient.OnActivated()` は `Messenger.RegisterAll(this)` を呼び出します。
これは、型によって実装されたすべての `IRecipient<T>` インターフェイスをサブスクライブします。
`ObservableRecipient` を使用していない場合は、手動で登録します。
```csharp
WeakReferenceMessenger.Default.RegisterAll(this);
```

---

## メッセージを送信する
```csharp
WeakReferenceMessenger.Default.Send(new ThemeChangedMessage(AppTheme.Dark));

// Empty payloads use the parameterless overload:
WeakReferenceMessenger.Default.Send<RefreshRequestedMessage>();
```

---

## チャネル (トークン)

トークン (等価な任意の) を使用して、メッセージのスコープをサブシステムまたはウィンドウに設定します。
値 — `int`、`string`、`Guid`):
```csharp
const int LeftPaneChannel = 1;

WeakReferenceMessenger.Default.Register<MyViewModel, RefreshRequestedMessage, int>(
    this, LeftPaneChannel,
    static (r, _) => r.RefreshLeft());

WeakReferenceMessenger.Default.Send(new RefreshRequestedMessage(), LeftPaneChannel);
```

トークンなしで送信されたメッセージは、デフォルトの共有チャネルを使用します。
**ではありません** チャネルスコープの受信者には配信されません。

---

## リクエスト・返信

受信者が値を提供する問い合わせスタイルのシナリオの場合、
送信者は、`RequestMessage<T>` ファミリを使用します。

### 同期リクエスト
```csharp
public sealed class CurrentUserRequest : RequestMessage<User> { }

WeakReferenceMessenger.Default.Register<UserService, CurrentUserRequest>(
    this,
    static (r, m) => m.Reply(r.CurrentUser));

User user = WeakReferenceMessenger.Default.Send<CurrentUserRequest>();
```

`CurrentUserRequest` から `User` への暗黙的な変換は、そうでない場合にスローされます。
受信者は `Reply` と呼ばれました。最初に確認するメッセージをキャプチャします。
```csharp
var request = WeakReferenceMessenger.Default.Send<CurrentUserRequest>();
if (request.HasReceivedResponse)
    User user = request.Response;
```

### 非同期リクエスト
```csharp
public sealed class CurrentUserRequest : AsyncRequestMessage<User> { }

WeakReferenceMessenger.Default.Register<UserService, CurrentUserRequest>(
    this,
    static (r, m) => m.Reply(r.GetCurrentUserAsync()));

User user = await WeakReferenceMessenger.Default.Send<CurrentUserRequest>();
```

### コレクションリクエスト（ファンイン）

`CollectionRequestMessage<T>` と `AsyncCollectionRequestMessage<T>` を収集します
応答するすべての受信者からの `Reply`:
```csharp
public sealed class OpenDocumentsRequest : CollectionRequestMessage<Document> { }

var docs = WeakReferenceMessenger.Default.Send<OpenDocumentsRequest>();
foreach (Document doc in docs) { /* ... */ }
```

---

## ライフサイクル

`WeakReferenceMessenger`を使用している場合でも、受信者が受信した場合は明示的に登録を解除してください
削除されています。無効なエントリが削除され、パフォーマンスが向上します。
```csharp
WeakReferenceMessenger.Default.Unregister<ThemeChangedMessage>(this);
WeakReferenceMessenger.Default.Unregister<ThemeChangedMessage, int>(this, LeftPaneChannel);
WeakReferenceMessenger.Default.UnregisterAll(this);
```

`ObservableRecipient.OnDeactivated()` の場合、これは自動的に行われます。
`IsActive` は `false` に切り替わります。アクティベーションフックから設定します。
```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    ViewModel.IsActive = true;
}

protected override void OnNavigatedFrom(NavigationEventArgs e)
{
    ViewModel.IsActive = false;
    base.OnNavigatedFrom(e);
}
```

---

## よくある落とし穴

1. **ラムダで `this` をキャプチャします。** `(r, m) => OnX(m)` を暗黙的に取得します
`this` をキャプチャします。クロージャを割り当て、ライフタイムを混乱させます。常に使用する
`(r, m) => r.OnX(m)` と `static`。
2. **`Unregister`.** なしの強参照受信者 ** あり
`StrongReferenceMessenger`、受信者 (およびそのオブジェクト グラフ全体)
永遠に固定されたままになります。 `ObservableRecipient` から継承するかのいずれか
(`OnDeactivated` で自動登録解除) または `UnregisterAll(this)` を呼び出します。
3. **継承されたメッセージ タイプ。** `BaseMessage` に登録されたハンドラーは次のとおりです。
**ではありません** `DerivedMessage : BaseMessage` に対して呼び出されています。それぞれ登録する
コンクリートタイプ。
4. **メッセンジャー インスタンスが間違っています。** `WeakReferenceMessenger.Default` 経由で送信しています
挿入されたウィンドウごとのメッセンジャーを介して登録すると、メッセージが
決して到着しません。どこでも同じ `IMessenger` を使用します (通常はインジェクトします)
`ObservableRecipient(messenger)`経由）。
5. **`OnActivated` は実行されません。** `ObservableRecipient` は登録のみです
`IsActive` が `false` から `true` に切り替わるときの `IRecipient<T>` ハンドラー。
6. **クロススレッド更新。** メッセンジャーはスレッドに依存しません。もし
ハンドラーは UI を更新し、手動でマーシャルします
(`DispatcherQueue.TryEnqueue` / `Dispatcher.BeginInvoke`)。

---

## 複数のメッセンジャー (ウィンドウごとのスコープ)
```csharp
services.AddSingleton<IMessenger>(WeakReferenceMessenger.Default); // app-wide
services.AddScoped<WindowScopedMessenger>();                       // per-window
```

適切な `IMessenger` を ViewModel コンストラクターに挿入します。
```csharp
public sealed partial class WindowViewModel(IMessenger messenger)
    : ObservableRecipient(messenger) { }
```

これにより、ブロードキャストが単一のウィンドウに分離されます。マルチウィンドウに便利です。
デスクトップ アプリ (WinUI 3、WPF、MAUI デスクトップ、Avalonia)。

---

## 参考文献

|トピック |ファイル |
|------|------|
|完全な詳細 (チャネル/ライフサイクルの例、診断の詳細) | [`references/messenger-patterns.md`](references/messenger-patterns.md) |

外部の：

- メッセンジャーのドキュメント: <https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/messenger>
- `WeakReferenceMessenger` API: <https://learn.microsoft.com/en-us/dotnet/api/communitytoolkit.mvvm.messaging.weakreferencemessenger>
- 出典: <https://github.com/CommunityToolkit/dotnet>