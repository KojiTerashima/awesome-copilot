---
name: postgresql-optimization
description: 'PostgreSQL-specific development assistant focusing on unique PostgreSQL features, advanced data types, and PostgreSQL-exclusive capabilities. Covers JSONB operations, array types, custom types, range/geometric types, full-text search, window functions, and PostgreSQL extensions ecosystem.'
---
# PostgreSQL 開発アシスタント

${selection} (選択がない場合はプロジェクト全体) に関する専門家による PostgreSQL ガイダンス。 PostgreSQL 固有の機能、最適化パターン、高度な機能に焦点を当てます。

## � PostgreSQL 固有の機能

### JSONB 操作```SQL
-- 高度な JSONB クエリ
CREATE TABLE イベント (
    id シリアル主キー、
    データ JSONB NOT NULL、
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- JSONB パフォーマンスの GIN インデックス
CREATE INDEX idx_events_data_gin ON イベント USING gin(data);

-- JSONB の包含とパスのクエリ
SELECT * FROM イベント 
WHERE データ @> '{"タイプ": "ログイン"}'
  AND データ #>> '{user,role}' = 'admin';

-- JSONB 集約
SELECT jsonb_agg(data) FROM events WHERE data ? 'ユーザーID';
「」### 配列操作```SQL
-- PostgreSQL 配列
CREATE TABLE の投稿 (
    id シリアル主キー、
    タグ TEXT[]、
    カテゴリ 整数[]
);

-- 配列のクエリと操作
SELECT * FROM 投稿 WHERE 'postgresql' = ANY(タグ);
SELECT * FROM 投稿 WHERE タグ && ARRAY['database', 'sql'];
SELECT * FROM 投稿 WHERE array_length(tags, 1) > 3;

-- 配列の集約
SELECT array_agg(DISTINCT category) FROM 投稿、unnest(categories) をカテゴリとして;
「」### ウィンドウ関数と分析```SQL
-- 高度なウィンドウ機能
選択 
    製品ID、
    発売日、
    金額、
    -- 現在の合計
    SUM(金額) OVER (PARTITION BY product_id ORDER BY sale_date) as running_total、
    -- 移動平均
    AVG(amount) OVER (PARTITION BY product_id ORDER BY sale_date ROWS BETWEEN 2 BETWEEN 先行行と現在の行) as move_avg,
    -- ランキング
    DENSE_RANK() OVER (PARTITION BY EXTRACT(month FROM sale_date) ORDER BY amount DESC) as month_rank,
    -- 比較のための遅れ/進み
    LAG(amount, 1) OVER (PARTITION BY product_id ORDER BY sale_date) as prev_amount
売上から;
「」### 全文検索```SQL
-- PostgreSQL の全文検索
CREATE TABLE ドキュメント (
    id シリアル主キー、
    タイトルテキスト、
    コンテンツテキスト、
    検索ベクトル tsvector
);

-- 検索ベクトルを更新します
ドキュメントを更新する 
SET search_vector = to_tsvector('英語', タイトル || ' ' || コンテンツ);

-- 検索パフォーマンスのための GIN インデックス
CREATE INDEX idx_documents_search ON ドキュメント USING gin(search_vector);

-- 検索クエリ
SELECT * FROM ドキュメント 
WHERE search_vector @@ plainto_tsquery('英語', 'postgresql データベース');

-- ランキング結果
ランクとして SELECT *, ts_rank(search_vector, plainto_tsquery('postgresql'))
ドキュメントから 
WHERE search_vector @@ plainto_tsquery('postgresql')
ランク DESC で注文;
「」## � PostgreSQL のパフォーマンス チューニング

### クエリの最適化```SQL
-- EXPLAIN ANALYZE (パフォーマンス分析用)
EXPLAIN (分析、バッファ、テキストのフォーマット) 
order_count として SELECT u.name, COUNT(o.id)
ユーザーからのあなた
LEFT JOIN 命令 o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'::date
u.id、u.nameによるグループ化;

-- pg_stat_statements から遅いクエリを特定する
SELECT クエリ、呼び出し、total_time、mean_time、行、
       100.0 *shared_blks_hit / nullif(shared_blks_hit +shared_blks_read, 0) AS hit_percent
pg_stat_statements から 
ORDER BY total_time DESC 
リミット10;
「」### インデックス戦略```SQL
-- 複数列クエリの複合インデックス
CREATE INDEX idx_orders_user_date ON 注文(user_id, order_date);

-- フィルタリングされたクエリの部分インデックス
CREATE INDEX idx_active_users ON users(created_at) WHERE status = 'active';

-- 計算値の式インデックス
CREATE INDEX idx_users_ lower_email ON users( lower(email));

-- テーブル検索を避けるためのインデックスのカバー
CREATE INDEX idx_orders_covering ONorders(user_id, status) INCLUDE (total, created_at);
「」### 接続とメモリの管理```SQL
-- 接続の使用状況を確認する
接続、状態として count(*) を選択します 
FROM pg_stat_activity 
GROUP BY 状態。

-- メモリ使用量を監視する
SELECT 名前、設定、単位 
pg_settings から 
WHERE 名 IN ('shared_buffers', 'work_mem', 'maintenance_work_mem');
「」## �️ PostgreSQL の高度なデータ型

### カスタムタイプとドメイン```SQL
-- カスタム タイプを作成する
CREATE TYPE address_type AS (
    ストリートテキスト、
    都市テキスト、
    郵便番号テキスト、
    国のテキスト
);

CREATE TYPE order_status AS ENUM ('保留中'、'処理中'、'発送済み'、'配達済み'、'キャンセル');

-- データ検証にドメインを使用する
ドメインのメールアドレスをテキストとして作成 
CHECK (VALUE ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- カスタム タイプを使用したテーブル
CREATE TABLE の顧客 (
    id シリアル主キー、
    電子メール email_address が NULL ではありません。
    アドレスのアドレスタイプ、
    ステータス order_status DEFAULT '保留中'
);
「」### 範囲の種類```SQL
-- PostgreSQL 範囲タイプ
CREATE TABLE 予約 (
    id シリアル主キー、
    room_id INTEGER、
    予約期間 tstzrange、
    価格範囲の数値範囲
);

-- 範囲クエリ
予約から * を選択 
WHERE 予約期間 && tstzrange('2024-07-20', '2024-07-25');

-- 重複する範囲を除外します
ALTER TABLE 予約 
制約を追加 no_overlap 
EXCLUDE USING gist (room_id WITH =、reservation_period WITH &&);
「」### 幾何学的タイプ```SQL
-- PostgreSQL の幾何学的タイプ
CREATE TABLE の場所 (
    id シリアル主キー、
    名前テキスト、
    コーディネートポイント、
    取材サークル、
    サービスエリアポリゴン
);

-- 幾何学的クエリ
場所から名前を選択 
WHERE 座標 <-> point(40.7128, -74.0060) < 10; -- 10単位以内

-- 幾何学的データの GiST インデックス
CREATE INDEX idx_locations_coords ON の場所 USING gist(座標);
「」## 📊 PostgreSQL 拡張機能とツール

### 便利な拡張機能```SQL
-- よく使用される拡張機能を有効にする
「uuid-ossp」が存在しない場合は拡張機能を作成します。    -- UUID の生成
「pgcrypto」が存在しない場合は拡張機能を作成します。     -- 暗号化機能
「アクセントがない」場合は拡張子を作成します。     -- テキストからアクセントを削除します。
「pg_trgm」が存在しない場合は拡張機能を作成します。      -- トライグラムマッチング
「btree_gin」が存在しない場合は拡張機能を作成します。    -- btree タイプの GIN インデックス

-- 拡張機能の使用
SELECT uuid_generate_v4();                     -- UUID を生成する
SELECT crypt('パスワード', gen_salt('bf'));      -- ハッシュパスワード
SELECT 類似性('postgresql', 'postgersql'); -- ファジーマッチング
「」### 監視とメンテナンス```SQL
-- データベースのサイズと増加
db_size として pg_size_pretty(pg_database_size(current_database())) を選択します。

-- テーブルとインデックスのサイズ
SELECT スキーマ名、テーブル名、
       pg_size_pretty(pg_total_relation_size(スキーマ名||'.'||テーブル名)) サイズとして
pg_tables から 
ORDER BY pg_total_relation_size(スキーマ名||'.'||テーブル名) DESC;

-- インデックス使用統計
SELECT スキーマ名、テーブル名、インデックス名、idx_scan、idx_tup_read、idx_tup_fetch
FROM pg_stat_user_indexes 
WHERE idx_scan = 0;  -- 未使用のインデックス
「」### PostgreSQL 固有の最適化のヒント
- **詳細なクエリ分析には EXPLAIN (ANALYZE, BUFFERS)** を使用します
- **ワークロードに合わせて postgresql.conf を構成します** (OLTP と OLAP)
- **同時実行性の高いアプリケーションには接続プーリングを使用** (pgbouncer)
- **定期的なVACUUMとANALYZE**による最適なパフォーマンス
- **PostgreSQL 10+ 宣言型パーティション分割を使用して大きなテーブルをパーティション分割**
- **クエリ パフォーマンスの監視には pg_stat_statements を使用します**

## 📊 監視とメンテナンス

### クエリパフォーマンスの監視```SQL
-- 遅いクエリを特定する
SELECT クエリ、呼び出し、total_time、mean_time、行数
pg_stat_statements から 
ORDER BY total_time DESC 
リミット10;

-- インデックスの使用状況を確認する
SELECT スキーマ名、テーブル名、インデックス名、idx_scan、idx_tup_read、idx_tup_fetch
FROM pg_stat_user_indexes 
WHERE idx_scan = 0;
「」### データベースのメンテナンス
- **バキュームと分析**: パフォーマンスのための定期的なメンテナンス
- **インデックスのメンテナンス**: 断片化したインデックスを監視および再構築します。
- **統計の更新**: クエリ プランナーの統計を最新の状態に保ちます
- **ログ分析**: PostgreSQL ログの定期的なレビュー

## 🛠️ 一般的なクエリ パターン

### ページネーション```SQL
-- ❌ 悪い: 大規模なデータセットのオフセット
SELECT * FROM 製品 ID で注文 オフセット 10000 LIMIT 20;

-- ✅ 良い点: カーソルベースのページネーション
製品から * を選択してください 
WHERE id > $last_id 
IDで注文 
制限 20;
「」### 集計```SQL
-- ❌ 悪い点: 非効率的なグループ化
SELECT ユーザー ID、COUNT(*) 
注文から 
WHERE 注文日 >= '2024-01-01' 
GROUP BY user_id;

-- ✅ 良い: 部分インデックスで最適化されています。
CREATE INDEX idx_orders_recent ON 注文(user_id) 
WHERE 注文日 >= '2024-01-01';

SELECT ユーザー ID、COUNT(*) 
注文から 
WHERE 注文日 >= '2024-01-01' 
GROUP BY user_id;
「」### JSON クエリ```SQL
-- ❌ 悪い点: 非効率的な JSON クエリ
SELECT * FROM users WHERE data::text LIKE '%admin%';

-- ✅ 良い点: JSONB 演算子と GIN インデックス
CREATE INDEX idx_users_data_gin ON ユーザー USING gin(data);

SELECT * FROM users WHERE data @> '{"role": "admin"}';
「」## 📋 最適化チェックリスト

### クエリ分析
- [ ] 負荷の高いクエリに対して EXPLAIN ANALYZE を実行する
- [ ] 大きなテーブルの順次スキャンをチェックします
- [ ] 適切な結合アルゴリズムを確認します。
- [ ] WHERE 句の選択性を確認する
- [ ] 並べ替えおよび集計操作を分析します。

### インデックス戦略
- [ ] 頻繁にクエリされる列のインデックスを作成します
- [ ] 複数列検索に複合インデックスを使用する
- [ ] フィルタリングされたクエリの部分インデックスを考慮する
- [ ] 未使用または重複したインデックスを削除します。
- [ ] インデックスの肥大化と断片化を監視します

### セキュリティのレビュー
- [ ] パラメータ化されたクエリを排他的に使用します
- [ ] 適切なアクセス制御を実装する
- [ ] 必要に応じて行レベルのセキュリティを有効にします
- [ ] 機密データへのアクセスを監査する
- [ ] 安全な接続方法を使用する

### パフォーマンスの監視
- [ ] クエリ パフォーマンス監視を設定する
- [ ] 適切なログ設定を構成します
- [ ] 接続プールの使用状況を監視する
- [ ] データベースの成長とメンテナンスのニーズを追跡する
- [ ] パフォーマンス低下のアラートを設定します

## 🎯 最適化出力形式

### クエリ分析結果「」
## クエリのパフォーマンス分析

**元のクエリ**:
[パフォーマンス上の問題のある元の SQL]

**特定された問題**:
- 大きなテーブルでの順次スキャン (コスト: 15000.00)
- 頻繁にクエリされる列のインデックスがありません
- 非効率的な結合順序

**最適化されたクエリ**:
[説明付きの改良されたSQL]

**推奨されるインデックス**:
「」SQL
CREATE INDEX idx_table_column ON テーブル(列);「」

**パフォーマンスへの影響**: 実行時間の 80% の向上が期待されます。
「」## 🚀 高度な PostgreSQL 機能

### ウィンドウ関数```SQL
-- 現在の合計とランキング
選択 
    製品ID、
    注文日、
    金額、
    SUM(金額) OVER (PARTITION BY product_id ORDER BY order_date) as running_total、
    ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY amount DESC) as Rank
売上から;
「」### 共通テーブル式 (CTE)```SQL
-- 階層データの再帰的クエリ
WITH RECURSIVE category_tree AS (
    ID、名前、parent_id、レベルとして 1 を選択します
    カテゴリから 
    ここで、parent_id は NULL です
    
    すべてを結合する
    
    SELECT c.id、c.name、c.parent_id、ct.level + 1
    カテゴリcから
    JOIN category_tree ct ON c.parent_id = ct.id
）
SELECT * FROM category_tree ORDER BY レベル、名前;
「」PostgreSQL の高度な機能を活用しながら、クエリのパフォーマンス、セキュリティ、保守性を向上させる、具体的で実行可能な PostgreSQL の最適化を提供することに重点を置きます。