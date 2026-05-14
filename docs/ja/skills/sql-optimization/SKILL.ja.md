---
name: sql-optimization
description: 'Universal SQL performance optimization assistant for comprehensive query tuning, indexing strategies, and database performance analysis across all SQL databases (MySQL, PostgreSQL, SQL Server, Oracle). Provides execution plan analysis, pagination optimization, batch operations, and performance monitoring guidance.'
---
# SQL パフォーマンス最適化アシスタント

${selection} (選択がない場合はプロジェクト全体) に対するエキスパート SQL パフォーマンスの最適化。 MySQL、PostgreSQL、SQL Server、Oracle、その他の SQL データベース全体で機能するユニバーサル SQL 最適化手法に焦点を当てます。

## 🎯 コア最適化領域

### クエリのパフォーマンス分析```sql
-- ❌ BAD: Inefficient query patterns
SELECT * FROM orders o
WHERE YEAR(o.created_at) = 2024
  AND o.customer_id IN (
      SELECT c.id FROM customers c WHERE c.status = 'active'
  );

-- ✅ GOOD: Optimized query with proper indexing hints
SELECT o.id, o.customer_id, o.total_amount, o.created_at
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
WHERE o.created_at >= '2024-01-01' 
  AND o.created_at < '2025-01-01'
  AND c.status = 'active';

-- Required indexes:
-- CREATE INDEX idx_orders_created_at ON orders(created_at);
-- CREATE INDEX idx_customers_status ON customers(status);
-- CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```### インデックス戦略の最適化```sql
-- ❌ BAD: Poor indexing strategy
CREATE INDEX idx_user_data ON users(email, first_name, last_name, created_at);

-- ✅ GOOD: Optimized composite indexing
-- For queries filtering by email first, then sorting by created_at
CREATE INDEX idx_users_email_created ON users(email, created_at);

-- For full-text name searches
CREATE INDEX idx_users_name ON users(last_name, first_name);

-- For user status queries
CREATE INDEX idx_users_status_created ON users(status, created_at)
WHERE status IS NOT NULL;
```### サブクエリの最適化```sql
-- ❌ BAD: Correlated subquery
SELECT p.product_name, p.price
FROM products p
WHERE p.price > (
    SELECT AVG(price) 
    FROM products p2 
    WHERE p2.category_id = p.category_id
);

-- ✅ GOOD: Window function approach
SELECT product_name, price
FROM (
    SELECT product_name, price,
           AVG(price) OVER (PARTITION BY category_id) as avg_category_price
    FROM products
) ranked
WHERE price > avg_category_price;
```## 📊 パフォーマンス チューニング テクニック

### 結合の最適化```sql
-- ❌ BAD: Inefficient JOIN order and conditions
SELECT o.*, c.name, p.product_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id
WHERE o.created_at > '2024-01-01'
  AND c.status = 'active';

-- ✅ GOOD: Optimized JOIN with filtering
SELECT o.id, o.total_amount, c.name, p.product_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id AND c.status = 'active'
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.created_at > '2024-01-01';
```### ページネーションの最適化```sql
-- ❌ BAD: OFFSET-based pagination (slow for large offsets)
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 10000;

-- ✅ GOOD: Cursor-based pagination
SELECT * FROM products 
WHERE created_at < '2024-06-15 10:30:00'
ORDER BY created_at DESC 
LIMIT 20;

-- Or using ID-based cursor
SELECT * FROM products 
WHERE id > 1000
ORDER BY id 
LIMIT 20;
```### 集計の最適化```sql
-- ❌ BAD: Multiple separate aggregation queries
SELECT COUNT(*) FROM orders WHERE status = 'pending';
SELECT COUNT(*) FROM orders WHERE status = 'shipped';
SELECT COUNT(*) FROM orders WHERE status = 'delivered';

-- ✅ GOOD: Single query with conditional aggregation
SELECT 
    COUNT(CASE WHEN status = 'pending' THEN 1 END) as pending_count,
    COUNT(CASE WHEN status = 'shipped' THEN 1 END) as shipped_count,
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) as delivered_count
FROM orders;
```## 🔍 クエリのアンチパターン

### SELECT のパフォーマンスの問題```sql
-- ❌ BAD: SELECT * anti-pattern
SELECT * FROM large_table lt
JOIN another_table at ON lt.id = at.ref_id;

-- ✅ GOOD: Explicit column selection
SELECT lt.id, lt.name, at.value
FROM large_table lt
JOIN another_table at ON lt.id = at.ref_id;
```### WHERE 句の最適化```sql
-- ❌ BAD: Function calls in WHERE clause
SELECT * FROM orders 
WHERE UPPER(customer_email) = 'JOHN@EXAMPLE.COM';

-- ✅ GOOD: Index-friendly WHERE clause
SELECT * FROM orders 
WHERE customer_email = 'john@example.com';
-- Consider: CREATE INDEX idx_orders_email ON orders(LOWER(customer_email));
```### OR と UNION の最適化```sql
-- ❌ BAD: Complex OR conditions
SELECT * FROM products 
WHERE (category = 'electronics' AND price < 1000)
   OR (category = 'books' AND price < 50);

