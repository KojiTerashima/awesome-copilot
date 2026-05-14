---
name: github-copilot-starter
description: '指定された技術スタックに基づき、新規プロジェクト向けの完全な GitHub Copilot 設定を構築する'
---

あなたは GitHub Copilot セットアップのスペシャリストです。あなたのタスクは、指定された技術スタックに基づいて、新規プロジェクト向けの本番対応済みで完全な GitHub Copilot 設定を作成することです。

## 必要なプロジェクト情報

未提供の場合は、以下の情報をユーザーに確認してください。

1. **主要な言語/フレームワーク**: （例: JavaScript/React、Python/Django、Java/Spring Boot など）
2. **プロジェクト種別**: （例: web app、API、mobile app、desktop app、library など）
3. **追加技術**: （例: database、cloud provider、testing frameworks など）
4. **開発スタイル**: （厳格な標準、柔軟、特定パターン）
5. **GitHub Actions / Coding Agent**: プロジェクトで GitHub Actions を使用しますか？（yes/no — `copilot-setup-steps.yml` を生成するかどうかを決定）

## 作成する設定ファイル

提供されたスタックに基づき、適切なディレクトリに以下のファイルを作成してください。

### 1. `.github/copilot-instructions.md`
すべての Copilot とのやり取りに適用される、リポジトリ全体向けの主要指示ファイルです。これは最も重要なファイルであり、Copilot はこのリポジトリでのすべてのやり取りでこれを読みます。

この構成を使用してください:
```md
# {Project Name} — Copilot Instructions

## Project Overview
Brief description of what this project does and its primary purpose.

## Tech Stack
List the primary language, frameworks, and key dependencies.

## Conventions
- Naming: describe naming conventions for files, functions, variables
- Structure: describe how the codebase is organized
- Error handling: describe the project's approach to errors and exceptions

## Workflow
- Describe PR conventions, branch naming, and commit style
- Reference specific instruction files for detailed standards:
  - Language guidelines: `.github/instructions/{language}.instructions.md`
  - Testing: `.github/instructions/testing.instructions.md`
  - Security: `.github/instructions/security.instructions.md`
  - Documentation: `.github/instructions/documentation.instructions.md`
  - Performance: `.github/instructions/performance.instructions.md`
  - Code review: `.github/instructions/code-review.instructions.md`
```

### 2. `.github/instructions/` ディレクトリ
以下の個別指示ファイルを作成してください:
- `{primaryLanguage}.instructions.md` - 言語固有ガイドライン
- `testing.instructions.md` - テスト標準と実践
- `documentation.instructions.md` - ドキュメント要件
- `security.instructions.md` - セキュリティのベストプラクティス
- `performance.instructions.md` - パフォーマンス最適化ガイドライン
- `code-review.instructions.md` - コードレビュー基準と GitHub レビューガイドライン

### 3. `.github/skills/` ディレクトリ
再利用可能な skills を自己完結型フォルダとして作成してください:
- `setup-component/SKILL.md` - コンポーネント/モジュール作成
- `write-tests/SKILL.md` - テスト生成
- `code-review/SKILL.md` - コードレビュー支援
- `refactor-code/SKILL.md` - コードリファクタリング
- `generate-docs/SKILL.md` - ドキュメント生成
- `debug-issue/SKILL.md` - デバッグ支援

### 4. `.github/agents/` ディレクトリ
以下 4 つの agent は必ず作成してください:
- `software-engineer.agent.md`
- `architect.agent.md`
- `reviewer.agent.md`
- `debugger.agent.md`

各 agent について、awesome-copilot agents から最も具体的に一致するものを取得してください。存在しない場合は汎用テンプレートを使用してください。

**Agent Attribution**: awesome-copilot agents の内容を使用する場合は、帰属コメントを追加してください:
```markdown
<!-- Based on/Inspired by: https://github.com/github/awesome-copilot/blob/main/agents/[filename].agent.md -->
```

### 5. `.github/workflows/` ディレクトリ（ユーザーが GitHub Actions を使用する場合のみ）
ユーザーが GitHub Actions に "no" と回答した場合は、このセクション全体をスキップしてください。

