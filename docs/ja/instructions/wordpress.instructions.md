---
applyTo: 'wp-content/plugins/**,wp-content/themes/**,**/*.php,**/*.inc,**/*.js,**/*.jsx,**/*.ts,**/*.tsx,**/*.css,**/*.scss,**/*.json'
description: 'WordPress plugin と theme のためのコーディング、セキュリティ、テストのルール'
---

# WordPress 開発 — Copilot 指示

**Goal:** 安全で、高性能で、テスト可能かつ公式 WordPress practice に準拠した WordPress code を生成すること。hook、小さな function、dependency injection (妥当な場合)、明確な責務分離を優先する。

## 1) 中核原則
- WordPress core を変更してはならない。**action** と **filter** で拡張する。
- plugin では、必ず header を含め、entry PHP file では direct execution を防ぐ。
- global collision を避けるため、固有の prefix または PHP namespace を使う。
- asset は enqueue し、PHP template 内に生の `<script>` / `<style>` を直接書かない。
- user-facing string は翻訳可能にし、正しい text domain を読み込む。

### 最小限の plugin header と guard
```php
<?php
defined('ABSPATH') || exit;
/**
 * Plugin Name: Awesome Feature
 * Description: Example plugin scaffold.
 * Version: 0.1.0
 * Author: Example
 * License: GPL-2.0-or-later
 * Text Domain: awesome-feature
 * Domain Path: /languages
 */
```

## 2) コーディング標準 (PHP、JS、CSS、HTML)
- **WordPress Coding Standards (WPCS)** に従い、公開 API には DocBlock を書く。
- PHP: 適切な場面では厳密比較 (`===`、`!==`) を優先する。配列構文と spacing は WPCS に従って一貫させる。
- JS: WordPress JS style に合わせ、block / editor code では `@wordpress/*` package を優先する。
- CSS: 有益な場合は BEM ライクな class 名を使い、過度に限定的な selector は避ける。
- project でより高い要件が指定されていない限り、PHP 7.4+ 互換の pattern を使う。対象の WP / PHP version で使えない feature は避ける。

### Linting 設定例
```xml
<!-- phpcs.xml -->
<?xml version="1.0"?>
<ruleset name="Project WPCS">
  <description>この project 向け WordPress Coding Standards。</description>
  <file>./</file>
  <exclude-pattern>vendor/*</exclude-pattern>
  <exclude-pattern>node_modules/*</exclude-pattern>
  <rule ref="WordPress"/>
  <rule ref="WordPress-Docs"/>
  <rule ref="WordPress-Extra"/>
  <rule ref="PHPCompatibility"/>
  <config name="testVersion" value="7.4-"/>
</ruleset>
```

```json
// composer.json (snippet)
{
  "require-dev": {
    "dealerdirect/phpcodesniffer-composer-installer": "^1.0",
    "wp-coding-standards/wpcs": "^3.0",
    "phpcompatibility/php-compatibility": "^9.0"
  },
  "scripts": {
    "lint:php": "phpcs -p",
    "fix:php": "phpcbf -p"
  }
}
```

```json
// package.json (snippet)
{
  "devDependencies": {
    "@wordpress/eslint-plugin": "^x.y.z"
  },
  "scripts": {
    "lint:js": "eslint ."
  }
}
```

## 3) セキュリティと data 処理
- **出力時に escape、入力時に sanitize する。**
  - Escape: `esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses_post()`.
  - Sanitize: `sanitize_text_field()`, `sanitize_email()`, `sanitize_key()`, `absint()`, `intval()`.
- **form、AJAX、REST には capability と nonce を使う:**
  - `wp_nonce_field()` で nonce を追加し、`check_admin_referer()` / `wp_verify_nonce()` で検証する。
  - 変更操作は `current_user_can( 'manage_options' /* or specific cap */ )` で制限する。
- **database:** 常に placeholder 付きの `$wpdb->prepare()` を使い、信頼できない入力を連結しない。
- **upload:** MIME / type を検証し、`wp_handle_upload()` / `media_handle_upload()` を使う。

