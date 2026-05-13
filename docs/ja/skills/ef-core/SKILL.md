---
name: ef-core
description: 'Entity Framework Core のベスト プラクティスを提供します'
---

# Entity Framework Core Best Practices

あなたの目標は、Entity Framework Core を使う際に私がベスト プラクティスに従えるよう支援することです。

## Data Context Design

- DbContext クラスは焦点を絞り、凝集度を高く保つ
- 構成オプションにはコンストラクター インジェクションを使う
- fluent API の構成には OnModelCreating をオーバーライドする
- IEntityTypeConfiguration を使って entity 構成を分離する
- console app や test では DbContextFactory パターンの利用を検討する

## Entity Design

- 意味のある主キーを使う (自然キーと代理キーを検討する)
- 適切なリレーションシップを実装する (one-to-one, one-to-many, many-to-many)
- 制約や検証には data annotations または fluent API を使う
- 適切な navigational property を実装する
- value object には owned entity type の利用を検討する

## Performance

- 読み取り専用クエリには AsNoTracking() を使う
- 大きな結果セットには Skip() と Take() によるページングを実装する
- 必要なときは Include() を使って関連 entity を eager load する
- 必要なフィールドだけ取得する projection (Select) を検討する
- 頻繁に実行されるクエリには compiled query を使う
- 関連データを適切に含めて N+1 クエリ問題を避ける

## Migrations

- 小さく焦点を絞った migration を作成する
- migration には分かりやすい名前を付ける
- 本番適用前に migration SQL script を検証する
- デプロイには migration bundle の利用を検討する
- 適切な場合は migration を通じてデータ シードを追加する

## Querying

- IQueryable は慎重に使い、いつクエリが実行されるかを理解する
- 生 SQL より強く型付けされた LINQ クエリを優先する
- 適切な query operator (Where, OrderBy, GroupBy) を使う
- 複雑な操作には database function の利用を検討する
- 再利用可能なクエリには specifications パターンを実装する

## Change Tracking & Saving

- 適切な change tracking 戦略を使う
- SaveChanges() の呼び出しはバッチ化する
- 複数ユーザー環境では同時実行制御を実装する
- 複数操作には transaction の利用を検討する
- 適切な DbContext lifetime を使う (web app では scoped)

## Security

- parameterized query を使って SQL injection を避ける
- 適切なデータ アクセス権限を実装する
- 生 SQL クエリの使用には注意する
- 機密情報にはデータ暗号化を検討する
- migration を使ってデータベース ユーザー権限を管理する

## Testing

- unit test には in-memory database provider を使う
- integration test には SQLite を使った専用の testing context を作る
- 純粋な unit test では DbContext と DbSet をモックする
- isolated な環境で migration をテストする
- model 変更には snapshot testing を検討する

EF Core コードをレビューするときは、これらのベスト プラクティスに従って問題を指摘し、改善案を提案してください。
