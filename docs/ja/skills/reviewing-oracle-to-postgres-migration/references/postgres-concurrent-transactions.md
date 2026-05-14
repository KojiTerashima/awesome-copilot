# Oracle から PostgreSQL: 同時トランザクション処理

## 内容

- 概要
- 核心的な違い
- 一般的なエラーの症状
- 問題のシナリオ
- ソリューション — 結果を具体化、接続を分離、単一のクエリ
- 検出戦略
- 注意すべきエラー メッセージ
- 比較表
- ベストプラクティス
- 移行チェックリスト

## 概要

Oracle から PostgreSQL に移行する場合、**単一データベース接続での同時操作**の処理方法に重大な違いが存在します。 Oracle の ODP.NET ドライバーでは、同じ接続上で同時に複数のアクティブなコマンドと結果セットが許可されますが、PostgreSQL の Npgsql ドライバーでは、**接続ごとに 1 つのアクティブなコマンド** という厳密なルールが適用されます。 Oracle でシームレスに動作するコードは、同時操作が接続を共有する場合、PostgreSQL ではランタイム例外をスローします。

## 核心的な違い

**オラクルの動作:**

- 単一の接続で複数のアクティブなコマンドを同時に実行できる
- 別の `DataReader` が開いている間に 2 番目の `DataReader` を開くことは許可されます
- 同じ接続上のネストされたデータベース呼び出しまたは重複したデータベース呼び出しは透過的に機能します

**PostgreSQL の動作:**

- 接続は **一度に 1 つのアクティブ コマンドのみをサポートします**
- `DataReader` が開いているときに 2 番目のコマンドを実行しようとすると、例外がスローされます
- 遅延ロードされたナビゲーション プロパティまたは同じ接続上で追加のクエリをトリガーするコールバック駆動の読み取りは失敗します。

## 一般的なエラーの症状

この違いを考慮せずに Oracle コードを移行する場合は、次のようになります。```
System.InvalidOperationException: An operation is already in progress.
```

```
Npgsql.NpgsqlOperationInProgressException: A command is already in progress: <SQL text>
```これらは、アクティブな `DataReader` または実行中のコミットされていないコマンドがすでに存在する接続上で、アプリケーション コードが新しいコマンドを実行しようとすると発生します。

---

## 問題のシナリオ

### シナリオ 1: 別のコマンドの実行中に DataReader を反復する```csharp
using (var reader = command1.ExecuteReader())
{
    while (reader.Read())
    {
        // PROBLEM: executing a second command on the same connection
        // while the reader is still open
        using (var command2 = new NpgsqlCommand("SELECT ...", connection))
        {
            var value = command2.ExecuteScalar(); // FAILS
        }
    }
}
```### シナリオ 2: データ アクセス層での遅延読み込み/遅延実行```csharp
// Oracle: works because ODP.NET supports concurrent readers
var items = repository.GetItems(); // returns IEnumerable backed by open DataReader
foreach (var item in items)
{
    // PROBLEM: triggers a second query on the same connection
    var details = repository.GetDetails(item.Id); // FAILS on PostgreSQL
}
```### シナリオ 3: アプリケーション コードを介したネストされたストアド プロシージャ呼び出し```csharp
// Oracle: ODP.NET handles multiple active commands
command1.ExecuteNonQuery(); // starts a long-running operation
command2.ExecuteScalar();   // FAILS on PostgreSQL — command1 still in progress
```---

## ソリューション

### 解決策 1: 新しいコマンドを発行する前に結果を実体化する (推奨)

同じ接続上で後続のコマンドを実行する前に、最初の結果セットをメモリにロードして閉じます。```csharp
// Load all results into a list first
var items = new List<Item>();
using (var reader = command1.ExecuteReader())
{
    while (reader.Read())
    {
        items.Add(MapItem(reader));
    }
} // reader is closed and disposed here

// Now safe to execute another command on the same connection
foreach (var item in items)
{
    using (var command2 = new NpgsqlCommand("SELECT ...", connection))
    {
        command2.Parameters.AddWithValue("id", item.Id);
        var value = command2.ExecuteScalar(); // Works
    }
}
```LINQ / EF Core シナリオの場合、`.ToList()` を使用して実体化を強制します。```csharp
// Before (fails on PostgreSQL — deferred execution keeps connection busy)
var items = dbContext.Items.Where(i => i.Active);
foreach (var item in items)
{
    var details = dbContext.Details.FirstOrDefault(d => d.ItemId == item.Id);
}

// After (materializes first query before issuing second)
var items = dbContext.Items.Where(i => i.Active).ToList();
foreach (var item in items)
{
    var details = dbContext.Details.FirstOrDefault(d => d.ItemId == item.Id);
}
```### 解決策 2: 同時操作には別の接続を使用する

操作を本当に同時に実行する必要がある場合は、それぞれに専用の接続を開きます。```csharp
using (var reader = command1.ExecuteReader())
{
    while (reader.Read())
    {
        // Use a separate connection for the nested query
        using (var connection2 = new NpgsqlConnection(connectionString))
        {
            connection2.Open();
            using (var command2 = new NpgsqlCommand("SELECT ...", connection2))
            {
                var value = command2.ExecuteScalar(); // Works — different connection
            }
        }
    }
}
```### 解決策 3: 単一のクエリに再構築する

可能であれば、JOIN またはサブクエリを使用してネストされたルックアップを 1 つのクエリに結合し、同時コマンドの必要性を完全に排除します。```csharp
// Before: two sequential queries on the same connection
var order = GetOrder(orderId);          // query 1
var details = GetOrderDetails(orderId); // query 2 (fails if query 1 reader still open)

