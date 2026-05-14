# Oracle から PostgreSQL へのソート移行ガイド

目的: クエリを PostgreSQL に移動するときに、Oracle のような並べ替えセマンティクスを維持します。

## 重要なポイント
- Oracle は多くの場合、プレーン `ORDER BY` をバイナリ/バイト単位で扱い、ASCII では大文字と小文字を区別しない順序付けを行います。
- PostgreSQL のデフォルトは異なります。 Oracle の動作と一致させるには、並べ替え式で `COLLATE "C"` を使用します。

## 1) 標準 `SELECT … ORDER BY`
**目標:** Oracle スタイルの順序を維持します。

**パターン：**```sql
SELECT col1
FROM your_table
ORDER BY col1 COLLATE "C";
```**注:**
- Oracle を模倣する必要がある各ソート式に `COLLATE "C"` を適用します。
- 昇順/降順および複数列のソートで動作します。 @@コード1@@。

## 2) `SELECT DISTINCT … ORDER BY`
**問題:** PostgreSQL では、`ORDER BY` 式が `DISTINCT` の `SELECT` リストに表示されるように強制し、次のような問題が発生します。
@@コード6@@

**Oracle の違い:** Oracle では、`DISTINCT` を使用する場合、投影されない式による順序付けが可能でした。

**推奨パターン (ラップとソート):**```sql
SELECT *
FROM (
  SELECT DISTINCT col1, col2
  FROM your_table
) AS distinct_results
ORDER BY col2 COLLATE "C";
```**理由:**
- 内部クエリは `DISTINCT` プロジェクションを実行します。
- 外側のクエリは結果セットを安全に順序付けし、Oracle の並べ替えに合わせて `COLLATE "C"` を追加します。

**ヒント:**
- 外側の `ORDER BY` で使用されている列がすべて内側の射影に含まれていることを確認します。
- 複数列のソートの場合は、関連する各式を照合します: `ORDER BY col2 COLLATE "C", col3 COLLATE "C" DESC`。

## 検証チェックリスト
- [ ] Oracle の並べ替えルールに従う必要があるすべての `ORDER BY` に `COLLATE "C"` を追加しました。
- [ ] `DISTINCT` クエリの場合、射影をラップし、外側のクエリでソートします。
- [ ] 内部投影に順序付けされた列が存在することを確認しました。
- [ ] テストまたは代表的なクエリを再実行して、順序が Oracle 出力と一致することを確認します。