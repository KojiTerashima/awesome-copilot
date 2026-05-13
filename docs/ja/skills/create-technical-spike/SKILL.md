---
name: create-technical-spike
description: '実装前に解決すべき重要な開発判断を調査するため、時間制限付きの technical spike 文書を作成します。'
---

# Technical Spike 文書を作成する

開発を進める前に答えを出す必要がある、重要な技術的問いに対して、time-box 付きの technical spike 文書を作成してください。各 spike は、明確な成果物と timeline を持つ、特定の技術判断に集中します。

## 文書構成

`${input:FolderPath|docs/spikes}` directory に個別 file を作成します。file 名は `[category]-[short-description]-spike.md` の pattern を使います（例: `api-copilot-integration-spike.md`、`performance-realtime-audio-spike.md`）。

```md
---
title: "${input:SpikeTitle}"
category: "${input:Category|Technical}"
status: "🔴 Not Started"
priority: "${input:Priority|High}"
timebox: "${input:Timebox|1 week}"
created: [YYYY-MM-DD]
updated: [YYYY-MM-DD]
owner: "${input:Owner}"
tags: ["technical-spike", "${input:Category|technical}", "research"]
---

# ${input:SpikeTitle}

## Summary

**Spike Objective:** [解決が必要な、明確で具体的な問いまたは判断]

**Why This Matters:** [開発上または architecture 上の判断にどう影響するか]

**Timebox:** [この spike に割り当てる時間]

**Decision Deadline:** [開発のブロッカーにならないために、いつまでに解決する必要があるか]

## Research Question(s)

**Primary Question:** [答えを出す必要がある主たる技術的問い]

**Secondary Questions:**

- [関連する問い 1]
- [関連する問い 2]
- [関連する問い 3]

## Investigation Plan

### Research Tasks

- [ ] [具体的な調査 task 1]
- [ ] [具体的な調査 task 2]
- [ ] [具体的な調査 task 3]
- [ ] [proof of concept / prototype を作成する]
- [ ] [調査結果と recommendation を文書化する]

### Success Criteria

**この spike は次の条件を満たしたら完了です:**

- [ ] [具体的な基準 1]
- [ ] [具体的な基準 2]
- [ ] [明確な recommendation が文書化されている]
- [ ] [proof of concept が完了している（必要な場合）]

## Technical Context

**Related Components:** [この判断の影響を受ける system component を列挙する]

**Dependencies:** [この判断の解決に依存する、他の spike や判断]

**Constraints:** [solution に影響する既知の制約や requirement]

## Research Findings

### Investigation Results

[調査結果、test 結果、収集した証拠を記録する]

### Prototype/Testing Notes

[prototype、spike、技術検証の結果を記録する]

### External Resources

- [関連 documentation への link]
- [API reference への link]
- [community discussion への link]
- [example/tutorial への link]

## Decision

### Recommendation

[調査結果に基づく明確な recommendation]

### Rationale

[なぜ他の選択肢ではなく、この approach を選んだのか]

### Implementation Notes

[実装時の重要な考慮点]

### Follow-up Actions

- [ ] [action item 1]
- [ ] [action item 2]
- [ ] [architecture document を更新する]
- [ ] [implementation task を作成する]

## Status History

| Date   | Status         | Notes                      |
| ------ | -------------- | -------------------------- |
| [Date] | 🔴 Not Started | Spike を作成し、scope を定義した |
| [Date] | 🟡 In Progress | 調査を開始した |
| [Date] | 🟢 Complete    | [解決内容の要約] |

---

_Last updated: [Date] by [Name]_
```

## Technical Spike のカテゴリ

### API Integration

- third-party API の capability と limitation
- integration pattern と authentication
- rate limit と performance 特性

### Architecture & Design

- system architecture の判断
- design pattern の適用可否
- component 間 interaction model

### Performance & Scalability

- performance requirement と制約
- scalability の bottleneck と解決策
- resource utilization pattern

### Platform & Infrastructure

- platform の capability と limitation
- infrastructure requirement
- deployment と hosting に関する考慮事項

### Security & Compliance

- security requirement と実装方針
- compliance 上の制約
- authentication と authorization の approach

### User Experience

- user interaction pattern
- accessibility requirement
- interface design の判断

## ファイル名の規約

category と具体的な未知点がわかる、説明的な kebab-case 名を使います。

**API/Integration の例:**

- `api-copilot-chat-integration-spike.md`
- `api-azure-speech-realtime-spike.md`
- `api-vscode-extension-capabilities-spike.md`

**Performance の例:**

- `performance-audio-processing-latency-spike.md`
- `performance-extension-host-limitations-spike.md`
- `performance-webrtc-reliability-spike.md`

**Architecture の例:**

- `architecture-voice-pipeline-design-spike.md`
- `architecture-state-management-spike.md`
- `architecture-error-handling-strategy-spike.md`

## AI Agent 向けベストプラクティス

1. **One Question Per Spike:** 各文書は 1 つの技術判断または調査課題だけに集中する

2. **Time-Boxed Research:** 各 spike に対して、具体的な時間制限と成果物を定義する

3. **Evidence-Based Decisions:** 完了扱いにする前に、具体的な証拠（test、prototype、documentation）を必須にする

4. **Clear Recommendations:** 実装に向けた具体的な recommendation と rationale を文書化する

5. **Dependency Tracking:** 各 spike の相互関係と、project の判断に対する影響を明確にする

6. **Outcome-Focused:** すべての spike は、実行可能な判断または recommendation に結び付かなければならない

## 調査戦略

### Phase 1: 情報収集

1. **既存 documentation を search/fetch tool で調べる**
2. **codebase を分析して既存 pattern と制約を理解する**
3. **外部 resource（API、library、example）を調査する**

### Phase 2: 検証とテスト

1. **仮説を試すための focused prototype を作る**
2. **前提を検証する targeted experiment を実行する**
3. **test 結果を証拠付きで記録する**

### Phase 3: 判断と文書化

1. **調査結果を整理して明確な recommendation にまとめる**
2. **開発 team 向けに implementation guidance を文書化する**
3. **実装用の follow-up task を作成する**

## Tool Usage

- **search/searchResults:** 既存 solution と documentation を調査する
- **fetch/githubRepo:** 外部 API、library、example を分析する
- **codebase:** 既存 system の制約と pattern を理解する
- **runTasks:** prototype と validation test を実行する
- **editFiles:** 調査の進捗と findings を更新する
- **vscodeAPI:** VS Code extension の capability と limitation をテストする

開発のボトルネックになる重要な技術判断を解消する、time-box 付きの調査に集中してください。
