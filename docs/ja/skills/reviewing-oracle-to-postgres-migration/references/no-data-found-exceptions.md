# PostgreSQL 例外処理: SELECT INTO データが見つかりません

## 概要

Oracle から PostgreSQL に移行する際の一般的な問題には、行が見つからない場合に例外が発生することを期待する `SELECT INTO` ステートメントが関係します。このパターンの違いにより、適切に処理されないと統合テストが失敗し、アプリケーション ロジックが誤動作する可能性があります。

---

## 問題の説明

### シナリオ

ストアド プロシージャは、`SELECT INTO` を使用して検索操作を実行し、必要な値を取得します。```sql
SELECT column_name
INTO variable_name
FROM table1, table2 
WHERE table1.id = table2.id AND table1.id = parameter_value;
```### オラクルの動作

Oracle の `SELECT INTO` ステートメントで **行が見つからない**場合、自動的に次のエラーが発生します。```
ORA-01403: no data found
```この例外はプロシージャの例外ハンドラによってキャッチされ、呼び出し側アプリケーションに再送出されます。

### PostgreSQL の動作 (プレフィックス)

PostgreSQL の `SELECT INTO` ステートメントで **行が見つからない**場合、次のようになります。

- `FOUND` 変数を `false` に設定します
- **例外を発生させずに実行をサイレントに続行します**

この根本的な違いにより、テストが通知なしで失敗し、実稼働コードでロジック エラーが発生する可能性があります。

---

## 根本原因の分析

PostgreSQL バージョンには、`SELECT INTO` ステートメントの後の `NOT FOUND` 条件に対する明示的なエラー処理がありませんでした。

**元のコード (問題あり):**```plpgsql
SELECT column_name
INTO variable_name
FROM table1, table2 
WHERE table1.id = table2.id AND table1.id = parameter_value;

IF variable_name = 'X' THEN
 result_variable := 1;
ELSE
 result_variable := 2;
END IF;
```**問題:** `NOT FOUND` 条件のチェックがありません。無効なパラメータが渡されると、SELECT は行を返さず、`FOUND` は `false` になり、初期化されていない変数で実行が続行されます。

---

## 主な違い: Oracle と PostgreSQL

Oracle の動作に合わせて明示的な `NOT FOUND` エラー処理を追加します。

**修正コード:**```plpgsql
SELECT column_name
INTO variable_name
FROM table1, table2 
WHERE table1.id = table2.id AND table1.id = parameter_value;

-- Explicitly raise exception if no data found (matching Oracle behavior)
IF NOT FOUND THEN
    RAISE EXCEPTION 'no data found';
END IF;

IF variable_name = 'X' THEN
 result_variable := 1;
ELSE
 result_variable := 2;
END IF;
```---

## 同様の問題に関する移行メモ

この問題を解決するときは、次のことを確認してください。

1. **成功パス テスト** - 有効なパラメータが引き続き正しく機能することを確認します。
2. **例外テスト** - 無効なパラメーターで例外が発生することを確認します。
3. **トランザクションのロールバック** - エラーが発生した場合に適切にクリーンアップすることを保証します。
4. **データの整合性** - 成功した場合、すべてのフィールドが正しく入力されていることを確認します