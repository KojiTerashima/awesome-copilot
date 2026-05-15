---
name: mvvm-toolkit
description: 'CommunityToolkit.Mvvm (MVVM ツールキット) コア: ソース ジェネレーター ([ObservableProperty]、[RelayCommand]、[NotifyPropertyChangedFor]、[NotifyCanExecuteChangedFor]、[NotifyDataErrorInfo])、基本クラス (ObservableObject / ObservableValidator / ObservableRecipient)、コマンド (RelayCommand / AsyncRelayCommand)、および検証。コンパニオン スキル: パブリッシュ/サブスクライブ用の mvvm-toolkit-messenger、Microsoft.Extensions.DependencyInjection ワイヤリング用の mvvm-toolkit-di。 WPF、WinUI 3、MAUI、Uno、Avalonia で動作します。'
---

# CommunityToolkit.Mvvm (コア)

ViewModel、プロパティ、
コマンド、または `CommunityToolkit.Mvvm` 8.x を使用するアプリでの検証。

> **コンパニオン スキル。** `IMessenger` の **`mvvm-toolkit-messenger`** をロードします
> パブ/サブのパターン。 **`mvvm-toolkit-di`** をロードします
> `Microsoft.Extensions.DependencyInjection` の統合。

> **簡単な要約。** `partial` のプライベート フィールドに関する `[ObservableProperty]`
> クラス。 `[RelayCommand]` インスタンス メソッドの場合。から継承する
> `ObservableObject` (入力フォームの場合は `ObservableValidator`、
> `IMessenger` を使用する場合は `ObservableRecipient`)。

---

## パッケージとセットアップ
```xml
<ItemGroup>
  <PackageReference Include="CommunityToolkit.Mvvm" Version="8.*" />
</ItemGroup>
```

ターゲット: `netstandard2.0`、`netstandard2.1`、`net6.0`+。 .NET、.NET上で動作します
フレームワーク、モノ。ソース ジェネレーターは同じ NuGet に同梱されており、追加のものはありません
アナライザーのリファレンスが必要です。

名前空間:
```csharp
using CommunityToolkit.Mvvm.ComponentModel;   // ObservableObject, [ObservableProperty]
using CommunityToolkit.Mvvm.Input;             // [RelayCommand], RelayCommand, AsyncRelayCommand
```

> **普遍的なルール。** `[ObservableProperty]` または
> `[RelayCommand]` — ネストされている場合、すべての囲み型は次のようにする必要があります
> `partial` を宣言しました。それがないと、発電機は
> `MVVMTK0008` / `MVVMTK0042`。

---

## ソースジェネレーターのチートシート

|属性 |適用先 | |を生成します
|----------|-----------|----------|
| `[ObservableProperty]` |プライベートフィールド |パブリック `INotifyPropertyChanged` プロパティ + `OnXxxChanging`/`OnXxxChanged` 部分メソッド フック |
| `[NotifyPropertyChangedFor(nameof(Other))]` |観測可能なフィールド |リストされたプロパティに対して `PropertyChanged` も発生します。
| `[NotifyCanExecuteChangedFor(nameof(MyCommand))]` |観測可能なフィールド |変更時に `MyCommand.NotifyCanExecuteChanged()` を呼び出します |
| `[NotifyDataErrorInfo]` | `ObservableValidator` の監視可能なフィールド |セッターから `ValidateProperty(value)` を呼び出します |
| `[NotifyPropertyChangedRecipients]` | `ObservableRecipient` の監視可能なフィールド | `Broadcast(old, new)` 変更後 |
| `[RelayCommand]` |インスタンスメソッド |怠惰な `RelayCommand` / `AsyncRelayCommand` が `IRelayCommand` / `IAsyncRelayCommand` として公開される |
| `[RelayCommand(CanExecute = nameof(CanX))]` |インスタンスメソッド | `CanExecute` をメソッドまたはプロパティに接続します。
| `[RelayCommand(IncludeCancelCommand = true)]` | `CancellationToken` を使用した非同期メソッド | `XxxCancelCommand` も生成します。
| `[RelayCommand(AllowConcurrentExecutions = true)]` |非同期メソッド |キューに入れられた/並列呼び出しを許可します (実行中はデフォルトで無効になります)。
| `[RelayCommand(FlowExceptionsToTaskScheduler = true)]` |非同期メソッド |待機して再スローする代わりに、`ExecutionTask` を介して例外を表示します。
| `[property: SomeAttr]` |監視可能なフィールドまたは `[RelayCommand]` メソッド | `SomeAttr` を生成されたプロパティに転送します (例: `[JsonIgnore]`)。

