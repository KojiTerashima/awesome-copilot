---
description: 'Razor Pages コンポーネントとアプリケーション パターン'
applyTo: '**/*.cshtml, **/*.cshtml.cs'
---

## Razor ページのコード スタイルと構造

- 慣用的で効率的な Razor ページと C# を作成します。
- ページに組み込まれた MVC コントローラー パターンではなく、ハンドラー ベースの PageModel という、フレームワークの構築に基づいた規則に従ってください。
- PageModel はリクエスト/レスポンス オーケストレーションに重点を置いてください。ビジネス ロジックは、挿入されたドメイン サービスに属します。
- 自明なハンドラはインラインに留まることができます。多数のハンドラーと依存関係を含むページの場合は、MediatR などのメディエーターを使用してください。
- ハンドラーがリクエスト パイプラインをブロックしないように、エンドツーエンドで async/await を使用します。

## 命名規則

- PageModel クラス、ハンドラー メソッド、およびパブリック メンバー (`CreateModel`、`OnPostAsync`、`OnPostDeleteAsync`) 用の PascalCase。
- プライベート フィールドとローカルにはキャメルケース。.NET 規約 (`_context`、`_logger`) に従ってプライベート フィールドに `_` プレフィックスが付きます。
- インターフェイス名は「I」で始まります (`IEmailService`)。
- 名前付きハンドラーは、ルーティング時に `OnPost`/`Async` 接辞を削除します。 `OnPostJoinListAsync` は `handler=JoinList` として到達されます。

## モデルバインディングとオーバーポスト

- `[BindProperty]` を EF またはドメイン エンティティに直接置かないでください。攻撃者は、`IsAdmin` や `Secret` などの追加フィールドを投稿することができ、フォームがそれらをレンダリングしない場合でも、バインダーは喜んでそれらを設定します。
- ページが受け入れることを許可されているプロパティのみを公開する専用の入力モデルまたはビュー モデルにバインドして、エンティティにマップします。
- 特に編集シナリオでは、プロパティの明示的な許可リストを指定した `TryUpdateModelAsync<T>` も別のオプションです。
- 編集には `[Bind]` を避けてください。除外されたプロパティは、そのままではなく `default(T)` にリセットされますが、これは希望通りになることはほとんどありません。入力モデルを優先します。
- `[BindProperty(SupportsGet = true)]` を広く有効にしないでください。 Razor Pages は、理由によりデフォルトで GET バインディングをスキップします。プロパティごとにオプトインし、何が入っているかを検証します。
- カスタム タイプ (厳密に型指定された ID を含む) の場合は、`TryParse` または `TypeConverter` を実装して、ルートおよびクエリ値からバインドできるようにします。これがないと、バインダーはそれらを複合型として扱い、バインディングは静かに失敗します。午後を無駄にするバグの 1 つ。
- `[BindRequired]` と `[Required]` は同じものではありません。投稿されたフォームにソース値が *存在しない*場合、`[BindRequired]` エラーが発生します。 `[Required]` は、バインドされた値が null/空ではないことを検証します。 `[BindRequired]` は、JSON と XML が代わりに入力フォーマッタを通過するため、フォーム バインディングにのみ適用されます。

## ハンドラーメソッドとリクエストフロー

- POST が成功した場合は、常に Post-Redirect-Get を使用してください。 `RedirectToPage("./Index")` を返します。`Page()` は返しません。成功時に `Page()` が返されるということは、ブラウザを更新してフォームを再送信することを意味します。
```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid) return Page();          // re-render on error
    await _service.CreateAsync(Input);
    return RedirectToPage("./Index");                // PRG on success
}
```

- すべての永続パスを `if (!ModelState.IsValid) return Page();` で保護します。クライアント側の検証はバイパスできます。サーバーには権限があります。
- 単一リクエストのルートまたはクエリ値には、ハンドラー パラメーター (`OnGetAsync(int id)`) を使用します。検証エラーのビューに往復する必要がある POST データには `[BindProperty]` を使用します。
- 名前付きハンドラー (`OnPostDeleteAsync`、`OnPostApproveAsync`) には、送信ボタンに `asp-page-handler` タグ ヘルパーが必要です。これがないと、通常のボタンは `OnPostAsync` または 404 に戻ります。
- `OnGet` が負荷の高い作業を行う場合は、軽量の `OnHead` を追加します。それ以外の場合、Razor Pages は HEAD 要求の場合は `OnGet` にフォールバックするため、すべてのプローブが GET コストの全額を支払います。
- ここでのフィルターの動作は MVC とは異なります。`[ActionFilter]` 属性はページ ハンドラーでは暗黙的に無視されます。 `IPageFilter` / `IAsyncPageFilter` を使用するか、`Program.cs` の `options.Conventions` を通じてグローバル規則を登録します。

