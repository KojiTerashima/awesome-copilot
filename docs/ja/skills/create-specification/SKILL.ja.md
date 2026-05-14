---
name: create-specification
description: 'Generative AI が利用しやすいよう最適化された、新しい solution 用 specification file を作成します。'
---

# Create Specification

あなたの目標は、`${input:SpecPurpose}` のための新しい specification file を作成することです。

specification file は、solution component の requirement、constraint、interface を、Generative AI が効果的に利用できるよう、明確で曖昧さがなく、構造化された形で定義しなければなりません。確立された documentation standard に従い、content が machine-readable かつ self-contained であることを保証してください。

## Best Practices for AI-Ready Specifications

- 正確で、明示的で、曖昧さのない言語を使う。
- requirement、constraint、recommendation を明確に区別する。
- parsing しやすいよう、構造化された formatting（heading、list、table）を使う。
- idiom、metaphor、文脈依存の表現を避ける。
- すべての acronym と domain-specific term を定義する。
- 必要に応じて example と edge case を含める。
- 文書が self-contained であり、外部文脈に依存しないことを保証する。

specification は [/spec/](/spec/) directory に保存し、`spec-[a-z0-9-]+.md` という命名規則に従ってください。名前は specification の内容を説明的に示し、先頭は [schema, tool, data, infrastructure, process, architecture, or design] のいずれかで始まる highlevel purpose にしてください。

specification file は、整形式の Markdown でなければなりません。

specification file は次の template に従い、すべての section を適切に埋める必要があります。markdown の front matter は、以下の例のとおり正しく構成してください。

```md
---
title: [Specification の焦点を表す簡潔な title]
version: [Optional: 例 1.0、Date]
date_created: [YYYY-MM-DD]
last_updated: [Optional: YYYY-MM-DD]
owner: [Optional: この spec の担当 Team/Individual]
tags: [Optional: 関連する tag や category の一覧。例 `infrastructure`、`process`、`design`、`app` など]
---

# Introduction

[specification の概要と、達成を意図する目標を簡潔に説明する。]

## 1. Purpose & Scope

[specification の目的と適用範囲を、明確かつ簡潔に説明する。想定読者と前提も記載する。]

## 2. Definitions

[この specification で使う acronym、abbreviation、domain-specific term をすべて列挙して定義する。]

## 3. Requirements, Constraints & Guidelines

[すべての requirement、constraint、rule、guideline を明示的に列挙する。明確さのため bullet point や table を使う。]

- **REQ-001**: Requirement 1
- **SEC-001**: Security Requirement 1
- **[3 LETTERS]-001**: Other Requirement 1
- **CON-001**: Constraint 1
- **GUD-001**: Guideline 1
- **PAT-001**: Pattern to follow 1

## 4. Interfaces & Data Contracts

[interface、API、data contract、integration point を記述する。schema や example には table や code block を使う。]

## 5. Acceptance Criteria

[必要に応じて Given-When-Then format を用い、各 requirement に対する明確で test 可能な acceptance criteria を定義する。]

- **AC-001**: Given [context], When [action], Then [expected outcome]
- **AC-002**: The system shall [specific behavior] when [condition]
- **AC-003**: [必要に応じた追加 acceptance criteria]

## 6. Test Automation Strategy

[testing approach、framework、automation requirement を定義する。]

- **Test Levels**: Unit、Integration、End-to-End
- **Frameworks**: MSTest、FluentAssertions、Moq（.NET application の場合）
- **Test Data Management**: [test data 作成と cleanup の approach]
- **CI/CD Integration**: [GitHub Actions pipeline における automated testing]
- **Coverage Requirements**: [最小 code coverage threshold]
- **Performance Testing**: [load/performance testing の approach]

## 7. Rationale & Context

[requirement、constraint、guideline の背景理由を説明する。design decision の context を提供する。]

## 8. Dependencies & External Integrations

[この specification に必要な external system、service、architectural dependency を定義する。実装方法ではなく **what** が必要かに焦点を当てる。architectural constraint でない限り、特定 package や library の version は避ける。]

### External Systems
- **EXT-001**: [外部 system 名] - [目的と integration type]

### Third-Party Services
- **SVC-001**: [service 名] - [必要な capability と SLA requirement]

### Infrastructure Dependencies
- **INF-001**: [infrastructure component] - [requirement と constraint]

### Data Dependencies
- **DAT-001**: [外部 data source] - [format、frequency、access requirement]

### Technology Platform Dependencies
- **PLT-001**: [platform/runtime requirement] - [version constraint と理由]

### Compliance Dependencies
- **COM-001**: [regulatory または compliance requirement] - [実装への影響]

**Note**: この section は architectural/business dependency に焦点を当てるべきで、特定 package の実装詳細は扱いません。たとえば "Microsoft.AspNetCore.Authentication.JwtBearer v6.0.1" ではなく "OAuth 2.0 authentication library" と記述します。

## 9. Examples & Edge Cases

    ```code
    // ガイドラインの正しい適用と edge case を示す code snippet または data example
    ```

## 10. Validation Criteria

[この specification への準拠を満たすための criteria または test を列挙する。]

## 11. Related Specifications / Further Reading

[関連 spec 1 へのリンク]
[関連する外部 documentation へのリンク]

```
