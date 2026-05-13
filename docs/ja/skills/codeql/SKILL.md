---
name: codeql
description: GitHub Actions ワークフローと CodeQL CLI を使った CodeQL コードスキャン設定の包括ガイドです。コードスキャン構成、CodeQL ワークフローファイル、CodeQL CLI コマンド、SARIF 出力、セキュリティ分析セットアップ、CodeQL 分析のトラブルシューティングが必要な場合に使用してください。
---

# CodeQL Code Scanning

このスキルは、CodeQL コードスキャンを設定・実行するための手順を提供します。対象は GitHub Actions ワークフローとスタンドアロンの CodeQL CLI の両方です。

## このスキルを使う場面

- `codeql.yml` GitHub Actions ワークフローの作成/カスタマイズ
- コードスキャンの default setup と advanced setup の選択
- CodeQL の言語マトリクス、build mode、query suite の設定
- ローカルでの CodeQL CLI 実行（`codeql database create`, `database analyze`, `github upload-results`）
- CodeQL の SARIF 出力の理解や解釈
- CodeQL 分析失敗のトラブルシュート（build mode、コンパイル言語、runner 要件）
- モノレポ向けのコンポーネント別 CodeQL 設定
- dependency caching、custom query pack、model pack の設定

## 対応言語

| Language | Identifier | Alternatives |
|---|---|---|
| C/C++ | `c-cpp` | `c`, `cpp` |
| C# | `csharp` | — |
| Go | `go` | — |
| Java/Kotlin | `java-kotlin` | `java`, `kotlin` |
| JavaScript/TypeScript | `javascript-typescript` | `javascript`, `typescript` |
| Python | `python` | — |
| Ruby | `ruby` | — |
| Rust | `rust` | — |
| Swift | `swift` | — |
| GitHub Actions | `actions` | — |

> 代替 identifier は標準 identifier と等価です（例: `javascript` でも TypeScript 解析は除外されません）。

## Core Workflow — GitHub Actions

### Step 1: セットアップ種別を選ぶ
- **Default setup** — repository Settings → Advanced Security → CodeQL analysis で有効化。素早く始めるのに最適。多くの言語で `none` build mode を利用。
- **Advanced setup** — `.github/workflows/codeql.yml` を作成し、trigger、build mode、query suite、matrix strategy を完全制御。

default から advanced へ切り替える場合は、先に default setup を無効化してから workflow ファイルをコミットします。

### Step 2: ワークフロートリガー設定
```yaml
on:
  push:
    branches: [main, protected]
  pull_request:
    branches: [main]
  schedule:
    - cron: '30 6 * * 1'  # 毎週月曜 6:30 UTC
```

- `push` — 指定ブランチへの push ごとにスキャン（結果は Security タブ）
- `pull_request` — PR マージコミットをスキャン（結果は PR 注釈）
- `schedule` — default branch の定期スキャン（cron は default branch 上に必要）
- `merge_group` — merge queue 利用時に追加

ドキュメントのみの PR でスキャンをスキップする例:
```yaml
on:
  pull_request:
    paths-ignore:
      - '**/*.md'
      - '**/*.txt'
```
> `paths-ignore` は「workflow を実行するか」を制御し、「解析対象ファイル」を直接制御しません。

### Step 3: 権限設定
```yaml
permissions:
  security-events: write   # SARIF アップロードに必須
  contents: read            # checkout に必須
  actions: read             # private repo で codeql-action 利用時に必須
```

### Step 4: 言語マトリクス設定
```yaml
jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        include:
          - language: javascript-typescript
            build-mode: none
          - language: python
            build-mode: none
```

コンパイル言語では適切な `build-mode` を設定します:
- `none` — ビルド不要（C/C++, C#, Java, Rust でサポート）
- `autobuild` — 自動ビルド検出
- `manual` — カスタムビルドコマンド（advanced setup のみ）

