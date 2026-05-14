---
name: postgresql-code-review
description: 'PostgreSQL-specific code review assistant focusing on PostgreSQL best practices, anti-patterns, and unique quality standards. Covers JSONB operations, array usage, custom types, schema design, function optimization, and PostgreSQL-exclusive security features like Row Level Security (RLS).'
---
# PostgreSQL コードレビューアシスタント

${selection} (選択がない場合はプロジェクト全体) の専門家による PostgreSQL コード レビュー。 PostgreSQL 固有のベスト プラクティス、アンチパターン、および PostgreSQL に固有の品質基準に焦点を当てます。

## 🎯 PostgreSQL 固有のレビュー領域

### JSONB のベスト プラクティス```SQL
-- ❌ 悪い点: 非効率的な JSONB の使用法
SELECT * FROM 注文 WHERE data->>'ステータス' = '発送済み';  -- インデックスはサポートされていません

-- ✅ 良い: インデックス可能な JSONB クエリ
CREATE INDEX idx_orders_status ON 注文 USING gin((data->'status'));
SELECT * FROM 注文 WHERE データ @> '{"ステータス": "発送済み"}';

-- ❌ 悪い点: 考慮せずに深いネストを作成する
UPDATE 命令 SET データ = データ || '{"配送":{"追跡":{"番号":"123"}}}';

-- ✅ 良い: 検証付きの構造化 JSONB
ALTER TABLE は ADD CONSTRAINT を命令します valid_status 
CHECK (データ->>'ステータス' IN ('保留', '発送済み', '配達済み'));
「」### 配列操作のレビュー```SQL
-- ❌ 悪い点: 非効率的な配列操作
SELECT * FROM products WHERE 'エレクトロニクス' = ANY(カテゴリ);  -- インデックスなし

-- ✅ 良い: GIN インデックス付き配列クエリ
ジン(カテゴリ)を使用して製品に関するインデックスidx_products_categoriesを作成します。
SELECT * FROM products WHERE カテゴリ @> ARRAY['electronics'];

-- ❌ 悪い: ループ内の配列の連結
-- これは関数/プロシージャでは非効率的です

-- ✅ 良い点: 一括配列操作
商品を更新 カテゴリ = カテゴリを設定 || ARRAY['新しいカテゴリ']
WHERE id IN (製品 WHERE 条件から ID を選択);
「」### PostgreSQL スキーマ設計のレビュー```SQL
-- ❌ 悪い点: PostgreSQL の機能を使用していない
CREATE TABLE ユーザー (
    id 整数、
    電子メール VARCHAR(255)、
    created_at TIMESTAMP
);

