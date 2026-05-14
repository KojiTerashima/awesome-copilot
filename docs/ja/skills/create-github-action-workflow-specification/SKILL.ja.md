---
name: create-github-action-workflow-specification
description: 'Create a formal specification for an existing GitHub Actions CI/CD workflow, optimized for AI consumption and workflow maintenance.'
---
# GitHub アクションのワークフロー仕様を作成する

GitHub Actions ワークフローの包括的な仕様を作成します: `${input:WorkflowFile}`。

この仕様は、ワークフローの動作、要件、および制約の仕様として機能します。実装に依存せず、ワークフローが**どのように**実装されるかではなく、ワークフローが**何を達成するか**に重点を置く必要があります。

## AI に最適化された要件

- **トークンの効率**: 明瞭さを犠牲にすることなく簡潔な言葉を使用します。
- **構造化データ**: 表、リスト、図を活用して緻密な情報を得る
- **意味の明確さ**: 全体を通して一貫して正確な用語を使用します。
- **実装の抽象化**: 特定の構文、コマンド、またはツールのバージョンを避ける
- **保守性**: ワークフローの進化に応じて簡単に更新できるように設計

## 仕様テンプレート

名前を付けて保存: `/spec/spec-process-cicd-[workflow-name].md````md
---
title: CI/CD Workflow Specification - [Workflow Name]
version: 1.0
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
owner: DevOps Team
tags: [process, cicd, github-actions, automation, [domain-specific-tags]]
---

## Workflow Overview

**Purpose**: [One sentence describing workflow's primary goal]
**Trigger Events**: [List trigger conditions]
**Target Environments**: [Environment scope]

## Execution Flow Diagram

```人魚
グラフTD
    A[トリガーイベント] --> B[ジョブ1]
    B --> C[ジョブ 2]
    C --> D[ジョブ 3]
    D --> E[終了]
    
    B --> F[並列ジョブ]
    F --> D
    
    スタイル A 塗りつぶし:#e1f5fe
    スタイル E 塗りつぶし:#e8f5e8```

## Jobs & Dependencies

| Job Name | Purpose | Dependencies | Execution Context |
|----------|---------|--------------|-------------------|
| job-1 | [Purpose] | [Prerequisites] | [Runner/Environment] |
| job-2 | [Purpose] | job-1 | [Runner/Environment] |

## Requirements Matrix

### Functional Requirements
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|-------------------|
| REQ-001 | [Requirement] | High | [Testable criteria] |
| REQ-002 | [Requirement] | Medium | [Testable criteria] |

### Security Requirements
| ID | Requirement | Implementation Constraint |
|----|-------------|---------------------------|
| SEC-001 | [Security requirement] | [Constraint description] |

### Performance Requirements
| ID | Metric | Target | Measurement Method |
|----|-------|--------|-------------------|
| PERF-001 | [Metric] | [Target value] | [How measured] |

## Input/Output Contracts

### Inputs

```ヤムル
# 環境変数
ENV_VAR_1: 文字列 # 目的: [説明]
ENV_VAR_2: シークレット # 目的: [説明]

# リポジトリトリガー
paths: [パスフィルターのリスト]
分岐: [分岐パターンのリスト]```

### Outputs

```ヤムル
# ジョブ出力
job_1_output: string # 説明: [目的]
build_artifact: ファイル # 説明: [コンテンツ タイプ]```

### Secrets & Variables

| Type | Name | Purpose | Scope |
|------|------|---------|-------|
| Secret | SECRET_1 | [Purpose] | Workflow |
| Variable | VAR_1 | [Purpose] | Repository |

## Execution Constraints

### Runtime Constraints

- **Timeout**: [Maximum execution time]
- **Concurrency**: [Parallel execution limits]
- **Resource Limits**: [Memory/CPU constraints]

### Environmental Constraints

- **Runner Requirements**: [OS/hardware needs]
- **Network Access**: [External connectivity needs]
- **Permissions**: [Required access levels]

## Error Handling Strategy

| Error Type | Response | Recovery Action |
|------------|----------|-----------------|
| Build Failure | [Response] | [Recovery steps] |
| Test Failure | [Response] | [Recovery steps] |
| Deployment Failure | [Response] | [Recovery steps] |

