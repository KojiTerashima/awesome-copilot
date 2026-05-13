# Dependabot YAML Options Reference

`.github/dependabot.yml` におけるすべての設定 option の完全リファレンスです。

## File Structure

```yaml
version: 2                    # 必須。常に 2

registries:                   # 任意: private registry へのアクセス
  REGISTRY_NAME:
    type: "..."
    url: "..."

multi-ecosystem-groups:       # 任意: ecosystem 横断の grouping
  GROUP_NAME:
    schedule:
      interval: "..."

updates:                      # 必須: ecosystem 設定の一覧
  - package-ecosystem: "..."  # 必須
    directory: "/"            # 必須 (または directories)
    schedule:                 # 必須
      interval: "..."
```

## Required Keys

### `version`

常に `2` です。トップレベルに置く必要があります。

### `package-ecosystem`

監視する package manager を定義します。ecosystem ごとに 1 entry が必要です (同一 ecosystem でも directory が異なれば複数 entry を持てます)。

| Package Manager | YAML Value | Manifest Files |
|---|---|---|
| Bazel | `bazel` | `MODULE.bazel`, `WORKSPACE` |
| Bun | `bun` | `bun.lockb` |
| Bundler (Ruby) | `bundler` | `Gemfile`, `Gemfile.lock` |
| Cargo (Rust) | `cargo` | `Cargo.toml`, `Cargo.lock` |
| Composer (PHP) | `composer` | `composer.json`, `composer.lock` |
| Conda | `conda` | `environment.yml` |
| Dev Containers | `devcontainers` | `devcontainer.json` |
| Docker | `docker` | `Dockerfile` |
| Docker Compose | `docker-compose` | `docker-compose.yml` |
| .NET SDK | `dotnet-sdk` | `global.json` |
| Elm | `elm` | `elm.json` |
| Git Submodules | `gitsubmodule` | `.gitmodules` |
| GitHub Actions | `github-actions` | `.github/workflows/*.yml` |
| Go Modules | `gomod` | `go.mod`, `go.sum` |
| Gradle | `gradle` | `build.gradle`, `build.gradle.kts` |
| Helm | `helm` | `Chart.yaml` |
| Hex (Elixir) | `mix` | `mix.exs`, `mix.lock` |
| Julia | `julia` | `Project.toml`, `Manifest.toml` |
| Maven | `maven` | `pom.xml` |
| npm/pnpm/yarn | `npm` | `package.json`, lockfile |
| NuGet | `nuget` | `*.csproj`, `packages.config` |
| OpenTofu | `opentofu` | `*.tf` |
| pip/pipenv/poetry/uv | `pip` | `requirements.txt`, `Pipfile`, `pyproject.toml` |
| Pre-commit | `pre-commit` | `.pre-commit-config.yaml` |
| Pub (Dart/Flutter) | `pub` | `pubspec.yaml` |
| Rust Toolchain | `rust-toolchain` | `rust-toolchain.toml` |
| Swift | `swift` | `Package.swift` |
| Terraform | `terraform` | `*.tf` |
| uv | `uv` | `uv.lock`, `pyproject.toml` |
| vcpkg | `vcpkg` | `vcpkg.json` |

### `directory` / `directories`

repo root からの相対パスで package manifest の場所を指定します。

- `directory` — 単一パス (glob 非対応)
- `directories` — パス一覧 (`*` と `**` の glob 対応)

```yaml
# 単一ディレクトリ
directory: "/"

# glob 付きの複数ディレクトリ
directories:
  - "/"
  - "/apps/*"
  - "/packages/*"
```

GitHub Actions では `/` を使ってください。Dependabot が `.github/workflows/` を自動で検索します。

### `schedule`

更新確認の頻度を指定します。

| Parameter | Values | Notes |
|---|---|---|
| `interval` | `daily`, `weekly`, `monthly`, `quarterly`, `semiannually`, `yearly`, `cron` | 必須 |
| `day` | `monday`–`sunday` | weekly のみ |
| `time` | `HH:MM` | 既定では UTC |
| `timezone` | IANA timezone string | 例: `America/New_York` |
| `cronjob` | Cron expression | `interval` が `cron` のとき必須 |

```yaml
schedule:
  interval: "weekly"
  day: "tuesday"
  time: "09:00"
  timezone: "Europe/London"
```

## Grouping Options

### `groups`

依存関係をまとめて PR 数を減らします。

| Parameter | Purpose | Values |
|---|---|---|
| `IDENTIFIER` | Group 名 (branch/PR title で使用) | Letters, pipes, underscores, hyphens |
| `applies-to` | 更新種別 | `version-updates` (default), `security-updates` |
| `dependency-type` | 種別で絞り込み | `development`, `production` |
| `patterns` | 一致する名前を含める | `*` ワイルドカード付き文字列リスト |
| `exclude-patterns` | 一致する名前を除外する | `*` ワイルドカード付き文字列リスト |
| `update-types` | SemVer で絞り込み | `major`, `minor`, `patch` |
| `group-by` | ディレクトリ横断 grouping | `dependency-name` |

```yaml
groups:
  dev-deps:
    dependency-type: "development"
    update-types: ["minor", "patch"]
  angular:
    patterns: ["@angular*"]
    exclude-patterns: ["@angular/cdk"]
  monorepo:
    group-by: dependency-name
```

### `multi-ecosystem-groups` (top-level)

異なる ecosystem の更新を 1 つの PR にまとめます。

```yaml
multi-ecosystem-groups:
  GROUP_NAME:
    schedule:
      interval: "weekly"
    labels: ["infrastructure"]
    assignees: ["@platform-team"]
```

各 `updates` entry で `multi-ecosystem-group: "GROUP_NAME"` を設定して割り当てます。この機能を使う場合、各 ecosystem entry で `patterns` キーが必須です。