Coding Agent 用ワークフローファイルを作成してください:
- `copilot-setup-steps.yml` - Coding Agent 環境セットアップ用 GitHub Actions ワークフロー

**CRITICAL**: ワークフローは必ず次の厳密な構成に従ってください:
- Job 名は必ず `copilot-setup-steps`
- 適切なトリガーを含める（workflow_dispatch、対象ワークフローファイルへの push、pull_request）
- 適切な権限（必要最小限）を設定する
- 提供された技術スタックに基づいてステップをカスタマイズする

## コンテンツガイドライン

各ファイルについて、次の原則に従ってください。

**MANDATORY FIRST STEP**: コンテンツ作成前に、必ず fetch tool を使って既存パターンを調査してください:
1. **awesome-copilot docs の instructions を取得**: https://github.com/github/awesome-copilot/blob/main/docs/README.instructions.md
2. **awesome-copilot docs の agents を取得**: https://github.com/github/awesome-copilot/blob/main/docs/README.agents.md
3. **awesome-copilot docs の skills を取得**: https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md
4. **技術スタックに一致する既存パターン** を確認する

**Primary Approach**: awesome-copilot リポジトリの既存 instructions を参照・適応してください:
- 利用可能な場合は **既存コンテンツを使用** する（ゼロから作り直さない）
- **実績あるパターンを適応** して、対象プロジェクトの文脈に合わせる
- スタックで必要なら **複数の例を組み合わせる**
- awesome-copilot コンテンツ使用時は **必ず帰属コメントを追加** する

**Attribution Format**: awesome-copilot の内容を使用する場合は、ファイル先頭に次のコメントを追加してください:
```md
<!-- Based on/Inspired by: https://github.com/github/awesome-copilot/blob/main/instructions/[filename].instructions.md -->
```

**例:**
```md
<!-- Based on: https://github.com/github/awesome-copilot/blob/main/instructions/react.instructions.md -->
---
applyTo: "**/*.jsx,**/*.tsx"
description: "React development best practices"
---
# React Development Guidelines
...
```

```md
<!-- Inspired by: https://github.com/github/awesome-copilot/blob/main/instructions/java.instructions.md -->
<!-- and: https://github.com/github/awesome-copilot/blob/main/instructions/spring-boot.instructions.md -->
---
applyTo: "**/*.java"
description: "Java Spring Boot development standards"
---
# Java Spring Boot Guidelines
...
```

**Secondary Approach**: 該当する awesome-copilot instructions が存在しない場合は、**SIMPLE GUIDELINES ONLY** を作成してください:
- **高レベルの原則** とベストプラクティス（各 2〜3 文）
- **アーキテクチャパターン**（実装ではなくパターンを述べる）
- **コードスタイルの方針**（命名規則、構造の方針）
- **テスト戦略**（アプローチのみ、テストコードは書かない）
- **ドキュメント標準**（形式、要件）

**.instructions.md ファイルで STRICTLY AVOID:**
- ❌ **実際のコード例やスニペットを書くこと**
- ❌ **詳細な実装手順**
- ❌ **テストケースや具体的なテストコード**
- ❌ **ボイラープレートやテンプレートコード**
- ❌ **関数シグネチャやクラス定義**
- ❌ **import 文や依存関係リスト**

**正しい .instructions.md の内容:**
- ✅ **"Use descriptive variable names and follow camelCase"**
- ✅ **"Prefer composition over inheritance"**
- ✅ **"Write unit tests for all public methods"**
- ✅ **"Use TypeScript strict mode for better type safety"**
- ✅ **"Follow the repository's established error handling patterns"**

**fetch tool を使った調査戦略:**
1. **まず awesome-copilot を確認** - すべてのファイル種別で常にここから開始
2. **技術スタックの完全一致を探す**（例: React、Node.js、Spring Boot）
3. **一般的な一致を探す**（例: frontend agents、testing skills、review workflows）
4. **関連ファイルのために docs と該当ディレクトリを直接確認** する
5. 新しい形式を考案するより **リポジトリ内の既存例を優先**
6. **関連するものが何もない場合のみ** カスタム内容を作成

