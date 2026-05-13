---
description: 'Specification-Driven Workflow v1 は、要件を明確に定義し、設計を綿密に計画し、実装を十分に文書化して検証することで、構造化された software development アプローチを提供します。'
applyTo: '**'
---
# Spec Driven Workflow v1

**Specification-Driven Workflow:**
要件と実装の間にあるギャップを埋める。

**常に次の artifact を維持すること:**

- **`requirements.md`**: 構造化された EARS notation による user story と acceptance criteria。
- **`design.md`**: technical architecture、sequence diagram、実装上の考慮事項。
- **`tasks.md`**: 詳細で追跡可能な実装 plan。

## 汎用ドキュメントフレームワーク

**Documentation Rule:**
すべての documentation で、詳細 template を **唯一の正** として使う。

**Summary format:**
changelog や pull request description のような簡潔な artifact にのみ使う。

### 詳細 documentation template

#### Action Documentation Template (すべての step / execution / test)

```bash
### [TYPE] - [ACTION] - [TIMESTAMP]
**Objective**: [達成する目標]
**Context**: [現在の状態、要件、過去 step への参照]
**Decision**: [選んだアプローチと根拠。該当する場合は Decision Record を参照]
**Execution**: [実施した手順、使用した parameter と command。code の場合は file path を含める]
**Output**: [完全で省略のない結果、log、command output、metric]
**Validation**: [成功確認方法と結果。失敗時は remediation plan を含める]
**Next**: [次の具体的 action への自動継続 plan]
```

#### Decision Record Template (すべての意思決定)

```bash
### Decision - [TIMESTAMP]
**Decision**: [何を決めたか]
**Context**: [判断が必要になった状況と、それを駆動した data]
**Options**: [評価した代替案と、それぞれの簡潔な pros / cons]
**Rationale**: [選択した案が優れている理由。trade-off を明示する]
**Impact**: [実装、保守性、性能に対する見込み影響]
**Review**: [この判断を再評価する条件または時期]
```

### Summary format (report 用)

#### 簡略 Action Log

簡潔な changelog を生成するために使う。各 log entry は完全な Action Document から派生する。

`[TYPE][TIMESTAMP] Goal: [X] → Action: [Y] → Result: [Z] → Next: [W]`

#### 圧縮された Decision Record

pull request summary や executive summary で使う。

`Decision: [X] | Rationale: [Y] | Impact: [Z] | Review: [Date]`

## 実行ワークフロー (6-Phase Loop)

**どの step も飛ばさないこと。一貫した用語を使い、曖昧さを減らすこと。**

### **Phase 1: ANALYZE**

**Objective:**

- 問題を理解する。
- 既存 system を分析する。
- 明確でテスト可能な要件セットを作る。
- あり得る解決策とその影響を考える。

**Checklist:**

- [ ] 提供された code、documentation、test、log をすべて読む。
      - file inventory、要約、初期分析結果を文書化する。
- [ ] **EARS Notation** で要件を定義する:
      - feature request を構造化されテスト可能な要件に変換する。
      - 形式: `WHEN [a condition or event], THE SYSTEM SHALL [expected behavior]`
- [ ] dependency と constraint を特定する。
      - risk と mitigation strategy を含む dependency graph を文書化する。
- [ ] data flow と相互作用を整理する。
      - system interaction diagram と data model を文書化する。
- [ ] edge case と failure をカタログ化する。
      - 包括的な edge case matrix と潜在的な failure point を文書化する。
- [ ] confidence を評価する。
      - 要件の明確さ、複雑さ、問題 scope に基づいて **Confidence Score (0-100%)** を生成する。
      - score とその根拠を文書化する。

**Critical Constraint:**

- **すべての要件が明確に文書化されるまで先に進まないこと。**

### **Phase 2: DESIGN**

**Objective:**

- 包括的な technical design と詳細な実装 plan を作る。

**Checklist:**

