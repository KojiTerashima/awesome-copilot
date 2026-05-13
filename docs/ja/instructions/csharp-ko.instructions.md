---
description: 'C#アプリケーション開発のためのコード記述規則by @jgkim999'
applyTo: '**/*.cs'
---

# C#コード作成規則

## 命名規則 (Naming Conventions)

一貫した命名規則はコード可読性の鍵です。 Microsoftのガイドラインに従うことをお勧めします。

| 要素 | 命名規則 | 例 |
|------|-----------|------|
| インターフェース | プレフィックス 'I' + PascalCase | `IAsyncRepository`、`ILogger` |
| 公開(public)メンバー | パスカルケース(PascalCase) | `public int MaxCount;`、`public void GetData()` |
| パラメータ、ローカル変数 | キャメルケース(camelCase) | `int userCount`、`string customerName` |
| プライベート/内部フィールド | アンダースコア（_）+キャメルケース | `private string _connectionString;` |
| 定数(const) | パスカルケース(PascalCase) | `public const int DefaultTimeout = 5000;` |
| ジェネリック型パラメータ | プレフィックス「T」+説明的な名前 | `TKey`、`TValue`、`TResult` |
| 非同期メソッド | 'Async'サフィックス | `GetUserAsync`、`DownloadFileAsync` |

## コードの書式と読みやすさ (Formatting & Readability)

一貫した書式は、コードを視覚的に解析しやすくします。

| アイテム | ルール | 説明 |
|------|------|------|
| インデント | 4つのスペースを使用 | タブの代わりに4つのスペースを使用します。 csファイルは必ず4つのスペースを使用します。 |
| かっこ | 常に中括弧{}を使用 | 制御ステートメント（if、for、whileなど）が1行でも、常に中括弧を使用してください。 |
| 空行 | 論理的分離 | メソッド定義、プロパティ定義、論理的に分離されたコードブロックの間に空白行を追加します。 |
| 文章を書く | 1行に1つの文 | 1行には1つの文だけを書きます。 |
| varキーワード | 形式が明確な場合にのみ使用 | 変数の型が右から明確に推論できる場合にのみvarを使用します。 |
| 名前空間 | ファイル範囲名前空間の使用 | C#10以降では、ファイル範囲の名前空間を使用して不要なインデントを減らします。 |
| コメント | XML形式のコメントの作成 | 作成したクラスまたは関数に常にxml形式のコメントを作成します。 |

## 言語機能の使用 (Language Features)

最新のC#機能を活用して、コードをより簡潔で効率的にしましょう。

| 機能 | 説明 | 例/参考 |
|------|------|------|
| 非同期プログラミング | I/O バインド操作に async/await を使用する | `async Task<string> GetDataAsync()` |
| 構成待機 | ライブラリコードでコンテキスト切り替えオーバーヘッドを削減 | `await SomeMethodAsync().ConfigureAwait(false)` |
| リンク | コレクションデータの照会と操作 | `users.Where(u => u.IsActive).ToList()` |
| 式ベースのメンバー | 単純なメソッド/プロパティを簡潔に表現 | `public string Name => _name;` |
| Null 許容参照型 | コンパイル時のNullReferenceExceptionの防止 | `#nullable enable` |
| using 宣言 | IDisposable オブジェクトの簡潔な処理 | `using var stream = new FileStream(...);` |

## パフォーマンスおよび例外処理(Performance & Exception Handling)

堅牢で迅速なアプリケーションのためのガイドラインです。

### 例外処理

処理できる具体的な例外だけをキャッチしてください。 catch（Exception）のような一般的な例外をキャッチすることは避けるべきです。

例外はプログラムフロー制御には使用しないでください。例外は、予期しないエラー状況にのみ使用する必要があります。

### パフォーマンス
s
文字列を繰り返し連結する場合は、+演算子の代わりにStringBuilderを使用してください。

Entity Framework Coreを使用する場合は、読み取り専用クエリには.AsNoTracking（）を使用してパフォーマンスを向上させます。

不要なオブジェクトの割り当てを避け、特にループ内で注意してください。

## セキュリティ(Security)

安全なコードを書くための基本原則です。

| セキュリティゾーン | ルール | 説明 |
|------|------|------|
| 入力検証 | すべての外部データ検証 | 外部（ユーザー、APIなど）からのすべてのデータは信頼せず、常に検証してください。 |
| SQL挿入防止 | パラメータ化されたクエリの使用 | 常にパラメータ化されたクエリやEntity FrameworkなどのORMを使用してSQL挿入攻撃を防ぎます。 |
| 機密データ保護 | 構成管理ツールの使用 | パスワード、接続文字列、APIキーなどは、ソースコードにハードコードしないで、Secret Manager、Azure Key Vaultなどを使用してください。 |

これらのルールをプロジェクトの.editorconfigファイルとチームのコードレビュープロセスに統合して、常に高品質のコードを維持することを目指してください。
