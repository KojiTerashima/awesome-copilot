---
name: create-architectural-decision-record
description: 'AI 最適化された意思決定 documentation のための Architectural Decision Record (ADR) document を作成します。'
---

# Create Architectural Decision Record

`${input:DecisionTitle}` に対する ADR document を、AI が扱いやすく人間にも読みやすい構造化 format で作成してください。

## Inputs

- **Context**: `${input:Context}`
- **Decision**: `${input:Decision}`
- **Alternatives**: `${input:Alternatives}`
- **Stakeholders**: `${input:Stakeholders}`

## Input Validation
必要な input が不足している、または会話履歴から判断できない場合は、ADR 生成を進める前に不足情報の提供をユーザーに依頼してください。

## Requirements

- 正確で曖昧さのない表現を使う
- front matter を含む標準 ADR format に従う
- positive consequence と negative consequence の両方を含める
- alternatives と、その不採用理由を記録する
- machine parsing と human reference の両方に適した構造にする
- 複数項目 section では coded bullet point（3-4 文字 code + 3 桁番号）を使う

ADR は `/docs/adr/` directory に、`adr-NNNN-[title-slug].md` という naming convention で保存する必要があります。NNNN は次の連番 4 桁です（例: `adr-0001-database-selection.md`）。

## Required Documentation Structure

documentation file は次の template に従い、すべての section を適切に埋めてください。markdown の front matter は、以下の例のとおり正しく構成する必要があります。

```md
---
title: "ADR-NNNN: [Decision Title]"
status: "Proposed"
date: "YYYY-MM-DD"
authors: "[Stakeholder Names/Roles]"
tags: ["architecture", "decision"]
supersedes: ""
superseded_by: ""
---

# ADR-NNNN: [Decision Title]

## Status

**Proposed** | Accepted | Rejected | Superseded | Deprecated

## Context

[この判断を必要とする問題定義、技術制約、business requirement、環境要因。]

## Decision

[採用した解決策と、その選定理由を明確に記す。]

## Consequences

### Positive

- **POS-001**: [望ましい結果と利点]
- **POS-002**: [performance、maintainability、scalability の改善]
- **POS-003**: [architectural principle との整合]

### Negative

- **NEG-001**: [trade-off、制約、欠点]
- **NEG-002**: [持ち込まれる technical debt や複雑性]
- **NEG-003**: [リスクと将来の課題]

## Alternatives Considered

### [Alternative 1 Name]

- **ALT-001**: **Description**: [技術的な概要]
- **ALT-002**: **Rejection Reason**: [この案を選ばなかった理由]

### [Alternative 2 Name]

- **ALT-003**: **Description**: [技術的な概要]
- **ALT-004**: **Rejection Reason**: [この案を選ばなかった理由]

## Implementation Notes

- **IMP-001**: [主要な実装上の考慮点]
- **IMP-002**: [該当する場合の移行または rollout 戦略]
- **IMP-003**: [monitoring と成功基準]

## References

- **REF-001**: [関連 ADR]
- **REF-002**: [外部 documentation]
- **REF-003**: [参照した標準または framework]
```
