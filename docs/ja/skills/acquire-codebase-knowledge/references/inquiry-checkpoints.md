# 調査チェックポイント

acquire-codebase-knowledge ワークフローのフェーズ 2 における、テンプレートごとの調査質問です。各テンプレート領域について、まずはスキャン出力から回答を探し、その後不足分を埋めるためにソースファイルを確認してください。

---

## 1. STACK.md — 技術スタック

- 主要言語とその正確なバージョンは何か？（`.nvmrc`、`go.mod`、`pyproject.toml`、Docker の `FROM` 行を確認）
- どのパッケージマネージャーが使われているか？（`npm`、`yarn`、`pnpm`、`go mod`、`pip`、`uv`）
- 主要なランタイムフレームワークは何か？（Web サーバー、ORM、DI コンテナ）
- `dependencies`（本番）と `devDependencies`（開発ツール）には何が含まれているか？
- Docker イメージはあるか？ある場合、どのベースイメージを使っているか？
- `package.json` / `Makefile` / `pyproject.toml` の主要スクリプトは何か？

## 2. STRUCTURE.md — ディレクトリ構成

- ソースコードはどこにあるか？（通常は `src/`、`lib/`、または Go の場合はプロジェクトルート）
- エントリーポイントはどこか？（`package.json` の `main`、`scripts.start`、`cmd/main.go`、`app.py` を確認）
- 各トップレベルディレクトリの目的は何と説明されているか？
- 一見して分かりにくいディレクトリはあるか？（例: `eng/`、`platform/`、`infra/`）
- 隠し設定ディレクトリはあるか？（`.github/`、`.vscode/`、`.husky/`）
- ディレクトリ名の命名規則はどうなっているか？（camelCase、kebab-case、ドメインベース vs レイヤーベース）

## 3. ARCHITECTURE.md — パターン

- コードはレイヤー単位（controllers → services → repos）で構成されているか、それとも機能単位か？
- 主なデータフローは何か？1 つのリクエストまたはコマンドを、入口からデータストアまで追跡する。
- シングルトン、依存性注入パターン、または明示的な初期化順序の要件はあるか？
- バックグラウンドワーカー、キュー、またはイベント駆動コンポーネントはあるか？
- 繰り返し現れるデザインパターンは何か？（Factory、Repository、Decorator、Strategy）

## 4. CONVENTIONS.md — コーディング規約

- ファイル命名規則は何か？（10 ファイル以上を確認 — camelCase、kebab-case、PascalCase）
- 関数名と変数名の命名規則は何か？
- private メソッド/フィールドに接頭辞は付いているか？（例: `_methodName`、`#field`）
- どの linter と formatter が設定されているか？（`.eslintrc`、`.prettierrc`、`golangci.yml` を確認）
- TypeScript の厳格性設定はどうなっているか？（`strict`、`noImplicitAny` など）
- 各レイヤーでエラーはどう処理されているか？（throw するか、構造化エラーを返すか）
- どのロギングライブラリが使われており、ログメッセージの形式はどうなっているか？
- import はどのように整理されているか？（barrel exports、path aliases、グルーピング規則）

## 5. INTEGRATIONS.md — 外部サービス

- どの外部 API を呼び出しているか？（`axios.`、`fetch(`、`http.Get(`、定数内のベース URL を検索）
- 認証情報はどのように保存・参照されているか？（`.env`、secrets manager、環境変数）
- どのデータベースに接続しているか？（`pg`、`mongoose`、`prisma`、`typeorm`、`sqlalchemy` がマニフェストにあるか確認）
- アプリと外部サービスの間に API ゲートウェイ、サービスメッシュ、またはプロキシはあるか？
- どの監視/可観測性ツールが使われているか？（APM、Prometheus、ロギングパイプライン）
- メッセージキューやイベントバスはあるか？（Kafka、RabbitMQ、SQS、Pub/Sub）

## 6. TESTING.md — テスト設定

- どのテストランナーが設定されているか？（`package.json` の `scripts.test`、`pytest.ini`、`go test` を確認）
- テストファイルはどこにあるか？（ソースと同階層、`tests/`、`__tests__/`）
- どのアサーションライブラリが使われているか？（Jest expect、Chai、pytest assert）
- 外部依存はどのようにモックしているか？（jest.mock、依存性注入、fixtures）
- 実サービスにアクセスする統合テストはあるか？それともモックを使う単体テストか？
- カバレッジしきい値は強制されているか？（`jest.config.js`、`.nycrc`、`pyproject.toml` を確認）

## 7. CONCERNS.md — 既知の課題

- 本番コード内に TODO/FIXME/HACK はいくつあるか？（スキャン出力を参照）
- 過去 90 日で git の変更頻度（churn）が最も高いファイルはどれか？（スキャン出力を参照）
- 500 行を超え、複数の責務が混在しているファイルはあるか？
- 並列化できるのに逐次呼び出しをしているサービスはあるか？
- ハードコードされた値（URL、ID、マジックナンバー）で設定化すべきものはあるか？
- どのようなセキュリティリスクがあるか？（入力検証の欠如、生のエラーメッセージのクライアント露出、認可チェック不足）
- スケールしないパフォーマンスパターンはあるか？（N+1 クエリ、マルチインスタンス構成でのインメモリキャッシュ）