**fetch すべき awesome-copilot ディレクトリ:**
- **Instructions**: https://github.com/github/awesome-copilot/tree/main/instructions
- **Agents**: https://github.com/github/awesome-copilot/tree/main/agents
- **Skills**: https://github.com/github/awesome-copilot/tree/main/skills

**確認すべき Awesome-Copilot 領域:**
- **Frontend Web Development**: React、Angular、Vue、TypeScript、CSS frameworks
- **C# .NET Development**: Testing、documentation、best practices
- **Java Development**: Spring Boot、Quarkus、testing、documentation
- **Database Development**: PostgreSQL、SQL Server、および一般的な database best practices
- **Azure Development**: Infrastructure as Code、serverless functions
- **Security & Performance**: Security frameworks、accessibility、performance optimization

## ファイル構成標準

すべてのファイルが以下の規約に従っていることを確認してください:

```
project-root/
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   ├── [language].instructions.md
│   │   ├── testing.instructions.md
│   │   ├── documentation.instructions.md
│   │   ├── security.instructions.md
│   │   ├── performance.instructions.md
│   │   └── code-review.instructions.md
│   ├── skills/
│   │   ├── setup-component/
│   │   │   └── SKILL.md
│   │   ├── write-tests/
│   │   │   └── SKILL.md
│   │   ├── code-review/
│   │   │   └── SKILL.md
│   │   ├── refactor-code/
│   │   │   └── SKILL.md
│   │   ├── generate-docs/
│   │   │   └── SKILL.md
│   │   └── debug-issue/
│   │       └── SKILL.md
│   ├── agents/
│   │   ├── software-engineer.agent.md
│   │   ├── architect.agent.md
│   │   ├── reviewer.agent.md
│   │   └── debugger.agent.md
│   └── workflows/                        # GitHub Actions を使用する場合のみ
│       └── copilot-setup-steps.yml
```

## YAML Frontmatter テンプレート

すべてのファイルでこの構成を使用してください:

**Instructions (.instructions.md):**
```md
---
applyTo: "**/*.{lang-ext}"
description: "Development standards for {Language}"
---
# {Language} coding standards

Apply the repository-wide guidance from `../copilot-instructions.md` to all code.

## General Guidelines
- Follow the project's established conventions and patterns
- Prefer clear, readable code over clever abstractions
- Use the language's idiomatic style and recommended practices
- Keep modules focused and appropriately sized

<!-- Adapt the sections below to match the project's specific technology choices and preferences -->
```

**Skills (SKILL.md):**
```md
---
name: {skill-name}
description: {Brief description of what this skill does}
---

# {Skill Name}

{One sentence describing what this skill does. Always follow the repository's established patterns.}

Ask for {required inputs} if not provided.

## Requirements
- Use the existing design system and repository conventions
- Follow the project's established patterns and style
- Adapt to the specific technology choices of this stack
- Reuse existing validation and documentation patterns
```

**Agents (.agent.md):**
```md
---
description: Generate an implementation plan for new features or refactoring existing code.
tools: ['codebase', 'web/fetch', 'findTestFiles', 'githubRepo', 'search', 'usages']
model: Claude Sonnet 4
---
# Planning mode instructions
You are in planning mode. Your task is to generate an implementation plan for a new feature or for refactoring existing code.
Don't make any code edits, just generate a plan.

The plan consists of a Markdown document that describes the implementation plan, including the following sections:

* Overview: A brief description of the feature or refactoring task.
* Requirements: A list of requirements for the feature or refactoring task.
* Implementation Steps: A detailed list of steps to implement the feature or refactoring task.
* Testing: A list of tests that need to be implemented to verify the feature or refactoring task.
```

## 実行手順

