# Oracle から PostgreSQL への型強制の問題

## 内容

- 概要
- 問題 — 症状、根本原因、例
- 解決策 — 文字列リテラル、明示的なキャスト
- 影響を受ける一般的な比較演算子
- 検出戦略
- 現実世界の例
- 予防のベストプラクティス

## 概要

このドキュメントでは、SQL コードを Oracle から PostgreSQL に移植するときに発生する一般的な移行の問題について説明します。この問題は、これらのデータベースが比較演算子で暗黙的な型変換を処理する方法の根本的な違いに起因します。

## 問題

### 症状

SQL クエリを Oracle から PostgreSQL に移行するときに、次のエラーが発生する場合があります。```
Npgsql.PostgresException: 42883: operator does not exist: character varying <> integer
POSITION: [line_number]
```### 根本原因

PostgreSQL には **厳密な型強制**があり、比較演算子で暗黙的な型強制は実行されません。対照的に、Oracle は比較操作中にオペランドを互換性のある型に自動的に変換します。

#### 不一致の例

**Oracle SQL (正常に動作):**```sql
AND physical_address.pcountry_cd <> 124
```- `pcountry_cd` は `VARCHAR2` です
- `124` は整数リテラルです
- Oracle は比較のために `124` を文字列にサイレントに変換します

**PostgreSQL (失敗):**```sql
AND physical_address.pcountry_cd <> 124
```

```
42883: operator does not exist: character varying <> integer
```- `pcountry_cd` は `character varying` です
- `124` は整数リテラルです
- 型が一致しないため、PostgreSQL は比較を拒否します。

## 解決策

### アプローチ 1: 文字列リテラルを使用する (推奨)

整数リテラルを文字列リテラルに変換します。```sql
AND physical_address.pcountry_cd <> '124'
```**長所:**

- 意味的に正しい (国コードは通常、文字列として保存されます)
- 最も効率的
- 最も明確な意図

**短所:**

- なし

### アプローチ 2: 明示的な型キャスト

整数を文字列型に明示的にキャストします。```sql
AND physical_address.pcountry_cd <> CAST(124 AS VARCHAR)
```**長所:**

- 変換を明示的かつ可視化します。
- 値がパラメータまたは複雑な式の場合に便利です

**短所:**

- 効率がわずかに低下する
- より冗長

## 影響を受ける一般的な比較演算子

すべての比較演算子がこの問題を引き起こす可能性があります。

- `<>` (等しくない)
- `=` (等しい)
- `<` (未満)
- `>` (より大きい)
- `<=` (以下)
- `>=` (以上)

## 検出戦略

Oracle から PostgreSQL に移行する場合:

1. **WHERE 句内の数値リテラルを検索**、string/varchar 列と比較します
2. **次のようなパターンを探します。**
   - `column_name <> 123` (列は VARCHAR/CHAR)
   - `column_name = 456` (列は VARCHAR/CHAR)
   - `column_name IN (1, 2, 3)` (列は VARCHAR/CHAR)

3. **コードレビューのチェックリスト:**
   - すべての比較値が正しく入力されていますか?
   - 文字列列は常に文字列リテラルを使用しますか?
   - 数値列は常に数値と比較されますか?

## 実際の例

**元の Oracle クエリ:**```sql
SELECT ac040.stakeholder_id,
       ac006.organization_etxt
  FROM ac040_stakeholder ac040
  INNER JOIN ac006_organization ac006 ON ac040.stakeholder_id = ac006.organization_id
 WHERE physical_address.pcountry_cd <> 124
   AND LOWER(ac006.organization_etxt) LIKE '%' || @orgtxt || '%'
 ORDER BY UPPER(ac006.organization_etxt)
```**修正された PostgreSQL クエリ:**```sql
SELECT ac040.stakeholder_id,
       ac006.organization_etxt
  FROM ac040_stakeholder ac040
  INNER JOIN ac006_organization ac006 ON ac040.stakeholder_id = ac006.organization_id
 WHERE physical_address.pcountry_cd <> '124'
   AND LOWER(ac006.organization_etxt) LIKE '%' || @orgtxt || '%'
 ORDER BY UPPER(ac006.organization_etxt)
```**変更:** `124` → `'124'`

## 予防のベストプラクティス

1. **型一貫性のあるリテラルを使用する:**
   - 文字列列の場合: 常に文字列リテラル (`'value'`) を使用します。
   - 数値列の場合: 常に数値リテラル (`123`) を使用します。
   - 日付の場合: 常に日付リテラル (`DATE '2024-01-01'`) を使用します。

2. **データベース ツールを活用する:**
   - IDE の SQL リンターを使用して型の不一致を検出する
   - コードレビュー中にPostgreSQL構文検証を実行します。

3. **早めにテストしてください:**
   - 導入前に PostgreSQL に対して移行クエリを実行します。
   - すべての比較演算子を実行する統合テストを含めます。

4. **ドキュメント:**
   - コメント内の型強制を文書化します。
   - 移行されたコードにリビジョン履歴をマークします

## 参考文献

- [PostgreSQL の型キャストに関するドキュメント](https://www.postgresql.org/docs/current/sql-syntax.html)
- [Oracle 型変換ドキュメント](https://docs.oracle.com/database/121/SQLRF/sql_elements003.htm)
- [Npgsql 例外: 演算子が存在しません](https://www.npgsql.org/doc/api/NpgsqlException.html)

## 関連する問題

この問題は、Oracle → PostgreSQL へのより広範な移行課題の一部です。

- 暗黙的な関数変換 (例: `TO_CHAR`、`TO_DATE`)
- 文字列連結演算子の違い (`||` は両方で動作しますが、動作が異なります)
- 数値の精度と丸めの違い
- 比較時のNULL処理