### Step 5: CodeQL 初期化と解析
```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4

  - name: Initialize CodeQL
    uses: github/codeql-action/init@v4
    with:
      languages: ${{ matrix.language }}
      build-mode: ${{ matrix.build-mode }}
      queries: security-extended
      dependency-caching: true

  - name: Perform CodeQL Analysis
    uses: github/codeql-action/analyze@v4
    with:
      category: "/language:${{ matrix.language }}"
```

**Query suite options:**
- `security-extended`
- `security-and-quality`
- `packs:` で custom query pack を指定

### Step 6: モノレポ設定
```yaml
category: "/language:${{ matrix.language }}/component:frontend"
```

```yaml
paths:
  - apps/
  - services/
paths-ignore:
  - node_modules/
  - '**/test/**'
```

```yaml
- uses: github/codeql-action/init@v4
  with:
    config-file: .github/codeql/codeql-config.yml
```

### Step 7: 手動ビルド（コンパイル言語）
```yaml
- language: c-cpp
  build-mode: manual
```

```yaml
- if: matrix.build-mode == 'manual'
  name: Build
  run: |
    make bootstrap
    make release
```

## Core Workflow — CodeQL CLI

### Step 1: CodeQL CLI のインストール
```bash
# Download from https://github.com/github/codeql-action/releases
# Extract and add to PATH
export PATH="$HOME/codeql:$PATH"

# Verify installation
codeql resolve packs
codeql resolve languages
```

### Step 2: CodeQL Database 作成
```bash
codeql database create codeql-db \
  --language=javascript-typescript \
  --source-root=src

codeql database create codeql-dbs \
  --db-cluster \
  --language=java,python \
  --command=./build.sh \
  --source-root=src
```

### Step 3: Database 解析
```bash
codeql database analyze codeql-db \
  javascript-code-scanning.qls \
  --format=sarif-latest \
  --sarif-category=javascript \
  --output=results.sarif
```

### Step 4: 結果を GitHub へアップロード
```bash
codeql github upload-results \
  --repository=owner/repo \
  --ref=refs/heads/main \
  --commit=<commit-sha> \
  --sarif=results.sarif
```

`security-events: write` 権限を持つ `GITHUB_TOKEN` が必要です。

### CLI Server Mode
```bash
codeql execute cli-server
```

## Alert Management

- **Standard severity:** `Error`, `Warning`, `Note`
- **Security severity:** `Critical`, `High`, `Medium`, `Low`（CVSS 由来、表示優先）
- PR では changed lines に注釈が表示され、既定で `error`/`critical`/`high` でチェック失敗
- 偽陽性の dismiss では監査のため理由を明記

## Custom Queries and Packs
```yaml
- uses: github/codeql-action/init@v4
  with:
    packs: |
      my-org/my-security-queries@1.0.0
      codeql/javascript-queries:AlertSuppression.ql
```

```bash
codeql pack init my-org/my-queries
codeql pack install
codeql pack publish
```

```yaml
paths:
  - apps/
  - services/
paths-ignore:
  - '**/test/**'
  - node_modules/
queries:
  - uses: security-extended
packs:
  javascript-typescript:
    - my-org/my-custom-queries
```

## Troubleshooting（要点）

| Problem | Solution |
|---|---|
| Workflow not triggering | `on:`、`paths/branches`、対象 branch 上の workflow 存在を確認 |
| `Resource not accessible` | `security-events: write` と `contents: read` を付与 |
| Autobuild failure | `build-mode: manual` + 明示 build コマンドへ切替 |
| SARIF upload fails | トークン権限と 10 MB 制限を確認 |
| Two CodeQL workflows | default setup と advanced setup の重複を解消 |

## Reference Files

- `references/workflow-configuration.md`
- `references/cli-commands.md`
- `references/sarif-output.md`
- `references/compiled-languages.md`
- `references/troubleshooting.md`
- `references/alert-management.md`