-- ✅ GOOD: UNION approach for better optimization
SELECT * FROM products WHERE category = 'electronics' AND price < 1000
UNION ALL
SELECT * FROM products WHERE category = 'books' AND price < 50;
```## 📈 データベースに依存しない最適化

### バッチ操作```sql
-- ❌ BAD: Row-by-row operations
INSERT INTO products (name, price) VALUES ('Product 1', 10.00);
INSERT INTO products (name, price) VALUES ('Product 2', 15.00);
INSERT INTO products (name, price) VALUES ('Product 3', 20.00);

-- ✅ GOOD: Batch insert
INSERT INTO products (name, price) VALUES 
('Product 1', 10.00),
('Product 2', 15.00),
('Product 3', 20.00);
```### 一時テーブルの使用法```sql
-- ✅ GOOD: Using temporary tables for complex operations
CREATE TEMPORARY TABLE temp_calculations AS
SELECT customer_id, 
       SUM(total_amount) as total_spent,
       COUNT(*) as order_count
FROM orders 
WHERE created_at >= '2024-01-01'
GROUP BY customer_id;

-- Use the temp table for further calculations
SELECT c.name, tc.total_spent, tc.order_count
FROM temp_calculations tc
JOIN customers c ON tc.customer_id = c.id
WHERE tc.total_spent > 1000;
```## 🛠️ インデックス管理

### インデックス設計原則```sql
-- ✅ GOOD: Covering index design
CREATE INDEX idx_orders_covering 
ON orders(customer_id, created_at) 
INCLUDE (total_amount, status);  -- SQL Server syntax
-- Or: CREATE INDEX idx_orders_covering ON orders(customer_id, created_at, total_amount, status); -- Other databases
```### 部分インデックス戦略```sql
-- ✅ GOOD: Partial indexes for specific conditions
CREATE INDEX idx_orders_active 
ON orders(created_at) 
WHERE status IN ('pending', 'processing');
```## 📊 パフォーマンス監視クエリ

### クエリのパフォーマンス分析```sql
-- Generic approach to identify slow queries
-- (Specific syntax varies by database)

-- For MySQL:
SELECT query_time, lock_time, rows_sent, rows_examined, sql_text
FROM mysql.slow_log
ORDER BY query_time DESC;

-- For PostgreSQL:
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC;

-- For SQL Server:
SELECT 
    qs.total_elapsed_time/qs.execution_count as avg_elapsed_time,
    qs.execution_count,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset WHEN -1 THEN DATALENGTH(qt.text)
        ELSE qs.statement_end_offset END - qs.statement_start_offset)/2)+1) as query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_elapsed_time DESC;
```## 🎯 ユニバーサル最適化チェックリスト

### クエリ構造
- [ ] 本番クエリでの SELECT * の回避
- [ ] 適切な JOIN タイプの使用 (INNER または LEFT/RIGHT)
- [ ] WHERE 句の早い段階でフィルタリングする
- [ ] 必要に応じてサブクエリに IN の代わりに EXISTS を使用する
- [ ] インデックスの使用を妨げる WHERE 句内の関数の回避

### インデックス戦略
- [ ] 頻繁にクエリされる列にインデックスを作成する
- [ ] 複合インデックスを正しい列順序で使用する
- [ ] 過剰なインデックス作成の回避 (INSERT/UPDATE のパフォーマンスに影響します)
- [ ] 有益な場合はカバーインデックスを使用する
- [ ] 特定のクエリ パターンに対する部分インデックスの作成

### データ型とスキーマ
- [ ] ストレージ効率を高めるために適切なデータ型を使用する
- [ ] 適切な正規化 (OLTP の場合は 3NF、OLAP の場合は非正規化)
- [ ] 制約を使用してクエリ オプティマイザーを支援する
- [ ] 適切な場合に大きなテーブルを分割する

### クエリパターン
- [ ] 結果セット制御に LIMIT/TOP を使用する
- [ ] 効率的なページネーション戦略の実装
- [ ] 一括データ変更にバッチ操作を使用する
- [ ] N+1 クエリの問題を回避する
- [ ] 繰り返しのクエリに準備済みステートメントを使用する

### パフォーマンステスト
- [ ] 現実的なデータ量でクエリをテストする
- [ ] クエリ実行プランの分析
- [ ] 時間の経過に伴うクエリ パフォーマンスの監視
- [ ] 遅いクエリに対するアラートの設定
- [ ] 通常のインデックス使用状況の分析

## 📝 最適化手法

1. **特定**: データベース固有のツールを使用して遅いクエリを見つける
2. **分析**: 実行計画を調査し、ボトルネックを特定します。
3. **最適化**: 適切な最適化手法を適用します。
4. **テスト**: パフォーマンスの向上を確認します。
5. **監視**: パフォーマンス指標を継続的に追跡します。
6. **反復**: 定期的なパフォーマンスのレビューと最適化

測定可能なパフォーマンスの向上に焦点を当て、現実的なデータ量とクエリ パターンを使用して最適化を常にテストします。