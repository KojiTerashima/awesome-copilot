# Oracle から PostgreSQL: CURRENT_TIMESTAMP および NOW() タイムゾーンの処理

## 内容

- 問題
- 動作の比較
- PostgreSQL のタイムゾーンの優先順位
- 一般的なエラーの症状
- 移行アクション — Npgsql 設定、DateTime 正規化、ストアド プロシージャ、セッション タイムゾーン、アプリケーション コード
- 統合テストパターン
- チェックリスト

## 問題

Oracle の `CURRENT_TIMESTAMP` は、**セッション タイムゾーン**の値を返し、それを列の宣言された精度に格納します。 .NET が ODP.NET 経由でこの値を読み取ると、クライアントの OS タイムゾーンを反映して、`Kind=Local` を含む `DateTime` として表示されます。

PostgreSQL の `CURRENT_TIMESTAMP` と `NOW()` はどちらも、セッションのタイムゾーン設定に関係なく、**UTC** に固定された `timestamptz` (タイムゾーン付きのタイムスタンプ) を返します。 Npgsql がこの値を表示する方法は、ドライバーのバージョンと構成によって異なります。

- **Npgsql < 6 / レガシー モード (`EnableLegacyTimestampBehavior = true`):** `timestamptz` 列は、`Kind=Unspecified` を含む `DateTime` として返されます。これは、Oracle から移行する際のサイレント タイムゾーンのバグの原因です。
- **レガシー モードが無効になっている Npgsql 6 以降 (新しいデフォルト):** `timestamptz` 列は `Kind=Utc` を含む `DateTime` として返され、`Kind=Unspecified` 値を書き込むと挿入時に例外がスローされます。

まだ Npgsql 6 以降にアップグレードしていないプロジェクト、または明示的にレガシー モードに戻っているプロジェクトは、`Kind=Unspecified` 問題に対して依然として脆弱です。この不一致と、誤ってレガシー モードを再度有効にしてしまう可能性が高いため、サイレント データ破損、誤った比較、追跡が非常に困難な N 時間単位のバグが発生します。

---

## 動作の比較

|側面 |オラクル |ポストグレSQL |
|---|---|---|
| `CURRENT_TIMESTAMP` タイプ | `TIMESTAMP WITH LOCAL TIME ZONE` | `timestamptz` (UTC 正規化) |
|クライアント `DateTime.Kind` ドライバー経由 | `Local` | `Unspecified` (Npgsql < 6 / レガシー モード); `Utc` (Npgsql 6 以降のデフォルト) |
|セッションのタイムゾーンの影響 |はい - 格納/戻り値に影響します | *表示*のみに影響します。 UTC が内部に保存される |
| NOW() と同等 | `SYSDATE` / `CURRENT_TIMESTAMP` | `NOW()` = `CURRENT_TIMESTAMP` (両方とも `timestamptz` を返します) |
|比較時の暗黙的な変換 | Oracle はセッション TZ オフセットを適用します。 PostgreSQL は UTC を比較します。セッション TZ は表示専用です |

---

## PostgreSQL のタイムゾーンの優先順位

PostgreSQL は、次の階層を使用して有効なセッション タイムゾーンを解決します (最も高い優先順位が優先されます)。

|レベル |設定方法 |
|---|---|
| **セッション** | `SET TimeZone = 'UTC'` 接続オープン時に送信 |
| **役割** | `ALTER ROLE app_user SET TimeZone = 'UTC'` |
| **データベース** | `ALTER DATABASE mydb SET TimeZone = 'UTC'` |
| **サーバー** | `postgresql.conf` → `TimeZone = 'America/New_York'` |セッション タイムゾーンは、`timestamptz` 列の保存された UTC 値には**影響しません**。`SHOW timezone` および `::text` が表示用に値をキャストする方法を制御するだけです。 `DateTime.Kind` に依存するアプリケーション コードや、明示的なタイムゾーンを指定せずにタイムスタンプを比較するアプリケーション コードは、サーバーのデフォルトのタイムゾーンが UTC でない場合、誤った結果を生成する可能性があります。

---

## 一般的なエラーの症状

- PostgreSQL から読み取られたタイムスタンプには `Kind=Unspecified` が付いています。 `DateTime.UtcNow` または `DateTime.Now` との比較では、間違った結果が生成されます。
- WHERE 句の比較が、保存されている UTC 値とは異なるタイムゾーンで評価されるため、日付範囲クエリで返される行が少なすぎるか多すぎます。
- 統合テストは開発者マシン (UTC OS タイムゾーン) では成功しますが、CI または運用環境 (非 UTC タイムゾーン) では失敗します。
- タイムスタンプを含むストアド プロシージャの出力パラメータは、サーバーによって適用されたセッション オフセットとともに到着しますが、その後アプリケーションの UTC 値と比較されます。

---

## 移行アクション

### 1. 接続文字列または AppContext を使用して Npgsql を UTC 用に構成する

Npgsql 6 以降では、`EnableLegacyTimestampBehavior` がデフォルトで `false` に設定された状態で出荷されます。そのため、`timestamptz` 値は `Kind=Utc` を含む `DateTime` として返されます。誤ってレガシー モードにオプトインすることを防止し (構成ファイルや推移的な依存関係などを介して)、将来のメンテナにその意図が見えるようにするために、起動時にスイッチを明示的に設定することを引き続き推奨します。```csharp
// Program.cs / Startup.cs — apply once at application start
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", false);
```このスイッチを無効にすると、`Kind=Unspecified` を含む `DateTime` を `timestamptz` 列に書き込もうとすると Npgsql がスローされ、タイムゾーンのバグがクエリ時に静かに発生するのではなく、挿入時に検出可能になります。

