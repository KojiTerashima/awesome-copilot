# Oracle から PostgreSQL: クライアント アプリケーションでの Refcursor の処理

## 核心的な違い

Oracle のドライバーは `SYS_REFCURSOR` 出力パラメーターを自動的にアンラップし、結果セットをデータ リーダーに直接公開します。 PostgreSQL の Npgsql ドライバーは、代わりに **カーソル名** (例: `"<unnamed portal 1>"`) を返します。実際の行を取得するには、クライアントは別の `FETCH ALL FROM "<cursor_name>"` コマンドを発行する必要があります。

これを考慮しないと、次のような原因が発生します。```
System.IndexOutOfRangeException: Field not found in row: <column_name>
```リーダーには、カーソル名パラメータのみが含まれており、予期される結果列は含まれません。

> **トランザクション要件:** PostgreSQL refcursor のスコープはトランザクションに限定されます。プロシージャ呼び出しと `FETCH` の両方を同じ明示的なトランザクション内で実行する必要があります。そうしないと、自動コミットでフェッチが完了する前にカーソルが閉じられる可能性があります。

## 解決策: 明示的な Refcursor アンラップ (C#)```csharp
public IEnumerable<User> GetUsers(int departmentId)
{
    var users = new List<User>();
    using var connection = new NpgsqlConnection(connectionString);
    connection.Open();

    // Refcursors are transaction-scoped — wrap both the call and FETCH in one transaction.
    using var tx = connection.BeginTransaction();

    using var command = new NpgsqlCommand("get_users", connection, tx)
    {
        CommandType = CommandType.StoredProcedure
    };
    command.Parameters.AddWithValue("p_department_id", departmentId);
    var refcursorParam = new NpgsqlParameter("cur_result", NpgsqlDbType.Refcursor)
    {
        Direction = ParameterDirection.Output
    };
    command.Parameters.Add(refcursorParam);

    // Execute the procedure to open the cursor.
    command.ExecuteNonQuery();

    // Retrieve the cursor name, then fetch the actual data.
    string cursorName = (string)refcursorParam.Value;
    using var fetchCommand = new NpgsqlCommand($"FETCH ALL FROM \"{cursorName}\"", connection, tx);
    using var reader = fetchCommand.ExecuteReader();
    while (reader.Read())
    {
        users.Add(new User
        {
            UserId   = reader.GetInt32(reader.GetOrdinal("user_id")),
            UserName = reader.GetString(reader.GetOrdinal("user_name")),
            Email    = reader.GetString(reader.GetOrdinal("email"))
        });
    }

    tx.Commit();
    return users;
}
```## 再利用可能なヘルパー

ヘルパーからライブ `NpgsqlDataReader` を返すと、基礎となる `NpgsqlCommand` が破棄されず、所有権があいまいになります。代わりに、ヘルパー内で結果を具体化することを好みます。```csharp
public static class PostgresHelpers
{
    public static List<T> ExecuteRefcursorProcedure<T>(
        NpgsqlConnection connection,
        NpgsqlTransaction transaction,
        string procedureName,
        Dictionary<string, object> parameters,
        string refcursorParameterName,
        Func<NpgsqlDataReader, T> map)
    {
        using var command = new NpgsqlCommand(procedureName, connection, transaction)
        {
            CommandType = CommandType.StoredProcedure
        };
        foreach (var (key, value) in parameters)
            command.Parameters.AddWithValue(key, value);

        var refcursorParam = new NpgsqlParameter(refcursorParameterName, NpgsqlDbType.Refcursor)
        {
            Direction = ParameterDirection.Output
        };
        command.Parameters.Add(refcursorParam);
        command.ExecuteNonQuery();

        string cursorName = (string)refcursorParam.Value;
        if (string.IsNullOrEmpty(cursorName))
            return new List<T>();

        // fetchCommand is disposed here; results are fully materialized before returning.
        using var fetchCommand = new NpgsqlCommand($"FETCH ALL FROM \"{cursorName}\"", connection, transaction);
        using var reader = fetchCommand.ExecuteReader();

        var results = new List<T>();
        while (reader.Read())
            results.Add(map(reader));
        return results;
    }
}

// Usage:
using var connection = new NpgsqlConnection(connectionString);
connection.Open();
using var tx = connection.BeginTransaction();

var users = PostgresHelpers.ExecuteRefcursorProcedure(
    connection, tx,
    "get_users",
    new Dictionary<string, object> { { "p_department_id", departmentId } },
    "cur_result",
    r => new User
    {
        UserId   = r.GetInt32(r.GetOrdinal("user_id")),
        UserName = r.GetString(r.GetOrdinal("user_name")),
        Email    = r.GetString(r.GetOrdinal("email"))
    });

tx.Commit();
```## Oracle と PostgreSQL の概要

|側面 |オラクル (ODP.NET) | PostgreSQL (Npgsql) |
|----------|------|----------|
| **カーソルリターン** |データ リーダーで直接公開される結果セット |出力パラメータのカーソル名文字列 |
| **データ アクセス** | `ExecuteReader()` はすぐに行を返します。 `ExecuteNonQuery()` → カーソル名を取得 → `FETCH ALL FROM` |
| **トランザクション** |透明 | CALL と FETCH は同じトランザクションを共有する必要があります。
| **複数のカーソル** |自動 |それぞれに個別の `FETCH` コマンドが必要です。
| **リソースの有効期間** |ドライバー管理 |カーソルはフェッチされるかトランザクションが終了するまで開いています。

## 移行チェックリスト

- [ ] `SYS_REFCURSOR` (Oracle) / `refcursor` (PostgreSQL) を返すすべてのプロシージャを特定します
- [ ] `ExecuteReader()`を`ExecuteNonQuery()`に置換 → カーソル名 → `FETCH ALL FROM`
- [ ] 各呼び出しとフェッチのペアを明示的なトランザクションでラップします。
- [ ] コマンドとリーダーが破棄されていることを確認します (ヘルパー内で結果を具体化することを推奨します)。
- [ ] 単体テストと統合テストを更新します

## 参考文献

- [PostgreSQL ドキュメント: カーソル](https://www.postgresql.org/docs/current/plpgsql-cursors.html)
- [PostgreSQL FETCHコマンド](https://www.postgresql.org/docs/current/sql-fetch.html)
- [Npgsql Refcursor サポート](https://github.com/npgsql/npgsql/issues/1887)