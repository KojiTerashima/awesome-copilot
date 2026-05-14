# Oracle から PostgreSQL: TO_CHAR() 数値変換

## 内容

- 問題
- 根本原因
- ソリューション パターン — CAST、フォーマット文字列、連結
- 移行チェックリスト
- アプリケーションコードのレビュー
- テストの推奨事項
- 一般的な場所
- 注意すべきエラー メッセージ

## 問題

Oracle では、`TO_CHAR()` が形式指定子なしで数値型を文字列に変換できます。```sql
-- Oracle: Works fine
SELECT TO_CHAR(vessel_id) FROM vessels;
SELECT TO_CHAR(fiscal_year) FROM certificates;
```PostgreSQL では `TO_CHAR()` を数値型で使用する場合にフォーマット文字列が必要ですが、それ以外の場合は次のエラーが発生します。```
42883: function to_char(numeric) does not exist
```## 根本原因

- **Oracle**: 書式マスクを使用しない `TO_CHAR(number)` は、デフォルトの書式設定を使用して数値を文字列に暗黙的に変換します。
- **PostgreSQL**: `TO_CHAR()` では常に数値型の明示的なフォーマット文字列が必要です (例: `'999999'`、`'FM999999'`)

## 解決策のパターン

### パターン 1: CAST を使用する (推奨)

最もクリーンな移行アプローチは、`TO_CHAR(numeric_column)` を `CAST(numeric_column AS TEXT)` に置き換えることです。```sql
-- Oracle
SELECT TO_CHAR(vessel_id) AS vessel_item FROM vessels;

-- PostgreSQL (preferred)
SELECT CAST(vessel_id AS TEXT) AS vessel_item FROM vessels;
```**利点:**

- PostgreSQL ではより慣用的です
- より明確な意図
- フォーマット文字列は必要ありません

### パターン 2: フォーマット文字列を指定する

特定の数値書式設定が必要な場合は、明示的な書式マスクを使用します。```sql
-- PostgreSQL with format
SELECT TO_CHAR(vessel_id, 'FM999999') AS vessel_item FROM vessels;
SELECT TO_CHAR(amount, 'FM999999.00') AS amount_text FROM payments;
```**マスクの書式設定:**

- `'FM999999'`: 固定幅整数 (FM = Fill Mode、先頭のスペースを削除)
- `'FM999999.00'`: 2 桁の 10 進数
- `'999,999.00'`: 千の区切り文字あり

### パターン 3: 文字列の連結

数値変換が暗黙的に行われる単純な連結の場合:```sql
-- Oracle
WHERE TO_CHAR(fiscal_year) = '2024'

-- PostgreSQL (using concatenation)
WHERE fiscal_year::TEXT = '2024'
-- or
WHERE CAST(fiscal_year AS TEXT) = '2024'
```## 移行チェックリスト

`TO_CHAR()` を含む SQL を移行する場合:

1. **すべての TO_CHAR() 呼び出しを特定します**: SQL 文字列、ストアド プロシージャ、およびアプリケーション クエリで `TO_CHAR\(` を検索します。
2. **引数の型を確認します**:
   - **DATE/TIMESTAMP**: `TO_CHAR()` をフォーマット文字列とともに保持します (例: `TO_CHAR(date_col, 'YYYY-MM-DD')`)
   - **NUMERIC/INTEGER**: `CAST(... AS TEXT)` に置き換えるか、形式文字列を追加します
3. **出力をテスト**: 文字列表現が期待どおりであることを確認します (予期しないスペース、小数などが含まれていない)。
4. **比較ロジックを更新**: 数値と文字列を比較する場合は、両側で型が一貫していることを確認します。

## アプリケーションコードのレビュー

### C# の例```csharp
// Before (Oracle)
var sql = "SELECT TO_CHAR(id) AS id_text FROM entities WHERE TO_CHAR(status) = @status";

// After (PostgreSQL)
var sql = "SELECT CAST(id AS TEXT) AS id_text FROM entities WHERE CAST(status AS TEXT) = @status";
```## テストに関する推奨事項

1. **単体テスト**: 数値から文字列への変換が期待値を返すことを確認します。```csharp
   [Fact]
   public void GetVesselNumbers_ReturnsVesselIdsAsStrings()
   {
       var results = dal.GetVesselNumbers(certificateType);
       Assert.All(results, item => Assert.True(int.TryParse(item.DISPLAY_MEMBER, out _)));
   }
   ```2. **統合テスト**: `CAST()` を使用したクエリがエラーなしで実行されることを確認します。
3. **比較テスト**: 数値と文字列の比較フィルターを使用して WHERE 句が正しく検証されることを確認します。

## 一般的な場所

次の場所で `TO_CHAR` を検索します。

- ✅ ストアド プロシージャと関数 (DDL スクリプト)
- ✅ アプリケーション データ アクセス レイヤー (DAL クラス)
- ✅ 動的 SQL ビルダー
- ✅ クエリのレポート
- ✅ ORM/Entity Framework の生の SQL

## 注意すべきエラー メッセージ```
Npgsql.PostgresException: 42883: function to_char(numeric) does not exist
Npgsql.PostgresException: 42883: function to_char(integer) does not exist
Npgsql.PostgresException: 42883: function to_char(bigint) does not exist
```## 関連項目

- [oracle-to-postgres-type-coercion.md](oracle-to-postgres-type-coercion.md) - 関連する型変換の問題
- PostgreSQL ドキュメント: [データ型フォーマット関数](https://www.postgresql.org/docs/current/functions-formatting.html)