### 2. 永続化の前に DateTime 値を正規化する

移行されたコードベース全体で `DateTime.Now` を `DateTime.UtcNow` に置き換えます。外部入力に由来する値 (JSON から逆シリアル化されたユーザー指定の日付など) の場合は、保存する前に UTC に変換されていることを確認してください。```csharp
// Before (Oracle-era code — relied on session/OS timezone)
var timestamp = DateTime.Now;

// After (PostgreSQL-compatible)
var timestamp = DateTime.UtcNow;

// For externally-supplied values
var utcTimestamp = dateTimeInput.Kind == DateTimeKind.Utc
    ? dateTimeInput
    : dateTimeInput.ToUniversalTime();
```### 3. CURRENT_TIMESTAMP / NOW() を使用してストアド プロシージャを修正する

`CURRENT_TIMESTAMP` または `NOW()` を `timestamp without time zone` (`timestamp`) 列に割り当てるストアド プロシージャを確認する必要があります。 `timestamptz` 列を優先するか、明示的にキャストします。```sql
-- Ambiguous: server timezone influences interpretation
INSERT INTO audit_log (created_at) VALUES (NOW()::timestamp);

-- Safe: always UTC
INSERT INTO audit_log (created_at) VALUES (NOW() AT TIME ZONE 'UTC');

-- Or: use timestamptz column type and let PostgreSQL store UTC natively
INSERT INTO audit_log (created_at) VALUES (CURRENT_TIMESTAMP);
```### 4. 接続時にセッション タイムゾーンを強制的に開く (多層防御)

ロールやデータベースのデフォルトに関係なく、接続を開くときにセッションのタイムゾーンを明示的に設定します。これにより、サーバー構成に関係なく一貫した動作が保証されます。```csharp
// Npgsql connection string approach
var connString = "Host=localhost;Database=mydb;Username=app;Password=...;Timezone=UTC";

// Or: apply via NpgsqlDataSourceBuilder
var dataSource = new NpgsqlDataSourceBuilder(connString)
    .Build();

// Or: execute on every new connection
await using var conn = new NpgsqlConnection(connString);
await conn.OpenAsync();
await using var cmd = new NpgsqlCommand("SET TimeZone = 'UTC'", conn);
await cmd.ExecuteNonQueryAsync();
```### 5. アプリケーション コード — DateTime.Kind=Unspecified を避ける

タイムスタンプ列を読み取るすべてのリポジトリおよびデータ アクセス コードを監査します。 Npgsql が `Unspecified` を返す場合、データ ソースをグローバルに設定するか (上記のオプション 1)、読み取りをラップします。```csharp
// Safe reader helper — convert Unspecified to Utc at the boundary
DateTime ReadUtcDateTime(NpgsqlDataReader reader, int ordinal)
{
    var dt = reader.GetDateTime(ordinal);
    return dt.Kind == DateTimeKind.Unspecified
        ? DateTime.SpecifyKind(dt, DateTimeKind.Utc)
        : dt.ToUniversalTime();
}
```---

## 統合テスト パターン

### テスト: タイムスタンプが保持され、UTC として返されることを確認します。```csharp
[Fact]
public async Task InsertedTimestamp_ShouldRoundTripAsUtc()
{
    var before = DateTime.UtcNow;

    await repository.InsertAuditEntryAsync(/* ... */);

    var retrieved = await repository.GetLatestAuditEntryAsync();

    Assert.Equal(DateTimeKind.Utc, retrieved.CreatedAt.Kind);
    Assert.True(retrieved.CreatedAt >= before,
        "Persisted CreatedAt should not be earlier than the pre-insert UTC timestamp.");
}
```### テスト: Oracle ベースラインと PostgreSQL ベースライン間のタイムスタンプの比較を検証する```csharp
[Fact]
public async Task TimestampComparison_ShouldReturnSameRowsAsOracle()
{
    var cutoff = DateTime.UtcNow.AddDays(-1);

    var oracleResults = await oracleRepository.GetEntriesAfter(cutoff);
    var postgresResults = await postgresRepository.GetEntriesAfter(cutoff);

    Assert.Equal(oracleResults.Count, postgresResults.Count);
}
```---

## チェックリスト

- [ ] `AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", false)` はアプリケーション起動時に適用されます。
- [ ] データ アクセス コード内のすべての `DateTime.Now` の使用は `DateTime.UtcNow` に置き換えられます。
- [ ] 接続文字列または接続オープン フックは `Timezone=UTC` / `SET TimeZone = 'UTC'` を設定します。
- [ ] `CURRENT_TIMESTAMP` または `NOW()` を使用するストアド プロシージャをレビューしました。 `timestamp without time zone` 列は明示的にキャストまたは `timestamptz` に置き換えられます。
- [ ] 統合テストは、取得したタイムスタンプ値に対して `DateTime.Kind == Utc` をアサートします。
- [ ] テストは日付範囲クエリを対象として、行数が Oracle ベースラインと一致することを確認します。