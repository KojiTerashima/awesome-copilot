---
applyTo: '**/*.php, **/*.js, **/*.mustache, **/*.xml, **/*.css, **/*.scss'
description: 'Moodle プロジェクトの文脈で GitHub Copilot がコード生成を行うための指示。'
---

# プロジェクトコンテキスト

このリポジトリには Moodle プロジェクトが含まれています。生成するコードは、このプロジェクトで使われている特定の Moodle バージョン（たとえば Moodle 3.11、4.1 LTS、またはそれ以降）と互換性があることを確認してください。

含まれるもの:
- Plugin development (local, block, mod, auth, enrol, tool など)
- Theme customization
- CLI scripts
- Moodle API を使用した外部サービスとの連携

# コード標準

- 公式 Moodle Coding guidelines に従ってください: https://moodledev.io/general/development/policies/codingstyle
- PHP はコアバージョンと互換性がある必要があります（例: PHP 7.4 / 8.0 / 8.1）。
- 互換性を壊す場合は、コアでサポートされていないモダン構文を使用しないでください。
- クラス名には Moodle namespaces を使用する必要があります。
- Moodle 標準の plugin directory layout（例: classes/output、classes/form、db/、lang/、templates/…）に従ってください。
- Moodle のセキュリティ関数を必ず使ってください:
  - SQL placeholder 付きの `$DB`
  - `require_login()`, `require_capability()`
  - `required_param()` / `optional_param()` によるパラメーター処理

# コード生成ルール

- Plugin 内で新しい PHP class を作成する場合は、plugin の component 名に一致する Moodle component (Frankenstyle) namespace を使ってください。例: `local_myplugin`, `mod_forum`, `block_mycatalog`, `tool_mytool`。
- Plugin では常に次の構造を尊重してください:
  - /db
  - /lang
  - /classes
  - /templates
  - /version.php
  - /settings.php
  - /lib.php (必要な場合のみ)

- HTML には renderer と Mustache template を使ってください。PHP の中に HTML を混在させないでください。
- JavaScript コードでは inline script ではなく AMD module を使ってください。
- 可能な限り手書きコードよりも Moodle API 関数を優先してください。
- 存在しない Moodle 関数を作り出してはいけません。

# Copilot が答えられるべき内容の例

- "version.php、settings.php、lib.php を含む基本的な local plugin を生成して。"
- "db/install.xml に新しいテーブルを作り、db/upgrade.php に upgrade script を生成して。"
- "moodleform を使った Moodle form を生成して。"
- "テーブルを表示するための renderer と Mustache を作って。"

# 期待されるスタイル

- Moodle コンテキストに即した、明確で具体的な回答。
- 常に完全なパス付きでファイルを含めること。
- 同じことに複数の方法がある場合は、Moodle が推奨するアプローチを使うこと。
