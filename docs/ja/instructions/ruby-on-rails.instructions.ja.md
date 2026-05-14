---
description: 'Ruby on Rails のコーディング規約とガイドライン'
applyTo: '**/*.rb'
---

# Ruby on Rails

## 一般ガイドライン

- 一貫した formatting のため、RuboCop Style Guide に従い、`rubocop`、`standardrb`、`rufo` のような tool を使います。
- 変数 / method には snake_case、class / module には CamelCase を使います。
- method は短く焦点を絞り、early return、guard clause、private method を使って複雑さを減らします。
- 短い名前や汎用的な名前より、意味のある名前を優先します。
- コメントは必要なときだけ書き、自明な内容の説明は避けます。
- class、method、module に単一責任の原則を適用します。
- 継承より composition を優先し、再利用可能なロジックは module や service に抽出します。
- controller は薄く保ち、business logic は model、service、command / query object に移します。
- 「fat model, skinny controller」パターンは、きれいな抽象化とともに慎重に適用します。
- 再利用性とテスト容易性のため、business logic は service object に抽出します。
- partial や view component を使って重複を減らし、view を単純化します。
- 否定条件には `unless` を使いますが、明確さのため `else` と組み合わせるのは避けます。
- 深い条件ネストは避け、guard clause と method 抽出を優先します。
- 複数の `nil` チェックの代わりに safe navigation (`&.`) を使います。
- 手動の nil / empty チェックより `.present?`、`.blank?`、`.any?` を優先します。
- routing と controller action では RESTful 規約に従います。
- resource を一貫して scaffold するため Rails generator を使います。
- attribute を安全に whitelist するため strong parameters を使います。
- model の明確さと validation のため、enum や typed attribute を優先します。
- migration は database 非依存に保ち、可能なら raw SQL を避けます。
- foreign key と頻繁に query される column には必ず index を追加します。
- `null: false` と `unique: true` は model だけでなく DB level で定義します。
- 大量データの反復処理では、メモリ使用量を抑えるため `find_each` を使います。
- query は model で scope 化するか、明確さと再利用性のため query object を使います。
- `before_action` callback の使いすぎを避け、そこに business logic を置きません。
- 高コストな計算や頻繁に使うデータの保存には `Rails.cache` を使います。
- file path はハードコードせず `Rails.root.join(...)` で組み立てます。
- association では、関係を明示するため `class_name` と `foreign_key` を使います。
- secret と config は `Rails.application.credentials` や ENV variable を使って codebase から分離します。
- model、service、helper には独立した unit test を書きます。
- request / system test で end-to-end ロジックをカバーします。
- email 送信や API 呼び出しのような非ブロッキング処理には background job (ActiveJob) を使います。
- test data の準備には `FactoryBot` (RSpec) または fixture (Minitest) を使って整理します。
- `puts` は避け、`byebug`、`pry`、logger utility で debug します。
- 複雑な code path や method は YARD または RDoc で文書化します。

## App Directory Structure

- business logic をカプセル化する service object は `app/services` directory に定義します。
- validation と submit ロジックを扱う form object は `app/forms` に配置します。
- API response を整形する JSON serializer は `app/serializers` directory に実装します。
- resource へのユーザー access を制御する authorization policy は `app/policies` に定義します。
- GraphQL API は `app/graphql` 内で schema、query、mutation を整理して構成します。
- 特殊な validation ロジックを強制する custom validator は `app/validators` に作成します。
- 複雑な ActiveRecord query は、再利用性とテスト容易性のため `app/queries` に隔離してカプセル化します。
- ActiveModel type の振る舞いを拡張または上書きする custom data type と coercion ロジックは `app/types` directory に定義します。

## Commands

- 新しい model、controller、migration の作成には `rails generate` を使います。
- database migration の適用には `rails db:migrate` を使います。
- 初期データ投入には `rails db:seed` を使います。
- 直前の migration を戻すには `rails db:rollback` を使います。
- REPL 環境で Rails application と対話するには `rails console` を使います。
- 開発 server の起動には `rails server` を使います。
- test suite の実行には `rails test` を使います。
- application に定義された route を一覧表示するには `rails routes` を使います。
- 本番用 asset の compile には `rails assets:precompile` を使います。


## API 開発のベスト プラクティス