## Filtering Options

### `allow`

保守対象にする依存関係を明示的に定義します。

| Parameter | Purpose |
|---|---|
| `dependency-name` | 名前で一致 (`*` ワイルドカード対応) |
| `dependency-type` | `direct`, `indirect`, `all`, `production`, `development` |

```yaml
allow:
  - dependency-type: "production"
  - dependency-name: "express"
```

### `ignore`

更新対象から依存関係またはバージョンを除外します。

| Parameter | Purpose |
|---|---|
| `dependency-name` | 名前で一致 (`*` ワイルドカード対応) |
| `versions` | 特定 version または range (例: `["5.x"]`, `[">=2.0.0"]`) |
| `update-types` | SemVer レベル: `version-update:semver-major`, `version-update:semver-minor`, `version-update:semver-patch` |

```yaml
ignore:
  - dependency-name: "lodash"
  - dependency-name: "@types/node"
    update-types: ["version-update:semver-patch"]
  - dependency-name: "express"
    versions: ["5.x"]
```

ルール: dependency が `allow` と `ignore` の両方に一致する場合、**ignored** になります。

### `exclude-paths`

manifest の走査対象から特定ディレクトリやファイルを除外します。

```yaml
exclude-paths:
  - "vendor/**"
  - "test/fixtures/**"
  - "*.lock"
```

glob pattern をサポートします: `*` (単一セグメント)、`**` (再帰)、特定ファイル パス。

## PR Customization Options

### `labels`

```yaml
labels:
  - "dependencies"
  - "npm"
```

`labels: []` を設定するとすべてのラベルを無効にできます。SemVer ラベルは、リポジトリに存在する場合は常に適用されます。

### `assignees`

```yaml
assignees:
  - "user1"
  - "user2"
```

assignee には write access が必要です (org repo では read access でも可)。

### `milestone`

```yaml
milestone: 4  # milestone URL の数値 ID
```

### `commit-message`

```yaml
commit-message:
  prefix: "deps"              # 最大 50 文字。末尾が英数字なら自動でコロン付与
  prefix-development: "deps-dev"  # dev dependency 向けの別 prefix
  include: "scope"            # prefix の後に deps/deps-dev を追加
```

### `pull-request-branch-name`

```yaml
pull-request-branch-name:
  separator: "-"  # options: "-", "_", "/"
```

### `target-branch`

```yaml
target-branch: "develop"
```

これを設定すると、version update 向けの設定は version update にしか適用されません。security update は常に default branch を対象にします。

## Scheduling & Rate Limiting

### `cooldown`

新しく公開された version に対する version update を遅らせます。

| Parameter | Purpose |
|---|---|
| `default-days` | 既定の cooldown (1–90 日) |
| `semver-major-days` | major update の cooldown |
| `semver-minor-days` | minor update の cooldown |
| `semver-patch-days` | patch update の cooldown |
| `include` | cooldown を適用する dependency (最大 150、`*` 対応) |
| `exclude` | cooldown 免除の dependency (最大 150、こちらが優先) |

```yaml
cooldown:
  default-days: 5
  semver-major-days: 30
  semver-minor-days: 7
  semver-patch-days: 3
  include: ["*"]
  exclude: ["critical-security-lib"]
```

### `open-pull-requests-limit`

```yaml
open-pull-requests-limit: 10  # default: version update は 5
```

`0` にすると version update を完全に無効化できます。security update には別途 10 の内部上限があります。

## Advanced Options

### `versioning-strategy`

サポート対象: `bundler`, `cargo`, `composer`, `mix`, `npm`, `pip`, `pub`, `uv`。

| Value | Behavior |
|---|---|
| `auto` | 既定: app では increase、library では widen |
| `increase` | 最小 version を常に引き上げる |
| `increase-if-necessary` | 現在の range が新 version を含まない場合のみ変更 |
| `lockfile-only` | lockfile のみ更新 |
| `widen` | 古い version と新しい version をともに含むよう range を広げる |

### `rebase-strategy`

```yaml
rebase-strategy: "disabled"
```

既定の挙動では、Dependabot は競合時に PR を自動 rebase します。rebase は PR を開いてから 30 日で停止します。

追加 commit の上から Dependabot に force push させるには、commit message に `[dependabot skip]` を含めてください。

### `vendor`

サポート対象: `bundler`, `gomod`。

```yaml
vendor: true  # vendored dependency を保守する
```

Go modules は vendored dependency を自動検出します。

### `insecure-external-code-execution`

サポート対象: `bundler`, `mix`, `pip`。

```yaml
insecure-external-code-execution: "allow"
```

更新時に manifest 内のコード実行を Dependabot に許可します。解決時にコード実行が必要な ecosystem では必須になることがあります。

## Private Registries

### Top-Level Registry Definition

```yaml
registries:
  npm-private:
    type: npm-registry
    url: https://npm.example.com
    token: ${{secrets.NPM_TOKEN}}

  maven-central:
    type: maven-repository
    url: https://repo.maven.apache.org/maven2
    username: ""
    password: ""

  docker-ghcr:
    type: docker-registry
    url: https://ghcr.io
    username: ${{secrets.GHCR_USER}}
    password: ${{secrets.GHCR_TOKEN}}

  python-private:
    type: python-index
    url: https://pypi.example.com/simple
    token: ${{secrets.PYPI_TOKEN}}
```

### Associating Registries with Ecosystems

```yaml
updates:
  - package-ecosystem: "npm"
    directory: "/"
    registries:
      - npm-private
    schedule:
      interval: "weekly"
```

`registries: "*"` を使うと、定義済みのすべての registry へのアクセスを許可します。
