---
name: sql-code-review
description: 'Universal SQL code review assistant that performs comprehensive security, maintainability, and code quality analysis across all SQL databases (MySQL, PostgreSQL, SQL Server, Oracle). Focuses on SQL injection prevention, access control, code standards, and anti-pattern detection. Complements SQL optimization prompt for complete development coverage.'
---
# SQL コードのレビュー

セキュリティ、パフォーマンス、保守性、データベースのベスト プラクティスに焦点を当てて、${selection} (または選択がない場合はプロジェクト全体) の徹底的な SQL コード レビューを実行します。

## 🔒 セキュリティ分析

### SQL インジェクションの防止```sql
-- ❌ CRITICAL: SQL Injection vulnerability
query = "SELECT * FROM users WHERE id = " + userInput;
query = f"DELETE FROM orders WHERE user_id = {user_id}";

-- ✅ SECURE: Parameterized queries
-- PostgreSQL/MySQL
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
EXECUTE stmt USING @user_id;

-- SQL Server
EXEC sp_executesql N'SELECT * FROM users WHERE id = @id', N'@id INT', @id = @user_id;
```### アクセス制御と権限
- **最小特権の原則**: 必要な最小限の権限を付与します。
- **ロールベースのアクセス**: 直接のユーザー権限の代わりにデータベース ロールを使用します。
- **スキーマのセキュリティ**: 適切なスキーマの所有権とアクセス制御
- **関数/プロシージャのセキュリティ**: DEFINER と INVOKER の権限を確認する

### データ保護
- **機密データの漏洩**: 機密列を含むテーブルでは SELECT * を避けてください。
- **監査ログ**: 機密性の高い操作が確実にログに記録されるようにします
- **データ マスキング**: ビューまたは関数を使用して機密データをマスクします。
- **暗号化**: 機密データの暗号化されたストレージを検証します。

## ⚡ パフォーマンスの最適化

### クエリ構造の分析```sql
-- ❌ BAD: Inefficient query patterns
SELECT DISTINCT u.* 
FROM users u, orders o, products p
WHERE u.id = o.user_id 
AND o.product_id = p.id
AND YEAR(o.order_date) = 2024;

-- ✅ GOOD: Optimized structure
SELECT u.id, u.name, u.email
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.order_date >= '2024-01-01' 
AND o.order_date < '2025-01-01';
```### インデックス戦略のレビュー
- **欠落しているインデックス**: インデックス付けが必要な列を特定します。
- **過剰なインデックス作成**: 未使用または冗長なインデックスを検索します。
- **複合インデックス**: 複雑なクエリ用の複数列インデックス
- **インデックスのメンテナンス**: 断片化したインデックスや古いインデックスをチェックします。

### 結合の最適化
- **結合タイプ**: 適切な結合タイプを確認します (INNER、LEFT、EXISTS)。
- **結合順序**: 最初に小さい結果セットを最適化します。
- **デカルト積**: 欠落している結合条件を特定して修正します
- **サブクエリ vs JOIN**: 最も効率的なアプローチを選択してください

### 集計関数とウィンドウ関数```sql
-- ❌ BAD: Inefficient aggregation
SELECT user_id, 
       (SELECT COUNT(*) FROM orders o2 WHERE o2.user_id = o1.user_id) as order_count
FROM orders o1
GROUP BY user_id;

-- ✅ GOOD: Efficient aggregation
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id;
```## 🛠️ コードの品質と保守性

### SQL スタイルと書式設定```sql
-- ❌ BAD: Poor formatting and style
select u.id,u.name,o.total from users u left join orders o on u.id=o.user_id where u.status='active' and o.order_date>='2024-01-01';

-- ✅ GOOD: Clean, readable formatting
SELECT u.id,
       u.name,
       o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active'
  AND o.order_date >= '2024-01-01';
```### 命名規則
- **一貫した名前付け**: テーブル、列、制約は一貫したパターンに従います。
- **説明的な名前**: データベース オブジェクトの明確で意味のある名前
- **予約語**: データベースの予約語を識別子として使用しないでください。
- **大文字と小文字の区別**: スキーマ全体で一貫した大文字と小文字の使用

### スキーマ設計のレビュー
- **正規化**: 適切な正規化レベル (過剰または過小な正規化を避ける)
- **データ型**: ストレージとパフォーマンスに最適なデータ型の選択
- **制約**: PRIMARY KEY、FOREIGN KEY、CHECK、NOT NULL の適切な使用
- **デフォルト値**: 列の適切なデフォルト値

## 🗄️ データベース固有のベスト プラクティス

### PostgreSQL```sql
-- Use JSONB for JSON data
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- GIN index for JSONB queries
CREATE INDEX idx_events_data ON events USING gin(data);

-- Array types for multi-value columns
CREATE TABLE tags (
    post_id INT,
    tag_names TEXT[]
);
```### MySQL```sql
-- Use appropriate storage engines
CREATE TABLE sessions (
    id VARCHAR(128) PRIMARY KEY,
    data TEXT,
    expires TIMESTAMP
) ENGINE=InnoDB;

