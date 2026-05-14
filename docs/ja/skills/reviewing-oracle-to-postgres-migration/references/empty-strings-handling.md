# Oracle と PostgreSQL: 空文字列処理の違い

## 問題

Oracle は、VARCHAR2 列の空の文字列 (`''`) を `NULL` に自動的に変換します。 PostgreSQL は、空の文字列を `NULL` とは区別して保持します。この違いにより、移行中にアプリケーション ロジック エラーやテストの失敗が発生する可能性があります。

## 動作の比較

**オラクル:**
- 空の文字列 (`''`) は、VARCHAR2 列では **常に** `NULL` として扱われます
- `WHERE column = ''` は行と一致しません。 `WHERE column IS NULL`を使用してください
- 明示的な空文字列と `NULL` を区別できません

**PostgreSQL:**
- 空の文字列 (`''`) と `NULL` は **別の** 値です
- `WHERE column = ''` は空の文字列と一致します
- `WHERE column IS NULL` は `NULL` の値と一致します

## コード例```sql
-- Oracle behavior
INSERT INTO table (varchar_column) VALUES ('');
SELECT * FROM table WHERE varchar_column IS NULL;  -- Returns the row

-- PostgreSQL behavior  
INSERT INTO table (varchar_column) VALUES ('');
SELECT * FROM table WHERE varchar_column IS NULL;  -- Returns nothing
SELECT * FROM table WHERE varchar_column = '';     -- Returns the row
```## 移行アクション

### 1. ストアド プロシージャ
空の文字列が `NULL` に変換されることを前提とした更新ロジック:```sql
-- Preserve Oracle behavior (convert empty to NULL):
column = NULLIF(param, '')

-- Or accept PostgreSQL behavior (preserve empty string):
column = param
```### 2. アプリケーションコード
`NULL` をチェックするコードを確認し、空の文字列が適切に処理されることを確認します。```csharp
// Before (Oracle-specific)
if (value == null) { }

// After (PostgreSQL-compatible)
if (string.IsNullOrEmpty(value)) { }
```### 3. テスト
両方の動作と互換性があるようにアサーションを更新します。```csharp
// Migration-compatible test pattern
var value = reader.IsDBNull(columnIndex) ? null : reader.GetString(columnIndex);
Assert.IsTrue(string.IsNullOrEmpty(value));
```### 4. データ移行
次のことを行うかどうかを決定します。
- 既存の `NULL` 値を空の文字列に変換します
- `NULLIF(column, '')` を使用して空の文字列を `NULL` に変換します
- 値をそのままにして、アプリケーション ロジックを更新します