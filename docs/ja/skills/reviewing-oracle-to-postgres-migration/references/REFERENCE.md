# 参照インデックス

|ファイル |簡単な説明 |
| --- | --- |
| [空文字列処理.md](空文字列処理.md) | Oracle は '' を NULL として扱います。 PostgreSQL は空の文字列を区別し、コード、テスト、移行の動作を調整するパターンを維持します。 |
| [データが見つからない例外.md](データが見つからない例外.md) | Oracle SELECT INTO では「データが見つかりません」と表示されます。 PostgreSQL では、Oracle の動作を反映するために明示的な NOT FOUND 処理を追加しません。 |
| [句からの oracle-括弧.md](句からの oracle-括弧.md) | Oracle では `FROM(TABLE_NAME)` 構文が使用できます。 PostgreSQL には `FROM TABLE_NAME` が必要です。テーブル名を囲む不要な括弧を削除します。 |
| [oracle-to-postgres-sorting.md](oracle-to-postgres-sorting.md) | COLLATE "C" および DISTINCT ラッパー パターンを使用して、PostgreSQL で Oracle のような順序付けを維持する方法。 |
| [oracle-to-postgres-to-char-numeric.md](oracle-to-postgres-to-char-numeric.md) | Oracle では、フォーマットなしの TO_CHAR(数値) を許可します。 PostgreSQL にはフォーマット文字列が必要です。代わりに CAST(numeric AS TEXT) を使用してください。 |
| [oracle-to-postgres-type-coercion.md](oracle-to-postgres-type-coercion.md) | PostgreSQL の厳密な型チェックと Oracle の暗黙的な強制 - リテラルを引用符またはキャストすることで比較エラーを修正します。 |
| [postgres-concurrent-transactions.md](postgres-concurrent-transactions.md) | PostgreSQL では、接続ごとにアクティブなコマンドを 1 つだけ許可します。結果を具体化するか、同時操作エラーを避けるために別の接続を使用します。 |
| [postgres-refcursor-handling.md](postgres-refcursor-handling.md) |リカーサーの処理の違い。 PostgreSQL では、結果をアンラップして読み取るために、カーソル名によるフェッチ (C# パターン) が必要です。 |
| [oracle-to-postgres-timestamp-timezone.md](oracle-to-postgres-timestamp-timezone.md) | CURRENT_TIMESTAMP / NOW() は PostgreSQL で UTC 正規化されたタイムスタンプを返します。 Npgsql は DateTime.Kind=Unspecified を表示します。接続オープン時およびアプリケーション コード内で UTC を強制します。 |