## Quality Gates

### Gate Definitions

| Gate | Criteria | Bypass Conditions |
|------|----------|-------------------|
| Code Quality | [Standards] | [When allowed] |
| Security Scan | [Thresholds] | [When allowed] |
| Test Coverage | [Percentage] | [When allowed] |

## Monitoring & Observability

### Key Metrics

- **Success Rate**: [Target percentage]
- **Execution Time**: [Target duration]
- **Resource Usage**: [Monitoring approach]

### Alerting

| Condition | Severity | Notification Target |
|-----------|----------|-------------------|
| [Condition] | [Level] | [Who/Where] |

## Integration Points

### External Systems

| System | Integration Type | Data Exchange | SLA Requirements |
|--------|------------------|---------------|------------------|
| [System] | [Type] | [Data format] | [Requirements] |

### Dependent Workflows

| Workflow | Relationship | Trigger Mechanism |
|----------|--------------|-------------------|
| [Workflow] | [Type] | [How triggered] |

## Compliance & Governance

### Audit Requirements

- **Execution Logs**: [Retention policy]
- **Approval Gates**: [Required approvals]
- **Change Control**: [Update process]

### Security Controls

- **Access Control**: [Permission model]
- **Secret Management**: [Rotation policy]
- **Vulnerability Scanning**: [Scan frequency]

## Edge Cases & Exceptions

### Scenario Matrix

| Scenario | Expected Behavior | Validation Method |
|----------|-------------------|-------------------|
| [Edge case] | [Behavior] | [How to verify] |

## Validation Criteria

### Workflow Validation

- **VLD-001**: [Validation rule]
- **VLD-002**: [Validation rule]

### Performance Benchmarks

- **PERF-001**: [Benchmark criteria]
- **PERF-002**: [Benchmark criteria]

## Change Management

### Update Process

1. **Specification Update**: Modify this document first
2. **Review & Approval**: [Approval process]
3. **Implementation**: Apply changes to workflow
4. **Testing**: [Validation approach]
5. **Deployment**: [Release process]

### Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | [Date] | Initial specification | [Author] |

## Related Specifications

- [Link to related workflow specs]
- [Link to infrastructure specs]
- [Link to deployment specs]

```## 分析手順

ワークフロー ファイルを分析する場合:

1. **中心的な目的の抽出**: 主要なビジネス目標を特定します
2. **ジョブ フローのマップ**: 実行順序を示す依存関係グラフを作成します。
3. **契約の特定**: 入力、出力、インターフェイスを文書化する
4. **キャプチャ制約**: タイムアウト、権限、および制限を抽出します。
5. **品質ゲートの定義**: 検証ポイントと承認ポイントを特定する
6. **エラー パスを文書化**: 障害シナリオとリカバリをマップする
7. **抽象実装**: 構文ではなく動作に焦点を当てる

## マーメイドダイアグラムのガイドライン

### フローの種類
- **シーケンシャル**: `A --> B --> C`
- **パラレル**: `A --> B & A --> C; B --> D & C --> D`
- **条件付き**: `A --> B{Decision}; B -->|Yes| C; B -->|No| D`

### スタイリング```mermaid
style TriggerNode fill:#e1f5fe
style SuccessNode fill:#e8f5e8
style FailureNode fill:#ffebee
style ProcessNode fill:#f3e5f5
```### 複雑なワークフロー
5 つ以上のジョブを含むワークフローの場合は、サブグラフを使用します。```mermaid
graph TD
    subgraph "Build Phase"
        A[Lint] --> B[Test] --> C[Build]
    end
    subgraph "Deploy Phase"  
        D[Staging] --> E[Production]
    end
    C --> D
```## トークンの最適化戦略

1. **表を使用**: 構造化された形式での高密度の情報
2. **一貫して省略**: 一度定義すれば、全体で使用します
3. **箇条書き**: 散文的な段落は避ける
4. **コード ブロック**: ナラティブよりも構造化データ
5. **相互参照**: 情報を繰り返すのではなくリンクする

ドキュメントとワークフロー更新のテンプレートの両方として機能する仕様の作成に重点を置きます。