- [ ] **Confidence Score に基づく適応的な実行戦略を定義する:**
  - **High Confidence (>85%)**
    - 包括的で step-by-step の実装 plan を作成する。
    - proof-of-concept step は省略する。
    - 完全な自動実装へ進む。
    - 標準的な包括 documentation を維持する。
  - **Medium Confidence (66–85%)**
    - **Proof-of-Concept (PoC)** または **Minimum Viable Product (MVP)** を優先する。
    - PoC / MVP の明確な成功条件を定義する。
    - 先に PoC / MVP を構築して検証し、その後 plan を段階的に拡張する。
    - PoC / MVP の目標、実行、検証結果を文書化する。
  - **Low Confidence (<66%)**
    - 最初の phase を research と知識形成に充てる。
    - semantic search を使い、類似実装を分析する。
    - 得られた内容を research document にまとめる。
    - research 後に ANALYZE phase を再実行する。
    - confidence が低いままなら escalate する。

- [ ] **`design.md` に technical design を記録する:**
  - **Architecture:** component と相互作用の高レベル概要。
  - **Data Flow:** diagram と説明。
  - **Interfaces:** API contract、schema、公開 function signature。
  - **Data Models:** data structure と database schema。

- [ ] **error handling を文書化する:**
  - 手順と期待 response を持つ error matrix を作成する。

- [ ] **unit test 戦略を定義する。**

- [ ] **`tasks.md` に実装 plan を作成する:**
  - 各 task について、説明、期待結果、dependency を含める。

**Critical Constraint:**

- **design と plan が完成し、検証されるまで実装へ進まないこと。**

### **Phase 3: IMPLEMENT**

**Objective:**

- design と plan に従って、本番品質の code を書く。

**Checklist:**

- [ ] 小さくテスト可能な increment で code を書く。
      - 各 increment について、code change、結果、test link を文書化する。
- [ ] dependency から上位へ向かって実装する。
      - 解決順序、根拠、検証を文書化する。
- [ ] convention に従う。
      - 準拠状況と逸脱があれば Decision Record とともに文書化する。
- [ ] 意味のある comment を追加する。
      - 仕組み ("what") ではなく意図 ("why") に焦点を当てる。
- [ ] 計画どおりに file を作成する。
      - file 作成 log を文書化する。
- [ ] task status をリアルタイムで更新する。

**Critical Constraint:**

- **すべての実装 step が文書化され test 済みになるまで、code を merge / deploy してはならない。**

### **Phase 4: VALIDATE**

**Objective:**

- 実装がすべての要件と品質基準を満たすことを確認する。

**Checklist:**

- [ ] automated test を実行する。
      - output、log、coverage report を文書化する。
      - failure があれば、root cause analysis と remediation を文書化する。
- [ ] 必要に応じて manual verification を行う。
      - 手順、checklist、結果を文書化する。
- [ ] edge case と error をテストする。
      - 結果と、正しい error handling の証拠を文書化する。
- [ ] performance を確認する。
      - metric を文書化し、重要 section を profile する。
- [ ] 実行 trace を記録する。
      - path analysis と runtime behavior を文書化する。

**Critical Constraint:**

- **すべての validation step が完了し、すべての issue が解決されるまで先に進まないこと。**

### **Phase 5: REFLECT**

**Objective:**

- codebase を改善し、documentation を更新し、performance を分析する。

**Checklist:**

- [ ] 保守性向上のために refactor する。
      - decision、before / after 比較、impact を文書化する。
- [ ] project documentation をすべて更新する。
      - すべての README、diagram、comment が最新であることを確認する。
- [ ] 潜在的な改善点を特定する。
      - backlog を優先度付きで文書化する。
- [ ] success criteria を確認する。
      - 最終 verification matrix を文書化する。
- [ ] meta-analysis を行う。
      - 効率、tool usage、protocol 遵守を振り返る。
- [ ] technical debt issue を自動作成する。
      - inventory と remediation plan を文書化する。

**Critical Constraint:**

- **すべての documentation と改善 action が記録されるまで、この phase を閉じてはならない。**

### **Phase 6: HANDOFF**

**Objective:**

- 作業を review / deploy 用にまとめ、次の task へ引き継ぐ。

**Checklist:**

- [ ] executive summary を生成する。
      - **Compressed Decision Record** format を使う。
