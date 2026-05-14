---
name: update-implementation-plan
description: 'Update an existing implementation plan file with new or update requirements to provide new features, refactoring existing code or upgrading packages, design, architecture or infrastructure.'
---
# 実装計画の更新

## 主なディレクティブ

あなたは、新規または更新された要件に基づいて実装計画ファイル `${file}` を更新する任務を負った AI エージェントです。出力は機械可読で、決定論的であり、他の AI システムや人間による自律実行のために構造化されている必要があります。

## 実行コンテキスト

このプロンプトは、AI 間の通信と自動処理のために設計されています。すべての指示は文字通りに解釈され、人間による解釈や説明なしに体系的に実行される必要があります。

## コア要件

- AI エージェントまたは人間によって完全に実行可能な実装計画を生成します
- 曖昧さのない決定論的な言語を使用する
- 自動解析と実行のためにすべてのコンテンツを構造化する
- 理解のための外部依存がない完全な自己完結型を確保します。

## 計画構造の要件

計画は、実行可能なタスクを含む個別のアトミックなフェーズで構成されている必要があります。各フェーズは、明示的に宣言されない限り、フェーズ間の依存関係なしに AI エージェントまたは人間によって独立して処理可能でなければなりません。

## フェーズアーキテクチャ

- 各フェーズには測定可能な完了基準が必要です
- 依存関係が指定されていない限り、フェーズ内のタスクは並列実行可能である必要があります
- すべてのタスクの説明には、特定のファイル パス、関数名、および正確な実装の詳細が含まれている必要があります
- 人間による解釈や意思決定を必要とするタスクがあってはなりません

## AI に最適化された実装基準

- 解釈を必要とせず、明示的で明確な言語を使用する
- すべてのコンテンツを機械解析可能な形式 (テーブル、リスト、構造化データ) として構造化します。
- 該当する場合、特定のファイル パス、行番号、正確なコード参照を含めます。
- すべての変数、定数、構成値を明示的に定義します。
- 各タスクの説明内に完全なコンテキストを提供します
- すべての識別子 (REQ-、TASK- など) に標準化されたプレフィックスを使用します。
- 自動的に検証できる検証基準を含める

## 出力ファイルの仕様

- 実装計画ファイルを `/plan/` ディレクトリに保存します
- 命名規則を使用します: `[purpose]-[component]-[version].md`
- 目的の接頭辞: `upgrade|refactor|feature|data|infrastructure|process|architecture|design`
- 例: `upgrade-system-command-4.md`、`feature-auth-module-1.md`
- ファイルは適切な前付構造を備えた有効なマークダウンである必要があります

## 必須のテンプレート構造すべての実装計画は、次のテンプレートに厳密に従う必要があります。各セクションは必須であり、具体的で実用的なコンテンツを入力する必要があります。 AI エージェントは実行前にテンプレートのコンプライアンスを検証する必要があります。

## テンプレート検証ルール

- すべての前付けフィールドが存在し、適切にフォーマットされている必要があります
- すべてのセクションヘッダーは正確に一致する必要があります (大文字と小文字が区別されます)。
- すべての識別子のプレフィックスは、指定された形式に従う必要があります
- テーブルには必要な列がすべて含まれている必要があります
- 最終出力にプレースホルダー テキストを残すことはできません

## ステータス

実施計画のステータスは前付で明確に定義し、計画の現在の状態を反映する必要があります。ステータスは次のいずれかになります (括弧内の status_color): `Completed` (明るい緑色のバッジ)、`In progress` (黄色のバッジ)、`Planned` (青色のバッジ)、`Deprecated` (赤色のバッジ)、または `On Hold` (オレンジ色のバッジ)。導入セクションにもバッジとして表示される必要があります。```md
---
goal: [Concise Title Describing the Package Implementation Plan's Goal]
version: [Optional: e.g., 1.0, Date]
date_created: [YYYY-MM-DD]
last_updated: [Optional: YYYY-MM-DD]
owner: [Optional: Team/Individual responsible for this spec]
status: 'Completed'|'In progress'|'Planned'|'Deprecated'|'On Hold'
tags: [Optional: List of relevant tags or categories, e.g., `feature`, `upgrade`, `chore`, `architecture`, `migration`, `bug` etc]
---

# Introduction

![Status: <status>](https://img.shields.io/badge/status-<status>-<status_color>)

[A short concise introduction to the plan and the goal it is intended to achieve.]

## 1. Requirements & Constraints

[Explicitly list all requirements & constraints that affect the plan and constrain how it is implemented. Use bullet points or tables for clarity.]

- **REQ-001**: Requirement 1
- **SEC-001**: Security Requirement 1
- **[3 LETTERS]-001**: Other Requirement 1
- **CON-001**: Constraint 1
- **GUD-001**: Guideline 1
- **PAT-001**: Pattern to follow 1

## 2. Implementation Steps

### Implementation Phase 1

- GOAL-001: [Describe the goal of this phase, e.g., "Implement feature X", "Refactor module Y", etc.]

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-001 | Description of task 1 | ✅ | 2025-04-25 |
| TASK-002 | Description of task 2 | |  |
| TASK-003 | Description of task 3 | |  |

### Implementation Phase 2

- GOAL-002: [Describe the goal of this phase, e.g., "Implement feature X", "Refactor module Y", etc.]

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-004 | Description of task 4 | |  |
| TASK-005 | Description of task 5 | |  |
| TASK-006 | Description of task 6 | |  |

## 3. Alternatives

[A bullet point list of any alternative approaches that were considered and why they were not chosen. This helps to provide context and rationale for the chosen approach.]

- **ALT-001**: Alternative approach 1
- **ALT-002**: Alternative approach 2

## 4. Dependencies

[List any dependencies that need to be addressed, such as libraries, frameworks, or other components that the plan relies on.]

- **DEP-001**: Dependency 1
- **DEP-002**: Dependency 2

## 5. Files

[List the files that will be affected by the feature or refactoring task.]

- **FILE-001**: Description of file 1
- **FILE-002**: Description of file 2

## 6. Testing

[List the tests that need to be implemented to verify the feature or refactoring task.]

- **TEST-001**: Description of test 1
- **TEST-002**: Description of test 2

## 7. Risks & Assumptions

[List any risks or assumptions related to the implementation of the plan.]

- **RISK-001**: Risk 1
- **ASSUMPTION-001**: Assumption 1

## 8. Related Specifications / Further Reading

[Link to related spec 1]
[Link to relevant external documentation]
```
