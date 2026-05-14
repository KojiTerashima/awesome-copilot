---
name: copilot-instructions-blueprint-generator
description: '既存 codebase の pattern を分析し、推測を避けながら、project の standard、architecture pattern、正確な technology version に沿って GitHub Copilot を導く包括的な copilot-instructions.md file を作成するための、技術非依存 blueprint generator。'
---

# Copilot Instructions Blueprint Generator

## Configuration Variables
${PROJECT_TYPE="Auto-detect|.NET|Java|JavaScript|TypeScript|React|Angular|Python|Multiple|Other"} <!-- 主要技術 -->
${ARCHITECTURE_STYLE="Layered|Microservices|Monolithic|Domain-Driven|Event-Driven|Serverless|Mixed"} <!-- Architecture の方向性 -->
${CODE_QUALITY_FOCUS="Maintainability|Performance|Security|Accessibility|Testability|All"} <!-- 品質上の優先事項 -->
${DOCUMENTATION_LEVEL="Minimal|Standard|Comprehensive"} <!-- Documentation 要件 -->
${TESTING_REQUIREMENTS="Unit|Integration|E2E|TDD|BDD|All"} <!-- Testing の方針 -->
${VERSIONING="Semantic|CalVer|Custom"} <!-- Versioning の方針 -->

## Generated Prompt

"GitHub Copilot がこの project の standard、architecture、technology version に整合した code を生成できるよう導く、包括的な copilot-instructions.md file を生成してください。instruction は、実際の codebase pattern に厳密に基づき、推測を避ける必要があります。次の approach に従ってください。

### 1. Core Instruction Structure