- RESTful 規約に従うため、route は Rails の `resources` で構成します。
- version 管理と将来互換性のため、namespaced route (例: `/api/v1/`) を使います。
- 出力を一貫させるため、response は `ActiveModel::Serializer` または `fast_jsonapi` で serialize します。
- 各 response には適切な HTTP status code を返します (例: 200 OK、201 Created、422 Unprocessable Entity)。
- `before_action` filter は resource の読み込みと認可に使い、business logic には使いません。
- 大量データを返す endpoint には pagination (`kaminari` や `pagy` など) を活用します。
- middleware や `rack-attack` のような gem を使って、重要な endpoint に rate limiting / throttling を適用します。
- error は error code、message、detail を含む構造化 JSON 形式で返します。
- strong parameters を使って入力 parameter を sanitize し whitelist します。
- internal logic と response formatting を切り離すため、custom serializer や presenter を使います。
- 関連データの eager loading には `includes` を使い、N+1 query を避けます。
- email 送信や外部 API 連携のような非ブロッキング処理には background job を実装します。
- debug、observability、audit のため request / response metadata を log します。
- endpoint は OpenAPI (Swagger)、`rswag`、`apipie-rails` を使って文書化します。
- 必要に応じて API への cross-origin access を許可するため、CORS header (`rack-cors`) を使います。
- API response や error message に機密データを決して露出しないようにします。

## Frontend 開発のベスト プラクティス

- Rails 6+ で Webpacker や esbuild を使う場合、JavaScript pack、module、frontend logic の主 directory には `app/javascript` を使います。
- JavaScript は file type 別ではなく component や domain 別に構成し、modular に保ちます。
- Rails ネイティブ app では、real-time update と最小限の JavaScript のために Hotwire (Turbo + Stimulus) を活用します。
- HTML への振る舞い付与と UI logic の宣言的な管理には Stimulus controller を使います。
- style は `app/assets/stylesheets` 配下で SCSS module、Tailwind、BEM 規約を使って整理します。
- 繰り返しの markup は partial や component に抽出し、view logic をきれいに保ちます。
- すべての view で semantic HTML tag を使い、accessibility (a11y) best practice に従います。
- inline JavaScript や style は避け、代わりにロジックを別の `.js` または `.scss` file に移して明確さと再利用性を確保します。
- image、font、icon などの asset は asset pipeline や bundler で最適化し、cache と compression を有効にします。
- `data-*` attribute を使って frontend の interactivity と Rails が生成する HTML を Stimulus と橋渡しします。
- frontend 機能は system test (Capybara) または Cypress / Playwright などの integration test で検証します。
- 本番で不要な script や style を防ぐため environment ごとの asset loading を使います。
- UI の一貫性と拡張性のため、design system や component library に従います。
- lazy loading、Turbo Frames、JS の遅延読み込みを使って TTFP と asset loading を最適化します。

## テスト ガイドライン

- business logic を検証するため、model の unit test は `test/models` (Minitest) または `spec/models` (RSpec) に書きます。
- test data は fixture (Minitest) または `FactoryBot` (RSpec) で整理し、一貫して管理します。
- RESTful API の振る舞いをテストする controller spec は `test/controllers` または `spec/requests` に配置します。
- 共通の test data 初期化には RSpec の `before` block や Minitest の `setup` を優先します。
- test 環境を分離するため、外部 API は test で直接叩かず、`WebMock`、`VCR`、`stub_request` を使います。
- 完全なユーザー フローの再現には、Minitest の `system tests` または RSpec + Capybara の `feature specs` を使います。
- 遅く高コストな test (外部 service、file upload など) は別の test type や tag に分離します。
- 十分な code coverage を確認するため `SimpleCov` のような tool を実行します。
- test で `sleep` は使わず、Minitest では `perform_enqueued_jobs`、RSpec では `ActiveJob::TestHelper` を使います。
- test 間の state をクリーンに保つため、`rails test:prepare`、`DatabaseCleaner`、`transactional_fixtures` のような database cleaning tool を使います。
- background job は `ActiveJob::TestHelper` や `have_enqueued_job` matcher を使って enqueue / 実行を検証します。
- GitHub Actions や CircleCI などの CI tool を使い、test が環境をまたいで一貫して動くことを確認します。
- 再利用しやすく表現力のある test logic のため、custom matcher (RSpec) や custom assertion (Minitest) を使います。
- より高速で狙いを絞った test 実行のため、test を `:model`、`:request`、`:feature` のような type で tag 付けします。
- brittle な test は避け、明示的に必要な場合を除き、特定の timestamp、ランダム データ、順序に依存しません。
- model、view、controller にまたがる end-to-end flow には integration test を書きます。
- test は高速で信頼性が高く、本番 code と同じくらい DRY に保ちます。