// After: single query with JOIN
using (var command = new NpgsqlCommand(
    "SELECT o.*, d.* FROM orders o JOIN order_details d ON o.id = d.order_id WHERE o.id = @id",
    connection))
{
    command.Parameters.AddWithValue("id", orderId);
    using (var reader = command.ExecuteReader())
    {
        // Process combined result set
    }
}
```---

## 検出戦略

### コードレビューのチェックリスト

- [ ] `DataReader` を開き、閉じる前に他のデータベース メソッドを呼び出すメソッドを検索します。
- [ ] 実行を延期するデータ アクセス メソッドからの `IEnumerable` 戻り値の型を探します (オープン リーダーを示します)。
- [ ] さらなるクエリの発行中に反復される `.ToList()` / `.ToArray()` のない EF Core クエリを識別します
- [ ] 接続を共有するアプリケーション コード内のネストされたストアド プロシージャ呼び出しをチェックします。

### 一般的な検索場所

- データ アクセス レイヤーとリポジトリ クラス
- 複数のリポジトリ呼び出しを調整するサービス メソッド
- クエリ結果を反復し、行ごとにルックアップを実行するコード パス
- データの反復中にトリガーされるイベント ハンドラーまたはコールバック

### 検索パターン```regex
ExecuteReader\(.*\)[\s\S]*?Execute(Scalar|NonQuery|Reader)\(
```

```regex
\.Where\(.*\)[\s\S]*?foreach[\s\S]*?dbContext\.
```---

## 注意すべきエラー メッセージ

|エラーメッセージ |考えられる原因 |
|--------------|--------------|
| `An operation is already in progress` | `DataReader` が同じ接続上で開いている間に実行された 2 番目のコマンド |
| `A command is already in progress: <SQL>` | Npgsql が単一の接続で重複したコマンドの実行を検出しました。
| `The connection is already in state 'Executing'` |同時使用による接続状態の競合 |

---

## 比較表: Oracle と PostgreSQL

|側面 |オラクル (ODP.NET) | PostgreSQL (Npgsql) |
|----------|------|----------|
| **同時コマンド** |接続ごとに複数のアクティブなコマンド |接続ごとに 1 つのアクティブなコマンド |
| **複数の開いている DataReader** |サポートされている |サポートされていません - 最初に閉じる/実体化する必要があります |
| **反復中のネストされた DB 呼び出し** |透明 | `InvalidOperationException` をスローします |
| **遅延実行の安全性** |安全に反復およびクエリを実行できます。新しいクエリを発行する前に実体化 (`.ToList()`) する必要があります。
| **接続プーリングの影響** |接続需要の低下 |解決策 2 を使用する場合は、さらにプールされた接続が必要になる場合があります。

---

## ベストプラクティス

1. **早期にマテリアライズ** — さらにデータベース呼び出しを繰り返し発行する前に、クエリ結果に対して `.ToList()` または `.ToArray()` を呼び出します。これは最も簡単で信頼性の高い修正です。

2. **データ アクセス パターンの監査** — 呼び出し元が追加のクエリを発行しながら反復する遅延実行の戻り値の型 (`IEnumerable`、`IQueryable`) のすべてのリポジトリとデータ アクセス メソッドを確認します。

3. **単一クエリを優先** — 可能な場合は、ネストされたルックアップを JOIN またはサブクエリに結合して、同時コマンド パターンを完全に排除します。

4. **必要に応じて接続を分離します** — 同時操作が本当に必要な場合は、1 つの接続を共有しようとするのではなく、別の接続を使用します。

5. **反復ワークフローのテスト** — 統合テストでは、コードが結果セットを反復し、行ごとに追加のデータベース操作を実行するシナリオをカバーする必要があります。これらは最も一般的な障害点であるためです。

## 移行チェックリスト- [ ] 単一の接続で複数のコマンドを同時に実行するすべてのコード パスを特定します。
- [ ] オープン リーダーでの実行を延期する `IEnumerable` をサポートするデータ アクセス メソッドを見つけます。
- [ ] `.ToList()` / `.ToArray()` マテリアライゼーションを追加します。ここでは、遅延された結果がさらなるクエリとともに反復されます。
- [ ] ネストされたデータベース呼び出しをリファクタリングして、必要に応じて個別の接続または組み合わせたクエリを使用します。
- [ ] EF Core ナビゲーション プロパティと遅延読み込みによって同時接続の使用がトリガーされないことを確認します。
- [ ] 反復的なデータ アクセス パターンをカバーするために統合テストを更新します
- [ ] 解決策 2 (個別の接続) が広範囲に使用される場合の負荷テスト接続プールのサイジング

## 参考文献

- [Npgsql ドキュメント: 基本的な使用法](https://www.npgsql.org/doc/basic-usage.html)
- [PostgreSQL ドキュメント: 同時実行制御](https://www.postgresql.org/docs/current/mvcc.html)
- [Npgsql GitHub: 複数のアクティブな結果セットのディスカッション](https://github.com/npgsql/npgsql/issues/462)