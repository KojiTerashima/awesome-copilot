---
name: create-agentsmd
description: 'リポジトリ向けの AGENTS.md ファイルを生成するための prompt'
---

# 高品質な AGENTS.md ファイルを作成する

あなたは code agent です。このリポジトリの root に、https://agents.md/ の公開ガイダンスに従った、完全で正確な AGENTS.md を作成してください。

AGENTS.md は、coding agent が project 上で効果的に作業するために必要な context と instruction を提供するための open format です。

## AGENTS.md とは何か

AGENTS.md は、「agent 向けの README」として機能する Markdown file です。AI coding agent が project で作業しやすくなるよう、専用で予測しやすい場所に context と instruction を置きます。人間向けの README.md を置き換えるのではなく、人間向け README では煩雑になりがちな、agent に必要な詳細な technical context を補完します。

## 主要原則

- **Agent-focused**: automated tool 向けの詳細な technical instruction を含む
- **Complements README.md**: 人間向け documentation を置き換えず、agent 固有の context を追加する
- **Standardized location**: repository root（monorepo では subproject root も可）に置く
- **Open format**: 柔軟な構造を持つ標準 Markdown を使う
- **Ecosystem compatibility**: 20 を超えるさまざまな AI coding tool と agent で動作する

## ファイル構成と内容のガイドライン

### 1. 必須のセットアップ

- file は repository root に `AGENTS.md` として作成する
- 標準 Markdown formatting を使う
- 必須 field はない。project に応じて柔軟な構造にする

### 2. 含めるべき主要 section

#### Project Overview

- project が何をするかの簡潔な説明
- 複雑な場合は architecture overview
- 使用している主要技術と framework

#### Setup Commands

- installation 手順
- environment setup 手順
- dependency management command
- 該当する場合は database setup

#### Development Workflow

- development server の起動方法
- build command
- watch/hot-reload の設定
- package manager 固有事項（npm、pnpm、yarn など）

#### Testing Instructions

- test の実行方法（unit、integration、e2e）
- test file の配置場所と naming convention
- coverage requirement
- 特定の test pattern や framework
- 一部の test だけを実行する方法、特定領域に絞る方法

#### Code Style Guidelines

- 言語固有の規約
- linting と formatting の rule
- file organization pattern
- naming convention
- import/export pattern

#### Build and Deployment

- build command と出力先
- environment 設定
- deployment 手順と requirement
- CI/CD pipeline 情報

### 3. 任意だが推奨される section

#### Security Considerations

- security test requirement
- secret 管理
- authentication pattern
- permission model

#### Monorepo Instructions（該当する場合）

- 複数 package の扱い方
- package 間 dependency
- 選択的な build/test
- package 固有の command

#### Pull Request Guidelines

- title format requirement
- 提出前に必要な check
- review process
- commit message convention

#### Debugging and Troubleshooting

- よくある issue と解決策
- logging pattern
- debug 設定
- performance 上の考慮点

## Example Template

これを出発点として使い、project に合わせて調整してください。

```markdown
# AGENTS.md

## Project Overview

[project の概要、目的、主要技術を簡潔に記載]

## Setup Commands

- dependency を install する: `[package manager] install`
- development server を起動する: `[command]`
- production build を実行する: `[command]`

## Development Workflow

- [development server の起動手順]
- [hot reload/watch mode の情報]
- [environment variable の設定]

## Testing Instructions

- すべての test を実行する: `[command]`
- unit test を実行する: `[command]`
- integration test を実行する: `[command]`
- test coverage を確認する: `[command]`
- [特定の test pattern や requirement]

## Code Style

- [言語と framework の規約]
- [linting rule と command]
- [formatting requirement]
- [file organization pattern]

## Build and Deployment

- [build process の詳細]
- [output directory]
- [environment ごとの build]
- [deployment command]

## Pull Request Guidelines

- Title format: [component] Brief description
- Required checks: `[lint command]`, `[test command]`
- [review requirement]

## Additional Notes

- [project 固有の context]
- [よくある落とし穴や troubleshooting のコツ]
- [performance 上の考慮点]
```

## agents.md の Working Example

以下は agents.md サイトにある実例です。

```markdown
# Sample AGENTS.md file

## Dev environment tips

- Use `pnpm dlx turbo run where <project_name>` to jump to a package instead of scanning with `ls`.
- Run `pnpm install --filter <project_name>` to add the package to your workspace so Vite, ESLint, and TypeScript can see it.
- Use `pnpm create vite@latest <project_name> -- --template react-ts` to spin up a new React + Vite package with TypeScript checks ready.
- Check the name field inside each package's package.json to confirm the right name—skip the top-level one.

## Testing instructions

- Find the CI plan in the .github/workflows folder.
- Run `pnpm turbo run test --filter <project_name>` to run every check defined for that package.
- From the package root you can just call `pnpm test`. The commit should pass all tests before you merge.
- To focus on one step, add the Vitest pattern: `pnpm vitest run -t "<test name>"`.
- Fix any test or type errors until the whole suite is green.
- After moving files or changing imports, run `pnpm lint --filter <project_name>` to be sure ESLint and TypeScript rules still pass.
- Add or update tests for the code you change, even if nobody asked.

## PR instructions

- Title format: [<project_name>] <Title>
- Always run `pnpm lint` and `pnpm test` before committing.
```

## 実装手順

1. **project structure を分析する**。次を理解するためです。

   - 使用している programming language と framework
   - package manager と build tool
   - test framework
   - project architecture（monorepo か single package かなど）

2. **主要 workflow を特定する**。次を確認します。

   - package.json scripts
   - Makefile やその他の build file
   - CI/CD 設定 file
   - documentation file

3. **包括的な section を作成する**。次を含めます。

   - 主要な setup と development command すべて
   - testing 戦略と command
   - code style と規約
   - build と deployment の流れ

4. **agent がそのまま実行できる、具体的で実用的な command** を含める

5. **instruction をテストする**。記載した command が documentation どおりに動くことを確認する

6. **agent が知るべきことに集中する**。一般的な project 情報ではなく、agent が作業に必要な内容を優先する

## ベストプラクティス

- **Be specific**: 曖昧な説明ではなく、正確な command を記載する
- **Use code blocks**: command は clarity のため backtick で囲む
- **Include context**: その手順が必要な理由を説明する
- **Stay current**: project の変化に合わせて更新する
- **Test commands**: 記載する command が実際に動くことを確認する
- **Consider nested files**: monorepo では必要に応じて subproject にも AGENTS.md を作成する

## Monorepo に関する考慮点

大規模 monorepo では次を行います。

- repository root に main の AGENTS.md を置く
- subproject directory に追加の AGENTS.md を作成する
- ある場所に対しては、もっとも近い AGENTS.md が優先される
- package / project 間を移動するための navigation tip を含める

## 最後のメモ

- AGENTS.md は Cursor、Aider、Gemini CLI などを含む 20 以上の AI coding tool で利用できます
- format は意図的に柔軟です。project の needs に合わせて調整してください
- focus すべきは、agent がコードを理解して作業するのに役立つ、実行可能な instruction です
- これは living documentation です。project が進化したら更新してください

AGENTS.md を作成するときは、明確さ、完全性、実用性を最優先してください。目的は、追加の人手 guidance なしでどの coding agent でも project に効果的に貢献できるだけの context を与えることです。
