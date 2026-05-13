---
description: '.NET MAUI コンポーネントおよびアプリケーション パターン'
applyTo: '**/*.xaml, **/*.cs'
---

# .NET MAUI

## .NET MAUI のコードスタイルと構造

- .NET MAUI と C# の慣用的で効率的なコードを書きます。
- .NET と .NET MAUI の規約に従います。
- UI (View) はレイアウトとバインディングに集中させ、ロジックは ViewModel とサービスに置きます。
- I/O や長時間実行処理には async/await を使い、UI の応答性を保ちます。

## 命名規則

- コンポーネント名、メソッド名、公開メンバーには PascalCase を使用します。
- private フィールドとローカル変数には camelCase を使用します。
- インターフェイス名には "I" を接頭辞として付けます (例: `IUserService`)。

## .NET MAUI および .NET 固有のガイドライン

- コンポーネント ライフサイクルには .NET MAUI の組み込み機能 (例: OnAppearing, OnDisappearing) を活用します。
- `{Binding}` と MVVM パターンを使ってデータ バインディングを効果的に使用します。
- .NET MAUI のコンポーネントとサービスは、関心の分離に従って構成します。
- 言語バージョンは、リポジトリの対象 .NET SDK と設定がサポートするものを使用します。プロジェクトがすでにそうなっていない限り、プレビュー機能を前提にしないでください。

## 重要ルール (一貫性)

- ListView は **絶対に** 使用しないでください (非推奨)。CollectionView を使います。
- TableView は **絶対に** 使用しないでください (非推奨)。CollectionView または Grid/VerticalStackLayout などのレイアウトを優先します。
- Frame は **絶対に** 使用しないでください (非推奨)。代わりに Border を使います。
- `*AndExpand` レイアウト オプションは **絶対に** 使用しないでください (非推奨)。代わりに Grid と明示的なサイズ指定を使います。
- ScrollView や CollectionView を StackLayout/VerticalStackLayout/HorizontalStackLayout の中に **絶対に** 置かないでください (スクロールや仮想化が壊れることがあります)。親レイアウトには Grid を使います。
- 実行時に画像を `.svg` として参照してはいけません。PNG/JPG リソースを使います。
- Shell ナビゲーションと NavigationPage/TabbedPage/FlyoutPage を混在させてはいけません。
- renderer は使わず、handler を使います。
- `BackgroundColor` は設定せず、`Background` を使います (グラデーション/brush をサポートし、現在はこちらが推奨 API です)。

## レイアウトとコントロールの選択

- `StackLayout Orientation="..."` より `VerticalStackLayout`/`HorizontalStackLayout` を優先します (より高性能です)。
- 小規模でスクロールしないリスト (20 件以下) には `BindableLayout` を使います。大きいリストやスクロールするリストには `CollectionView` を使います。
- 複雑なレイアウトや領域分割が必要な場合は `Grid` を優先します。
- 枠線や背景を持つコンテナーには `Frame` より `Border` を使います。

## Shell ナビゲーション

- 主要なナビゲーション ホストとして Shell を使用します。
- `Routing.RegisterRoute(...)` でルートを登録し、`Shell.Current.GoToAsync(...)` で遷移します。
- `MainPage` は起動時に 1 回だけ設定し、頻繁に変更しないでください。
- Shell の中にタブを入れ子にしないでください。

## エラー処理と検証

- .NET MAUI のページと API 呼び出しには適切なエラー処理を実装します。
- アプリ レベルのエラーにはログを使い、回復可能な失敗ではログを残したうえで、ユーザー向けに分かりやすいメッセージを表示します。
- フォームの検証には FluentValidation または DataAnnotations を使います。

## MAUI API とパフォーマンス最適化

- パフォーマンスと正確性のため、コンパイル済みバインディングを優先します。
	- XAML では、ページ/ビュー/テンプレートに `x:DataType` を設定します。
	- C# では可能な限り式ベースのバインディングを優先します。
	- 特に CI では、プロジェクト設定でより厳格な XAML コンパイル (例: `MauiStrictXamlCompilation=true`) を有効化することを検討します。
- 深いレイアウトの入れ子 (特に StackLayout の入れ子) は避けます。複雑なレイアウトでは Grid を優先します。
- バインディングは意図を持って使います。
	- 値が変わらない場合は `OneTime` を使用します。
	- 編集可能な値にだけ `TwoWay` を使います。
	- 静的定数はバインドせず、直接設定します。
- バックグラウンド処理から UI を更新する場合は `Dispatcher.Dispatch()` または `Dispatcher.DispatchAsync()` を使用します。
	- Page、View、その他の BindableObject への参照がある場合は、`BindableObject.Dispatcher` を優先します。
	- BindableObject に直接アクセスできないサービスや ViewModel では、DI で `IDispatcher` を注入します。
	- Dispatcher が利用できない場合のみ、代替として `MainThread.BeginInvokeOnMainThread(...)` を使います。
	- 廃止された `Device.BeginInvokeOnMainThread` パターンは **避けてください**。

## リソースとアセット

- 画像は `Resources/Images/`、フォントは `Resources/Fonts/`、raw アセットは `Resources/Raw/` に配置します。
- 画像は `.svg` ではなく PNG/JPG として参照します (例: `<Image Source="logo.png" />`)。
- メモリ肥大化を避けるため、適切なサイズの画像を使います。

## 状態管理

- 共有状態や横断的関心事には DI 管理のサービスを優先し、ViewModel はナビゲーション/ページのライフタイムに合わせてスコープします。

## API 設計と統合

- 外部 API や独自バックエンドとの通信には HttpClient または適切なサービスを使います。
- API 呼び出しでは try-catch によるエラー処理を実装し、UI で適切なユーザー フィードバックを提供します。

## ストレージとシークレット

- シークレット (トークン、リフレッシュ トークン) には `SecureStorage` を使い、例外 (未対応デバイス、キー変更、破損) はクリア/リセットと再認証で処理します。
- シークレットを Preferences に保存してはいけません。

## テストとデバッグ

- コンポーネントとサービスは xUnit、NUnit、または MSTest でテストします。
- テスト時の依存関係のモックには Moq または NSubstitute を使います。

## セキュリティと認証

- 必要に応じて MAUI アプリに認証と認可を実装し、API 認証には OAuth または JWT トークンを使用します。
- すべての Web 通信に HTTPS を使用し、適切な CORS ポリシーが実装されていることを確認します。

## よくある落とし穴

- `MainPage` を頻繁に変更すると、ナビゲーション上の問題を引き起こすことがあります。
- 親ビューと子ビューの両方にジェスチャー認識を付けると競合することがあります。必要に応じて `InputTransparent = true` を使用します。
- イベント購読解除漏れによるメモリリークに注意し、常に購読解除とリソース破棄を行います。
- 深いレイアウトの入れ子はパフォーマンスを悪化させるため、視覚ツリーはフラットに保ちます。
- エミュレーターだけのテストでは実機特有のケースを見落とします。物理デバイスでもテストしてください。
