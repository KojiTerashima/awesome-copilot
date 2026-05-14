# Oracle から PostgreSQL: FROM 句のかっこ

## 内容

- 問題
- 根本原因
- 解決パターン
- 例
- 移行チェックリスト
- 一般的な場所
- アプリケーションコード例
- 注意すべきエラー メッセージ
- テストの推奨事項

## 問題

Oracle では、FROM 句でテーブル名を括弧で囲むことができます。```sql
-- Oracle: Both are valid
SELECT * FROM (TABLE_NAME) WHERE id = 1;
SELECT * FROM TABLE_NAME WHERE id = 1;
```PostgreSQL では、派生テーブルまたはサブクエリでない場合、FROM 句内の単一のテーブル名を囲む追加の括弧は**許可されません**。このパターンを使用しようとすると、次のような結果になります。```
Npgsql.PostgresException: 42601: syntax error at or near ")"
```## 根本原因

- **Oracle**: `FROM(TABLE_NAME)` を `FROM TABLE_NAME` と同等として扱います
- **PostgreSQL**: FROM 句のかっこは次の場合にのみ有効です。
  - サブクエリ: `FROM (SELECT * FROM table)`
  - 結合構文の一部である明示的なテーブル参照
  - 共通テーブル式 (CTE)
  - 有効な SELECT または結合コンテキストがないと、PostgreSQL で構文エラーが発生します

## 解決策のパターン

テーブル名を囲んでいる不要な括弧を削除します。```sql
-- Oracle (problematic in PostgreSQL)
SELECT col1, col2
FROM (TABLE_NAME)
WHERE id = 1;

-- PostgreSQL (correct)
SELECT col1, col2
FROM TABLE_NAME
WHERE id = 1;
```## 例

### 例 1: 単純なテーブル参照```sql
-- Oracle
SELECT employee_id, employee_name
FROM (EMPLOYEES)
WHERE department_id = 10;

-- PostgreSQL (fixed)
SELECT employee_id, employee_name
FROM EMPLOYEES
WHERE department_id = 10;
```### 例 2: 括弧を使用した結合```sql
-- Oracle (problematic)
SELECT e.employee_id, d.department_name
FROM (EMPLOYEES) e
JOIN (DEPARTMENTS) d ON e.department_id = d.department_id;

-- PostgreSQL (fixed)
SELECT e.employee_id, d.department_name
FROM EMPLOYEES e
JOIN DEPARTMENTS d ON e.department_id = d.department_id;
```### 例 3: 有効なサブクエリのかっこ (両方で機能します)```sql
-- Both Oracle and PostgreSQL
SELECT *
FROM (SELECT employee_id, employee_name FROM EMPLOYEES WHERE department_id = 10) sub;
```## 移行チェックリスト

この問題を解決するときは、次のことを確認してください。

1. **問題のある FROM 句をすべて特定します**:
   - SQLで`FROM (`パターンを検索
   - 開き括弧が `FROM` の直後にあり、その後にテーブル名が続くことを確認してください。
   - サブクエリではないことを確認します (内部に SELECT キーワードがありません)

2. **有効な括弧を区別する**:
   - ✅ `FROM (SELECT ...)` - 有効なサブクエリ
   - ✅ `FROM (table_name` の後に結合 - JOIN キーワードが続くかどうかを確認します
   - ❌ `FROM (TABLE_NAME)` - 無効です。括弧を削除してください

3. **修正を適用**:
   - テーブル名の周囲のかっこを削除します。
   - 正当なサブクエリには括弧を付けてください

4. **徹底的にテストします**:
   - PostgreSQLでクエリを実行します。
   - 結果セットが元の Oracle クエリと一致することを確認する
   - 統合テストに含める

## 一般的な場所

次の場所で `FROM (` を検索します。

- ✅ ストアド プロシージャと関数 (DDL スクリプト)
- ✅ アプリケーション データ アクセス レイヤー (DAL クラス)
- ✅ 動的 SQL ビルダー
- ✅ クエリのレポート
- ✅ ビューとマテリアライズド ビュー
- ✅ 複数の結合を含む複雑なクエリ

## アプリケーションコードの例

### VB.NET```vb
' Before (Oracle)
StrSQL = "SELECT employee_id, NAME " _
       & "FROM (EMPLOYEES) e " _
       & "WHERE e.department_id = 10"

' After (PostgreSQL)
StrSQL = "SELECT employee_id, NAME " _
       & "FROM EMPLOYEES e " _
       & "WHERE e.department_id = 10"
```###C#```csharp
// Before (Oracle)
var sql = "SELECT id, name FROM (USERS) WHERE status = @status";

// After (PostgreSQL)
var sql = "SELECT id, name FROM USERS WHERE status = @status";
```## 注意すべきエラー メッセージ```
Npgsql.PostgresException: 42601: syntax error at or near ")"
ERROR: syntax error at or near ")"
LINE 1: SELECT * FROM (TABLE_NAME) WHERE ...
                      ^
```## テストに関する推奨事項

1. **構文検証**: 移行されたすべてのクエリを解析して、構文エラーなしで実行されることを確認します。```csharp
   [Fact]
   public void GetEmployees_ExecutesWithoutSyntaxError()
   {
       // Should not throw PostgresException with error code 42601
       var employees = dal.GetEmployees(departmentId: 10);
       Assert.NotEmpty(employees);
   }
   ```2. **結果の比較**: 移行前と移行後の結果セットが同一であることを確認します。
3. **正規表現ベースの検索**: パターン `FROM\s*\(\s*[A-Za-z_][A-Za-z0-9_]*\s*\)` を使用して候補を特定します

## 関連ファイル

- 参考: [oracle-to-postgres-type-coercion.md](oracle-to-postgres-type-coercion.md) - その他の構文の違い
- PostgreSQL ドキュメント: [SELECT ステートメント](https://www.postgresql.org/docs/current/sql-select.html)

## 移行メモ

- これは、意味的な意味を持たない単純な構文修正です。
- データ変換は必要ありません
- 自動検索と置換を安全に適用できますが、複雑なクエリは手動で検証します
- 移行されたクエリを実行するために統合テストを更新します