```markdown
# GitHub Copilot Instructions

## Priority Guidelines

この repository 向けに code を生成するときは、次を守ること:

1. **Version Compatibility**: この project で使われている言語、framework、library の正確な version を必ず検出して尊重する
2. **Context Files**: `.github/copilot` directory で定義されている pattern と standard を優先する
3. **Codebase Patterns**: context file に具体的 guidance がない場合は、codebase を走査して既存 pattern を見つける
4. **Architectural Consistency**: ${ARCHITECTURE_STYLE} の architecture style と、確立済みの boundary を維持する
5. **Code Quality**: 生成するすべての code で ${CODE_QUALITY_FOCUS == "All" ? "maintainability、performance、security、accessibility、testability" : CODE_QUALITY_FOCUS} を優先する

## Technology Version Detection

code を生成する前に、codebase を走査して次を特定する:

1. **Language Versions**: 使用中の programming language の正確な version を検出する
   - project file、configuration file、package manager を確認する
   - 言語固有の version indicator（例: .NET project の `<LangVersion>`）を探す
   - 検出した version を超える language feature は使わない

2. **Framework Versions**: すべての framework の正確な version を特定する
   - package.json、.csproj、pom.xml、requirements.txt などを確認する
   - code 生成時は version constraint を尊重する
   - 検出した framework version に存在しない feature は提案しない

3. **Library Versions**: 主要 library と dependency の正確な version を把握する
   - これらの specific version に適合する code を生成する
   - 検出した version に存在しない API や feature は使わない

## Context Files

`.github/copilot` directory に次の file があれば優先する:

- **architecture.md**: system architecture guideline
- **tech-stack.md**: technology version と framework の詳細
- **coding-standards.md**: code style と formatting standard
- **folder-structure.md**: project organization guideline
- **exemplars.md**: 従うべき exemplar code pattern

## Codebase Scanning Instructions

context file に具体的 guidance がない場合:

1. 修正または作成する file に近い file を見つける
2. 次の pattern を分析する:
   - naming convention
   - code organization
   - error handling
   - logging approach
   - documentation style
   - testing pattern

3. codebase で最も一貫している pattern に従う
4. pattern が競合する場合は、より新しい file、または test coverage が高い file の pattern を優先する
5. 既存 codebase に存在しない pattern を導入しない

## Code Quality Standards

${CODE_QUALITY_FOCUS.includes("Maintainability") || CODE_QUALITY_FOCUS == "All" ? `### Maintainability
- 自己文書化される、明確な naming の code を書く
- codebase に見られる naming と organization の規約に従う
- 一貫性のため既存 pattern に従う
- function は単一責務に集中させる
- function の複雑さと長さは既存 pattern に合わせる` : ""}

${CODE_QUALITY_FOCUS.includes("Performance") || CODE_QUALITY_FOCUS == "All" ? `### Performance
- memory と resource management では既存 pattern に従う
- 計算コストの高い処理では codebase に見られる pattern に合わせる
- async operation では確立済みの pattern に従う
- caching は既存 pattern に整合する形で適用する
- codebase に見られる pattern に基づいて最適化する` : ""}

${CODE_QUALITY_FOCUS.includes("Security") || CODE_QUALITY_FOCUS == "All" ? `### Security
- input validation は既存 pattern に従う
- codebase で使われている sanitization 手法を適用する
- parameterized query は既存 pattern と同じ形で使う
- authentication と authorization は確立済み pattern に従う
- sensitive data は既存 pattern に沿って扱う` : ""}

${CODE_QUALITY_FOCUS.includes("Accessibility") || CODE_QUALITY_FOCUS == "All" ? `### Accessibility
- codebase にある accessibility pattern に従う
- ARIA attribute の使い方は既存 component に合わせる
- keyboard navigation support は既存 code と同じ水準で維持する
- color と contrast は確立済み pattern に従う
- text alternative は codebase と一貫した pattern を使う` : ""}

${CODE_QUALITY_FOCUS.includes("Testability") || CODE_QUALITY_FOCUS == "All" ? `### Testability
- testable な code に関する既存 pattern に従う
- dependency injection の approach は codebase で使われているものに合わせる
- dependency 管理は同じ pattern を適用する
- mocking と test double は既存 pattern に従う
- testing style は既存 test に合わせる` : ""}

## Documentation Requirements

${DOCUMENTATION_LEVEL == "Minimal" ?
`- 既存 code にある comment の量と style に合わせる
- codebase で観測された pattern に従って documentation を書く
- 自明でない挙動の説明は既存 pattern に従う
- parameter description の format は既存 code と同じにする` : ""}

${DOCUMENTATION_LEVEL == "Standard" ?
`- codebase にある documentation format に正確に従う
- XML/JSDoc style と comment の充実度を既存 code に合わせる
- parameter、return、exception の記述を同じ style で書く
- usage example は既存 pattern に従う
- class-level documentation の style と内容を合わせる` : ""}

${DOCUMENTATION_LEVEL == "Comprehensive" ?
`- codebase で見つかる中でもっとも詳細な documentation pattern に従う
- 最もよく文書化された code の style と充実度に合わせる
- documentation は、十分に整備された既存 file と同じ水準で書く
- documentation link は既存 pattern に従う
- design decision の説明は既存 code と同じレベルの詳細さに合わせる` : ""}

## Testing Approach

${TESTING_REQUIREMENTS.includes("Unit") || TESTING_REQUIREMENTS == "All" ?
`### Unit Testing
- existing unit test の構造と style に正確に合わせる
- test class と method の naming convention は既存 code に従う
- assertion pattern は既存 test に合わせる
- mocking approach は codebase のものを使う
- test isolation に関する既存 pattern に従う` : ""}

${TESTING_REQUIREMENTS.includes("Integration") || TESTING_REQUIREMENTS == "All" ?
`### Integration Testing
- codebase にある integration test pattern に従う
- test data の setup と teardown は既存 pattern に合わせる
- component 間 interaction の test は同じ approach を使う
- system behavior の検証方法は既存 pattern に従う` : ""}

${TESTING_REQUIREMENTS.includes("E2E") || TESTING_REQUIREMENTS == "All" ?
`### End-to-End Testing
- 既存の E2E test の構造と pattern に合わせる
- UI testing は確立済み pattern に従う
- user journey の検証方法は既存の approach を使う` : ""}

${TESTING_REQUIREMENTS.includes("TDD") || TESTING_REQUIREMENTS == "All" ?
`### Test-Driven Development
- codebase に見られる TDD pattern に従う
- 既存 code に見られる test case の進め方に合わせる
- test 通過後の refactoring pattern も既存 code に合わせる` : ""}

${TESTING_REQUIREMENTS.includes("BDD") || TESTING_REQUIREMENTS == "All" ?
`### Behavior-Driven Development
- test にある Given-When-Then structure に合わせる
- 振る舞いの記述方法は既存 pattern に従う
- business 観点の強さは既存 test と同じ水準に保つ` : ""}

## Technology-Specific Guidelines

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### .NET Guidelines
- 使用中の .NET version を検出し、厳密に従う
- 検出した version に対応する C# language feature だけを使う
- LINQ の使い方は codebase にある pattern に正確に合わせる
- async/await の使い方は既存 code に合わせる
- dependency injection の approach は codebase のものを適用する
- collection type とその使い方は既存 code に合わせる` : ""}

