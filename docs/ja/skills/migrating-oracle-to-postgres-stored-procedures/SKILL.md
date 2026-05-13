---
name: migrating-oracle-to-postgres-stored-procedures
description: 'OracleのPL/SQLストアドプロシージャをPostgreSQLのPL/pgSQLに移行します。Oracle固有の構文を翻訳し、メソッドシグネチャと型連動パラメータを保持し、適宜orafceを活用し、Oracle互換のテキストソートのためにCOLLATE "C"を適用します。データベース移行時にOracleのストアドプロシージャや関数をPostgreSQLの同等物に変換する際に使用してください。'
---

# OracleからPostgreSQLへのストアドプロシージャ移行

OracleのPL/SQLストアドプロシージャおよび関数をPostgreSQLのPL/pgSQLの同等物に翻訳します。

## ワークフロー

```
進捗:
- [ ] ステップ1: Oracleのソースプロシージャを読み込む
- [ ] ステップ2: PostgreSQLのPL/pgSQLに翻訳する
- [ ] ステップ3: 移行済みプロシージャをPostgresの出力ディレクトリに書き込む
```

**ステップ1: Oracleのソースプロシージャを読み込む**

`.github/oracle-to-postgres-migration/DDL/Oracle/Procedures and Functions/`からOracleのストアドプロシージャを読み込みます。型解決のために`.github/oracle-to-postgres-migration/DDL/Oracle/Tables and Views/`のOracleのテーブル・ビュー定義を参照してください。

**ステップ2: PostgreSQLのPL/pgSQLに翻訳する**

以下の翻訳ルールを適用します：

- Oracle固有の構文はすべてPostgreSQLの同等のものに翻訳する。
- 元の機能と制御フローのロジックを保持する。
- 型連動の入力パラメータ（例：`PARAM_NAME IN table_name.column_name%TYPE`）は保持する。
- 他のプロシージャに渡される出力パラメータには明示的な型（`NUMERIC`、`VARCHAR`、`INTEGER`など）を使用し、型連動はしない。
- メソッドシグネチャは変更しない。
- Oracleソースにスキーマ名が付いていない限り、オブジェクト名にスキーマ名を付けない。
- 例外処理やロールバックロジックは変更しない。
- `COMMENT`や`GRANT`文は生成しない。
- テキストフィールドでの並び替えにはOracle互換のソートのために`COLLATE "C"`を使用する。
- 明確さや忠実度が向上する場合は`orafce`拡張機能を活用する。

ターゲットスキーマの詳細は`.github/oracle-to-postgres-migration/DDL/Postgres/Tables and Views/`のPostgreSQLのテーブル・ビュー定義を参照してください。

**ステップ3: 移行済みプロシージャをPostgresの出力ディレクトリに書き込む**

移行済みの各プロシージャは`.github/oracle-to-postgres-migration/DDL/Postgres/Procedures and Functions/{PACKAGE_NAME_IF_APPLICABLE}/`のそれぞれのファイルに1つずつ配置してください。
