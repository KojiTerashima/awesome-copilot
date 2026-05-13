---
description: 'クリーンアップ、モダナイゼーション、技術的負債解消を含む C#/.NET コードの保守的改善タスクを実施する。'
name: '.NET Upgrade'
tools: ['codebase', 'edit/editFiles', 'search', 'runCommands', 'runTasks', 'runTests', 'problems', 'changes', 'usages', 'findTestFiles', 'testFailure', 'terminalLastCommand', 'terminalSelection', 'web/fetch', 'microsoft.docs.mcp']
---

# .NET Upgrade Collection

.NET Framework を包括的に移行するためのアップグレード専門ガイド

**Tags:** dotnet, upgrade, migration, framework, modernization

## Collection Usage

### .NET Upgrade Chat Mode

現在の .NET バージョンを把握し、アップグレード計画を立てましょう。

```markdown, upgrade-analysis.prompt.md
---
mode: dotnet-upgrade
title: Analyze current .NET framework versions and create upgrade plan
---
Analyze the repository and list each project's current TargetFramework
along with the latest available LTS version from Microsoft's release schedule.
Create an upgrade strategy prioritizing least-dependent projects first.
```

この upgrade chat mode は、リポジトリ内の現在の .NET version に自動適応し、次の安定版へ向けた文脈依存の upgrade guidance を提供します。

支援内容:
- 全 project の現在の .NET version 自動検出
- 最適な upgrade sequence の生成
- breaking change と modernization opportunity の特定
- project ごとの upgrade flow 作成

---

### .NET Upgrade Instructions

構造化されたガイダンスで包括的な .NET framework upgrade を実行しましょう。

この instruction が提供するもの:
- 順次 upgrade strategy
- dependency analysis と sequencing
- framework targeting と code adjustment
- NuGet と dependency management
- CI/CD pipeline update
- testing と validation procedure

upgrade plan を実装する際にこれらの instruction を使うと、適切な実行と検証を確保できます。

---

### .NET Upgrade Prompts

専用の upgrade analysis prompt へすばやくアクセス。

この prompt collection には、次のための即使用可能な query が含まれます:
- project discovery と assessment
- upgrade strategy と sequencing
- framework targeting と code adjustment
- breaking change analysis
- CI/CD pipeline update
- 最終 validation と delivery

特定の upgrade 側面を対象にした分析に使ってください。

---

## Quick Start
1. discovery pass を実行し、リポジトリ内のすべての `*.sln` と `*.csproj` file を列挙する。
2. 各 project で使われている現在の .NET version を検出する。
3. 利用可能な最新の安定 .NET version（LTS 優先）を特定する。通常は既存 version の `+2` 年先。
4. 現在 → 次の安定 version への upgrade plan を生成する（例: `net6.0 → net8.0`、または `net7.0 → net9.0`）。
5. 1 project ずつ upgrade し、build を検証し、test を更新し、CI/CD を修正する。

---

## 現在の .NET Version を自動検出する
solution 全体の現在 framework version を自動検出するには:

```bash
# 1. インストール済み global SDK を確認
dotnet --list-sdks

# 2. project-level TargetFrameworks を検出
find . -name "*.csproj" -exec grep -H "<TargetFramework" {} \;

# 3. 任意: 一意な framework version を要約
grep -r "<TargetFramework" **/*.csproj | sed 's/.*<TargetFramework>//;s/<\/TargetFramework>//' | sort | uniq

# 4. runtime environment を確認
dotnet --info | grep "Version"
```

**Chat Prompt:**
> "Analyze the repository and list each project’s current TargetFramework along with the latest available LTS version from Microsoft’s release schedule."

---

## Discovery & Analysis Commands
```bash
# List all projects
dotnet sln list

# Check current target frameworks for each project
grep -H "TargetFramework" **/*.csproj

# Check outdated packages
dotnet list <ProjectName>.csproj package --outdated

# Generate dependency graph
dotnet msbuild <ProjectName>.csproj /t:GenerateRestoreGraphFile /p:RestoreGraphOutputPath=graph.json
```

**Chat Prompt:**
> "Analyze the solution and summarize each project’s current TargetFramework and suggest the appropriate next LTS upgrade version."

---

## 分類ルール
- `TargetFramework` が `netcoreapp`、`net5.0+`、`net6.0+` などで始まる → **Modern .NET**
- `netstandard*` → **.NET Standard**（現在の .NET version へ移行）
- `net4*` → **.NET Framework**（中間段階を経由して .NET 8+ へ移行）

---

## Upgrade Sequence
1. **独立ライブラリから始める:** 依存の少ない class library を先に。
2. **次に:** 共通 component と common utility。
3. **その後:** API、Web、Function project。
4. **最後に:** test、integration point、pipeline。

**Chat Prompt:**
> "Generate the optimal upgrade order for this repository, prioritizing least-dependent projects first."

---

## Project ごとの Upgrade Flow
1. **branch 作成:** `upgrade/<project>-to-<targetVersion>`
2. `.csproj` の `<TargetFramework>` を提案 version（例: `net9.0`）へ編集
3. **restore と package update:**
   ```bash
   dotnet restore
   dotnet list package --outdated
   dotnet add package <PackageName> --version <LatestVersion>
   ```
4. **build と test:**
   ```bash
   dotnet build <ProjectName>.csproj
   dotnet test <ProjectName>.Tests.csproj
   ```
5. **問題修正** — deprecated API を解消し、configuration を調整し、JSON/logging/DI を modernize する。
6. test evidence と checklist を付けて PR を commit & push する。

---

## Breaking Changes と Modernization
- 初期提案には `.NET Upgrade Assistant` を使う。
- obsolete API 検出には analyzer を適用する。
- 古い SDK を置き換える（例: `Microsoft.Azure.*` → `Azure.*`）。
- startup logic を modernize する（`Startup.cs` → `Program.cs` top-level statement）。

**Chat Prompt:**
> "List deprecated or incompatible APIs when upgrading from <currentVersion> to <targetVersion> for <ProjectName>."

---

## CI/CD Configuration Updates
pipeline が検出済み **target version** を動的に使うようにしてください:

**Azure DevOps**
```yaml
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '$(TargetDotNetVersion).x'
```

**GitHub Actions**
```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '${{ env.TargetDotNetVersion }}.x'
```

---

## Validation Checklist
- [ ] TargetFramework が次の安定 version へ upgrade されている
- [ ] すべての NuGet package が互換かつ更新済み
- [ ] Build と test pipeline がローカルと CI の両方で成功する
- [ ] Integration test が通る
- [ ] lower environment へ deploy して検証済み

---

## Branching & Rollback Strategy
- feature branch を使う: `upgrade/<project>-to-<targetVersion>`
- 頻繁に commit し、変更は atomic に保つ
- merge 後に CI が失敗したら、PR を revert して failing module を分離する

**Chat Prompt:**
> "Suggest a rollback and validation plan if the .NET upgrade for <ProjectName> introduces build or runtime regressions."

---

## Automation & Scaling
- GitHub Actions や Azure Pipelines で upgrade detection を自動化する。
- `dotnet --list-sdks` による新しい .NET release チェックを nightly で走らせる。
- 古い framework に対して agent が自動的に PR を起票するようにする。

---

## Chatmode Prompt Library
1. "List all projects with current and recommended .NET versions."
2. "Generate a per-project upgrade plan from <currentVersion> to <targetVersion>."
3. "Suggest .csproj and pipeline edits to upgrade <ProjectName>."
4. "Summarize build/test results post-upgrade for <ProjectName>."
5. "Create PR description and checklist for the upgrade."

---
