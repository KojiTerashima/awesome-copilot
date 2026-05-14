---
description: "Symfony 公式 Best Practices に準拠した Symfony 開発標準"
applyTo: "**/*.php, **/*.yaml, **/*.yml, **/*.xml, **/*.twig"
---

# Symfony 開発 instruction

Symfony 公式 Best Practices とフレームワークの中核思想に従って Symfony アプリケーションを開発するための instruction です。

## プロジェクト コンテキスト
- Symfony (最新 stable または LTS)
- 既定の Symfony directory structure
- autowiring と autoconfiguration を有効化
- 永続化が必要な場合は Doctrine ORM
- templating には Twig
- 必要に応じて Symfony Forms、Validator、Security、Messenger
- テストには PHPUnit
- サポートされている場合は attribute-based configuration

## プロジェクト構造
- 既定の Symfony directory structure を使う
- アプリケーション code 用の bundle を作成しない
- PHP namespace を使ってアプリケーション code を整理する
- 設定は `config/`、アプリケーション code は `src/`、template は `templates/` に置く

## 設定

### 環境設定
- インフラ関連の設定には environment variable を使う
- 環境固有の値の定義には `.env` file を使う
- アプリケーションの振る舞いを制御するために environment variable を使わない

### 機密設定
- secret (API key、credential) は Symfony Secrets を使って保存する
- secret を repository に commit しない

### アプリケーション設定
- アプリケーションの振る舞い設定には `config/services.yaml` 内の parameter を使う
- environment ごとの override は必要な場合にだけ行う
- 衝突回避のため parameter には `app.` 接頭辞を付ける
- parameter 名は短く説明的にする
- めったに変わらない設定値には PHP constant を使う

## Services と Dependency Injection
- dependency injection のみを使う
- constructor injection を優先する
- 既定では autowiring と autoconfiguration を使う
- 可能な限り service は private に保つ
- `$container->get()` 経由で service にアクセスしない
- service 設定の推奨形式には YAML を使う
- 疎結合化や明確さの向上に役立つなら interface を使う

## Controllers
- `AbstractController` を継承する
- controller は薄く保ち、glue code に集中させる
- business logic を controller に置かない
- routing、caching、security の設定には attribute を使う
- service には dependency injection を使う
- 便利かつ適切な場面では Entity Value Resolver を使う
- 複雑な query は必要に応じて repository 経由で明示的に実行する

## Doctrine と永続化
- Doctrine entity は素の PHP object として扱う
- Doctrine mapping は PHP attribute で定義する
- データ取得には repository を使う
- business logic を repository に置かない
- schema 変更にはすべて migration を使う

## Templates (Twig)
- template 名、directory 名、変数名には snake_case を使う
- template fragment には underscore を接頭辞として付ける
- template は presentation に集中させる
- business logic を Twig template に置かない
- 既定で output は escape する
- content が信頼でき、かつ sanitize 済みでない限り `|raw` を使わない

## Forms
- form は PHP class として定義する
- form を controller で直接構築しない
- form button は form class ではなく template に追加する
- validation constraint は基になる object に定義する
- 各 form は 1 つの controller action で render と process を行う
- 複数 submit が必要な場合にだけ controller で submit button を定義する

## Validation
- Symfony Validator constraint を使う
- データはアプリケーション境界で検証する
- 再利用が必要なら form 専用の validation より object-level validation を優先する

## Internationalization
- 翻訳 file には XLIFF を使う
- 文字列リテラルではなく translation key を使う
- 位置ではなく目的を表す説明的な key を使う

## Security
- 複数システムが必要な場合を除き、single firewall を優先する
- auto password hasher を使う
- 複雑な認可ロジックには voter を使う
- attribute 内で複雑な security expression を避ける

## Web Assets
- web asset 管理には AssetMapper を使う
- 必要でない限り、不要な frontend build の複雑さを持ち込まない

## 非同期処理
- async / background task には Symfony Messenger を使う
- message handler は小さく、焦点を絞る
- 失敗した message 用に failure transport を設定する

## Testing
- functional test は `WebTestCase` を使って書く
- すべての public URL が正常に応答することを確認する smoke test を追加する
- functional test では route を生成せず URL をハードコードする
- 分離されたロジックには適切に unit test を使う
- アプリケーションの進化に合わせて、より具体的な test を段階的に追加する

## 一般ガイドライン
- 抽象化より明確さを優先する
- custom pattern を導入する前に Symfony の規約に従う
- 設定は明示的で読みやすく保つ
- 早すぎる最適化を避ける
- 参照実装として Symfony Demo を使う
