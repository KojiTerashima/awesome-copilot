# Dependabot Configuration Examples

一般的なシナリオ向けの、実運用に近い `dependabot.yml` 設定例です。

---

## 1. Basic Single Ecosystem

単一の npm project 向け最小構成:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

## 2. Monorepo with Glob Patterns

複数 workspace package を持つ Turborepo/pnpm monorepo:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directories:
      - "/"
      - "/apps/*"
      - "/packages/*"
      - "/services/*"
    schedule:
      interval: "weekly"
      day: "monday"
    groups:
      dev-dependencies:
        dependency-type: "development"
        update-types: ["minor", "patch"]
      production-dependencies:
        dependency-type: "production"
        update-types: ["minor", "patch"]
    labels:
      - "dependencies"
      - "npm"
    commit-message:
      prefix: "deps"
      include: "scope"
```

---

## 3. Grouped Dev vs Production Dependencies

production 変更のレビューを優先しやすいよう、dev と production の更新を分離します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      production-deps:
        dependency-type: "production"
      dev-deps:
        dependency-type: "development"
        exclude-patterns:
          - "eslint*"
      linting:
        patterns:
          - "eslint*"
          - "prettier*"
          - "@typescript-eslint*"
```

---

## 4. Cross-Directory Grouping (Monorepo)

ディレクトリをまたいで共通 dependency ごとに 1 つの PR を作成します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directories:
      - "/frontend"
      - "/admin-panel"
      - "/mobile-app"
    schedule:
      interval: "weekly"
    groups:
      monorepo-dependencies:
        group-by: dependency-name
```

`lodash` が 3 つのディレクトリすべてで更新されると、Dependabot は 1 つの PR を作成します。

---

## 5. Multi-Ecosystem Group (Docker + Terraform)

インフラ依存関係の更新を 1 つの PR に集約します。

```yaml
version: 2

multi-ecosystem-groups:
  infrastructure:
    schedule:
      interval: "weekly"
    labels: ["infrastructure", "dependencies"]
    assignees: ["@platform-team"]

updates:
  - package-ecosystem: "docker"
    directory: "/"
    patterns: ["nginx", "redis", "postgres"]
    multi-ecosystem-group: "infrastructure"

  - package-ecosystem: "terraform"
    directory: "/"
    patterns: ["aws*", "terraform-*"]
    multi-ecosystem-group: "infrastructure"
```

---

## 6. Security Updates Only (Version Updates Disabled)

version update PR を出さずに security vulnerability を監視します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 0  # version update PR を無効化
    groups:
      security-all:
        applies-to: security-updates
        patterns: ["*"]
        update-types: ["patch", "minor"]

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 0
```

---

## 7. Private Registries

private npm registry と Docker registry にアクセスします。

```yaml
version: 2

registries:
  npm-private:
    type: npm-registry
    url: https://npm.internal.example.com
    token: ${{secrets.NPM_PRIVATE_TOKEN}}

  docker-ghcr:
    type: docker-registry
    url: https://ghcr.io
    username: ${{secrets.GHCR_USER}}
    password: ${{secrets.GHCR_TOKEN}}

updates:
  - package-ecosystem: "npm"
    directory: "/"
    registries:
      - npm-private
    schedule:
      interval: "weekly"

  - package-ecosystem: "docker"
    directory: "/"
    registries:
      - docker-ghcr
    schedule:
      interval: "weekly"
```

---

## 8. Cooldown Periods

公開直後の version 更新を遅らせ、早期採用由来の不具合を避けます。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 5
      semver-major-days: 30
      semver-minor-days: 14
      semver-patch-days: 3
      include: ["*"]
      exclude:
        - "security-critical-lib"
        - "@company/internal-*"
```

---

## 9. Cron Scheduling

cron expression を使って特定時刻に更新を実行します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "cron"
      cronjob: "0 9 * * 1"  # 毎週月曜 9:00 AM
      timezone: "America/New_York"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "cron"
      cronjob: "0 6 1 * *"  # 毎月 1 日の 6:00 AM
```

---

## 10. Full-Featured Configuration

複数の最適化を組み合わせた包括例です。

```yaml
version: 2

registries:
  npm-private:
    type: npm-registry
    url: https://npm.example.com
    token: ${{secrets.NPM_TOKEN}}

updates:
  # npm — monorepo workspaces
  - package-ecosystem: "npm"
    directories:
      - "/"
      - "/apps/*"
      - "/packages/*"
      - "/services/*"
    registries:
      - npm-private
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/New_York"
    groups:
      dev-dependencies:
        dependency-type: "development"
        update-types: ["minor", "patch"]
      production-dependencies:
        dependency-type: "production"
        update-types: ["minor", "patch"]
      angular:
        patterns: ["@angular*"]
        update-types: ["minor", "patch"]
      security-patches:
        applies-to: security-updates
        patterns: ["*"]
        update-types: ["patch", "minor"]
    ignore:
      - dependency-name: "aws-sdk"
        update-types: ["version-update:semver-major"]
    cooldown:
      default-days: 3
      semver-major-days: 14
    labels:
      - "dependencies"
      - "npm"
    commit-message:
      prefix: "deps"
      prefix-development: "deps-dev"
      include: "scope"
    assignees:
      - "security-lead"
    open-pull-requests-limit: 15

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    groups:
      actions:
        patterns: ["*"]
    labels:
      - "dependencies"
      - "ci"
    commit-message:
      prefix: "ci"

  # Docker
  - package-ecosystem: "docker"
    directories:
      - "/services/*"
    schedule:
      interval: "weekly"
    labels:
      - "dependencies"
      - "docker"
    commit-message:
      prefix: "deps"

  # pip
  - package-ecosystem: "pip"
    directory: "/scripts"
    schedule:
      interval: "monthly"
    labels:
      - "dependencies"
      - "python"
    versioning-strategy: "increase-if-necessary"
    commit-message:
      prefix: "deps"

  # Terraform
  - package-ecosystem: "terraform"
    directory: "/infra"
    schedule:
      interval: "weekly"
    labels:
      - "dependencies"
      - "terraform"
    commit-message:
      prefix: "infra"
```

---

## 11. Ignore Patterns and Versioning Strategy

何をどのように更新するかを厳密に制御します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    versioning-strategy: "increase"
    ignore:
      # Express 5.x には自動更新しない (破壊的変更)
      - dependency-name: "express"
        versions: ["5.x"]
      # 型定義の patch update はスキップ
      - dependency-name: "@types/*"
        update-types: ["version-update:semver-patch"]
      # vendored package のすべての更新を無視
      - dependency-name: "legacy-internal-lib"
    allow:
      - dependency-type: "all"
    exclude-paths:
      - "vendor/**"
      - "test/fixtures/**"
```

---

## 12. Target Non-Default Branch

本番前に development branch 上で更新を検証します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    target-branch: "develop"
    labels:
      - "dependencies"
      - "staging"

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    target-branch: "develop"
```

注: `target-branch` に関係なく、security update は常に default branch を対象にします。