${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Java Guidelines
- 使用中の Java version を検出し、それに従う
- design pattern は codebase にあるものと同じものを使う
- exception handling は既存 code の pattern に合わせる
- collection type と approach は codebase にあるものを使う
- dependency injection pattern は既存 code に見られるものに従う` : ""}

${PROJECT_TYPE == "JavaScript" || PROJECT_TYPE == "TypeScript" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### JavaScript/TypeScript Guidelines
- 使用中の ECMAScript/TypeScript version を検出し、それに従う
- module import/export pattern は codebase にあるものと同じにする
- TypeScript の型定義は既存 pattern に合わせる
- async pattern（promise、async/await）は既存 code と同じものを使う
- error handling は類似 file の pattern に従う` : ""}

${PROJECT_TYPE == "React" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### React Guidelines
- 使用中の React version を検出し、それに従う
- component structure は既存 component の pattern に合わせる
- hooks と lifecycle の使い方は codebase にある pattern に従う
- state management は既存 component で使われている approach を使う
- prop typing と validation は既存 code の pattern に合わせる` : ""}

${PROJECT_TYPE == "Angular" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Angular Guidelines
- 使用中の Angular version を検出し、それに従う
- component と module の pattern は codebase にあるものに合わせる
- decorator の使い方は既存 code に見られる形に正確に合わせる
- RxJS pattern は codebase にあるものを適用する
- component 間 communication は既存 pattern に従う` : ""}

${PROJECT_TYPE == "Python" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Python Guidelines
- 使用中の Python version を検出し、それに従う
- import の並びと organization は既存 module に合わせる
- type hinting を使っているなら、その approach を既存 code に合わせる
- error handling pattern は既存 code に従う
- module organization は既存 pattern に合わせる` : ""}

## Version Control Guidelines

${VERSIONING == "Semantic" ?
`- codebase に適用されている Semantic Versioning pattern に従う
- breaking change の文書化は既存 pattern に合わせる
- deprecation notice の出し方は既存 approach に従う` : ""}

${VERSIONING == "CalVer" ?
`- codebase に適用されている Calendar Versioning pattern に従う
- change の文書化は既存 pattern に合わせる
- 重要 change の強調方法は既存 approach に従う` : ""}

${VERSIONING == "Custom" ?
`- codebase に見られる正確な versioning pattern に合わせる
- changelog format は既存 documentation と同じにする
- tagging convention は project で使われているものに従う` : ""}

## General Best Practices

- naming convention は既存 code に見られるものをそのまま使う
- code organization pattern は類似 file に合わせる
- error handling は既存 pattern と一貫させる
- testing への approach は既存 code に見られるものに合わせる
- logging pattern は既存 code に合わせる
- configuration の扱い方は既存 code に見られるものに従う

## Project-Specific Guidance

- code を生成する前に、必ず codebase を十分に走査する
- 既存の architectural boundary は例外なく守る
- 周囲の code の style と pattern に合わせる
- 迷ったときは、外部の best practice や新しい language feature より、既存 code との一貫性を優先する
```

### 2. Codebase Analysis Instructions

copilot-instructions.md file を作成するために、まず codebase を分析して次を把握してください。

1. **Identify Exact Technology Versions**:
   - ${PROJECT_TYPE == "Auto-detect" ? "file extension と configuration file を走査して、すべての programming language、framework、library を検出する" : `Focus on ${PROJECT_TYPE} technologies`}
   - project file、package.json、.csproj などから precise な version 情報を抽出する
   - version constraint と互換性 requirement を文書化する

2. **Understand Architecture**:
   - folder structure と module organization を分析する
   - 明確な layer boundary と component relationship を特定する
   - component 間の communication pattern を文書化する

3. **Document Code Patterns**:
   - 各種 code 要素の naming convention を整理する
   - documentation style とその充実度を把握する
   - error handling pattern を文書化する
   - testing approach と coverage を整理する

4. **Note Quality Standards**:
   - 実際に使われている performance optimization technique を特定する
   - code に実装されている security practice を文書化する
   - accessibility feature があれば把握する
   - codebase に見られる code quality pattern を文書化する

### 3. Implementation Notes

最終的な copilot-instructions.md は次を満たす必要があります。
- `.github/copilot` directory に配置する
- codebase に実際に存在する pattern と standard だけを参照する
- version compatibility requirement を明示する
- codebase に見られない practice を規定しない
- codebase からの具体例を含める
- 包括的でありながら、Copilot が使いやすい程度に簡潔である

重要: guidance には、実際に codebase で観測された pattern だけを含めてください。既存 code との一貫性を、外部の best practice や新しい language feature より優先するよう、Copilot に明示的に指示してください。
"

## Expected Output

既存の technology version と完全に互換性があり、確立済みの pattern と architecture に従う code を GitHub Copilot が生成できるよう導く、包括的な copilot-instructions.md file。