-- ✅ 良い: PostgreSQL に最適化されたスキーマ
CREATE TABLE ユーザー (
    id BIGSERIAL 主キー、
    電子メール CITEXT UNIQUE NOT NULL、 -- 大文字と小文字を区別しない電子メール
    created_at TIMESTAMPTZ DEFAULT NOW()、
    メタデータ JSONB DEFAULT '{}'、
    CONSTRAINT valid_email CHECK (電子メール ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- メタデータ クエリ用の JSONB GIN インデックスを追加
CREATE INDEX idx_users_metadata ON users USING gin(metadata);
「」### カスタムタイプとドメイン```SQL
-- ❌ 悪い点: 特定のデータにジェネリック型を使用する
CREATE TABLE トランザクション (
    金額 DECIMAL(10,2)、
    通貨 VARCHAR(3)、
    ステータス VARCHAR(20)
);

-- ✅ 良い点: PostgreSQL のカスタム タイプ
CREATE TYPE 通貨コード AS ENUM ('USD', 'EUR', 'GBP', 'JPY');
CREATE TYPEtransaction_status AS ENUM ('保留中'、'完了'、'失敗'、'キャンセル');
CREATE DOMAIN 正の量 AS DECIMAL(10,2) CHECK (VALUE > 0);

CREATE TABLE トランザクション (
    金額positive_amount NOT NULL、
    通貨通貨コードが NULL ではありません。
    ステータスtransaction_status DEFAULT '保留中'
);
「」## 🔍 PostgreSQL 固有のアンチパターン

### パフォーマンスのアンチパターン
- **PostgreSQL 固有のインデックスの回避**: 適切なデータ型に GIN/GiST を使用しない
- **JSONB の誤用**: JSONB を単純な文字列フィールドのように扱う
- **配列演算子の無視**: 非効率的な配列演算の使用
- **パーティション キーの選択が不適切**: PostgreSQL のパーティション分割が効果的に活用されていない

### スキーマ設計の問題
- **ENUM 型を使用しない**: 制限された値セットには VARCHAR を使用する
- **制約の無視**: データ検証のための CHECK 制約がありません
- **間違ったデータ型**: TEXT または CITEXT の代わりに VARCHAR を使用しています
- **JSONB 構造がありません**: 検証されていない構造化されていない JSONB

### 関数とトリガーの問題```SQL
-- ❌ BAD: 非効率的なトリガー機能
関数の作成または置換 update_modified_time()
トリガーを $$ として返します
始める
    NEW.updated_at = NOW();  -- TIMESTAMPTZ を使用する必要があります
    新しいものを返します。
終わり;
$$ 言語 plpgsql;

-- ✅ 良い: 最適化されたトリガー機能
関数の作成または置換 update_modified_time()
トリガーを $$ として返します
始める
    NEW.updated_at = CURRENT_TIMESTAMP;
    新しいものを返します。
終わり;
$$ 言語 plpgsql;

-- 必要な場合にのみトリガーを起動するように設定します。
トリガーの作成 update_modified_time_trigger
    table_name の更新前
    行ごとに
    いつ (古い.* は新しい.* と区別されます)
    関数の実行 update_modified_time();
「」## 📊 PostgreSQL 拡張機能の使用状況のレビュー

### 拡張機能のベストプラクティス```SQL
-- ✅ 作成する前に拡張機能が存在するかどうかを確認してください
「uuid-ossp」が存在しない場合は拡張機能を作成します。
「pgcrypto」が存在しない場合は拡張機能を作成します。
「pg_trgm」が存在しない場合は拡張機能を作成します。

-- ✅ 拡張機能を適切に使用する
-- UUID の生成
SELECT uuid_generate_v4();

-- パスワードのハッシュ化
SELECT crypt('パスワード', gen_salt('bf'));

-- ファジーテキストマッチング
SELECT word_similarity('postgres', 'postgre');
「」## 🛡️ PostgreSQL セキュリティのレビュー

### 行レベルセキュリティ (RLS)```SQL
-- ✅ 良い点: RLS の実装
ALTER TABLEsensitive_data 行レベルのセキュリティを有効にする;

CREATE POLICY user_data_policy ONsensitive_data
    application_role に対するすべての者
    USING (user_id = current_setting('app.current_user_id')::INTEGER);
「」### 権限管理```SQL
-- ❌ 悪い点: 権限が広すぎる
スキーマ public 内のすべてのテーブルに対するすべての権限を app_user に付与します。

-- ✅ 良い点: きめ細かな権限
app_user に specific_table の SELECT、INSERT、UPDATE を許可します。
シーケンス specific_table_id_seq の使用を app_user に許可します。
「」## 🎯 PostgreSQL コード品質チェックリスト

### スキーマ設計
- [ ] 適切な PostgreSQL データ型 (CITEXT、JSONB、配列) の使用
- [ ] 制約された値に ENUM タイプを利用する
- [ ] 適切な CHECK 制約の実装
- [ ] TIMESTAMP の代わりに TIMESTAMPTZ を使用する
- [ ] 再利用可能な制約のためのカスタム ドメインの定義

### パフォーマンスに関する考慮事項
- [ ] 適切なインデックス タイプ (JSONB/配列の場合は GIN、範囲の場合は GiST)
- [ ] 包含演算子 (@>、?) を使用した JSONB クエリ
- [ ] PostgreSQL 固有の演算子を使用した配列操作
- [ ] ウィンドウ関数と CTE の適切な使用
- [ ] PostgreSQL 固有の機能の効率的な使用

### PostgreSQL の機能の利用
- [ ] 必要に応じて拡張機能を使用する
- [ ] PL/pgSQL にストアド プロシージャを実装すると効果的です
- [ ] PostgreSQL の高度な SQL 機能の活用
- [ ] PostgreSQL 固有の最適化手法の使用
- [ ] 関数での適切なエラー処理の実装

### セキュリティとコンプライアンス
- [ ] 必要に応じて行レベル セキュリティ (RLS) を実装
- [ ] 適切な役割と権限の管理
- [ ] PostgreSQL の組み込み暗号化関数の使用
- [ ] PostgreSQL 機能を使用した監査証跡の実装

## 📝 PostgreSQL 固有のレビュー ガイドライン

1. **データ型の最適化**: PostgreSQL 固有の型が適切に使用されていることを確認します。
2. **インデックス戦略**: インデックスの種類を確認し、PostgreSQL 固有のインデックスが確実に使用されるようにする
3. **JSONB 構造**: JSONB スキーマ設計とクエリ パターンを検証する
4. **関数の品質**: PL/pgSQL 関数の効率性とベスト プラクティスを確認します。
5. **拡張機能の使用方法**: PostgreSQL 拡張機能の適切な使用を確認します。
6. **パフォーマンス機能**: PostgreSQL の高度な機能の使用状況を確認する
7. **セキュリティの実装**: PostgreSQL 固有のセキュリティ機能を確認する

PostgreSQL の独自の機能に焦点を当て、コードが PostgreSQL を汎用 SQL データベースとして扱うのではなく、PostgreSQL の特別な点を活用していることを確認します。