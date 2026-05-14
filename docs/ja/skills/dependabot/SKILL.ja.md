---
name: dependabot
description: >-
  GitHub Dependabot の設定と管理に関する包括的ガイドです。dependabot.yml ファイルの作成や最適化、Dependabot pull request の管理、依存関係更新戦略の設定、グループ化更新の設定、monorepo パターン、multi-ecosystem group、security update の設定、auto-triage rule、または Dependabot に関連する GitHub Advanced Security (GHAS) のサプライチェーン セキュリティ トピックについてユーザーが尋ねるときにこのスキルを使用します。
---

# Dependabot Configuration & Management

## Overview

Dependabot は GitHub に組み込まれた依存関係管理ツールで、3 つの中核機能があります。

1. **Dependabot Alerts** — 依存関係に既知の脆弱性 (CVE) があると通知する
2. **Dependabot Security Updates** — 脆弱な依存関係を修正する PR を自動作成する
3. **Dependabot Version Updates** — 依存関係を最新に保つための PR を自動作成する

すべての設定はデフォルト ブランチ上の **単一ファイル** `.github/dependabot.yml` に置かれます。GitHub は 1 つのリポジトリにつき複数の `dependabot.yml` ファイルを **サポートしません**。

## Configuration Workflow

`dependabot.yml` を作成または最適化するときは、次のプロセスに従ってください。

### Step 1: Detect All Ecosystems

依存関係 manifest を探すためにリポジトリをスキャンします。次を確認してください。

| Ecosystem | YAML Value | Manifest Files |
|---|---|---|
| npm/pnpm/yarn | `npm` | `package.json`, `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock` |
| pip/pipenv/poetry/uv | `pip` | `requirements.txt`, `Pipfile`, `pyproject.toml`, `setup.py` |
| Docker | `docker` | `Dockerfile` |
| Docker Compose | `docker-compose` | `docker-compose.yml` |
| GitHub Actions | `github-actions` | `.github/workflows/*.yml` |
| Go modules | `gomod` | `go.mod` |
| Bundler (Ruby) | `bundler` | `Gemfile` |
| Cargo (Rust) | `cargo` | `Cargo.toml` |
| Composer (PHP) | `composer` | `composer.json` |
| NuGet (.NET) | `nuget` | `*.csproj`, `packages.config` |
| .NET SDK | `dotnet-sdk` | `global.json` |
| Maven (Java) | `maven` | `pom.xml` |
| Gradle (Java) | `gradle` | `build.gradle` |
| Terraform | `terraform` | `*.tf` |
| OpenTofu | `opentofu` | `*.tf` |
| Helm | `helm` | `Chart.yaml` |
| Hex (Elixir) | `mix` | `mix.exs` |
| Swift | `swift` | `Package.swift` |
| Pub (Dart) | `pub` | `pubspec.yaml` |
| Bun | `bun` | `bun.lockb` |
| Dev Containers | `devcontainers` | `devcontainer.json` |
| Git Submodules | `gitsubmodule` | `.gitmodules` |
| Pre-commit | `pre-commit` | `.pre-commit-config.yaml` |

注: pnpm と yarn はどちらも `npm` ecosystem value を使います。

### Step 2: Map Directory Locations

各 ecosystem について、manifest が置かれている場所を特定します。monorepo では glob pattern を使える `directories` (複数形) を利用してください。

```yaml
directories:
  - "/"           # ルート
  - "/apps/*"     # すべての app サブディレクトリ
  - "/packages/*" # すべての package サブディレクトリ
  - "/lib-*"      # lib- で始まるディレクトリ
  - "**/*"        # 再帰的 (すべてのサブディレクトリ)
```

重要: `directory` (単数形) は glob を **サポートしません**。ワイルドカードには `directories` (複数形) を使ってください。

### Step 3: Configure Each Ecosystem Entry

各 entry に最低限必要なのは次です。

```yaml
- package-ecosystem: "npm"
  directory: "/"
  schedule:
    interval: "weekly"
```

### Step 4: Optimize with Grouping, Labels, and Scheduling

各種の最適化手法については以下のセクションを参照してください。

## Monorepo Strategies

### Glob Patterns for Workspace Coverage

多くの package を持つ monorepo では、各ディレクトリを列挙する代わりに glob pattern を使います。

```yaml
- package-ecosystem: "npm"
  directories:
    - "/"
    - "/apps/*"
    - "/packages/*"
    - "/services/*"
  schedule:
    interval: "weekly"
```

