# 出力形式 — レポート ファイル テンプレート

⛔ **自己修正指示:** このドキュメントのテンプレートを使用してファイルを作成した後、すぐに下部にあるセルフチェック セクションを実行してください。オーケストレーターへの応答には、各項目について ✅/❌ を記入したチェックリストを含める必要があります。いずれかの項目が ❌ の場合は、次の手順に進む前にファイルを修正してください。

このファイルは、脅威モデル アナリストによって生成されるすべての出力ファイルの構造と内容を定義します。各セクションにはテンプレート、ルール、検証チェックリストが含まれています。

**図の規則** は別のファイルにあります: [diagram-conventions.md](./diagram-conventions.md)
**分析方法** は別のファイルにあります: [analysis-principles.md](./analysis-principles.md)

---

## 出力フォルダー

分析の開始時にタイムスタンプ付きのフォルダーを作成します。
- 形式: `threat-model-YYYYMMDD-HHmmss` (UTC 時間)
- 例: `threat-model-20260130-073845`
- すべての出力ファイルをこのフォルダーに書き込みます

---

## ファイルコンテンツのフォーマット - 重要なルール

**`.md` ファイルのコンテンツをコード フェンスでラップしないでください。** `create_file` または `edit_file` を使用する場合:
- このツールは生のコンテンツをディスクに書き込みます。 `を含めると```markdown ` at the start, it becomes literal text in the file.
- **WRONG**: Content starts with ` ```markdown ` — ファイルにはフェンスがリテラルテキストとして含まれます
- **正解**: コンテンツは 1 行目の `# Heading` で直接始まります
- これはすべての `.md` ファイルに適用されます: `0.1-architecture.md`、`0-assessment.md`、`1-threatmodel.md`、`2-stride-analysis.md`、`3-findings.md`

