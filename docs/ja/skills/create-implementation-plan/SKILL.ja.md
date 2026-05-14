---
name: create-implementation-plan
description: '新機能、既存コードの refactoring や package upgrade、design、architecture、infrastructure のための新しい implementation plan file を作成します。'
---

# Create Implementation Plan

## Primary Directive

あなたの目標は、`${input:PlanPurpose}` のための新しい implementation plan file を作成することです。出力は、他の AI system または人間が自律的に実行できるよう、machine-readable、deterministic、かつ構造化されていなければなりません。

## Execution Context

この prompt は AI-to-AI communication と automated processing のために設計されています。すべての instruction は文字どおりに解釈し、人間の解釈や追加確認なしで体系的に実行してください。

## Core Requirements

- AI agent または人間が完全に実行可能な implementation plan を生成する
- 曖昧さゼロの deterministic な言語を使う
- すべての content を automated parsing と execution 向けに構造化する
- 理解のために外部依存を必要としない、完全な self-containment を保証する

## Plan Structure Requirements

plan は、実行可能な task を含む離散的かつ atomic な phase で構成しなければなりません。phase 間 dependency が明示されていない限り、各 phase は AI agent または人間が独立して処理できる必要があります。

## Phase Architecture

- 各 phase には測定可能な completion criteria が必要
- dependency が指定されていない限り、phase 内 task は並列実行可能であること
- すべての task description に具体的な file path、function 名、正確な実装詳細を含める
- どの task も人間の解釈や判断を必要としてはならない

## AI-Optimized Implementation Standards

- 解釈不要で曖昧さゼロの明示的な言語を使う
- すべての content を machine-parseable format（table、list、structured data）として構造化する
- 必要に応じて具体的な file path、line number、正確な code reference を含める
- すべての variable、constant、configuration value を明示的に定義する
- 各 task description 内に完全な context を含める
- すべての identifier に標準 prefix（REQ-、TASK- など）を使う
- 自動検証可能な validation criteria を含める

## Output File Specifications

- implementation plan file は `/plan/` directory に保存する
- 命名規則は `[purpose]-[component]-[version].md`
- purpose prefix: `upgrade|refactor|feature|data|infrastructure|process|architecture|design`
- 例: `upgrade-system-command-4.md`、`feature-auth-module-1.md`
- file は適切な front matter structure を持つ valid Markdown であること

## Mandatory Template Structure

すべての implementation plan は、以下の template に厳密に従わなければなりません。各 section は必須であり、具体的で実行可能な content で埋める必要があります。AI agent は実行前に template compliance を検証しなければなりません。

## Template Validation Rules

- すべての front matter field が存在し、正しく format されていること
- すべての section header が完全一致すること（case-sensitive）
- すべての identifier prefix が指定 format に従うこと
- table が必要な列をすべて含むこと
- 最終出力に placeholder text が残らないこと

## Status

implementation plan の status は front matter で明確に定義し、plan の現在状態を反映していなければなりません。status に指定できる値は次のいずれかです（括弧内は status_color）: `Completed`（明るい緑 badge）、`In progress`（黄 badge）、`Planned`（青 badge）、`Deprecated`（赤 badge）、`On Hold`（橙 badge）。さらに introduction section に badge として表示してください。

```md
---
goal: [Package Implementation Plan の目標を表す簡潔な title]
version: [Optional: 例 1.0、Date]
date_created: [YYYY-MM-DD]
last_updated: [Optional: YYYY-MM-DD]
owner: [Optional: この spec の担当 Team/Individual]
status: 'Completed'|'In progress'|'Planned'|'Deprecated'|'On Hold'
tags: [Optional: 関連する tag や category の一覧。例 `feature`、`upgrade`、`chore`、`architecture`、`migration`、`bug` など]
---

# Introduction

![Status: <status>](https://img.shields.io/badge/status-<status>-<status_color>)

[plan の概要と、達成を意図する目標を簡潔に説明する。]

## 1. Requirements & Constraints

[plan に影響し、実装方法を制約する requirement と constraint を明示的に列挙する。明確さのため bullet point や table を使う。]

- **REQ-001**: Requirement 1
- **SEC-001**: Security Requirement 1
- **[3 LETTERS]-001**: Other Requirement 1
- **CON-001**: Constraint 1
- **GUD-001**: Guideline 1
- **PAT-001**: Pattern to follow 1

## 2. Implementation Steps

### Implementation Phase 1

- GOAL-001: [この phase の goal を記述する。例: "Implement feature X"、"Refactor module Y" など]

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-001 | task 1 の説明 | ✅ | 2025-04-25 |
| TASK-002 | task 2 の説明 | |  |
| TASK-003 | task 3 の説明 | |  |

### Implementation Phase 2

- GOAL-002: [この phase の goal を記述する。例: "Implement feature X"、"Refactor module Y" など]

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-004 | task 4 の説明 | |  |
| TASK-005 | task 5 の説明 | |  |
| TASK-006 | task 6 の説明 | |  |

## 3. Alternatives

[検討した代替アプローチと、それを選ばなかった理由を bullet point で列挙する。これは採用した approach の context と rationale を提供するためのもの。]

- **ALT-001**: Alternative approach 1
- **ALT-002**: Alternative approach 2

## 4. Dependencies

[plan が依存する library、framework、その他 component など、対応が必要な dependency を列挙する。]

- **DEP-001**: Dependency 1
- **DEP-002**: Dependency 2

## 5. Files

[feature または refactoring task の影響を受ける file を列挙する。]

- **FILE-001**: file 1 の説明
- **FILE-002**: file 2 の説明

## 6. Testing

[feature または refactoring task を検証するために実装すべき test を列挙する。]

- **TEST-001**: test 1 の説明
- **TEST-002**: test 2 の説明

## 7. Risks & Assumptions

[plan の実装に関する risk や assumption を列挙する。]

- **RISK-001**: Risk 1
- **ASSUMPTION-001**: Assumption 1

## 8. Related Specifications / Further Reading

[関連 spec 1 へのリンク]
[関連する外部 documentation へのリンク]
```