## プロジェクトの構造と規約

- 共有レイアウト、部分ファイル、およびテンプレートは、`Views/Shared/` ではなく `Pages/Shared/` に入ります。 Razor Pages は、ページのフォルダーから `Pages/` までのビューを階層的に解決します。MVC 規約を混ぜると、フレームワークと競合するだけです。
- `Pages/_ViewStart.cshtml`に`Layout`を設定します。 `@namespace`、`@addTagHelper`、および共有ディレクティブには `Pages/_ViewImports.cshtml` を使用します。
- `.cshtml` と `.cshtml.cs` を同じ場所に置きます。そもそも Razor ページを使用する主な理由の 1 つはページごとの局所性であり、フォルダー間で分割するとそれが失われます。

## 安全

- Razor のデフォルトの `@` 式 HTML エンコーディングを信頼します。ユーザーが提供したコンテンツでは `@Html.Raw()` にアクセスしないでください。エンコードを無効にし、XSS への扉を開きます。
- `<form method="post">` とフォーム タグ ヘルパーを使用して、偽造防止トークンが自動的に挿入されるようにします。 AJAX または `fetch` の場合、`@Html.AntiForgeryToken()` を使用してトークンをレンダリングし、`RequestVerificationToken` ヘッダーとして送信します。
- シークレットを `appsettings.json` にコミットしないでください。環境オーバーライド、ローカルでのユーザー シークレット (`dotnet user-secrets`)、運用環境での Azure Key Vault または環境変数には `appsettings.{Environment}.json` を使用します。 `IOptions<T>` 経由でバインドします。

## PageModel での依存関係の注入

- シングルトン内スコープのキャプティブ依存関係トラップに注意してください。シングルトンがスコープ付きサービス (EF `DbContext` など) への参照を保持している場合、そのインスタンスはリクエスト間でリークします。 PageModel に隣接するサービスによくあるバグ。
- `DbContext` を `Singleton` として登録しないでください。デフォルトの `AddDbContext` 登録は `Scoped` ですが、これには理由があります。

## ページ ハンドラーの Entity Framework コア

- EF エンティティをビューに返す前に、`.Select(...)` を使用して DTO またはビュー モデルに投影します。ナビゲーション プロパティを持つエンティティを直接渡すと、ビューのレンダリング時に遅延読み込み例外、N+1 クエリ、またはシリアル化サイクルが発生します。
- リスト ページや詳細ページなどの読み取り専用クエリでは、編集せずに `.AsNoTracking()` を使用します。変更トラッカーには不必要なオーバーヘッドがあります。
- `Include` を使用せずに主キーでフェッチする場合は、`FirstOrDefaultAsync(x => x.Id == key)` よりも `FindAsync(key)` を優先します。 `FindAsync` は、最初に変更トラッカーを確認します。

## 状態管理

- `TempData` は、PRG 後のフラッシュ通知などのワンショットのクロスリダイレクト メッセージ用です。これは、デフォルトで 1 回だけ読み取り、Cookie でシリアル化され、セッション ストレージの代わりにはなりません。
- 実際のユーザーごとのセッション状態については、`ISession` を使用します。リクエストごとのデータの場合は、`HttpContext.Items`。単一リクエスト内の共有状態の場合、リクエストスコープの DI サービス。
- 値が複数のリダイレクト後も消費されずに存続する必要がある場合は、`TempData.Keep()` または `TempData.Peek()` を呼び出します。

## テスト

- `PageModel` クラスを直接単体テストします。モック化された依存関係 (Moq、NSubstitute) を使用してそれらをインスタンス化し、返された `IActionResult` に対してアサートします。再レンダリングの場合は `PageResult`、成功した PRG の場合は `RedirectToPageResult`、404 パスの場合は `NotFoundResult` です。
- ルーティング、モデル バインディング、および偽造防止を実行する統合テストの場合は、`WebApplicationFactory<TEntryPoint>` を `Microsoft.AspNetCore.Mvc.Testing` とともに使用します。
- `ModelState` を読み取るハンドラーをテストする場合は、`PageModel.ModelState.AddModelError(...)` を手動で設定します。バインディング パイプラインは単体テストでは実行されません。