### Cross-Directory Grouping

複数ディレクトリで同じ dependency が更新されたときに 1 つの PR を作るには、`group-by: dependency-name` を使います。

```yaml
groups:
  monorepo-deps:
    group-by: dependency-name
```

これにより、指定したすべてのディレクトリをまたいで dependency ごとに 1 つの PR が作られ、CI コストとレビュー負荷を減らせます。

制約:
- すべてのディレクトリが同じ package ecosystem を使っている必要がある
- 適用されるのは version update のみ
- 互換性のない version constraint は別々の PR になる

### Standalone Packages Outside Workspaces

あるディレクトリが独自の lockfile を持ち、workspace の一部では **ない** 場合 (例: `.github/` 内の script) は、そのための別 ecosystem entry を作成してください。

## Dependency Grouping

関連する依存関係を単一の PR にまとめて、PR のノイズを減らします。

### By Dependency Type

```yaml
groups:
  dev-dependencies:
    dependency-type: "development"
    update-types: ["minor", "patch"]
  production-dependencies:
    dependency-type: "production"
    update-types: ["minor", "patch"]
```

### By Name Pattern

```yaml
groups:
  angular:
    patterns: ["@angular*"]
    update-types: ["minor", "patch"]
  testing:
    patterns: ["jest*", "@testing-library*", "ts-jest"]
```

### For Security Updates

```yaml
groups:
  security-patches:
    applies-to: security-updates
    patterns: ["*"]
    update-types: ["patch", "minor"]
```

主な挙動:
- 複数の group に一致する dependency は **最初に一致した** group に入る
- `applies-to` は省略時に `version-updates` が既定になる
- group に入らない dependency は個別 PR になる

## Multi-Ecosystem Groups

異なる package ecosystem をまたいで更新を 1 つの PR にまとめます。

```yaml
version: 2

multi-ecosystem-groups:
  infrastructure:
    schedule:
      interval: "weekly"
    labels: ["infrastructure", "dependencies"]

updates:
  - package-ecosystem: "docker"
    directory: "/"
    patterns: ["nginx", "redis"]
    multi-ecosystem-group: "infrastructure"

  - package-ecosystem: "terraform"
    directory: "/"
    patterns: ["aws*"]
    multi-ecosystem-group: "infrastructure"
```

`multi-ecosystem-group` を使う場合、`patterns` キーは必須です。

## PR Customization

### Labels

```yaml
labels:
  - "dependencies"
  - "npm"
```

`labels: []` を設定すると既定ラベルを含むすべてのラベルを無効化できます。SemVer ラベル (`major`, `minor`, `patch`) は、リポジトリ内に存在する場合は常に適用されます。

### Commit Messages

```yaml
commit-message:
  prefix: "deps"
  prefix-development: "deps-dev"
  include: "scope"  # prefix の後に deps/deps-dev scope を追加
```

### Assignees and Milestones

```yaml
assignees: ["security-team-lead"]
milestone: 4  # milestone URL にある数値 ID
```

### Branch Name Separator

```yaml
pull-request-branch-name:
  separator: "-"  # 既定値は /
```

### Target Branch

```yaml
target-branch: "develop"  # PR の送信先をデフォルト ブランチではなくこちらにする
```

注: `target-branch` を設定しても security update は引き続き default branch を対象にします。ecosystem の設定が適用されるのは version update のみです。

## Schedule Optimization

### Intervals

サポート: `daily`, `weekly`, `monthly`, `quarterly`, `semiannually`, `yearly`, `cron`

```yaml
schedule:
  interval: "weekly"
  day: "monday"         # weekly のみ
  time: "09:00"         # HH:MM 形式
  timezone: "America/New_York"
```

### Cron Expressions

```yaml
schedule:
  interval: "cron"
  cronjob: "0 9 * * 1"  # 毎週月曜 9 AM
```

### Cooldown Periods

新しく公開されたバージョンへの更新を遅らせ、早期採用による問題を避けます。

```yaml
cooldown:
  default-days: 5
  semver-major-days: 30
  semver-minor-days: 7
  semver-patch-days: 3
  include: ["*"]
  exclude: ["critical-lib"]
```

cooldown は version update のみに適用され、security update には適用されません。

## Security Updates Configuration

### Enable via Repository Settings

Settings → Advanced Security → Dependabot alerts、security updates、grouped security updates を有効にします。

### Group Security Updates in YAML

```yaml
groups:
  security-patches:
    applies-to: security-updates
    patterns: ["*"]
    update-types: ["patch", "minor"]
```