**名前。** フィールド `name` / `_name` / `m_name` → `Name`。メソッド `LoadAsync` →
`LoadCommand` (`Async` 接尾辞は削除され、先頭の `On` も削除されます)
剥がされた）。

詳細については、[`references/source-generators.md`](references/source-generators.md) を参照してください。
生成されたコードのサンプルを含む完全な属性リファレンス。

---

## ビューモデルパターン

### 単純な観察可能なプロパティ
```csharp
public partial class ContactViewModel : ObservableObject
{
    [ObservableProperty]
    private string? name;
}
```

### フック: `OnXxxChanging` / `OnXxxChanged`
```csharp
[ObservableProperty]
private string? name;

partial void OnNameChanged(string? value) =>
    Logger.LogInformation("Name changed to {Name}", value);
```

単一引数の `(value)` と 2 つの引数 `(oldValue, newValue)` の両方のオーバーロード
利用可能です。必要なものだけを実装してください。未実装のフックは
コンパイラによって無視されます (実行時コストはゼロ)。

### 依存プロパティ + 依存コマンド
```csharp
[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FullName))]
[NotifyCanExecuteChangedFor(nameof(SaveCommand))]
private string? firstName;

[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FullName))]
[NotifyCanExecuteChangedFor(nameof(SaveCommand))]
private string? lastName;

public string FullName => $"{FirstName} {LastName}".Trim();
```

### 観察不可能なモデルのラッピング
```csharp
public sealed class ObservableUser(User user) : ObservableObject
{
    public string Name
    {
        get => user.Name;
        set => SetProperty(user.Name, value, user, (u, n) => u.Name = n);
    }
}
```

静的ラムダ (キャプチャされた状態なし) を渡して、呼び出しを割り当てフリーに保ちます。

---

## コマンド
```csharp
[RelayCommand]
private void Refresh() => Items.Reset();

[RelayCommand]
private async Task LoadAsync()
{
    foreach (var item in await service.GetItemsAsync())
        Items.Add(item);
}

[RelayCommand(IncludeCancelCommand = true)]
private async Task DownloadAsync(CancellationToken token)
{
    await using var stream = await http.GetStreamAsync(url, token);
    // ...
}

[RelayCommand(CanExecute = nameof(CanSave))]
private Task SaveAsync() => repo.SaveAsync(Name!);

private bool CanSave() => !string.IsNullOrWhiteSpace(Name);
```

手動の `RelayCommand` / `AsyncRelayCommand` コンストラクターのみに適用されます
コマンドの有効期間を明示的に所有する必要がある場合、またはコマンドを次のように構成する必要がある場合
重要な情報源。属性スタイルは、ケースの最大 95% をカバーします。

[`references/relaycommand-cookbook.md`](references/relaycommand-cookbook.md) を参照してください。
同期 / 非同期 / キャンセル可能 / 同時実行 / エラー表面化レシピ用。

---

## 基本クラスの選択

|基本クラス | | の場合に使用します。
|-----------|-----------|
| `ObservableObject` |デフォルト。 `INotifyPropertyChanged` + `INotifyPropertyChanging` + `SetProperty` オーバーロード + `Task` プロパティの `SetPropertyAndNotifyOnCompletion` |
| `ObservableValidator` | VM には `INotifyDataErrorInfo` (フォーム、設定入力) が必要です。
| `ObservableRecipient` | VM は `IMessenger` メッセージを送信または受信します。 **`mvvm-toolkit-messenger`** スキルを参照してください。

C# は単一継承です: `ObservableValidator` および `ObservableRecipient`
どちらも `ObservableObject` を拡張するため、結合する必要があります
(例: `IMessenger` を `ObservableValidator` に挿入します)。

---

## 検証
```csharp
using System.ComponentModel.DataAnnotations;

public sealed partial class RegistrationViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required, MinLength(2), MaxLength(100)]
    private string? name;

    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required, EmailAddress]
    private string? email;

    [RelayCommand]
    private void Submit()
    {
        ValidateAllProperties();
        if (HasErrors) return;
        // submit...
    }
}
```