**`.mmd` ファイルのコンテンツをコード フェンスでラップしないでください。** `.mmd` ファイルは生の Mermaid ソースです。
- **間違い**: コンテンツは ` で始まります```plaintext ` or ` ```人魚
- **正解**: コンテンツは 1 行目の `%%{init:` で始まり、2 行目の `flowchart` または `graph` が続きます

**すべてのファイルに書き込む前にセルフチェックを行ってください:** コンテンツの最初の文字を確認してください。 ` の場合``` ` — STOP and remove the fence.

---

## File List

| File | Description | Always? |
|------|-------------|---------|
| `0-assessment.md` | Executive summary, risk rating, action plan, metadata | Yes |
| `0.1-architecture.md` | Architecture overview, components, scenarios, tech stack | Yes |
| `1-threatmodel.md` | Threat model DFD diagram + element/flow/boundary tables | Yes |
| `1.1-threatmodel.mmd` | Pure Mermaid DFD (source of truth for detailed diagram) | Yes |
| `1.2-threatmodel-summary.mmd` | Summary DFD (only if >15 elements or >4 boundaries) | Conditional |
| `2-stride-analysis.md` | Full STRIDE-A analysis for all components | Yes |
| `3-findings.md` | Prioritized security findings with remediation | Yes |
| `threat-inventory.json` | Structured JSON inventory for comparison matching | Yes |
| `incremental-comparison.html` | Visual HTML comparison report (incremental mode only) | Conditional |

---

## 0.1-architecture.md

**Purpose:** High-level architecture overview — generated FIRST, before threat modeling begins.

**When to generate:** Every run. Not conditional.

**Diagrams:** All inline Mermaid in the markdown. NO separate `.mmd` files for 0.1-architecture.md.

### Content Structure

```値下げ
# アーキテクチャの概要

## システムの目的
<!-- 2 ～ 4 文: このシステムは何ですか?それはどのような問題を解決しますか?ユーザーとは誰ですか? -->

## 主要コンポーネント
|コンポーネント |タイプ |説明 |
|----------|------|---------------|
| [名前] | [プロセス / データ ストア / 外部サービス / 外部インタラクター] | [役割を 1 行で説明] |

## コンポーネント図
<!-- service/external/datastore classDef (DFD サークルではありません) を使用したアーキテクチャ図。スタイルについては、diagram-conventions.md を参照してください。 -->

## トップのシナリオ
<!-- 3 ～ 5 つの最も重要なワークフロー。最初の 3 つはシーケンス図を含める必要があります。 -->

### シナリオ 1: [名前]
【2-3文の説明】
<!-- マーメイドシーケンス図はここにあります -->

### シナリオ 2: [名前]
### シナリオ 3: [名前]

## テクノロジースタック
|レイヤー |テクノロジー |
|------|--------------|
|言語 | ... |
|フレームワーク | ... |
|データストア | ... |
|インフラ | ... |
|セキュリティ | ... |

## 導入モデル
<!-- 導入方法は?オンプレミス、クラウド、ハイブリッド?コンテナ、VM? -->

## セキュリティ インフラストラクチャのインベントリ
|コンポーネント |セキュリティの役割 |構成 |メモ |
|----------|------|------|------|
| [例: MISE サイドカー] | [例: 認証プロキシ] | [例: Entra ID OIDC] | [例: すべての API ポッド] |

## リポジトリ構造
|ディレクトリ |目的 |
|----------|----------|
| [パス/] | 【目次】 |```

### Processing Rules

1. Generate **before** creating the threat model diagram
2. Derive all content from code analysis — do not speculate
3. If a section cannot be determined, state that explicitly
4. Target: **150-250 lines** minimum. Previous iterations produced only 100-150 lines which is too thin. Include detailed component descriptions, port/protocol info, and substantial scenario narratives.
5. Key Components table should align with threat model diagram elements
6. Use **architecture** diagram styles (not DFD) — see `diagram-conventions.md`
7. After writing, verify each Mermaid block has valid syntax
8. **Top Scenarios**: The first 3 scenarios MUST include Mermaid `sequenceDiagram` blocks showing the interaction flow. Each sequence diagram should show actual participants, messages with protocol details, and alt/opt blocks for error paths.
9. **Component alignment**: Every component listed in Key Components MUST later appear as a section in `2-stride-analysis.md`
10. **Deployment Model**: Must include specific details: ports, protocols, bind addresses, network exposure, and deployment topology (single machine / cluster / multi-tier)
11. **Security Infrastructure Inventory**: Populate with EVERY security-relevant component found in code (auth, encryption, access control, logging, secrets management)

---

## 1-threatmodel.md + 1.1-threatmodel.mmd

**Purpose:** System threat model as a Data Flow Diagram (DFD).

### Generation Steps

**Step 1:** Create `1.1-threatmodel.mmd` (source of truth)
- Pure Mermaid code, no markdown wrapper
- Use DFD shapes and styles from `diagram-conventions.md`

**Step 2:** Run the POST-DFD GATE from `orchestrator.md` Step 4 to evaluate and create `1.2-threatmodel-summary.mmd` if threshold is met. See `skeletons/skeleton-summary-dfd.md` for the template.

**Step 3:** Create `1-threatmodel.md` (include Summary View section if summary was generated)

### 1-threatmodel.md Content

```値下げ
# 脅威モデル

## データフロー図
<!-- でラップされた 1.1-threatmodel.mmd から正確な図をコピーします。```mermaid fence -->

## Element Table
| Element | Type | TMT Category | Description | Trust Boundary |
|---------|------|--------------|-------------|----------------|

- **Type** = high-level DFD category: `Process`, `External Interactor`, or `Data Store`
- **TMT Category** = specific TMT ID from tmt-element-taxonomy.md §1 (e.g. `SE.P.TMCore.WebSvc`, `SE.EI.TMCore.Browser`, `SE.DS.TMCore.SQL`)
- For Kubernetes-based applications where pods run sidecars, add an optional **Co-located Sidecars** column (e.g. `MISE, Dapr` or `—`)

## Data Flow Table
| ID | Source | Target | Protocol | Description |
|----|--------|--------|----------|-------------|

## Trust Boundary Table
| Boundary | Description | Contains |
|----------|-------------|----------|

## Summary View (only if summary diagram generated)
<!-- Copy from 1.2-threatmodel-summary.mmd -->

## Summary to Detailed Mapping
| Summary Element | Contains | Summary Flows | Maps to Detailed Flows |
```**重要なルール:**
- `.mmd` と `.md` の図は同一でなければなりません (コピーし、再生成しないでください)
- 詳細なフローには `DF01`、`DF02` を使用します。 `SDF01`、`SDF02` (概要フロー用)

---

## 2-ストライド分析.md

**目的:** すべてのコンポーネントの完全な STRIDE + Abuse Cases 脅威分析。

### 構造要件

1. 各コンポーネントの脅威 ** 別の表を使用して、Tier 1、Tier 2、Tier 3 のサブセクションに分割する必要があります**
2. 概要テーブル **T1、T2、T3 列を含める必要があります**
3. コンポーネントごとに 3 つの階層サブセクションがすべて表示されます (空の場合でも、「*階層 N の脅威は識別されません*」を使用します)。

### アンカーセーフな見出し (重要)

コンポーネント `## ` の見出しは、`3-findings.md` からのリンク ターゲットになります。
- 文字、数字、スペース、ハイフンのみ**を使用してください
- **見出しの禁止:** `&`、`/`、`(`、`)`、`.`、`:`、`'`、`"`、`+`、`@`、`!`
- 置換: `&` → `and`、`/` → `-`、括弧 → 削除

**アンカー ルール:** 見出し→小文字、スペース→ハイフン、ハイフン以外の英数字以外は削除します。

### テンプレート

> **⛔ 重要: `## Summary` テーブルは、ファイルの先頭、`## Exploitability Tiers` の直後、個々の `## Component` セクションの前になければなりません。これはナビゲーション補助です。読者はまずそれを必要とします。モデルは一貫してそれを一番下に移動します。これは間違いです。次の順序に従ってください: `# STRIDE + Abuse Cases — Threat Analysis` → `## Exploitability Tiers` → `## Summary` → `---` → `## Component 1` → `## Component 2` → ...**

> **⛔ 厳格な層の定義 — これらを正確に適用してください。主観的な判断は使用しないでください。** これはスキル ディレクティブです。この行を出力にコピーしないでください。以下の層テーブルは、このディレクティブ行を除いたレポートに記載されているものです。

> **⛔ LEAKED DIRECTIVE CHECK:** 出力ファイルには、テキスト「RIGID TIER DEFINITIONS」、「主観的判断を使用しないでください」、または `⛔` で始まる行を含めることはできません。これらはスキルの説明であり、レポートの内容ではありません。出力にそれらが含まれている場合は、ファイナライズする前に削除してください。```markdown
# STRIDE + Abuse Cases — Threat Analysis

## Exploitability Tiers

Threats are classified into three exploitability tiers based on the prerequisites an attacker needs:

| Tier | Label | Prerequisites | Assignment Rule |
|------|-------|---------------|----------------|
| **Tier 1** | Direct Exposure | `None` | Exploitable by unauthenticated external attacker with NO prior access. The prerequisite field MUST say `None`. |
| **Tier 2** | Conditional Risk | Single prerequisite: `Authenticated User`, `Privileged User`, `Internal Network`, or single `{Boundary} Access` | Requires exactly ONE form of access. The prerequisite field has ONE item. |
| **Tier 3** | Defense-in-Depth | `Host/OS Access`, `Admin Credentials`, `{Component} Compromise`, `Physical Access`, or MULTIPLE prerequisites joined with `+` | Requires significant prior breach, infrastructure access, or multiple combined prerequisites. |
```> **⛔ 階層テーブルをそのままコピーしてください。** 4 番目の列は `Assignment Rule` である必要があります (`Example`、`Description`、`Criteria`、またはその他の名前ではありません)。セルの値は上記のテキストとまったく同じである必要があります。配置固有の例に置き換えないでください。表の後に「階層割り当てに影響を与えるデプロイメント コンテキスト」段落を追加しないでください。デプロイメント コンテキストは、階層定義ではなく、個々のコンポーネント セクションに属します。

> **⛔ ストライド A カテゴリのラベル (必須 - 「A」は「不正使用」であり、決して「認可」ではない):**
> すべてのテーブル (概要、コンポーネントごとの階層テーブル、threat-inventory.json) で使用される 7 つの STRIDE-A カテゴリは次のとおりです。
> **ふざけた行為 | **T**アンペアリング | **R**の証言 | **情報開示 | **サービス終了** | **E**特権の昇格 | **バス**
> 「悪用」には、ビジネス ロジックの悪用、ワークフローの操作、機能の誤用、正当な機能の意図しない使用が含まれます。
> モデルは A 列に「承認」を頻繁に生成します。これは誤りです。 STRIDE カテゴリ ラベルとして「Authorization」が表示されている場合は、「Abuse」に置き換えてください。脅威行のカテゴリ列には、(「認可」ではなく)「悪用」と表示する必要があります。 N/A エントリには、(「承認 - N/A」ではなく) 「不正使用 - N/A」と記載する必要があります。

## 概要
|コンポーネント |リンク | S |た | R |私 | D | E |あ |合計 | T1 | T2 | T3 |リスク |
|----------|------|---|---|---|---|---|---|---|----------|----|----|----|------|

---

## コンポーネント名

**信頼境界:** [境界名]
**役割:** [簡単な説明]
**データ フロー:** [DF ID のリスト]
**ポッドのコロケーション:** [K8 の場合はサイドカー — 図-conventions.md を参照]

### STRIDE-A 分析

> **⛔ カテゴリの名前: STRIDE-A の 7 つのカテゴリは、なりすまし、改ざん、否認、情報開示、サービス拒否、特権昇格、悪用です。 「A」カテゴリは常に「悪用」であり、決して「承認」ではありません。承認の問題は、特権の昇格 (E) に属します。これは、N/A 正当化ラベル、脅威テーブルのカテゴリ列、およびすべての散文に適用されます。**

#### Tier 1 — 直接暴露 (前提条件なし)
| ID |カテゴリー |脅威 |前提条件 |影響を受けるフロー |緩和 |ステータス |
|----|----------|--------|---------------|--------------|------------|--------|

#### 階層 2 — 条件付きリスク
| ID |カテゴリー |脅威 |前提条件 |影響を受けるフロー |緩和 |ステータス |

#### Tier 3 — 多層防御
| ID |カテゴリー |脅威 |前提条件 |影響を受けるフロー |緩和 |ステータス |```

**⛔ STRIDE Status Column — Valid Values (must match Coverage table):**
The `Status` column in each threat row MUST use exactly one of these values:
- `Open` — Threat is not mitigated; MUST map to a finding (`✅ Covered` in Coverage table). The finding documents the vulnerability and remediation guidance.
- `Mitigated` — Threat is mitigated by the engineering team's own code, configuration, or design decisions in THIS repository. Maps to `✅ Mitigated (FIND-XX)` in Coverage table. A finding MUST be created that documents WHAT the team did, WHERE in the code, and HOW it mitigates the threat. This gives credit to the engineering team for security work they've already done.
- `Platform` — Threat is mitigated by an EXTERNAL platform that is NOT part of the analyzed codebase. See strict definition below. Maps to `🔄 Mitigated by Platform` in Coverage table. NO finding is created — the mitigation is outside this team's control.

**How to distinguish Mitigated vs Platform:**
| Question | If YES → | If NO → |
|----------|----------|---------|
| Is the mitigation implemented in code within THIS repository? | `Mitigated` | Check next |
| Is the mitigation in deployment config controlled by THIS team? | `Mitigated` | Check next |
| Is the mitigation provided by a completely external system? | `Platform` | `Open` (no mitigation exists) |

**Examples of `Mitigated` (team's own work — create finding to document it):**
- Auth middleware validating JWT tokens — the team wrote this code
- TLS certificate generation and configuration — the team implemented this
- File permissions set to 0600 in the code — the team chose secure defaults
- Input validation or sanitization functions — the team built defenses
- Rate limiting middleware — the team added throttling
- Localhost-only binding — the team made an architectural security decision

**The finding for a `Mitigated` threat documents the existing control:**
- Title: descriptive of what IS in place (e.g., "JWT Authentication Middleware on API Endpoints")
- Severity: Low (existing control) or Moderate (if control has gaps)
- Mitigation Type: `Existing Control`
- Remediation section: describes what's already implemented + any hardening recommendations
- This ensures the Coverage table shows the team's security work, not just gaps

**⛔ STRICT DEFINITION OF "PLATFORM" (MANDATORY):**
`Platform` status is ONLY valid when ALL of these conditions are true:
1. The mitigation is provided by a system **completely outside** the analyzed repository's code
2. The mitigation is **managed by a different team/organization** (e.g., Azure AD is managed by Microsoft Identity team, not by this repo's team)
3. The mitigation **cannot be disabled or weakened** by modifying code in this repository

**Examples of LEGITIMATE Platform mitigations:**
- Azure AD token signing (managed by Microsoft Identity, not this code)
- K8s RBAC (managed by K8s control plane, not this operator)
- Azure Arc tunnel encryption (managed by Arc team, not this agent)
- TPM hardware security (hardware, not software)

**Examples of things that are NOT "Platform" — they are `Mitigated` (team's work):**
- ✅ "Auth middleware on endpoints" → `Mitigated` — team wrote the auth code. Create finding documenting it.
- ✅ "TLS on localhost" → `Mitigated` — team implemented TLS. Create finding documenting the implementation.
- ✅ "File permissions 0600" → `Mitigated` — team set secure defaults. Create finding documenting the choice.
- ✅ "Localhost binding" → `Mitigated` — team made architectural security decision. Create finding.
- ✅ "Input validation" → `Mitigated` — team built defense. Create finding documenting what's validated.
- ✅ "Operation state machine" → `Mitigated` — team's logic prevents abuse. Create finding.

**⛔ MAXIMUM PLATFORM RATIO:** If more than 20% of threats are classified as "🔄 Mitigated by Platform", re-examine each. Many should be `Mitigated` (team's code) not `Platform` (external). In a typical application, 5-15% are genuinely platform-mitigated, 20-40% are mitigated by the team's own code, and the rest are `Open` (needing remediation).

**⛔ NEVER use these values:**
- ❌ `Partial` — ambiguous. If partially mitigated, it's `Open` (the remaining gap is the finding)
- ❌ `N/A` — every threat is applicable if it's in the table
- ❌ `Accepted` — the tool does not accept risks
- ❌ `Needs Review` — every threat must be either Covered, Mitigated, or Platform

**Consistency rule:** The STRIDE `Status` column and the Findings Coverage table `Status` MUST agree:
| STRIDE Status | Coverage Table Status | Meaning |
|---|---|---|
| `Open` | `✅ Covered (FIND-XX)` | Finding documents a vulnerability needing remediation |
| `Mitigated` | `✅ Mitigated (FIND-XX)` | Finding documents an existing control the team built — gives credit for security work |
| `Platform` | `🔄 Mitigated by Platform` | External platform handles it — no finding needed |

**⛔ "Accepted Risk" and "Needs Review" are FORBIDDEN.** The tool does NOT have authority to accept risks or defer threats. Every threat maps to either a finding (Covered or Mitigated) or a genuine external platform mitigation. There is no middle ground.

### Arithmetic Verification (MANDATORY)

After writing ALL component tables:
1. Count actual threat rows per component per category (S,T,R,I,D,E,A) — compare with summary table
2. Verify Total = S+T+R+I+D+E+A for each row
3. Verify T1+T2+T3 = Total for each row
4. Verify Totals row = column-wise sum
5. Row count cross-check: threat rows in detail = Total in summary

---

## 3-findings.md

**Purpose:** Prioritized security findings with evidence and remediation.

> **⛔ IMPORTANT: Before writing this file, read [skeleton-findings.md](./skeletons/skeleton-findings.md) and copy the skeleton VERBATIM for each finding. Fill in the `[FILL]` placeholders. This prevents template drift.**

### Structure Requirements

Organized by **Exploitability Tier** (NOT by severity):
1. `## Tier 1 — Direct Exposure (No Prerequisites)`
2. `## Tier 2 — Conditional Risk (Authenticated / Single Prerequisite)`
3. `## Tier 3 — Defense-in-Depth (Prior Compromise / Host Access)`

**DO NOT** use `## Critical Findings`, `## Important Findings`, etc.
Sort by severity **within** each tier, then by CVSS descending.

**Tier Assignment for Findings:**
- A finding's tier is determined by its `Exploitation Prerequisites` value, using the same rules as STRIDE-A tier assignment (see [analysis-principles.md](./analysis-principles.md))
- If a finding covers threats from multiple tiers (via Related Threats), assign it to the **highest-priority tier** (lowest tier number) among its related threats

**Ordering within each tier:** Sort findings by:
1. **SDL Bugbar Severity** descending: Critical → Important → Moderate → Low
2. **Within each severity band**, sort by CVSS 4.0 score descending (highest first)

**After writing all findings**, verify the sort order:
- List all findings with their severity, CVSS score, and tier
- Confirm no finding with higher CVSS appears after a lower CVSS finding within the same severity band and tier
- If misordered, renumber and reorder before finalizing

**Finding ID Numbering — MUST be sequential:**
- Use `FIND-01`, `FIND-02`, `FIND-03`, ... only. `F-01`, `F01`, or `Finding 1` formats are NOT allowed.
- IDs MUST appear in order in the document: FIND-01 before FIND-02 before FIND-03, etc.
- ❌ NEVER have FIND-06 appear before FIND-04 in the document. If reordering findings, renumber ALL IDs to maintain sequential order.
- After final sort, scan the document top-to-bottom: the first finding heading must be FIND-01, the next FIND-02, etc. No gaps, no out-of-order.

### Finding Attributes (ALL MANDATORY)

| Attribute | Description |
|-----------|-------------|
| SDL Bugbar Severity | Critical / Important / Moderate / Low |
| CVSS 4.0 | Score AND full vector string (e.g., `9.3 (CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N)`) — BOTH are mandatory |
| CWE | ID, name, AND hyperlink (e.g., `[CWE-306](https://cwe.mitre.org/data/definitions/306.html): Missing Authentication for Critical Function`) |
| OWASP | Top 10:2025 mapping (A01:2025 format — never :2021) |
| Exploitation Prerequisites | From tier definitions |
| Exploitability Tier | Tier 1 / Tier 2 / Tier 3 |
| Remediation Effort | Low / Medium / High |
| Mitigation Type | Redesign / Standard Mitigation / Custom Mitigation / Existing Control / Accept Risk / Transfer Risk |
| Component | Affected component |
| Related Threats | Individual links to `2-stride-analysis.md#component-anchor` |

### Full Finding Example

```値下げ
### FIND-01: API で認証がありません

|属性 |値 |
|----------|----------|
| SDL バグバーの重大度 |クリティカル |
| CVSS4.0 | 9.3 (CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N) |
| CWE | [CWE-306](https://cwe.mitre.org/data/settings/306.html): 重要な機能の認証がありません |
|オワスプ | A07:2025 – 認証の失敗 |
|悪用の前提条件 |なし (外部攻撃者) |
|悪用可能性の層 | Tier 1 — 直接暴露 |
|修復の取り組み |中 |
|緩和タイプ |標準的な緩和策 |
|コンポーネント | APIゲートウェイ |
|関連する脅威 | [T01.S](2-ストライド分析.md#api-ゲートウェイ)、[T01.R](2-ストライド分析.md#api-ゲートウェイ) |

#### 説明

API エンドポイント /api/v1/resources は、認証チェックなしでリクエストを受け入れます...

#### 証拠

`src/Controllers/ResourceController.cs` 45 行目 — コントローラーに `[Authorize]` 属性がありません。

#### 修復

コントローラークラスに `[Authorize]` 属性を追加し、`Program.cs` に JWT ベアラー認証を設定します。

#### 検証

認証されていない GET リクエストを `/api/v1/resources` に送信します。401 Unauthorized が返されるはずです。```

### Related Threats Link Format

> **⛔ CRITICAL: Related Threats MUST be hyperlinks, NOT plain text. The model consistently outputs plain text like `T-02, T-17` — this is WRONG. Each threat ID must link to the specific component section in stride analysis.**

- Individual links per threat ID: `[T01.S](2-stride-analysis.md#component-name)`
- **WRONG**: `T-02, T-17, T-23` (plain text, no links)
- **WRONG**: `[T08.S, T08.T](2-stride-analysis.md)` (grouped, no anchor)
- **CORRECT**: `[T08.S](2-stride-analysis.md#redis-state-store), [T08.T](2-stride-analysis.md#redis-state-store)`
- Every `| **Related Threats** |` cell must contain ONLY `[Txx.Y](2-stride-analysis.md#anchor)` format links separated by commas

### Post-Write Checks

1. **Anchor spot-check**: Verify 3+ Related Threats links resolve to real `##` headings
2. **Threat coverage check**: Every threat ID in `2-stride-analysis.md` must be referenced by at least one finding
3. **Sort order check**: Within each tier, no higher-CVSS finding appears after a lower-CVSS finding in the same severity band
4. **CVSS-to-Tier consistency**: Scan every finding — if CVSS has `AV:L` or `PR:H`, finding MUST NOT be in Tier 1. Fix by downgrading the tier, not by changing the CVSS.
5. **Threat Coverage Verification table**: At the end of `3-findings.md`, include:

```値下げ
## 脅威カバレッジの検証

|脅威ID | IDを探す |ステータス |
|----------|---------------|----------|
| T01.S |ファインド-01 | ✅ 対象 |
| T01.T | FIND-05 | ✅ 緩和されました (チーム実装の TLS) |
| T02.I | — | 🔄 プラットフォーム (Azure AD) によって軽減 |```

Every threat from `2-stride-analysis.md` must appear in this table. Status is one of:
- `✅ Covered (FIND-XX)` — finding documents a vulnerability that needs remediation
- `✅ Mitigated (FIND-XX)` — finding documents an existing control the team built (gives credit for security work done)
- `🔄 Mitigated by Platform` — external system handles it (only for genuinely external platforms)

**⛔ THIS TABLE IS A FEEDBACK LOOP, NOT DOCUMENTATION:**
The purpose of this table is to force you to check your work. After filling it out:
1. If ANY threat has a `—` dash in the Finding ID column with status other than `🔄 Mitigated by Platform` → **you missed a finding. Go back and create one.**
2. If Platform count > 20% of total threats → **you are overusing Platform as an escape hatch. Re-examine.**
3. If any threat is listed as `⚠️ Accepted Risk` or `⚠️ Needs Review` → **VIOLATION. Create a finding or verify it's genuinely Platform.**

The table should drive you to 100% coverage: every threat maps to either a finding (`✅ Covered`) or a legitimate external platform mitigation (`🔄 Mitigated by Platform`). There is no third option.

**⛔ FINDING GENERATION RULE:**
If a threat in `2-stride-analysis.md` has a non-empty `Mitigation` column, it MUST become a finding. The mitigation text provides the remediation — use it. The only exception is threats genuinely mitigated by an EXTERNAL platform (Azure AD, K8s RBAC, TPM hardware) that this code cannot disable.

---

## 0-assessment.md

**Purpose:** Executive summary, risk rating, action plan, and metadata. The "front page" of the report.

> **⛔ IMPORTANT: Before writing this file, read [skeleton-assessment.md](./skeletons/skeleton-assessment.md) and copy the skeleton VERBATIM. Fill in the `[FILL]` placeholders. This prevents template drift.**

### Section Order (MANDATORY — ALL 7 sections REQUIRED, do NOT skip any)

1. **Report Files** (REQUIRED) — Links to all report deliverables
2. **Executive Summary** (REQUIRED) — Risk rating + coverage. NO separate "Key Recommendations" subsection
3. **Action Summary** (REQUIRED) — Tier-based prioritized action plan with `### Quick Wins` subsection
4. **Analysis Context & Assumptions** (REQUIRED) — Scope, infrastructure context, `### Needs Verification` table, finding overrides
5. **References Consulted** (REQUIRED) — Security standards + component documentation
6. **Report Metadata** (REQUIRED) — Model, timestamps, duration, git info
7. **Classification Reference** (REQUIRED) — MUST be the last section. Static table copied from skeleton.

⚠️ **Enforcement:** If a section has no data, include it with empty tables or "N/A" notes — NEVER omit the section entirely. The agent in previous iterations skipped sections 1, 4, 5, and 6 entirely. ALL SEVEN must be present.

### Report Files Template

The Report Files table MUST list `0-assessment.md` (this file) as the FIRST row, followed by the other files:

```値下げ
## レポート ファイル

|ファイル |説明 |
|------|---------------|
| [0-評価.md](0-評価.md) |この文書 — エグゼクティブ サマリー、リスク評価、アクション プラン、メタデータ |
| [0.1-アーキテクチャ.md](0.1-アーキテクチャ.md) |アーキテクチャの概要、コンポーネント、シナリオ、技術スタック |
| [1-threatmodel.md](1-threatmodel.md) |要素、フロー、境界テーブルを含む脅威モデルの DFD 図 |
| [1.1-threatmodel.mmd](1.1-threatmodel.mmd) |ピュアマーメイド DFD ソース ファイル |
| [1.2-threatmodel-summary.mmd](1.2-threatmodel-summary.mmd) |概要 DFD (生成された場合のみ) |
| [2-ストライド分析.md](2-ストライド分析.md) |すべてのコンポーネントの完全な STRIDE-A 分析 |
| [3-所見.md](3-所見.md) |セキュリティに関する発見事項を優先的に修正 |```

⚠️ **`0-assessment.md` MUST be the first row.** The model consistently lists `0.1-architecture.md` first — that is WRONG. This file IS the front page of the report and lists itself first.

### Risk Rating

The heading must be plain text with NO emojis: `### Risk Rating: Elevated`, NOT `### Risk Rating: 🟠 Elevated`

### Threat Count Context Paragraph

Include at end of Executive Summary:

```値下げ
> **脅威数に関する注意:** この分析では、[M] 個のコンポーネントにわたって [N] 個の脅威が特定されました。この数は、体系的な不安ではなく、包括的な STRIDE-A の対象範囲を反映しています。このうち、**[T1 カウント] は前提条件なしで直接悪用可能です** (Tier 1)。残りの [T2+T3 カウント] は、条件付きリスクと多層防御の考慮事項を表します。```

### Action Summary Template

> **⛔ FIXED PRIORITY MAPPING — The Priority column values are DETERMINISTIC, not judgment-based:**
> | Tier | Priority | Always |
> |------|----------|--------|
> | Tier 1 | 🔴 Critical Risk | ALWAYS — regardless of threat/finding count |
> | Tier 2 | 🟠 Elevated Risk | ALWAYS — regardless of threat/finding count |
> | Tier 3 | 🟡 Moderate Risk | ALWAYS — regardless of threat/finding count |
>
> **NEVER change the priority based on how many threats or findings exist in that tier.** Even if Tier 1 has 0 threats and 0 findings, the priority is still 🔴 Critical Risk — because IF a Tier 1 threat existed, it would be critical. The priority reflects the tier's inherent severity, not the count. A report with Tier 1 = "🟢 Low Risk" is WRONG and must be fixed.

```値下げ
## アクションの概要

|階層 |説明 |脅威 |調査結果 |優先順位 |
|------|---------------|----------|----------|----------|
|ティア 1 |直接悪用可能 | 5 | 3 | 🔴 重大なリスク |
|階層 2 |認証されたアクセスが必要です | 8 | 4 | 🟠 リスクの上昇 |
|ティア 3 |事前の妥協が必要 | 12 | 5 | 🟡 中程度のリスク |
| **合計** | | **25** | **12** | |```

> **⛔ EXACTLY 4 ROWS: The Action Summary table MUST have exactly 4 data rows: Tier 1, Tier 2, Tier 3, and Total. Do NOT add rows for "Mitigated", "Platform", "Fixed", "Accepted", or any other status. Mitigated threats are distributed across their respective tiers — they are NOT a separate tier. If you find yourself adding a "Mitigated" row, STOP and remove it.**

```値下げ

### 即効性
<!-- 修復作業が少ない Tier 1 の結果 — 影響が大きく、迅速な修正 -->
|発見 |タイトル |なぜ速いのか |
|----------|----------|----------|
|検索-XX | [タイトル] | 【理由】 |```

⚠️ **Quick Wins is a REQUIRED subsection.** The `### Quick Wins` heading and table MUST appear after the tier summary table inside Action Summary. If no low-effort findings exist, write: `### Quick Wins\n\nNo low-effort findings identified. All findings require Medium or High effort.`

**Processing Rules for Action Summary:**
1. Populate the tier table with actual counts from `3-findings.md` (findings per tier) and `2-stride-analysis.md` (threats per tier from T1/T2/T3 columns in summary table)
2. Quick Wins lists only Tier 1 findings with `Remediation Effort: Low` — highest-impact, lowest-effort items
3. If no Tier 1 Low-effort findings exist, show Tier 2 Low-effort findings instead, with a note: "No Tier 1 quick wins identified. These Tier 2 items offer the best effort-to-impact ratio:"
4. If no Low-effort findings exist at all, keep `### Quick Wins` heading and add: `No low-effort findings identified. All findings require Medium or High effort.`
5. Verify: Findings column sums must equal total findings count in `3-findings.md`
6. Verify: Threats column sums must equal total threats count in `2-stride-analysis.md` summary table

### ⛔ PROHIBITED Content in Action Summary and All Output Files

**NEVER generate ANY of the following:**
- `### Priority Remediation by Phase` or any phase-based remediation roadmap
- Sprint references (`Sprint 1-2`, `Sprint 3-4`, etc.)
- Time-based phases (`Phase 1 — Immediate`, `Phase 2 — Short-term`, `Phase 3 — Medium-term`, `Phase 4 — Long-term`, `Backlog`)
- Time-to-fix estimates (`~1 hour`, `~2 hours`, `~4 hours`, `1-2 days`, etc.)
- Timeline or scheduling language (`immediately`, `next quarter`, `within 30 days`, `addressed within`)
- Effort duration labels (`(hours)`, `(days)`, `(weeks)`) after Low/Medium/High effort levels

**The report identifies WHAT to fix and WHY (tier + severity + effort level). It does NOT prescribe WHEN to fix it.** Scheduling is the team's responsibility. Only use `Low`, `Medium`, `High` for remediation effort — never attach time durations.

### Analysis Context & Assumptions Template

⚠️ **This ENTIRE section is REQUIRED.** Previous iterations skipped it entirely. Include ALL sub-sections below, even if tables are empty.

```値下げ
## 分析のコンテキストと仮定

### 分析範囲
|制約 |説明 |
|-----------|---------------|
|範囲 | [完全なリポジトリまたは特定の領域] |
|除外 | 【対象外となるもの】 |
|重点分野 | [特別な焦点がある場合] |

### インフラストラクチャコンテキスト
|カテゴリー |コードベースから発見 |影響を受ける調査結果 |
|----------|--------------------------|----------|

**「コードベースから発見」のすべてのエントリには、情報が推測されたソース ファイルまたはドキュメントへの相対リンクが含まれなければなりません。** 例:```
| Deployment Model | Air-gapped, single-admin workstation ([daemon.json](src/Container/Moby/daemon.json), [InstallAzureEdgeDiagnosticTool.ps1](src/Setup/InstallArtifacts/InstallAzureEdgeDiagnosticTool.ps1)) | All findings — no Tier 1 |
| Network Exposure | All services bind to localhost:80 only ([KustoContainerHelper.psm1](src/Container/Kusto/KustoContainerHelper.psm1)) | FIND-01, FIND-03 |
```### 確認が必要です
|アイテム |質問 |何を確認するか |なぜ不確実なのか |
|------|----------|---------------|---------------|

### オーバーライドの検索
| IDを探す |元の重大度 |オーバーライド |正当化 |新しいステータス |
|-----------|---------------------|----------|------|------------|
| — | — | — |オーバーライドは適用されません。レビュー後にこのセクションを更新します。 | — |

### 追加メモ
<!-- ユーザーのプロンプトからのその他のコンテキスト -->

[ユーザーが提供する自由形式のメモ]```

### References Consulted Template

> **⛔ CRITICAL: This section MUST have TWO subsections with THREE-column tables including full URLs. Do NOT flatten into a simple 2-column `| Reference | Usage |` table. The model ALWAYS tries to simplify this — do NOT simplify it.**

```値下げ
## 参照した参考文献

### セキュリティ基準
|標準 | URL |使用方法 |
|----------|-----|----------|
| Microsoft SDL バグ バー | https://www.microsoft.com/en-us/msrc/sdlbugbar |重大度分類 |
| OWASP トップ 10:2025 | https://owasp.org/Top10/2025/ |脅威の分類 |
| CVSS4.0 | https://www.first.org/cvss/v4.0/specation-document |リスクスコア |
| CWE | https://cwe.mitre.org/ |弱点分類 |
|ストライド | https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats |脅威の列挙方法 |
| NIST SP 800-53 Rev. 5 | https://csrc.nist.gov/pubs/sp/800-53/r5/upd1/final |コントロールマッピング |

### コンポーネントのドキュメント
|コンポーネント |ドキュメントの URL |関連セクション |
|-----------|---------------|------|
| [例: Dapr] | [例: https://docs.dapr.io/operations/security/] | [例: mTLS 構成] |
| [例: Redis] | [例: https://redis.io/docs/management/security/] | [例: 認証] |```

**Processing Rules:**
1. Always include the Security Standards table — populate with actual standards consulted
2. Every row MUST have a full URL (https://...) — never omit the URL column
3. Populate Component Documentation with technologies actually consulted during analysis
4. Do not add documentation that was not used

### Report Metadata Template

> **⛔ CRITICAL: ALL fields below are MANDATORY. Do NOT skip Model, Analysis Started, Analysis Completed, or Duration. The previous run omitted these — that is a critical failure. Run `Get-Date` at start and end to compute Duration.**

```値下げ
## レポートのメタデータ

|フィールド |値 |
|------|------|
|ソースの場所 | `[Full path]` |
| Git リポジトリ | `[Remote URL or "Unavailable"]` |
| Git ブランチ | `[Branch name or "Unavailable"]` |
| Git コミット | `[Short SHA]` (`[YYYY-MM-DD]` — `git log -1 --format="%cs" [SHA]` を実行してコミット日を取得します) |
|モデル | `[Model name — ask the system or state the model you are running as]` |
|マシン名 | `[hostname]` |
|分析開始 | `[UTC timestamp from command]` |
|分析が完了しました | `[UTC timestamp from command]` |
|期間 | `[Computed difference between started and completed]` |
|出力フォルダー | `[folder name]` |
|プロンプト | `[The user's prompt text that triggered this analysis]` |```

**Gathering rules:**
- START_TIME: Run `Get-Date -Format "yyyy-MM-dd HH:mm:ss" -AsUTC` at workflow Step 1
- END_TIME: Run again before writing 0-assessment.md
- Git fields: `git remote get-url origin`, `git branch --show-current`, `git rev-parse --short HEAD`
- If any command fails → "Unavailable"
- **NEVER estimate timestamps** from folder names
- Model: State the model you are currently running as (e.g., `Claude Opus 4.6`, `GPT-5.3 Codex`, `Gemini 3 Pro`)
- Machine: run `hostname`

### Coverage Counts Consistency

Before writing 0-assessment.md:
- Count elements from `1-threatmodel.md` Element Table
- Count findings from `3-findings.md`
- Count threats from `2-stride-analysis.md` summary table
- Use these exact numbers in Executive Summary and Action Summary

### Formatting Rules

1. `---` horizontal rules between every `##` section
2. Report Metadata values all wrapped in backticks
3. Finding Overrides always uses table format (even when empty)
4. Report Files section always first
5. `0.1-architecture.md` always listed in Report Files table

---

## Common Mistakes Checklist

These are the most observed deviations. Check after writing each file:

1. ❌ Organizing by severity → ✅ Organize by **Exploitability Tier**
2. ❌ Flat STRIDE tables → ✅ Split into Tier 1/2/3 sub-sections per component
3. ❌ Missing `Exploitability Tier` and `Remediation Effort` → ✅ MANDATORY on every finding
4. ❌ STRIDE summary missing T1/T2/T3 columns → ✅ Include T1|T2|T3 columns
5. ❌ Wrapping `.md` in ` ```markdown ` code fences → ✅ Start with `# Heading` on line 1. The `create_file` ツールは生のコンテンツを書き込みます。フェンスはファイル内のリテラル テキストになります。
6. ❌ `.mmd` を ` で囲む```plaintext ` or ` ```mermaid ` → ✅ Start with `%%{init:` on line 1. The `.mmd` ファイルは生の Mermaid ソースです。
7. ❌ アクションの概要が欠落しています → ✅ セクションのタイトルは正確に `## Action Summary` にする必要があります。 Tier 1 の低労力の結果の表を含む `### Quick Wins` サブセクションを含めなければなりません。
8. ❌ 脅威カウントのコンテキスト段落が欠落している → ✅ エグゼクティブサマリーに `> **Note on threat counts:**` ブロック引用符を含める
9. ❌ 空の層セクションを省略する → ✅ コンポーネントごとに 3 つの層すべてを常に含める
10. ❌ 個別の `### Key Recommendations` または `### Top Recommendations` または `### Priority Remediation Roadmap` を追加する → ✅ アクションの概要は推奨事項であり、他の名前はありません。
11. ❌ サイドカーを別のノードとして描画 → ✅ `diagram-conventions.md` ルール 1 を参照
12. ❌ CVSS 4.0 ベクトル文字列が欠落しています → ✅ すべての結果にはスコアと完全なベクトルの両方が必要です (例: `CVSS:4.0/AV:N/AC:L/...`)
13. ❌ 所見に CWE または OWASP がない → ✅ すべての所見に必須
14. ❌ OWASP `:2021` サフィックスを使用する → ✅ 常に `:2025` を使用します (例: `A01:2025 – Broken Access Control`)。 2025 年版が最新です。
15. ❌ 脅威カバレッジ検証テーブルが欠落しています → ✅ `3-findings.md` の末尾に必須
16. ❌ STRIDE 解析に含まれないアーキテクチャコンポーネント → ✅ 0.1-architecture.md 内のすべてのコンポーネントには STRIDE セクションが必要
17. ❌ 最上位シナリオのシーケンス図が欠落している → ✅ 0.1-architecture.md の最初の 3 つのシナリオには Mermaid シーケンス図が必要です
18. ❌ 0-assesment.md に「ニーズ検証」セクションが存在しない → ✅ 分析コンテキストと仮定の下に含める
19. ❌ `## Analysis Context & Assumptions` セクションが完全に欠落しています → ✅ 必須。前回の反復ではこのセクションをスキップしました。 「範囲」、「検証が必要」、および「オーバーライドの検索」サブテーブルを含める必要があります。
20. ❌ `### Quick Wins` サブセクションがありません → ✅ [アクションの概要] で必須です。 Tier 1 の低労力の調査結果をリストします。存在しない場合は、見出しに注記を含めます。
21. ❌ `## Report Files`、`## References Consulted`、または `## Report Metadata` をスキップ → ✅ 0-assessment.md の 7 つのセクションはすべて必須です。決して省略しないでください。
22. ❌ ID の検索順序が間違っている (FIND-04 の前の FIND-06) → ✅ ID の検索は上から下に連続していなければなりません: FIND-01、FIND-02、FIND-03、... ソート後に番号を付け直します。
23. ❌ ハイパーリンクのない CWE → ✅ CWE にはハイパーリンクを含める必要があります: `[CWE-306](https://cwe.mitre.org/data/definitions/306.html): Missing Authentication for Critical Function`
24. ❌ 出力での時間の見積もりやスケジュール設定 → ✅ いかなる出力ファイルでも `~1 hour`、`Sprint 1-2`、`Phase 1 — Immediate`、`(hours)`、またはタイムライン/期間を決して生成しないでください。レポートには、「いつ」ではなく、「何を修正するか」が記載されています。

---

## 脅威インベントリー.json

**目的:** すべてのコンポーネント、データ フロー、境界、脅威、および調査結果の構造化された JSON インベントリ。
このファイルにより、2 つの脅威モデルの実行間の自動比較が可能になります。**いつ生成するか:** 実行ごと (ステップ 8b)。すべてのマークダウン ファイルが書き込まれた後に生成されます。

**`0-assessment.md`** にはリンクされていません - これは機械が読み取り可能なアーティファクトであり、人間が読めるレポート ファイルではありません。

### スキーマ```json
{
  "schema_version": "1.0",
  "commit": "abc1234",
  "commit_date": "2025-08-15",
  "branch": "main",
  "analysis_timestamp": "2025-08-15T14:30:00Z",
  "repository": "https://github.com/org/repo",
  "report_folder": "threat-model-20250815-143000",

  "components": [
    {
      "id": "RedisStateStore",
      "display": "Redis State Store",
      "aliases": ["Redis", "StateStoreRedis"],
      "type": "data_store",
      "tmt_type": "SE.DS.TMCore.NoSQL",
      "boundary": "DataLayer",
      "boundary_kind": "ClusterBoundary",
      "source_files": ["helmchart/myapp/templates/redis-statefulset.yaml"],
      "fingerprint": {
        "component_type": "data_store",
        "boundary_kind": "ClusterBoundary",
        "source_files": ["helmchart/myapp/templates/redis-statefulset.yaml"],
        "source_directories": ["helmchart/myapp/templates/"],
        "class_names": [],
        "namespace": "",
        "api_routes": [],
        "config_keys": ["REDIS_HOST", "REDIS_PORT"],
        "dependencies": [],
        "inbound_from": ["InferencingFlow"],
        "outbound_to": [],
        "protocols": ["TCP"]
      },
      "sidecars": []
    }
  ],

  "boundaries": [
    {
      "id": "DataLayer",
      "display": "Data Layer",
      "aliases": ["Data Boundary", "Persistence Layer"],
      "kind": "ClusterBoundary",
      "contains": ["RedisStateStore", "VectorDB"],
      "contains_fingerprint": "RedisStateStore|VectorDB"
    }
  ],

  "flows": [
    {
      "id": "DF_InferencingFlow_to_Redis",
      "display": "DF25: InferencingFlow → Redis",
      "from": "InferencingFlow",
      "to": "RedisStateStore",
      "protocol": "TCP",
      "label": "State store operations",
      "bidirectional": true,
      "security": {
        "encryption": "none",
        "authentication": "none"
      }
    }
  ],

  "threats": [
    {
      "id": "T05.I",
      "identity_key": {
        "component_id": "RedisStateStore",
        "stride_category": "I",
        "attack_surface": "helmchart/values.yaml:redis.tls.enabled",
        "data_flow_id": "DF_InferencingFlow_to_Redis"
      },
      "title": "Information Disclosure — Redis unencrypted traffic",
      "description": "Redis state store transmits data without TLS...",
      "tier": 1,
      "prerequisites": "None",
      "affected_flow": "DF25",
      "mitigation": "Enable TLS on Redis connections",
      "status": "Open"
    }
  ],

  "findings": [
    {
      "id": "FIND-01",
      "identity_key": {
        "component_id": "RedisStateStore",
        "vulnerability": "CWE-306",
        "attack_surface": "helmchart/values.yaml:redis.auth"
      },
      "title": "Redis state store has no authentication",
      "severity": "Critical",
      "cvss_score": 9.4,
      "cvss_vector": "CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N",
      "cwe": "CWE-306",
      "owasp": "A07:2025",
      "tier": 1,
      "effort": "Low",
      "related_threats": ["T05.I", "T05.T"],
      "evidence_files": ["helmchart/myapp/values.yaml"],
      "component": "Redis State Store"
    }
  ],

  "metrics": {
    "total_components": 15,
    "total_flows": 30,
    "total_boundaries": 7,
    "total_threats": 97,
    "total_findings": 18,
    "threats_by_tier": { "T1": 12, "T2": 53, "T3": 32 },
    "findings_by_tier": { "T1": 7, "T2": 7, "T3": 4 },
    "findings_by_severity": { "Critical": 4, "Important": 8, "Moderate": 6 },
    "threats_by_stride": { "S": 14, "T": 19, "R": 8, "I": 20, "D": 15, "E": 14, "A": 7 }
  }
}
```> **⛔ stride_category は単一の文字である必要があります:** `S`、`T`、`R`、`I`、`D`、`E`、または `A`。 `"Spoofing"` や `"Denial of Service"` のようなフルネームは決して使用しないでください。ヒートマップの計算と比較マッチングは、単一文字コードに依存します。 `"stride_category": "D"` の代わりに `"stride_category": "Denial of Service"` を記述すると、ヒートマップでは STRIDE 列にはすべてゼロが表示されますが、階層列には正しい値が表示されます。これはデータ整合性に関する重大なバグです。

### 増分分析拡張機能

**増分分析**用に `threat-inventory.json` を生成する場合 (`incremental-orchestrator.md` を参照)、次のフィールドを追加します。

**トップレベルのフィールド:**
- `"incremental": true` — これを増分レポートとしてマークします
- `"baseline_report": "threat-model-20260309-174425"` — ベースライン レポート フォルダーへのパス
- `"baseline_commit": "2dd84ab"` — ベースライン レポートのコミット SHA
- `"target_commit": "abc1234"` — 分析中のコミット
- `"schema_version": "1.1"` — 増分レポートはスキーマ バージョン 1.1 を使用します

**コンポーネントごと:** `"change_status"` — 次のいずれか:
- `"unchanged"` — ソース ファイルが同一か、表面のみの変更
- `"modified"` — セキュリティ関連のソース ファイルの変更
- `"restructured"` — ファイルの移動/名前変更、同じ論理コンポーネント
- `"removed"` — ソース ファイルが削除されました
- `"new"` — コンポーネントはベースラインに存在しませんでした
- `"merged_into:{id}"` — 別のコンポーネントにマージ
- `"split_into:{id1},{id2}"` — 複数のコンポーネントに分割

**脅威ごと:** `"change_status"` — 次のいずれか:
- `"still_present"` — 以前と同様に、現在のコードに脅威が存在します
- `"fixed"` — 脆弱性は修正されました (コード変更を引用する必要があります)
- `"mitigated"` — 部分的な修復が適用されました
- `"modified"` — 脅威は依然として存在しますが、詳細は変更されました
- `"new_code"` — まったく新しいコンポーネントによる脅威
- `"new_in_modified"` — 既存のコンポーネントのコード変更によってもたらされる脅威
- `"previously_unidentified"` — 脅威はベースライン コードに存在しましたが、古いレポートには存在しませんでした
- `"removed_with_component"` — コンポーネントが削除されました

**検出結果ごと:** `"change_status"` — 脅威ごとと同じ値に次の値を加えます。
- `"partially_mitigated"` — コードが部分的に変更され、脆弱性が部分的に残っています

**metrics.status_summary** — `change_status` ごとのコンポーネント、脅威、検出結果の数。完全なスキーマについては、`incremental-orchestrator.md` §4f を参照してください。

### 正規の命名規則

**コンポーネント ID** — 実際のクラス/ファイル名から派生した PascalCase:
- `SupportabilityAgent.cs` → `SupportabilityAgent`
- `PowerShellCommandExecutor.cs` → `PowerShellCommandExecutor`
- 「Redis ステート ストア」 → `RedisStateStore`
- 「Ingress-NGINX」 → `IngressNginx`

**フロー ID** — エンドポイントから決定的:
- 形式：`DF_{Source}_to_{Target}`
- `DF_Operator_to_TerminalUI`
- `DF_InferencingFlow_to_RedisStateStore`**ID キー** - 各脅威と検出結果は正規の ID キーを取得します。
- 脅威: `component_id` + `stride_category` + `attack_surface` + `data_flow_id`
- 調査結果: `component_id` + `vulnerability` (CWE) + `attack_surface`
- これらのキーは、LLM で生成された散文から独立しています。コード アーティファクトにアンカーされます。

### 決定的アイデンティティ ルール (必須)

これらのルールを使用して、変更されていないコードを繰り返し実行すると、同等のインベントリが生成されます。

1. **正規 ID と表示名**
  - `id` は安定した ID です。 `display` はプレゼンテーションのテキストです
  - 調査結果や図のラベルの散文的な表現からアイデンティティを導き出さないでください

2. **エイリアスのキャプチャ**
  - すべてのコンポーネントと境界には `aliases` 配列が含まれている必要があります
  - アーキテクチャ/DFD/STRIDE/調査結果から発見された同義語を含めます (重複排除、ソート)
  - 実行ごとに表示文言が変わっても、正規の `id` を安定させます

3. **境界種類分類法 (TMT に準拠)**
  - このセットの `boundary_kind`/`kind` を使用します。信頼の移行の内容ではなく、性質を説明します。
    - `MachineBoundary` — 異なるホスト/VM間 (例: ホスト ↔ ゲスト、VM1 ↔ VM2)
    - `NetworkBoundary` — ネットワーク ゾーン間 (例: 企業 LAN ↔ インターネット、DMZ ↔ 内部)
    - `ClusterBoundary` — K8/コンテナクラスタと外部の間（例：クラスタ ↔ 外部サービス）
    - `ProcessBoundary` — 同じホスト上の OS プロセスまたはコンテナ間 (例: サイドカー ↔ メイン コンテナ)
    - `PrivilegeBoundary` — 異なる権限レベル間（例：ユーザーモード ↔ カーネル、非特権 ↔ 管理者）
    - `SandboxBoundary` — サンドボックス実行とサンドボックスなしの実行の間 (ブラウザ サンドボックス、WASM など)
  - それぞれの値は、「この線を越えると何が変わるでしょうか?」と答えます。 (別のマシン、ネットワーク、クラスター、プロセス、特権、サンドボックス)
  - コンポーネント グループ ラベル (DataStorage、ApplicationCore、AgentExecution) を境界の種類として使用しないでください。これらは、内部の内容を説明するものであり、信頼遷移の性質を説明するものではありません。3b. **境界 ID の導出** (必須 — コンポーネントと同じ決定論的な名前を適用します)
  - 抽象的な概念ではなく、展開/インフラストラクチャ名から境界 ID を導き出します。
    - Docker ホスト → `Docker` (決して `DockerEnvironment` や `ContainerRuntime` ではありません)
    - Kubernetes クラスター → `K8sCluster` (決して `KubernetesEnvironment` ではありません)
    - オペレータのマシン → `OperatorWorkstation` (決して `HostOS` や `LocalMachine` ではありません)
    - 外部クラウド サービス → `ExternalServices` (決して `CloudBoundary` ではありません)
    - グループ化されたデータ ストレージ → `DataStorage` (決して `DataLayer` や `PersistenceLayer` ではありません)
    - バックエンド アプリケーション サービス → `BackendServices` (決して `AppBoundary` や `ApplicationCore` ではありません)
    - ML/AI 推論モデル → `MLModels` (`InferenceModels` や `ModelBoundary` は使用しないでください)
    - DMZ/パブリックゾーン → `PublicZone` (`DMZBoundary` または `IngressZone` は使用しないでください)
    - エージェントの実行 → `AgentExecution` (この正確な ID を保持)
    - ツール実行 → `ToolExecution` (この正確な ID を保持)
  - ステップ 1 で境界 ID を選択したら、それをあらゆる場所 (DFD、テーブル、JSON) で使用します。
  - 同じコードの実行間で包含を再構築しないでください (同じコンポーネント→同じ境界)4. **コンポーネントのフィンガープリント**
  - `fingerprint` は安定した証拠に基づいて構築する必要があります。
    - ソート済み `source_files` — プライマリ ソース ファイルへの完全なファイル パス
    - ソートされた `source_directories` — ソース ファイルの親ディレクトリ パス (リファクタリング間でのファイル名よりも安定しています)
    - ソートされた `class_names` — コンポーネントのソース ファイルで定義されたプライマリ クラス、構造体、またはインターフェイス名 (例: `["HealthServer", "IHealthService"]`)。コード以外のコンポーネント (データストア、外部サービス) の場合は、空のままにします。
    - `namespace` — プライマリ名前空間/パッケージ (例: C# の場合は `"MCP.Core.Servers.Health"`、Python の場合は `"ragapp.src.ingestflow"`)。コード以外のコンポーネントの場合は空です。
    - ソートされた `api_routes` — このコンポーネントによって公開される HTTP API エンドポイント パターン (例: `["/api/health", "/api/v1/chat"]`)。 HTTP サービスでない場合は空です。
    - ソートされた `config_keys` — このコンポーネントによって使用される環境変数と構成キー (例: `["AZURE_OPENAI_ENDPOINT", "REDIS_HOST"]`)。 appsettings.json、.env ファイル、Helm 値、または環境変数を読み取るコードから抽出します。
    - ソートされた `dependencies` — このコンポーネントに固有の外部パッケージ/ライブラリの依存関係 (例: NuGet の場合は `["Microsoft.SemanticKernel", "Azure.AI.OpenAI"]`、pip の場合は `["pymilvus", "fastapi"]`)。フレームワーク全体の依存関係ではなく、このコンポーネントの特徴であるパッケージのみを含めます。
    - ソートされた `inbound_from` および `outbound_to` コンポーネント ID
    - `protocols` を並べ替えました
    - `component_type` および `boundary_kind`
  - フィンガープリントに可変散文を含めないでください
  - **決定的マッチング優先度:** `source_directories` > `class_names` > `namespace` > `api_routes` > `config_keys` はすべて、コンポーネントの名前が変更されても存続する非常に安定した信号です。これらのいずれかを共有する 2 つのコンポーネントは、ほぼ確実に同じ実コンポーネントです。**指紋フィールド → 比較一致信号マップ:**
  |指紋フィールド |比較信号 |最大ポイント |安定性 |
  |---|---|---|---|
  | `source_files` |信号 2 — ソース ファイル/ディレクトリの重複 | +30 |高 (ファイルはめったに移動しない) |
  | `source_directories` |信号 2 — ソース ファイル/ディレクトリの重複 | +25 |非常に高い (ディレクトリはほとんど変更されません) |
  | `class_names` |信号 3 — クラス/名前空間の一致 | +25 |非常に高い (クラス名が変更されることはほとんどありません) |
  | `namespace` |信号 3 — クラス/名前空間の一致 | +20 |非常に高い (名前空間は構造的) |
  | `api_routes` |シグナル 4 — API ルート / 構成キーの重複 | +15 |高 (API コントラクトはバージョン管理されています) |
  | `config_keys` |シグナル 4 — API ルート / 構成キーの重複 | +10 |高 (設定キーは安定しています) |
  | `dependencies` |シグナル 4 — API ルート / 構成キーの重複 | +5 |中 (アップグレードによりパッケージが変更されます) |
  | `inbound_from` / `outbound_to` |信号 5 — トポロジの重複 | +15 |低 (ドリフトする可能性のあるコンポーネント ID を使用) |
  | `component_type` + `boundary_kind` |信号 6 — タイプ + 境界の種類 | +10 |中 (境界の名前は異なる場合があります) |
  | `protocols` | (直接得点ではない - タイブレークとして使用) | — |中 |

  **このテーブルのすべてのフィールドは、分析中に入力する必要があります (ステップ 8b)。** フィールドが実際には適用されない場合は、空の配列 `[]` を使用できます (データストアの場合は `api_routes` など)。ただし、`source_directories` と `class_names` は、プロセス タイプのコンポーネントでは決して空にしてはなりません。これらは主に一致するアンカーです。

5. **境界封じ込めの指紋**
  - `contains_fingerprint` = ソートされた `contains` と `|` が結合
  - 比較時の境界変更の検出に使用します。

6. **決定的な順序付け**
  - JSON を書き込む前に、すべての配列とネストされたリスト フィールドを並べ替えます。
  - これにより差分が安定し、偶発的なチャーンが防止されます。

### 処理ルール1. すべてのマークダウン ファイルが書き込まれた後に生成します (ステップ 8b)
2. マークダウン ファイルの作成に使用したのと同じ分析データからデータを入力します。
3. コンポーネント ID が実際のクラス/ファイル名から派生した PascalCase を使用していることを確認します。
4. フロー ID が正規の `DF_{Source}_to_{Target}` 形式を使用していることを確認します。
5. すべての脅威と検出 ID キーは、実際のコード アーティファクト (ファイル パス、構成キー) を参照する必要があります。
6. ステップ 1 の git メタデータ (コミット、ブランチ、日付) を含めます。
7. `metrics` オブジェクトは、マークダウン レポートのカウントと一致する必要があります
8. このファイルは `0-assessment.md` のレポート ファイル テーブルにリストされていません。
9. 決定的マッチングのために `aliases`、`boundary_kind`/`kind`、`fingerprint`、および `contains_fingerprint` を入力します
10. コンポーネントに同じ実行で複数の監視名がある場合は、1 つの正規 `id` を保持し、すべての代替名を `aliases` に保存します。

> **⚠️ クリティカル — 配列の完全性:**
> `threats` 配列には、`2-stride-analysis.md` にリストされている脅威ごとに 1 つのエントリが含まれなければなりません。
> `findings` 配列には、`3-findings.md` の検出結果ごとに 1 つのエントリが含まれなければなりません。
> `components` 配列には、要素テーブル内のコンポーネントごとに 1 つのエントリが含まれなければなりません。
> **確認:** `threats.length == metrics.total_threats`、`findings.length == metrics.total_findings`、
>`components.length == metrics.total_components`。一致しない場合、JSON は不完全です - 戻ってください
> 不足しているエントリを追加します。スペースを節約するために配列を切り詰めないでください。

---

## セルフチェック — 各ファイルの書き込み後に実行

⛔ **必須:** 各ファイルを書き込んだ後、これらのチェックを確認し、結果を報告します。続行する前に、❌ を修正してください。

### `2-stride-analysis.md` の後:
- [ ] 概要テーブルは、個々のコンポーネント セクションの前に表示されます。
- [ ] コンポーネントごとに 3 層のサブセクション (層 1、層 2、層 3)
- [ ] ステータス列は次のみを使用します: `Open`、`Mitigated`、`Platform` (`Accepted Risk`、`Needs Review` は不可)
- [ ] 制限内のプラットフォーム比率 (≤20% スタンドアロン、≤35% K8s オペレーター)
- [ ] すべての脅威には 1 文字の STRIDE カテゴリ (S/T/R/I/D/E/A) があります。

### `3-findings.md` の後:
- [ ] 3 層見出し: `## Tier 1`、`## Tier 2`、`## Tier 3` (すべて存在)
- [ ] ファイル内のどこにも「受け入れられたリスク」がゼロ発生
- [ ] すべての検出結果には CVSS 4.0 ベクトル文字列が含まれています
- [ ] アクションの概要: T1=重大、T2=昇格、T3=中優先度
- [ ] 4 列目のヘッダーは「割り当てルール」です (「例」ではありません)。### `threat-inventory.json` の後:
- [ ] `threats.length == metrics.total_threats` (ゼロトレランス)
- [ ] `findings.length == metrics.total_findings` (ゼロトレランス)
- [ ] 脅威が 50 を超える場合、サブエージェント/Python/チャンクが使用されます — 単一の `create_file` ではありません
- [ ] すべてのコンポーネントには空ではない `fingerprint.source_directories` があります
- [ ] 正規キーでソートされた配列
- [ ] **フィールド名はスキーマと正確に一致します:** コンポーネントは `display` (`display_name` ではありません) を使用し、脅威は `stride_category` (`category` ではありません) を使用し、脅威→コンポーネントのリンクは `identity_key.component_id` 内にあります (トップレベルの `component_id` ではありません)、脅威には両方の `title` が含まれています(短い名前) と `description` (長い散文) — `description` だけではありません

### `0-assessment.md` の後:
- [ ] 正確に 7 つのセクション: レポート ファイル、エグゼクティブ サマリー、アクション サマリー、分析コンテキストと前提条件、参照した参考文献、レポート メタデータ、分類リファレンス
- [ ] `---` `##` セクションの各ペア間の水平罫線

---

## 列挙型リファレンス

すべてのレポートはこれらの正確な値を使用しなければなりません。省略したり、置き換えたり、代替案を発明したりしないでください。

**コンポーネント タイプ:** `process` | `data_store` | `external_service` | `external_interactor`

**境界の種類 (TMT に合わせて):** `MachineBoundary` | `NetworkBoundary` | `ClusterBoundary` | `ProcessBoundary` | `PrivilegeBoundary` | `SandboxBoundary`

**悪用可能性の階層:** `Tier 1` (直接暴露 — 前提条件なし) | `Tier 2` (条件付きリスク — 単一の前提条件) | `Tier 3` (多層防御 — 複数の前提条件)

**STRIDE + 悪用カテゴリ:** `S` なりすまし | `T` 改ざん | `R` 否認 | `I` 情報開示 | `D` サービス拒否 | `E` 権限の昇格 | `A` 虐待

**SDL バグバーの重大度:** `Critical` | `Important` | `Moderate` | `Low`

**修復作業:** `Low` | `Medium` | `High`

**緩和タイプ (OWASP に準拠):** `Redesign` | `Standard Mitigation` | `Custom Mitigation` | `Existing Control` | `Accept Risk` | `Transfer Risk`

**脅威ステータス:** `Open` | `Mitigated` | `Platform`

**変更ステータスの検索 (増分):** `Still Present` | `Fixed` | `New` | `New (Code)` | `New (Previously Unidentified)` | `Removed`

**OWASP Top 10:2025 サフィックス:** 常に `:2025` (例: `A01:2025 – Broken Access Control`)
- [ ] クイック ウィン、要検証、オーバーライドの検索サブセクションが存在します
- [ ] 導入パターンを文書化 (K8s オペレーターとスタンドアロン)
- [ ] バックティック内のすべてのメタデータ値**また確認してください (すべてのファイルに適用):** リークしたディレクティブ (⛔、RIGID、出力内の NON-NEGOTIABLE)、時間の見積もり、ネストされた出力フォルダーはありません。完全な共通偏差リストについては、`verification-checklist.md` フェーズ 0 を参照してください。