- [ ] pull request を準備する (該当する場合):
    1. executive summary。
    2. **Streamlined Action Log** に基づく changelog。
    3. validation artifact と Decision Record への link。
    4. 最終的な `requirements.md`、`design.md`、`tasks.md` への link。
- [ ] workspace を最終化する。
      - 中間 file、log、一時 artifact を `.agent_work/` へ archive する。
- [ ] 次の task へ進む。
      - 移行または完了を文書化する。

**Critical Constraint:**

- **すべての handoff step が完了し文書化されるまで、task 完了と見なしてはならない。**

## トラブルシューティングと再試行プロトコル

**error、曖昧さ、blocker に遭遇した場合:**

**Checklist:**

1. **再分析する**:
   - ANALYZE phase を見直す。
   - すべての要件と制約が明確かつ完全かを確認する。
2. **再設計する**:
   - DESIGN phase を見直す。
   - 必要に応じて technical design、plan、dependency を更新する。
3. **再計画する**:
   - 新しい発見に対応するため、`tasks.md` の実装 plan を調整する。
4. **実行を再試行する**:
   - 修正した parameter や logic で失敗した step を再実行する。
5. **エスカレートする**:
   - 再試行後も問題が続く場合は、escalation protocol に従う。

**Critical Constraint:**

- **未解決の error や曖昧さを抱えたまま先に進まないこと。トラブルシューティングの手順と結果は必ず記録すること。**

## Technical Debt Management (自動化)

### 特定と文書化

- **Code Quality**: 実装中は static analysis を使って継続的に code quality を評価する。
- **Shortcut**: speed-over-quality の判断は、その結果とともに Decision Record に明示的に記録する。
- **Workspace**: 組織的 drift や命名の不一致を監視する。
- **Documentation**: 未完了、古い、欠落した documentation を追跡する。

### 自動 issue 作成 template

```text
**Title**: [Technical Debt] - [簡潔な説明]
**Priority**: [business impact と remediation cost に基づく High/Medium/Low]
**Location**: [file path と line number]
**Reason**: [debt が発生した理由。利用可能なら Decision Record へ link]
**Impact**: [現在および将来の影響 (例: 開発速度低下、bug risk 増加)]
**Remediation**: [具体的で実行可能な解決手順]
**Effort**: [解決見積もり (例: T-shirt size: S、M、L)]
```

### Remediation (自動優先付け)

- dependency analysis を伴う risk-based priority 付け。
- 将来計画の助けとなる effort estimate。
- 大規模 refactoring に対する migration strategy を提案する。

## Quality Assurance (自動化)

### 継続的監視

- **Static Analysis**: code style、quality、security vulnerability、architecture rule 遵守を linting する。
- **Dynamic Analysis**: staging environment で runtime behavior と performance を監視する。
- **Documentation**: documentation の完全性と正確さ (例: link、format) を自動確認する。

### 品質 metric (自動追跡)

- code coverage の割合と gap analysis。
- function / method ごとの cyclomatic complexity score。
- maintainability index の評価。
- technical debt ratio (例: remediation 想定時間 / development 時間)。
- documentation coverage の割合 (例: comment 付き public method)。

## EARS Notation リファレンス

**EARS (Easy Approach to Requirements Syntax)** - 要件の標準形式:

- **Ubiquitous**: `THE SYSTEM SHALL [expected behavior]`
- **Event-driven**: `WHEN [trigger event] THE SYSTEM SHALL [expected behavior]`
- **State-driven**: `WHILE [in specific state] THE SYSTEM SHALL [expected behavior]`
- **Unwanted behavior**: `IF [unwanted condition] THEN THE SYSTEM SHALL [required response]`
- **Optional**: `WHERE [feature is included] THE SYSTEM SHALL [expected behavior]`
- **Complex**: 高度な要件向けに、上記 pattern を組み合わせたもの

各要件は次を満たさなければならない:

- **Testable**: automated test または manual test で検証できる
- **Unambiguous**: 解釈が 1 つに定まる
- **Necessary**: system の目的に貢献する
- **Feasible**: 制約内で実装可能である
- **Traceable**: user need と design element に紐づけられる