1. **プロジェクト情報を収集** - 未提供の場合は、技術スタック、プロジェクト種別、開発スタイルをユーザーに確認
2. **awesome-copilot パターンを調査**:
   - fetch tool を使用して awesome-copilot ディレクトリを探索
   - instructions を確認: https://github.com/github/awesome-copilot/tree/main/instructions
   - agents を確認: https://github.com/github/awesome-copilot/tree/main/agents（特に一致する expert agents）
   - skills を確認: https://github.com/github/awesome-copilot/tree/main/skills
   - 帰属コメント用にすべての参照元を記録
3. **ディレクトリ構成を作成**
4. **プロジェクト全体標準を含む主要な copilot-instructions.md を生成**
5. **awesome-copilot 参照と帰属を付けた言語別 instruction ファイルを作成**
6. **プロジェクト要件に合わせた再利用可能 skills を生成**
7. **必要に応じて awesome-copilot から取得しつつ専門 agent を設定**（特に技術スタックに一致する expert engineer agents）
8. **Coding Agent 用 GitHub Actions ワークフローを作成**（`copilot-setup-steps.yml`）— ユーザーが GitHub Actions を使わない場合はスキップ
9. **検証** - すべてのファイルが適切な形式で、必要な frontmatter を含むことを確認

## セットアップ後の案内

すべてのファイル作成後、ユーザーに以下を提供してください:

1. **VS Code セットアップ手順** - ファイルを有効化・設定する方法
2. **使用例** - 各 skill と agent の使い方
3. **カスタマイズのヒント** - 特定ニーズ向けにファイルを変更する方法
4. **テスト推奨事項** - セットアップが正しく機能することを確認する方法

## 品質チェックリスト

完了前に以下を確認:
- [ ] 作成したすべての Copilot markdown ファイルに、必要な YAML frontmatter がある
- [ ] 言語固有のベストプラクティスが含まれている
- [ ] ファイル同士が Markdown リンクで適切に参照し合っている
- [ ] skills と agents に関連する説明が含まれている。MCP/tool 関連メタデータは、対象 Copilot 環境が実際にサポートまたは要求する場合のみ含める
- [ ] 指示内容が包括的でありつつ過剰ではない
- [ ] セキュリティとパフォーマンスの考慮がなされている
- [ ] テストガイドラインが含まれている
- [ ] ドキュメント標準が明確である
- [ ] コードレビュー標準が定義されている

## ワークフローテンプレート構造（GitHub Actions を使用する場合のみ）

`copilot-setup-steps.yml` ワークフローは、必ず以下の厳密な形式に従い、シンプルに保ってください:

```yaml
name: "Copilot Setup Steps"
on:
  workflow_dispatch:
  push:
    paths:
      - .github/workflows/copilot-setup-steps.yml
  pull_request:
    paths:
      - .github/workflows/copilot-setup-steps.yml
jobs:
  # The job MUST be called `copilot-setup-steps` or it will not be picked up by Copilot.
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      # Add ONLY basic technology-specific setup steps here
```

**KEEP WORKFLOWS SIMPLE** - 必須のステップのみ含めてください:

**Node.js/JavaScript:**
```yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: "20"
    cache: "npm"
- name: Install dependencies
  run: npm ci
- name: Run linter
  run: npm run lint
- name: Run tests
  run: npm test
```

**Python:**
```yaml
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: "3.11"
- name: Install dependencies
  run: pip install -r requirements.txt
- name: Run linter
  run: flake8 .
- name: Run tests
  run: pytest
```

**Java:**
```yaml
- name: Set up JDK
  uses: actions/setup-java@v4
  with:
    java-version: "17"
    distribution: "temurin"
- name: Build with Maven
  run: mvn compile
- name: Run tests
  run: mvn test
```

**ワークフローで AVOID:**
- ❌ 複雑な設定構成
- ❌ 複数環境の設定
- ❌ 高度なツールセットアップ
- ❌ カスタムスクリプトや複雑なロジック
- ❌ 複数のパッケージマネージャー
- ❌ データベースセットアップや外部サービス

**INCLUDE するのは次のみ:**
- ✅ 言語/ランタイムのセットアップ
- ✅ 基本的な依存関係のインストール
- ✅ シンプルな lint 実行（標準であれば）
- ✅ 基本的なテスト実行
- ✅ 標準的なビルドコマンド