その他のエントリ ポイント: `TrySetProperty`、`ValidateProperty(value, name)`、
`ClearAllErrors()`、`GetErrors(propertyName)`。カスタムルールのサポート
`[CustomValidation]` メソッドとカスタム `ValidationAttribute` サブクラス。

詳細については、[`references/validation.md`](references/validation.md) を参照してください。
バリデータの表面積。

---

## 主な落とし穴

1. **`partial`.** クラス (およびそれを囲むすべての型) は次のようにする必要があります。
@@コード0@@。コンパイル エラー `MVVMTK0008` / `MVVMTK0042`。
2. **PascalCase フィールド名。** `[ObservableProperty] private string Name;`
生成されたプロパティと衝突します。 `name`、`_name`、または `m_name` を使用します。
3. **`async void` `[RelayCommand]`.** ジェネレーターはラップのみです
`Task` - メソッドを `IAsyncRelayCommand` として返します。 `async void` になります
同期 `RelayCommand` と例外は観察されません。必ず戻る
@@コード0@@。
4. **`[NotifyCanExecuteChangedFor]` を忘れてください。** [保存] ボタンはそのままです
`CanSave()` が `true` を返すようになったとしても、無効になります。
5. **`[ObservableProperty]` が保持する同じ参照を変更する
field.** `EqualityComparer<T>.Default` は `true` を返しますが、通知はありません
火災が発生します。インスタンスを変更するのではなく、インスタンスを置き換えます。

完全な診断テーブル (`MVVMTK0xxx`) とその他の落とし穴については、を参照してください。
[`references/troubleshooting.md`](references/troubleshooting.md)。

---

## エンドツーエンドのミニウォークスルー

ジェネレーター + コマンド + をデモする 2 ペインの Notes アプリ
`[NotifyCanExecuteChangedFor]`:
```csharp
public sealed partial class NoteViewModel(INotesService notes,
    IMessenger messenger) : ObservableRecipient(messenger)
{
    [ObservableProperty]
    [NotifyCanExecuteChangedFor(nameof(SaveCommand))]
    [NotifyCanExecuteChangedFor(nameof(DeleteCommand))]
    private string? filename;

    [ObservableProperty]
    [NotifyCanExecuteChangedFor(nameof(SaveCommand))]
    private string? text;

    [RelayCommand(CanExecute = nameof(CanSave))]
    private Task SaveAsync()
    {
        Messenger.Send(new NoteSavedMessage(Filename!));
        return notes.SaveAsync(Filename!, Text!);
    }

    [RelayCommand(CanExecute = nameof(CanDelete))]
    private Task DeleteAsync() => notes.DeleteAsync(Filename!);

    private bool CanSave() =>
        !string.IsNullOrWhiteSpace(Filename) && !string.IsNullOrEmpty(Text);
    private bool CanDelete() => !string.IsNullOrWhiteSpace(Filename);
}
```

完全なサンプル (DI 配線、分離コードの表示、XAML、単体テスト) については、次を参照してください。
[`references/end-to-end-walkthrough.md`](references/end-to-end-walkthrough.md)。

---

## 参考文献とコンパニオンスキル

|トピック |どこ |
|------|------|
|ソース ジェネレーター属性リファレンス | [`references/source-generators.md`](references/source-generators.md) |
| RelayCommand レシピ | [`references/relaycommand-cookbook.md`](references/relaycommand-cookbook.md) |
|検証の詳細 | [`references/validation.md`](references/validation.md) |
|メモアプリの完全なウォークスルー | [`references/end-to-end-walkthrough.md`](references/end-to-end-walkthrough.md) |
| `MVVMTK0xxx` 診断と落とし穴 | [`references/troubleshooting.md`](参考文献/troubleshooting.md) |
| **メッセンジャー パブ/サブ** |コンパニオンスキル: **`mvvm-toolkit-messenger`** |
| **`Microsoft.Extensions.DependencyInjection` 配線** |コンパニオンスキル: **`mvvm-toolkit-di`** |

外部ソース:

- ツールキットの概要: <https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/>
- WinUI MVVM ツールキット チュートリアル: <https://learn.microsoft.com/en-us/windows/apps/tutorials/winui-mvvm-toolkit/intro>
- 出典: <https://github.com/CommunityToolkit/dotnet>
- サンプル: <https://github.com/CommunityToolkit/MVVM-Samples>