-- Optimize for InnoDB
ALTER TABLE large_table 
ADD INDEX idx_covering (status, created_at, id);
```### SQL サーバー```sql
-- Use appropriate data types
CREATE TABLE products (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at DATETIME2 DEFAULT GETUTCDATE()
);

-- Columnstore indexes for analytics
CREATE COLUMNSTORE INDEX idx_sales_cs ON sales;
```### オラクル```sql
-- Use sequences for auto-increment
CREATE SEQUENCE user_id_seq START WITH 1 INCREMENT BY 1;

CREATE TABLE users (
    id NUMBER DEFAULT user_id_seq.NEXTVAL PRIMARY KEY,
    name VARCHAR2(255) NOT NULL
);
```## 🧪 テストと検証

### データ整合性チェック```sql
-- Verify referential integrity
SELECT o.user_id 
FROM orders o 
LEFT JOIN users u ON o.user_id = u.id 
WHERE u.id IS NULL;

-- Check for data consistency
SELECT COUNT(*) as inconsistent_records
FROM products 
WHERE price < 0 OR stock_quantity < 0;
```### パフォーマンステスト
- **実行計画**: クエリ実行計画を確認します。
- **負荷テスト**: 現実的なデータ量でクエリをテストします。
- **ストレス テスト**: 同時負荷下でのパフォーマンスを検証します。
- **回帰テスト**: 最適化によって機能が損なわれないことを確認します

## 📊 一般的なアンチパターン

### N+1 クエリの問題```sql
-- ❌ BAD: N+1 queries in application code
for user in users:
    orders = query("SELECT * FROM orders WHERE user_id = ?", user.id)

-- ✅ GOOD: Single optimized query
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```### DISTINCT の乱用```sql
-- ❌ BAD: DISTINCT masking join issues
SELECT DISTINCT u.name 
FROM users u, orders o 
WHERE u.id = o.user_id;

-- ✅ GOOD: Proper join without DISTINCT
SELECT u.name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```### WHERE 句での関数の誤用```sql
-- ❌ BAD: Functions prevent index usage
SELECT * FROM orders 
WHERE YEAR(order_date) = 2024;

-- ✅ GOOD: Range conditions use indexes
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';
```## 📋 SQL レビューのチェックリスト

### セキュリティ
- [ ] すべてのユーザー入力はパラメータ化されます
- [ ] 文字列連結による動的 SQL 構築なし
- [ ] 適切なアクセス制御と権限
- [ ] 機密データは適切に保護されています
- [ ] SQL インジェクション攻撃ベクトルが排除される

### パフォーマンス
- [ ] 頻繁にクエリされる列にはインデックスが存在します
- [ ] 不要な SELECT * ステートメントは不要
- [ ] JOIN は最適化され、適切なタイプが使用されます
- [ ] WHERE 句は選択的であり、インデックスを使用します
- [ ] サブクエリは最適化されるか、JOIN に変換されます。

### コードの品質
- [ ] 一貫した命名規則
- [ ] 適切な書式設定とインデント
- [ ] 複雑なロジックに対する意味のあるコメント
- [ ] 適切なデータ型が使用されています
- [ ] エラー処理が実装されています

### スキーマ設計
- [ ] テーブルは適切に正規化されています
- [ ] 制約はデータの整合性を強制します
- [ ] インデックスはクエリ パターンをサポートします
- [ ] 外部キー関係が定義されています
- [ ] デフォルト値は適切です

## 🎯 出力形式を確認する

### 問題テンプレート```
## [PRIORITY] [CATEGORY]: [Brief Description]

**Location**: [Table/View/Procedure name and line number if applicable]
**Issue**: [Detailed explanation of the problem]
**Security Risk**: [If applicable - injection risk, data exposure, etc.]
**Performance Impact**: [Query cost, execution time impact]
**Recommendation**: [Specific fix with code example]

**Before**:
```SQL
-- 問題のある SQL```

**After**:
```SQL
-- SQL の改善```

**Expected Improvement**: [Performance gain, security benefit]
```### 要約評価
- **セキュリティ スコア**: [1-10] - SQL インジェクション保護、アクセス制御
- **パフォーマンス スコア**: [1-10] - クエリ効率、インデックス使用率
- **保守性スコア**: [1-10] - コードの品質、ドキュメント
- **スキーマ品質スコア**: [1-10] - 設計パターン、正規化

### 優先アクション上位 3
1. **[重要なセキュリティ修正]**: SQL インジェクションの脆弱性に対処します。
2. **[パフォーマンスの最適化]**: 不足しているインデックスを追加するか、クエリを最適化します。
3. **[コード品質]**: 命名規則とドキュメントを改善します。

プラットフォーム固有の最適化とベスト プラクティスを強調しながら、データベースに依存しない実用的な推奨事項を提供することに重点を置きます。