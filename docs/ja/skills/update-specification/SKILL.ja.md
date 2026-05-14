---
name: update-specification
description: 'Update an existing specification file for the solution, optimized for Generative AI consumption based on new requirements or updates to any existing code.'
---
# 仕様の更新

目標は、新しい要件に基づいて既存の仕様ファイル `${file}` を更新するか、既存のコードを更新することです。

仕様ファイルは、ソリューション コンポーネントの要件、制約、インターフェイスを、明確で曖昧さのない方法で定義し、生成 AI が効果的に使用できるように構造化する必要があります。確立された文書標準に従い、コンテンツが機械可読で自己完結型であることを確認してください。

## AI 対応仕様のベスト プラクティス

- 正確で明確かつ明確な言葉を使用します。
- 要件、制約、推奨事項を明確に区別します。
- 解析を容易にするために、構造化された書式設定 (見出し、リスト、表) を使用します。
- 慣用句、比喩、または文脈に依存した参照は避けてください。
- すべての頭字語とドメイン固有の用語を定義します。
- 該当する場合は、例と特殊なケースを含めます。
- ドキュメントが自己完結型であり、外部コンテキストに依存しないことを確認します。

仕様は [/spec/](/spec/) ディレクトリに保存し、`[a-z0-9-]+.md` の規則に従って名前を付ける必要があります。名前は仕様の内容を説明するものであり、[スキーマ、ツール、データ、インフラストラクチャ、プロセス、アーキテクチャ、または設計] の 1 つである高レベルの目的から始まる必要があります。

仕様ファイルは、整形式のマークダウンでフォーマットされている必要があります。

仕様ファイルは以下のテンプレートに従い、すべてのセクションが適切に入力されていることを確認する必要があります。マークダウンの前付けは、次の例のように正しく構成されている必要があります。```md
---
title: [Concise Title Describing the Specification's Focus]
version: [Optional: e.g., 1.0, Date]
date_created: [YYYY-MM-DD]
last_updated: [Optional: YYYY-MM-DD]
owner: [Optional: Team/Individual responsible for this spec]
tags: [Optional: List of relevant tags or categories, e.g., `infrastructure`, `process`, `design`, `app` etc]
---

# Introduction

[A short concise introduction to the specification and the goal it is intended to achieve.]

## 1. Purpose & Scope

[Provide a clear, concise description of the specification's purpose and the scope of its application. State the intended audience and any assumptions.]

## 2. Definitions

[List and define all acronyms, abbreviations, and domain-specific terms used in this specification.]

## 3. Requirements, Constraints & Guidelines

[Explicitly list all requirements, constraints, rules, and guidelines. Use bullet points or tables for clarity.]

- **REQ-001**: Requirement 1
- **SEC-001**: Security Requirement 1
- **[3 LETTERS]-001**: Other Requirement 1
- **CON-001**: Constraint 1
- **GUD-001**: Guideline 1
- **PAT-001**: Pattern to follow 1

## 4. Interfaces & Data Contracts

[Describe the interfaces, APIs, data contracts, or integration points. Use tables or code blocks for schemas and examples.]

## 5. Acceptance Criteria

[Define clear, testable acceptance criteria for each requirement using Given-When-Then format where appropriate.]

- **AC-001**: Given [context], When [action], Then [expected outcome]
- **AC-002**: The system shall [specific behavior] when [condition]
- **AC-003**: [Additional acceptance criteria as needed]

## 6. Test Automation Strategy

[Define the testing approach, frameworks, and automation requirements.]

- **Test Levels**: Unit, Integration, End-to-End
- **Frameworks**: MSTest, FluentAssertions, Moq (for .NET applications)
- **Test Data Management**: [approach for test data creation and cleanup]
- **CI/CD Integration**: [automated testing in GitHub Actions pipelines]
- **Coverage Requirements**: [minimum code coverage thresholds]
- **Performance Testing**: [approach for load and performance testing]

## 7. Rationale & Context

[Explain the reasoning behind the requirements, constraints, and guidelines. Provide context for design decisions.]

## 8. Dependencies & External Integrations

[Define the external systems, services, and architectural dependencies required for this specification. Focus on **what** is needed rather than **how** it's implemented. Avoid specific package or library versions unless they represent architectural constraints.]

### External Systems
- **EXT-001**: [External system name] - [Purpose and integration type]

### Third-Party Services
- **SVC-001**: [Service name] - [Required capabilities and SLA requirements]

### Infrastructure Dependencies
- **INF-001**: [Infrastructure component] - [Requirements and constraints]

### Data Dependencies
- **DAT-001**: [External data source] - [Format, frequency, and access requirements]

### Technology Platform Dependencies
- **PLT-001**: [Platform/runtime requirement] - [Version constraints and rationale]

### Compliance Dependencies
- **COM-001**: [Regulatory or compliance requirement] - [Impact on implementation]

**Note**: This section should focus on architectural and business dependencies, not specific package implementations. For example, specify "OAuth 2.0 authentication library" rather than "Microsoft.AspNetCore.Authentication.JwtBearer v6.0.1".

## 9. Examples & Edge Cases

```コード
// エッジケースを含む、ガイドラインの正しい適用を示すコード スニペットまたはデータ例```

## 10. Validation Criteria

[List the criteria or tests that must be satisfied for compliance with this specification.]

## 11. Related Specifications / Further Reading

[Link to related spec 1]
[Link to relevant external documentation]

```