### Disable Version Updates (Security Only)

```yaml
open-pull-requests-limit: 0  # version update PR を無効化
```

### Auto-Triage Rules

GitHub のプリセットは development dependency に対する低影響 alert を自動で却下します。カスタム rule では severity、package name、CWE などで絞り込めます。設定は repository Settings → Advanced Security で行います。

## PR Comment Commands

`@dependabot` コメントで Dependabot PR と対話できます。

> **Note:** 2026 年 1 月時点で、merge/close/reopen コマンドは廃止されています。
> 代わりに GitHub のネイティブ UI、CLI (`gh pr merge`)、または auto-merge を使ってください。

| Command | Effect |
|---|---|
| `@dependabot rebase` | PR を rebase する |
| `@dependabot recreate` | PR を最初から再作成する |
| `@dependabot ignore this dependency` | PR を閉じ、この dependency の今後の更新を止める |
| `@dependabot ignore this major version` | この major version を無視する |
| `@dependabot ignore this minor version` | この minor version を無視する |
| `@dependabot ignore this patch version` | この patch version を無視する |

grouped PR 用の追加コマンド:
- `@dependabot ignore DEPENDENCY_NAME` — group 内の特定 dependency を無視する
- `@dependabot unignore DEPENDENCY_NAME` — ignore を解除し、更新付きで再オープンする
- `@dependabot unignore *` — group 内のすべての dependency に対する全 ignore を解除する
- `@dependabot show DEPENDENCY_NAME ignore conditions` — 現在の ignore 条件を表示する

完全なコマンド リファレンスは `references/pr-commands.md` を参照してください。

## Ignore and Allow Rules

### Ignore Specific Dependencies

```yaml
ignore:
  - dependency-name: "lodash"
  - dependency-name: "@types/node"
    update-types: ["version-update:semver-patch"]
  - dependency-name: "express"
    versions: ["5.x"]
```

### Allow Only Specific Types

```yaml
allow:
  - dependency-type: "production"
  - dependency-name: "express"
```

ルール: dependency が `allow` と `ignore` の両方に一致する場合、**ignored** になります。

### Exclude Paths

```yaml
exclude-paths:
  - "vendor/**"
  - "test/fixtures/**"
```

## Advanced Options

### Versioning Strategy

Dependabot が version constraint をどう編集するかを制御します。

| Value | Behavior |
|---|---|
| `auto` | 既定 — app では increase、library では widen |
| `increase` | 最小 version を常に引き上げる |
| `increase-if-necessary` | 現在の range が新 version を含まない場合のみ変更する |
| `lockfile-only` | lockfile のみ更新し、manifest は無視する |
| `widen` | 古い version と新しい version の両方を含むよう range を広げる |

### Rebase Strategy

```yaml
rebase-strategy: "disabled"  # 自動 rebase を停止
```

commit message に `[dependabot skip]` を含めると、追加 commit の上から rebase することを許可できます。

### Open PR Limit

```yaml
open-pull-requests-limit: 10  # 既定は version 5、security 10
```

`0` に設定すると version update を完全に無効にできます。

### Private Registries

```yaml
registries:
  npm-private:
    type: npm-registry
    url: https://npm.example.com
    token: ${{secrets.NPM_TOKEN}}

updates:
  - package-ecosystem: "npm"
    directory: "/"
    registries:
      - npm-private
```

## FAQ

**Can I have multiple `dependabot.yml` files?**
いいえ。GitHub がサポートするのは `.github/dependabot.yml` に置かれた 1 ファイルだけです。異なる ecosystem や directory には、その中で複数の `updates` entry を使ってください。

**Does Dependabot support pnpm?**
はい。`package-ecosystem: "npm"` を使ってください。Dependabot は `pnpm-lock.yaml` を自動検出します。

**How do I reduce PR noise in a monorepo?**
更新のバッチ化には `groups`、カバレッジには glob 付きの `directories`、ディレクトリ横断の grouping には `group-by: dependency-name` を使ってください。優先度の低い ecosystem には `monthly` や `quarterly` の interval も検討してください。

**How do I handle dependencies outside the workspace?**
その場所を指す独立した `directory` を持つ ecosystem entry を別途作成してください。

## Resources

- `references/dependabot-yml-reference.md` — 完全な YAML option リファレンス
- `references/pr-commands.md` — PR comment command の完全リファレンス
- `references/example-configs.md` — 現実的な構成例