## 4) 国際化 (i18n)
- user-visible string は text domain を指定した翻訳 function で包む:
  - `__( 'Text', 'awesome-feature' )`, `_x()`, `esc_html__()`.
- 翻訳は `load_plugin_textdomain()` または `load_theme_textdomain()` で読み込む。
- `/languages` に `.pot` を置き、domain を一貫して使う。

## 5) パフォーマンス
- 重いロジックは特定 hook に遅延させる。必要がない限り `init` / `wp_loaded` で高コスト処理をしない。
- 高コスト query には transient または object cache を使い、無効化計画も用意する。
- asset は必要なものだけを conditionally enqueue する (front / admin、特定 screen / route など)。
- 無制限 loop より、pagination 済み / parameterized query を優先する。

## 6) Admin UI と設定
- option page には **Settings API** を使い、各 setting に `sanitize_callback` を用意する。
- table は `WP_List_Table` pattern に従う。notice には admin notices API を使う。
- 複雑な UI のために HTML を直接 echo しない。template または escaping を伴う小さな view helper を優先する。

## 7) REST API
- `register_rest_route()` で登録し、必ず `permission_callback` を設定する。
- request arg は `args` schema で validate / sanitize する。
- `WP_REST_Response` または JSON に自然に変換できる array / object を返す。

## 8) Block と Editor (Gutenberg)
- `block.json` + `register_block_type()` を使い、`@wordpress/*` package を活用する。
- 必要なら server render callback を提供する (dynamic block)。
- E2E test では次をカバーする: block 挿入 → 編集 → 保存 → front-end render。

## 9) Asset 読み込み
```php
add_action('wp_enqueue_scripts', function () {
  wp_enqueue_style(
    'af-frontend',
    plugins_url('assets/frontend.css', __FILE__),
    [],
    '0.1.0'
  );

  wp_enqueue_script(
    'af-frontend',
    plugins_url('assets/frontend.js', __FILE__),
    [ 'wp-i18n', 'wp-element' ],
    '0.1.0',
    true
  );
});
```
- 複数 component が同じ asset に依存する場合は、先に `wp_register_style/script` で登録する。
- admin screen では `admin_enqueue_scripts` に hook し、screen ID を確認する。

## 10) テスト
### PHP Unit / Integration
- **WordPress test suite** を `PHPUnit` と `WP_UnitTestCase` で使う。
- 次をテストする: sanitize、capability check、REST permission、DB query、hook。
- fixture 準備には factory (`self::factory()->post->create()` など) を優先する。

```xml
<!-- phpunit.xml.dist (minimal) -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap.php" colors="true">
  <testsuites>
    <testsuite name="Plugin Test Suite">
      <directory suffix="Test.php">tests/</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

```php
// tests/bootstrap.php (minimal sketch)
<?php
$_tests_dir = getenv('WP_TESTS_DIR') ?: '/tmp/wordpress-tests-lib';
require_once $_tests_dir . '/includes/functions.php';
tests_add_filter( 'muplugins_loaded', function () {
  require dirname(__DIR__) . '/awesome-feature.php';
} );
require $_tests_dir . '/includes/bootstrap.php';
```
### E2E
- editor / front-end flow には Playwright (または Puppeteer) を使う。
- 基本的な user journey と regression (block 挿入、setting 保存、front-end render) をカバーする。

## 11) ドキュメントと commit
- `README.md` は常に最新化する: install、usage、capability、hook / filter、test 手順を含める。
- 明確で命令形の commit message を使い、issue / ticket を参照し、影響を要約する。

## 12) Copilot が必ず満たすこと (Checklist)
- ✅ 固有 prefix / namespace を使い、意図しない global を作らない。
- ✅ 書き込み操作 (AJAX / REST / form) には nonce + capability check を入れる。
- ✅ input は sanitize し、output は escape する。
- ✅ user-visible string を正しい text domain で i18n 関数に包む。
- ✅ asset は API 経由で enqueue する (inline script / style を使わない)。
- ✅ 新しい振る舞いには test を追加 / 更新する。
- ✅ 該当する場合は code が PHPCS (WPCS) と ESLint を通過する。
- ✅ DB query を直接連結せず、常に prepare する。
