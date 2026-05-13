---
description: 'GitHub Copilot 用の高品質なエージェントスキルを作成するためのガイドライン'
applyTo: '**/skills/**/SKILL.md'
---

# エージェントスキル ファイルのガイドライン

特殊な機能、ワークフロー、バンドルされたリソースで GitHub Copilot を強化する、効果的で移植可能なエージェントスキルを作成する手順。

## エージェントスキルとは何ですか?

エージェントスキルは、AI エージェントに特殊な機能を教えるための手順とバンドルされたリソースが含まれる自己完結型のフォルダーです。カスタム命令 (コーディング標準を定義する) とは異なり、スキルにより、スクリプト、サンプル、テンプレート、参照データを含むタスク固有のワークフローが可能になります。

主な特徴:
- **ポータブル**: VS Code、Copilot CLI、および Copilot コーディングエージェント全体で動作します
- **プログレッシブロード**: ユーザーのリクエストに関連する場合にのみロードされます。
- **リソースのバンドル**: 手順とともにスクリプト、テンプレート、サンプルを含めることができます
- **オンデマンド**: プロンプトの関連性に基づいて自動的にアクティブ化されます

## ディレクトリ構造

スキルは特定の場所に保存されます。

| 位置 | 範囲 | おすすめ |
|----------|-------|----------------|
| `.github/skills/<skill-name>/` | プロジェクト/リポジトリ | プロジェクトスキルに推奨 |
| `.claude/skills/<skill-name>/` | プロジェクト/リポジトリ | レガシー、下位互換性のため |
| `~/.github/skills/<skill-name>/` | 個人用 (ユーザー全体) | 個人スキルにおすすめ |
| `~/.claude/skills/<skill-name>/` | 個人用 (ユーザー全体) | レガシー、下位互換性のため |

各スキルには、少なくとも `SKILL.md` ファイルを含む独自のサブディレクトリが必要です。

## 必要なSKILL.md形式

### 前付事項 (必須)

```yaml
---
name: webapp-testing
description: 'Toolkit for testing local web applications using Playwright. Use when asked to verify frontend functionality, debug UI behavior, capture browser screenshots, check for visual regressions, or view browser console logs. Supports Chrome, Firefox, and WebKit browsers.'
license: Complete terms in LICENSE.txt
---
```

| 分野 | 必須 | 制約 |
|-------|----------|-------------|
| `name` | はい | 小文字、スペースの場合はハイフン、最大 64 文字 (例: `webapp-testing`) |
| `description` | はい | 10 ～ 1024 文字、明確な機能とユースケース、一重引用符で囲む |
| `license` | いいえ | LICENSE.txt への参照 (例: `Complete terms in LICENSE.txt`) または SPDX 識別子 |

### 説明 ベストプラクティス

**重要**: `description` フィールドは、自動スキル検出の主要なメカニズムです。コパイロットは `name` と `description` のみを読み取り、スキルをロードするかどうかを決定します。説明が曖昧だとスキルは発動しません。

**説明に含める内容:**
1. **何を** スキルが行うこと (能力)
2. **いつ使用するか** (特定のトリガー、シナリオ、ファイルタイプ、またはユーザーリクエスト)
3. ユーザーがプロンプトで言及する可能性のある **キーワード**

**良い説明:**
```yaml
description: 'Toolkit for testing local web applications using Playwright. Use when asked to verify frontend functionality, debug UI behavior, capture browser screenshots, check for visual regressions, or view browser console logs. Supports Chrome, Firefox, and WebKit browsers.'
```

**不適切な説明:**
```yaml
description: 'Web testing helpers'
```

不十分な説明は次の理由で失敗します。
- 特定のトリガーはありません (Copilot はいつこれをロードする必要がありますか?)
- キーワードなし (どのユーザープロンプトが一致しますか?)
- 機能がない (実際に何ができるのか?)

### 本文の内容

本文には、スキルがアクティブ化された後に Copilot がロードする詳細な命令が含まれています。お勧めのセクション:

| セクション | 目的 |
|---------|---------|
| `# Title` | このスキルでできることの簡単な概要 |
| `## When to Use This Skill` | シナリオのリスト (説明トリガーを強化) |
| `## Prerequisites` | 必要なツール、依存関係、環境セットアップ (該当する場合) |
| `## Step-by-Step Workflows` | 繰り返し可能な手順 (ビルド、デプロイ、セットアップ) の番号付きステップ |
| `## Gotchas` | 明白ではない動作に関する事前の警告 (「Y があるので X を決して実行しないでください」) |
| `## Troubleshooting` | 既知の問題に対する事後対応​​型の修正 (「X が表示された場合は、Y を試してください」) |
| `## References` | バンドルされたドキュメントまたは外部リソースへのリンク |

すべてのスキルにすべてのセクションが必要なわけではありません。外部依存関係がない場合は、`## Prerequisites` をスキップします。スキルが単なるアドバイスである場合は、`## Step-by-Step Workflows` をスキップしてください。スキルに外部ツール、API、またはプラットフォーム固有の動作が含まれる場合は、常に `## Gotchas` を含めます。

コンテンツ品質の原則 (何を含め、何を省略するか) については、以下の [影響力の高いスキルを書く](#writing-high-impact-skills) を参照してください。

### 各セクションの書き方

**`# Title`** — スキルによって何が可能になるかを説明する 1 文。一般的な表現は避けてください。ドメインについて具体的にします。

**`## When to Use This Skill`** — 説明のトリガーを強化する具体的なシナリオの箇条書きリスト。これは、Copilot が正しいスキルをロードしたことを確認するのに役立ちます。

```markdown
## When to Use This Skill

- User asks to test a web application in a browser
- User needs to capture screenshots for visual regression testing
- User wants to debug frontend behavior with browser console logs
```

**`## Prerequisites`** — Copilot が利用可能であると想定できないツール、サービス、または構成がスキルに必要な場合にのみ含めます。正確なインストールコマンドをリストします。

```markdown
## Prerequisites

- [Playwright](https://playwright.dev/) installed: `npm install -D @playwright/test`
- At least one browser engine installed: `npx playwright install chromium`
```

**`## Step-by-Step Workflows`** — 順序が重要な場合の反復可能な手順 (ビルド、デプロイ、環境セットアップ) の番号付きステップ。ハードコーディングされたファイルパスや行番号ではなく、各段階で何を達成するかを説明します。ステップはさまざまなプロジェクト構造に適応できる必要があります。複雑なワークフロー (5 ステップ以上) の場合は、`references/` ファイルに分割し、それらのファイルにリンクします。

```markdown
## Step-by-Step Workflows

### Deploy to Staging

1. Build the project: `npm run build`
2. Run pre-deploy validation: `npm run validate`
3. Deploy to staging: `npm run deploy -- --env staging`
4. Verify the health endpoint returns 200
```

**`## Gotchas`** — 間違いを防ぐプロアクティブな警告。明らかではないデフォルト、API の癖、バージョン固有の動作、および一般的なトラップを文書化します。主要な制約を太字にして、その理由を説明します。

```markdown
## Gotchas

- **Never** call `billing.charge()` without checking `user.hasPaymentMethod` first —
  the SDK throws an unrecoverable error instead of returning a failure.
- The `currency` field expects ISO 4217 codes, not display names.
  Copilot often writes "dollars" instead of "USD".
```

**`## Troubleshooting`** — 既知の問題に対する事後対応​​型の修正。症状と解決策のペアの表として表示されます。各行は自己完結型であり、実行可能である必要があります。

```markdown
## Troubleshooting

| Issue | Solution |
|-------|----------|
| Plugin won't connect | Check servers are running (`npm run start:all`) |
| Browser blocks localhost | Allow local network access, or try a different browser |
| Tool execution times out | Ensure the plugin UI is open and shows "Connected" |
```

**`## References`** — `references/` にバンドルされているドキュメント、外部ドキュメント、または関連スキルへのリンク。バンドルされたファイルには相対パスを使用します。

## バンドルリソース

スキルには、Copilot がオンデマンドでアクセスする追加フ​​ァイルを含めることができます。

### サポートされているリソースの種類

| フォルダ | 目的 | コンテキストに読み込まれていますか? | サンプルファイル |
|--------|---------|---------------------|---------------|
| `scripts/` | 特定の操作を実行する実行可能な自動化 | 実行時 | `helper.py`、`validate.sh`、`build.ts` |
| `references/` | AI エージェントが意思決定を伝えるために読む文書 | はい、参照された場合 | `api_reference.md`、`schema.md`、`workflow_guide.md` |
| `assets/` | **出力では静的ファイルが現状のまま使用されます** (AI エージェントによって変更されません) | いいえ | `logo.png`、`brand-template.pptx`、`custom-font.ttf` |
| `templates/` | **AI エージェントが変更**し、その上に構築するスターターコード/スキャフォールド | はい、参照された場合 | `viewer.html` (アルゴリズムの挿入)、`hello-world/` (拡張) |

### ディレクトリ構造の例

```
.github/skills/my-skill/
├── SKILL.md              # Required: Main instructions
├── LICENSE.txt           # Recommended: License terms (Apache 2.0 typical)
├── scripts/              # Optional: Executable automation
│   ├── helper.py         # Python script
│   └── helper.ps1        # PowerShell script
├── references/           # Optional: Documentation loaded into context
│   ├── api_reference.md
│   ├── workflow-setup.md     # Detailed workflow (>5 steps)
│   └── workflow-deployment.md
├── assets/               # Optional: Static files used AS-IS in output
│   ├── baseline.png      # Reference image for comparison
│   └── report-template.html
└── templates/            # Optional: Starter code the AI agent modifies
    ├── scaffold.py       # Code scaffold the AI agent customizes
    └── config.template   # Config template the AI agent fills in
```

> **LICENSE.txt**: スキルを作成するときは、Apache 2.0 ライセンステキストを https://www.apache.org/licenses/LICENSE-2.0.txt からダウンロードし、`LICENSE.txt` として保存します。付録セクションの著作権発行年と所有者を更新します。

### アセットとテンプレート: 主な違い

**アセット**は、出力内で**変更されずに消費される**静的リソースです。
- 生成されたドキュメントに埋め込まれる `logo.png`
- `report-template.html` が出力形式としてコピーされました
- テキストレンダリングに適用される `custom-font.ttf`

**テンプレート**は、**AI エージェントが積極的に変更する**スターターコード/足場です。
- AI エージェントがロジックを挿入する `scaffold.py`
- `config.template` AI エージェントがユーザーの要件に基づいて値を入力します
- AI エージェントが新機能で拡張する `hello-world/` プロジェクトディレクトリ

**経験則**: AI エージェントがファイルの内容を読み取って構築する場合 → `templates/`。ファイルをそのまま出力に使用する場合→`assets/`。

### SKILL.mdのリソースの参照

相対パスを使用して、スキルディレクトリ内のファイルを参照します。

```markdown
## Available Scripts

Run the [helper script](./scripts/helper.py) to automate common tasks.

See [API reference](./references/api_reference.md) for detailed documentation.

Use the [scaffold](./templates/scaffold.py) as a starting point.
```

## プログレッシブローディングアーキテクチャ

スキルは効率を高めるために 3 つのレベルのロードを使用します。

| レベル | 何をロードするか | いつ |
|-------|------------|------|
| 1. 発見 | `name` と `description` のみ | 常に (軽量メタデータ) |
| 2. 説明書 | `SKILL.md` 本文全体 | リクエストが説明と一致する場合 |
| 3. リソース | スクリプト、サンプル、ドキュメント | Copilot がそれらを参照する場合のみ |

これはつまり：
- コンテキストを消費せずに多くのスキルをインストールする
- タスクごとに関連するコンテンツのみがロードされます
- リソースは明示的に必要になるまでロードされません

## コンテンツガイドライン

### 文体

- 命令的なムードを使用します: 「実行」、「作成」、「構成」 (「実行する必要があります」ではありません)
- 具体的かつ実行可能であること
- パラメータを含む正確なコマンドを含める
- 役立つ場合は期待される出力を表示する
- セクションに焦点を当ててスキャンしやすくする

### スクリプトの要件

スクリプトを含める場合は、クロスプラットフォーム言語を優先します。

| 言語 | 使用事例 |
|----------|----------|
| パイソン | 複雑な自動化、データ処理 |
| うわー | PowerShell コアスクリプト |
| Node.js | JavaScript ベースのツール |
| バッシュ/シェル | 単純な自動化タスク |

ベストプラクティス:
- ヘルプ/使用法ドキュメントを含める (`--help` フラグ)
- 明確なメッセージでエラーを適切に処理する
- 資格情報やシークレットの保存を避ける
- 可能な場合は相対パスを使用してください

### スクリプトをバンドルする場合

次の場合にスキルにスクリプトを含めます。
- 同じコードがエージェントによって繰り返し書き換えられる
- 決定的な信頼性が重要です (ファイル操作、API 呼び出しなど)
- 複雑なロジックは、毎回生成するのではなく、事前にテストすることでメリットが得られます。
- この作戦には独立して進化できる自己完結型の目的がある
- テスト容易性が重要 - スクリプトは単体テストと検証が可能
- 動的生成よりも予測可能な動作が優先されます

スクリプトは進化を可能にします。単純な操作であっても、操作が複雑になる場合、呼び出し間で一貫した動作が必要な場合、または将来の拡張性が必要な場合には、スクリプトとして実装することでメリットが得られます。

### セキュリティに関する考慮事項

- スクリプトは既存の資格情報ヘルパーに依存します (資格情報ストレージはありません)
- 破壊的な操作にのみ `--force` フラグを含めます
- 元に戻せないアクションの前にユーザーに警告する
- ネットワーク操作または外部呼び出しを文書化します。

## 影響力の高いスキルを書く

### 副操縦士が知らないことに焦点を当てる

Copilot がトレーニングデータからすでに知っている情報 (標準言語構文、一般的なライブラリの使用法、十分に文書化された API の動作など) を含めないでください。スキルのすべての行は、そうでなければ副操縦士が間違えたり完全に見逃したりするような内容を教えるべきです。公式ドキュメントの最初のページに情報が記載されている場合は、省略してください。 Copilot の動作を変更する内部規則、明白ではないデフォルト、バージョン固有の癖、およびドメイン固有のワークフローに焦点を当てます。

### コンテキスト予算の認識

すべてのスキルの説明は、検出中に利用可能なコンテキストウィンドウの限られた部分を共有します。あなたの説明は、副操縦士の注意を引くためにインストールされている他のすべてのスキルと競合します。説明は簡潔かつキーワードを密に保ち、何を、いつ、関連するキーワードを伝える最短のテキストを目指します。冗長な説明は自分の予算を無駄にするだけではありません。システム内の他のすべてのスキルの可視性が低下します。

### 落とし穴は最もシグナルの高いコンテンツです

`## Gotchas` セクションは、あらゆるスキルの中で常に最も価値のある部分であり、間違いが起こる前に予防的な警告を発します。これは、何か問題が発生した後に事後対応的な修正を提供する `## Troubleshooting` とは異なります。注意事項を生きたセクションとして扱います。Copilot が間違った結果を生成するたびに、注意事項を追加します。キーの制約を太字にして、その理由を説明します (例: 「最初に `Y` をチェックせずに `X()` を呼び出すことは決してしないでください。SDK は回復不可能なエラーをスローします。」)。

### 厳格な手順よりも柔軟なガイドラインを好む

番号付きのステップは、順序が本当に重要な具体的で反復可能な手順 (ビルド、デプロイ、環境セットアップ) にのみ使用してください。無制限のタスク (デバッグ、リファクタリング、コードレビュー) の場合は、代わりに決定基準と参照情報を提供します。Copilot には、ユーザーの特定の状況に適応する柔軟性が必要です。

```markdown
# ❌ Too rigid
1. Open the file at src/api/handlers.ts
2. Find the function named processOrder
3. Add a try-catch block around lines 45-60

# ✅ Flexible
When fixing error handling in API handlers:
- Ensure all database operations have proper error handling
- Use the project's ErrorHandler utility (see ./references/error-handling.md)
- Log errors with enough context to debug in production
```

### 大きなスキルには段階的開示を使用する

SKILL.md が約 200 行を超える場合は、詳細なコンテンツをサブディレクトリに分割することを検討してください。これによりコンテキストの消費が削減されます。Copilot は最初にコア命令のみをロードし、必要に応じて参照資料を取得します。

```markdown
## Reference Files

- `references/api.md` — complete function signatures and return types
- `references/error-codes.md` — every error code this service can return
- `scripts/validate.sh` — run this after making changes to verify correctness

Read these files as needed for your current task. Do not read them all upfront.
```

## よくあるパターン

### パラメータテーブルのパターン

パラメーターを明確に文書化します。

```markdown
| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `--input` | Yes | - | Input file or URL to process |
| `--action` | Yes | - | Action to perform |
| `--verbose` | No | `false` | Enable verbose output |
```

### ワークフローの実行パターン

複数ステップのワークフローを実行する場合は、各ステップが関連ドキュメントを参照する TODO リストを作成します。

```markdown
## TODO
- [ ] Step 1: Configure environment - see [workflow-setup.md](./references/workflow-setup.md#environment)
- [ ] Step 2: Build project - see [workflow-setup.md](./references/workflow-setup.md#build)
- [ ] Step 3: Deploy to staging - see [workflow-deployment.md](./references/workflow-deployment.md#staging)
- [ ] Step 4: Run validation - see [workflow-deployment.md](./references/workflow-deployment.md#validation)
- [ ] Step 5: Deploy to production - see [workflow-deployment.md](./references/workflow-deployment.md#production)
```

これによりトレーサビリティが確保され、ワー​​クフローが中断された場合でも再開できます。

## 検証チェックリスト

スキルを公開する前に:

- [ ] `SKILL.md` には `name` および `description` との有効な前付があります
- [ ] `name` は小文字でハイフンを含み、≤64 文字です
- [ ] `description` は、**何をするのか**、**いつ**使用するか、および関連する**キーワード**を明確に示しています
- [ ] `description` は簡潔でキーワードが豊富です (コンテキストの予算を考慮)
- [ ] 本体は、副操縦士がトレーニングデータからは知り得ない情報に焦点を当てます
- [ ] 本文には、いつ使用するか、前提条件 (該当する場合)、および主要な手順が含まれています
- [ ] `## Gotchas` セクションは、スキルに明らかではない動作、API の癖、または一般的なトラップが含まれる場合に存在します。
- [ ] SKILL.md 本文は 500 行未満 (最大 200 行で `references/` に分割することを検討してください。500 行が最大値です)
- [ ] 大規模なワークフロー (>5 ステップ) は、SKILL.md からの明確なリンクを含む `references/` フォルダーに分割されます
- [ ] スクリプトにはヘルプドキュメントとエラー処理が含まれます
- [ ] すべてのリソース参照に使用される相対パス
- [ ] ハードコードされた認証情報やシークレットはありません

## 関連リソース

- [エージェントのスキル仕様](https://agentskills.io/)
- [VS Code エージェントスキルのドキュメント](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [リファレンススキルリポジトリ](https://github.com/anthropics/skills)
- [素晴らしい副操縦士のスキ​​ル](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)
