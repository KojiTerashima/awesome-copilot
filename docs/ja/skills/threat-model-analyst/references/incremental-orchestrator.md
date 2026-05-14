# 増分オーケストレーター — 脅威モデル更新ワークフロー

このファイルには、**増分脅威モデル分析**を実行するための完全なオーケストレーション ロジックが含まれており、既存のベースライン レポートに基づいて新しい脅威モデル レポートを生成します。ユーザーが更新された分析をリクエストし、以前の `threat-model-*` フォルダーが存在する場合に呼び出されます。

**単一分析 (`orchestrator.md`) との主な違い:** このワークフローは、コンポーネントを最初から検出するのではなく、古いレポートのコンポーネント インベントリ、ID、および規則を継承します。次に、各項目を現在のコードと照合して検証し、新しい項目を検出します。

## ⚡ コンテキスト バジェット — ファイルを選択的に読み取る

**フェーズ 1 (セットアップ + 変更検出):** このファイル (`incremental-orchestrator.md`) のみを読み取ります。古い `threat-inventory.json` は構造的なスケルトンを提供します。まだ他のスキル ファイルを読み取る必要はありません。
**フェーズ 2 (レポート生成):** 各ファイルを書き込む前に、`orchestrator.md` (必須ルール 1 ～ 34 の場合)、`output-formats.md`、`diagram-conventions.md`、および `skeletons/` からの関連するスケルトンを読み取ります。以下のインクリメンタル固有のルールを参照してください。
**フェーズ 3 (検証):** `verification-checklist.md` を使用してサブエージェントに委任します (HTML 比較のためのフェーズ 8 を含む、9 つのフェーズすべて)。

---

## このワークフローを使用する場合

次の条件がすべて満たされる場合は、増分分析を使用します。
1. ユーザーのリクエストには、脅威モデルの更新、再実行、またはリフレッシュが含まれます。
2. 有効な `threat-inventory.json` を持つ以前の `threat-model-*` フォルダーがリポジトリに存在します。
3. ユーザーは、ベースライン レポート フォルダーとターゲット コミット (デフォルトは HEAD) の両方を提供または暗黙的に指定します。

**トリガーの例:**
- 「threat-model-20260309-174425 をベースラインとして使用して脅威モデルを更新します」
- 「以前のレポートに対して増分脅威モデル分析を実行します」
- 「前回の脅威モデル以降、セキュリティに関して何が変わりましたか?」
- 「最新のコミットの脅威モデルを更新」

**このワークフローではありません:**
- 初回分析(ベースラインなし) → `orchestrator.md`を使用
- 以前のレポートについては言及せずに「このリポジトリのセキュリティを分析する」 → `orchestrator.md` を使用します

---

## 入力

|入力 |出典 |必須？ |
|----------|----------|----------|
|ベースライン レポート フォルダー | `threat-model-*` ディレクトリへのパス |はい |
|ベースライン `threat-inventory.json` | `{baseline_folder}/threat-inventory.json` |はい |
|ベースラインコミット SHA | `{baseline_folder}/0-assessment.md` より レポートのメタデータ |はい |
|ターゲットコミット |ユーザー指定の SHA、またはデフォルトの HEAD |はい (デフォルト: HEAD) |

---**⛔ サブエージェント ガバナンスはすべてのフェーズに適用されます。** `orchestrator.md` サブエージェント ガバナンスのセクションを参照してください。サブエージェントは読み取り専用のヘルパーです。レポート ファイルに対して `create_file` を呼び出すことはありません。

## フェーズ 0: セットアップと検証

1. **録画開始時間:**```
   Get-Date -Format "yyyy-MM-dd HH:mm:ss" -AsUTC
   ````START_TIME` として保存します。

2. **git 情報を収集します:**```
   git remote get-url origin
   git branch --show-current
   git rev-parse --short HEAD
   hostname
   ```3. **入力を検証します:**
   - ベースライン フォルダーが存在することを確認します: `Test-Path {baseline_folder}/threat-inventory.json`
   - `0-assessment.md` からベースライン コミット SHA を読み取ります: `| Git Commit |` 行を検索します
   - ターゲットのコミットが解決可能であることを確認します: `git rev-parse {target_sha}`
   - **コミット日の取得:** `git log -1 --format="%ai" {baseline_sha}` および `git log -1 --format="%ai" {target_sha}` — 今日の日付ではありません
   - **コード変更数の取得** (HTML メトリクス バーの場合):```
     git rev-list --count {baseline_sha}..{target_sha}
     git log --oneline --merges --grep="Merged PR" {baseline_sha}..{target_sha} | wc -l
     ````COMMIT_COUNT` および `PR_COUNT` として保存します。

4. **ベースライン コード アクセス — ワークツリーを再利用または作成します:**```
   # Check for existing worktree
   git worktree list
   
   # If a worktree for baseline_sha exists → reuse it
   # Verify: git -C {worktree_path} rev-parse HEAD
   
   # If not → create one:
   git worktree add ../baseline-{baseline_sha_short} {baseline_sha}
   ```後のフェーズで古いコードを検証できるように、ワークツリーのパスを `BASELINE_WORKTREE` として保存します。

5. **出力フォルダーを作成します:**```
   threat-model-{YYYYMMDD-HHmmss}/
   ```---

## フェーズ 1: 古いレポート スケルトンをロードする

ベースライン `threat-inventory.json` を読み取り、構造スケルトンを抽出します。```
From threat-inventory.json, load:
  - components[]  → all component IDs, types, boundaries, source_files, fingerprints
  - flows[]       → all flow IDs, from/to, protocols
  - boundaries[]  → all boundary IDs, contains lists
  - threats[]     → all threat IDs, component mappings, stride categories, tiers
  - findings[]    → all finding IDs, titles, severities, CWEs, component mappings
  - metrics       → totals for validation

Store as the "inherited inventory" — the structural foundation.
```**古いレポートのマークダウン ファイルの全文はまだ読まないでください**。構造化データのみをロードします。次の場合に、古いレポートの散文をオンデマンドで読んでください。
- 特定のコードパターンが以前に分析されたかどうかを確認する
- コンポーネントの役割または分類に関する曖昧さを解決する
- 所見ステータスの決定に必要な歴史的背景

---

## フェーズ 2: コンポーネントごとの変更検出

継承されたインベントリ内の各コンポーネントについて、その変更ステータスを確認します。```
For EACH component in inherited inventory:

  1. Check source_files existence at target commit:
     git ls-tree {target_sha} -- {each source_file}
  
  2. If ALL source files missing:
     → change_status = "removed"
     → Mark all linked threats as "removed_with_component"
     → Mark all linked findings as "removed_with_component"
  
  3. If source files exist, check for changes:
     git diff --stat {baseline_sha} {target_sha} -- {source_files}
     
     If NO changes → change_status = "unchanged"
     
     If changes exist, check if security-relevant:
       Read the diff: git diff {baseline_sha} {target_sha} -- {source_files}
       Look for changes in:
       - Auth/credential patterns (tokens, passwords, certificates)
       - Network/API surface (new endpoints, changed listeners, port bindings)
       - Input validation (sanitization, parsing, deserialization)
       - Command execution patterns (shell exec, process spawn)
       - Config values (TLS settings, CORS, security headers)
       - Dependencies (new packages, version changes)
       
       If security-relevant → change_status = "modified"
       If cosmetic only (whitespace, comments, logging, docs) → change_status = "unchanged"
  
  4. If files moved or renamed:
     git log --follow --diff-filter=R {baseline_sha}..{target_sha} -- {source_files}
     → change_status = "restructured"
     → Update source_file references to new paths
```**すべてのコンポーネントの分類を記録します** - これにより、下流のすべての意思決定が促進されます。

---

## フェーズ 3: 新しいコンポーネントをスキャンする```
1. Enumerate source directories/files at {target_sha} that are NOT referenced
   by any existing component's source_files or source_directories.
   Focus on: new top-level directories, new *Service.cs/*Agent.cs/*Server.cs classes,
   new Helm deployments, new API controllers.

2. Apply the same component discovery rules from orchestrator.md:
   - Class-anchored naming (PascalCase from actual class names)
   - Component eligibility criteria (crosses trust boundary or handles security data)
   - Same naming procedure (primary class → script → config → directory → technology)

3. For each candidate new component:
   - Verify it didn't exist at baseline: git ls-tree {baseline_sha} -- {path}
   - If it existed at baseline → this is a "missed component" from the old analysis
     → Add to Needs Verification section with note: "Component existed at baseline
       but was not in the previous analysis. May indicate an analysis gap."
   - If genuinely new (files didn't exist at baseline):
     → change_status = "new"
     → Assign a new component ID following the same PascalCase naming rules
     → Full STRIDE analysis will be performed in Phase 4
```---

## フェーズ 4: レポート ファイルの生成

次に、すべてのレポート ファイルを生成します。 **開始する前に、関連するスキル ファイルをお読みください:**
- `orchestrator.md` — 必須ルール 1 ～ 34 がすべてのレポート ファイルに適用されます
- `output-formats.md` — テンプレートと形式ルール
- `diagram-conventions.md` — 図の色とスタイル
- **各ファイルを書き込む前に、`skeletons/skeleton-*.md`** から対応するスケルトンを読み取ります — VERBATIM をコピーし、`[FILL]` プレースホルダーを埋めます

**⛔ サブエージェント ガバナンス (必須 - 二重フォルダーのバグを防止):** 親エージェントはすべてのファイル作成を所有します。サブエージェントは、コードを検索し、コンテキストを収集し、検証を実行する読み取り専用のヘルパーです。レポート ファイルに対して `create_file` を呼び出すことはありません。 `orchestrator.md` の完全なサブエージェント ガバナンス ルールを参照してください。唯一の例外は、大規模なリポジトリの `threat-inventory.json` 委任です。その場合でも、サブエージェント プロンプトには、正確な出力ファイル パスと、その 1 つのファイルのみを書き込むための明示的な指示が含まれている必要があります。

**⛔ 重要: 増分レポートはスタンドアロン レポートです。** 古いレポートを使用せずにこのレポートを読む場合は、完全なセキュリティ体制を理解する必要があります。ステータスの注釈 ([STILL PRESENT]、[FIXED]、[NEW CODE] など) は、完全なコンテンツの上に追加されるものであり、コンテンツを置き換えるものではありません。

＃＃＃４ａ． 0.1-アーキテクチャ.md

- **最初に `skeletons/skeleton-architecture.md` をお読みください** — 構造テンプレートとして使用します
- 古いレポートのコンポーネント構造を開始テンプレートとしてコピーします。
- **変更されていないコンポーネント:** 現在のコードを使用して説明を再生成します (古いレポートからコピーアンドペーストするのではありません)。同じ ID、同じ規則。
- **変更されたコンポーネント:** コードの変更を反映するために説明を更新します。注釈を追加: `[MODIFIED — security-relevant changes detected]`
- **新しいコンポーネント:** 注釈付きで追加: `[NEW]`
- **削除されたコンポーネント:** 注釈を追加: `[REMOVED]` と簡単なメモ
- 技術スタック、展開モデル: 変更された場合は更新、そうでない場合は繰り越し

  ⛔ **展開の分類は必須です (増分モードでも):**
  `0.1-architecture.md` には以下を含める必要があります:
  1. `**Deployment Classification:** \`[値]\`` line (e.g., `K8S_SERVICE`, `LOCALHOST_DESKTOP`)
  2. `### Component Exposure Table` 列: コンポーネント、リッスンオン、認証が必要、到達可能性、最小前提条件、派生層
  ベースラインにこれらが含まれている場合は、それらを引き継いで、新しい/変更されたコンポーネント用に更新します。
  ベースラインにこれ​​らが含まれていない場合は、**今すぐコードから派生**してください。これらは後続のすべてのステップで必要です。
  **これら 2 つの要素を準備せずにステップ 4b に進まないでください。**- シナリオ: 古いシナリオを保持し、新しい機能のために新しいシナリオを追加します。
- `output-formats.md` のすべての標準 `0.1-architecture.md` ルールが適用されます

＃＃＃４ｂ． 1.1-threatmodel.mmd (DFD)

- **最初に `skeletons/skeleton-dfd.md` と `skeletons/skeleton-summary-dfd.md` をお読みください**
- 古い DFD の論理レイアウトから開始します
- **繰越コンポーネントの同じノード ID** (ID の安定性にとって重要)
- **新しいコンポーネント:** 独特のスタイルで追加 — `classDef newComponent fill:#d4edda,stroke:#28a745,stroke-width:3px` を使用
- **削除されたコンポーネント:** 灰色の塗りつぶしで破線で表示 — `classDef removedComponent fill:#e9ecef,stroke:#6c757d,stroke-width:1px,stroke-dasharray:5` を使用
- **変更されていないフローの **同じフロー ID**
- **新しいフロー:** シーケンスを継続する新しい ID
- `diagram-conventions.md` のすべての標準 DFD ルールが適用されます (フローチャート LR、カラー パレットなど)

  ⛔ **POST-DFD GATE:** `1.1-threatmodel.mmd` を作成した後、要素と境界をカウントします。要素 > 15 または境界 > 4 の場合 → `skeleton-summary-dfd.md` を使用して `1.2-threatmodel-summary.mmd` を今すぐ作成します。決定が下されるまではステップ 4c に進まないでください。

＃＃＃４ｃ． 1-threatmodel.md

- **最初に `skeletons/skeleton-threatmodel.md` を読んでください** - テーブル構造を使用します
- 要素テーブル: すべての古い要素 + 新しい要素 (`Status` 列が追加)
  - 値: `Unchanged`、`Modified`、`New`、`Removed`、`Restructured`
- フロー テーブル: すべての古いフロー + 新しいフロー、`Status` 列あり
- 境界テーブル: 継承された境界 + 任意の新しい境界
- `1.2-threatmodel-summary.mmd` が生成された場合は、概要図とマッピング テーブルを含む `## Summary View` セクションを含めます
- `output-formats.md` のすべての標準テーブル ルールが適用されます

### 4d。 2-ストライド分析.md

- **最初に `skeletons/skeleton-stride-analysis.md` をお読みください** — サマリー テーブルとコンポーネントごとの構造を使用します

**⛔ インクリメンタルストライドに関する重要な注意事項 (`orchestrator.md` のルールがここでも同様に適用されます):**
1. **STRIDE-A の「A」は常に「不正使用」** (ビジネス ロジックの不正使用、ワークフローの操作、機能の不正使用)。 STRIDE-A カテゴリ名として「Authorization」を決して使用しないでください。これは、脅威 ID サフィックス (T01.A)、N/A 正当化ラベル、およびすべての散文に適用されます。承認の問題は、A カテゴリーではなく、特権昇格 (E) に分類されます。
2. **`## Summary` テーブルは、ファイルの先頭、`## Exploitability Tiers` の直後、個々のコンポーネント セクションの前に存在する必要があります**。この正確な構造を上部で使用します。```markdown
# STRIDE-A Threat Analysis

## Exploitability Tiers
| Tier | Label | Prerequisites | Assignment Rule |
|------|-------|---------------|----------------|
| **Tier 1** | Direct Exposure | `None` | Exploitable by unauthenticated external attacker with NO prior access. |
| **Tier 2** | Conditional Risk | Single prerequisite | Requires exactly ONE form of access. |
| **Tier 3** | Defense-in-Depth | Multiple prerequisites or infrastructure access | Requires significant prior breach or multiple combined prerequisites. |

## Summary
| Component | Link | S | T | R | I | D | E | A | Total | T1 | T2 | T3 | Risk |
|-----------|------|---|---|---|---|---|---|---|-------|----|----|----|------|
<!-- one row per component with numeric counts, then Totals row -->

---
## [First Component Name]
```3. **STRIDE カテゴリでは、コンポーネントごとに 0、1、2、3+ の脅威が生成される可能性があります**。カテゴリごとに 1 つの脅威を上限にしないでください。豊富なセキュリティ面を備えたコンポーネントには、通常、関連するカテゴリごとに 2 ～ 4 つの脅威が存在するはずです。概要テーブルのすべての STRIDE セルが 0 または 1 の場合、分析は浅すぎます。戻って追加の脅威ベクトルを特定します。概要テーブルの列には、実際の脅威数が反映されています。
4. **⛔ 前提条件のフロア チェック (脅威ごと):** 脅威に前提条件を割り当てる前に、コンポーネント エクスポージャー テーブル (`0.1-architecture.md`) でコンポーネントの `Min Prerequisite` および `Derived Tier` を検索します。脅威の前提条件はコンポーネントのフロア以上である必要があります。脅威の層は、コンポーネントの派生層以上である必要があります。 `analysis-principles.md` の正規の前提条件→階層マッピングを使用します。前提条件では、正規の値のみを使用する必要があります: `None`、`Authenticated User`、`Privileged User`、`Internal Network`、`Local Process Access`、`Host/OS Access`、`Admin Credentials`、`Physical Access`、`{Component} Compromise`。 ⛔ `Application Access` および `Host Access` は禁止されています。

**⛔ 見出しアンカー規則 (すべての出力ファイルに適用):** すべての出力ファイルのすべての `##` および `###` 見出しはプレーン テキストである必要があります。見出しテキストにはステータス タグ (`[Existing]`、`[Fixed]`、`[Partial]`、`[New]`、`[Removed]`、または古い形式のタグ) は使用できません。タグはマークダウンのアンカー リンクを破壊し、目次を汚します。代わりに、ステータスの注釈をセクション/結果本文の最初の行に配置します。
- ✅ `## KmsPluginProvider` と最初の行 `> **[New]** Component added in this release.`
- ✅ `### FIND-01: Missing Auth Check` と最初の行 `> **[Existing]**`
- ❌ `## KmsPluginProvider [New]` (`#kmspluginprovider` アンカーを中断)
- ❌ `### FIND-01: Missing Auth Check [Existing]` (見出しを汚します)

このルールは、`0.1-architecture.md`、`2-stride-analysis.md`、`3-findings.md`、`1-threatmodel.md` に適用されます。

各コンポーネントの STRIDE 分析アプローチは、その変更ステータスによって異なります。

|コンポーネントのステータス | STRIDEアプローチ |
|-----------------|-----------------|
| **変更なし** | `[STILL PRESENT]` アノテーションが付いた古いレポートのすべての脅威エントリを引き継ぎます。現在のコードに対して各脅威の軽減ステータスを再検証します。 |
| **修正済み** | diff にアクセスしてコンポーネントを再分析します。古い脅威ごとに、`still_present`、`fixed`、`mitigated`、または `modified` のいずれかを判断します。コードの変更から新たな脅威を発見 → `new_in_modified` として分類。 |
| **新規** |完全に新鮮な STRIDE-A 解析 (単一解析モードと同じ)。すべての脅威は `new_code` として分類されます。 |
| **削除されました** |セクションヘッダーの注:「コンポーネントが削除されました — すべての脅威は `removed_with_component` ステータスで解決されました。」 |**脅威 ID の継続性:**
- 古い脅威は元の ID を保持します (例: T01.S、T02.T)
- 新しい脅威は、古いレポートの最大の脅威番号からのシーケンスを継続します。
- 古い脅威 ID を再割り当てしたり再利用したりしないでください。

**N/A カテゴリ (PRD の §3.7 より):**
- 各コンポーネントは 7 つの STRIDE-A カテゴリすべてに対応します
- 該当しないカテゴリ：`N/A — {1-sentence justification}`
- N/A エントリは脅威の合計にはカウントされません

**STRIDE テーブルのステータス注釈形式:**
次のいずれかを使用して、脅威テーブルの各行に `Change` 列を追加します。
- `Existing` — 脅威は以前と同様に現在のコードに存在します (詳細が若干変更された脅威を含む)
- `Fixed` — 脆弱性が修正されました (特定のコード変更を引用)
- `New` — 新しいコンポーネント、コード変更、または未確認の脅威による脅威
- `Removed` — コンポーネントが削除されました

<!-- 簡略化された表示タグ: マークダウン本文テキストに表示されるタグは 5 つだけです。
  [既存] = Still_present、修正済み、軽減済み (脅威はまだ存在します)
  [修正済み] = 修正済み (完全に修正済み)
  [部分的] = 部分的に軽減されました (コードは変更されましたが、脆弱性は軽減された形のままです)
  [新規] = new_code、new_in_modified、previous_unidentified (このレポートの新規)
  [削除] = Removed_with_component (コンポーネントが削除されました)
  JSON のchange_status は、プログラムで使用するための詳細な値を保持します。 -->

⛔ 事後チェック: すべての脅威の [変更] 列を書き込んだ後、次のことを確認します。
  1. すべての脅威行には、既存、修正済み、新規、削除済みのいずれかが含まれます。
  2. 古いスタイルのタグは使用しない: まだ存在する、新しい (コード)、新しい (変更された)、以前は識別されていなかった
  3. 修正された脅威は特定のコード変更を引用しています

＃＃＃４ｅ． 3-所見.md

⛔ **調査結果を書き込む前に、今すぐ `skeletons/skeleton-findings.md` を読み直してください。**
スケルトンは、`#### Evidence` セクションの必須の `**Prerequisite basis:**` 行を含む、各検索ブロックの正確な構造を定義します。すべての所見は、[既存]、[新規]、[修正]、または [部分] のいずれであっても、このスケルトン構造に従わなければなりません。⛔ **展開コンテキスト ゲート (失敗クローズ) — すべての結果 (新規および繰り越し) に適用されます:**
`0.1-architecture.md` 展開分類とコンポーネント公開表を参照してください。
分類が `LOCALHOST_DESKTOP` または `LOCALHOST_SERVICE` の場合:
- ゼロの結果は `Exploitation Prerequisites` = `None` になる可能性があります → `Local Process Access` または `Host/OS Access` に修正します
- `## Tier 1` にゼロの検出結果がある可能性があります → T2/T3 にダウングレード
- コンポーネントに `Reachability = External` がない限り、ゼロ CVSS ベクトルは `AV:N` を使用できます。
すべての分類の場合:
- 各検出結果の前提条件は、暴露表のコンポーネントの `Min Prerequisite` 以上である必要があります。
- 各検出結果の階層は、そのコンポーネントの `Derived Tier` 以上である必要があります。
- **すべての検出結果の `#### Evidence` セクションは、前提条件を決定する特定のコード/構成を引用する `**Prerequisite basis:**` 行で始まらなければなりません (例: 「ClusterIP サービス、イングレスなし - 公開テーブルごとの内部のみ」)。これは、[既存の] 結果にも当てはまります。現在のコードから再派生します。
- 前提条件では正規の値のみを使用しなければなりません。 ⛔ `Application Access` および `Host Access` は禁止されています。

古い検出結果ごとに、現在のコードと照合して検証します。

|状況 |変更ステータス |アクション |
|----------|------|----------|
|コードは変更されず、脆弱性はそのまま | `still_present` |本文の最初の行に `> **[Existing]**` を付けて次へ進みます。
|脆弱性を修正するためにコードが変更されました。 `fixed` | `> **[Fixed]**` でマークし、特定のコード変更を引用します。
|コードが部分的に変更されました | `partially_mitigated` | `> **[Partial]**` でマークを付け、何が変更され、何が残っているのか説明してください |
|コンポーネントが完全に削除されました | `removed_with_component` | `> **[Removed]**` でマーク |

新しい発見については、次のとおりです。

|状況 |変更ステータス |ラベル |
|-----------|------|------|
|新しいコンポーネント、新しい脆弱性 | `new_code` | `> **[New]**` |
|既存のコンポーネント、コード変更により脆弱性が発生 | `new_in_modified` | `> **[New]**` — 具体的な変更点を引用する |
|既存のコンポーネント、古いコードに脆弱性がありましたが、見つかりませんでした | `previously_unidentified` | `> **[New]**` — ベースラインのワークツリーに対して検証します |

<!-- ⛔ ステップ後のチェック: すべての検索結果の注釈を書き込んだ後:
  1. すべての所見本文は、[既存]、[修正]、[部分]、[新規]、[削除] のいずれかで始まります。
  2. タグは、### 見出しではなく、ブロック引用符 (> **[タグ]**) として本文に含まれます。
  3. 古いスタイルのタグはありません: [まだ存在します]、[新しいコード]、[変更された新しい]、[以前は未確認]、[部分的に緩和されました]、[コンポーネントとともに削除されました]
  4. JSON change_status は、プログラムによる比較に詳細な値 (still_present、new_code など) を使用します -->**ID の連続性の検索:**
- 古い結果は元の ID (FIND-01 から FIND-N) を保持します。
- 新しい発見は、FIND-N+1、FIND-N+2、... という順序で続きます。
- ギャップや重複はありません
- 修正された結果は保持されますが、注釈が付けられます。レポートからは削除されません。
- **ドキュメントの順序**: 調査結果は、階層 (1→2→3)、次に重大度 (重大→重要→中程度→低)、次に CVSS の降順で並べ替えられます。これはスタンドアロン分析と同じです。古い ID が保存されるため、ドキュメント内では ID 番号が昇順にならない場合があります。これはインクリメンタル モードで許容されます。クロスレポート トレースの ID の安定性は、シーケンシャルな順序付けよりも優先されます。 `### FIND-XX:` 見出しは、ID 順ではなく層/重大度順に表示されます。

**これまで未確認だった検証手順:**
1. 調査結果のコンポーネントと証拠ファイルを特定する
2. ベースラインコミット時に同じファイルを読み取ります: `cat {BASELINE_WORKTREE}/{file_path}`
3. 旧コードに脆弱性パターンが存在する場合 → `previously_unidentified`
4. 脆弱性パターンが古いコードに存在しない場合 → `new_in_modified`

### 4f。脅威インベントリ.json

- **最初に `skeletons/skeleton-inventory.md` をお読みください** — 正確なフィールド名とスキーマ構造を使用してください

単一分析と同じスキーマにフィールドが追加されています。```json
{
  "schema_version": "1.1",
  "incremental": true,
  "baseline_report": "threat-model-20260309-174425",
  "baseline_commit": "2dd84ab",
  "target_commit": "abc1234",
  
  "components": [
    {
      "id": "McpHost",
      "change_status": "unchanged",
      ...existing fields...
    }
  ],
  
  "threats": [
    {
      "id": "T01.S",
      "change_status": "still_present",
      ...existing fields...
    }
  ],
  
  "findings": [
    {
      "id": "FIND-01",
      "change_status": "still_present",
      ...existing fields...
    }
  ],
  
  "metrics": {
    ...existing fields...,
    "status_summary": {
      "components": {
        "unchanged": 15,
        "modified": 2,
        "new": 1,
        "removed": 1,
        "restructured": 0
      },
      "threats": {
        "still_present": 80,
        "fixed": 5,
        "mitigated": 3,
        "new_code": 10,
        "new_in_modified": 4,
        "previously_unidentified": 2,
        "removed_with_component": 8
      },
      "findings": {
        "still_present": 12,
        "fixed": 2,
        "partially_mitigated": 1,
        "new_code": 3,
        "new_in_modified": 2,
        "previously_unidentified": 1,
        "removed_with_component": 1
      }
    }
  }
}
```### 4g。 0-評価.md

- **最初に `skeletons/skeleton-assessment.md` を読んでください** — セクションの順序とテーブル構造を使用します

標準評価セクション (7 つすべて必須) と増分固有のセクション:

**標準セクション (単一分析と同じ):**
1. レポートファイル
2. エグゼクティブ サマリー (`> **Note on threat counts:**` ブロック引用付き)
3. アクションの概要 (`### Quick Wins` を使用)
4. 分析コンテキストと仮定 (`### Needs Verification` および `### Finding Overrides` を使用)
5. 参照した参考文献
6. レポートのメタデータ
7. 分類リファレンス (スケルトンからコピーされた静的テーブル)

**追加の増分セクション (アクションの概要と分析コンテキストの間に挿入):**```markdown
## Change Summary

### Component Changes
| Status | Count | Components |
|--------|-------|------------|
| Unchanged | X | ComponentA, ComponentB, ... |
| Modified | Y | ComponentC, ... |
| New | Z | ComponentD, ... |
| Removed | W | ComponentE, ... |

### Threat Status
| Status | Count |
|--------|-------|
| Still Present | X |
| Fixed | Y |
| New (Code) | Z |
| New (Modified) | M |
| Previously Unidentified | W |
| Removed with Component | V |

### Finding Status
| Status | Count |
|--------|-------|
| Still Present | X |
| Fixed | Y |
| Partially Mitigated | P |
| New (Code) | Z |
| New (Modified) | M |
| Previously Unidentified | W |
| Removed with Component | V |

### Risk Direction
[Improving / Worsening / Stable] — [1-2 sentence justification based on status distribution]

---

## Previously Unidentified Issues

These vulnerabilities were present in the baseline code at commit `{baseline_sha}` but were not identified in the prior analysis:

| Finding | Title | Component | Evidence |
|---------|-------|-----------|----------|
| FIND-XX | [title] | [component] | Baseline code at `{file}:{line}` |
```**レポート メタデータの追加:**```markdown
| Baseline Report | `{baseline_folder}` |
| Baseline Commit | `{baseline_sha}` (`{baseline_commit_date}` — run `git log -1 --format="%cs" {baseline_sha}`) |
| Target Commit | `{target_sha}` (`{target_commit_date}` — run `git log -1 --format="%cs" {target_sha}`) |
| Baseline Worktree | `{worktree_path}` |
| Analysis Mode | `Incremental` |
```### 4時間。インクリメンタル比較.html

- **最初に `skeletons/skeleton-incremental-html.md` を読んでください** — 8 セクション構造と CSS 変数を使用します

比較を視覚化する自己完結型 HTML ファイルを生成します。すべてのデータは、`threat-inventory.json` ですでに計算されている `change_status` フィールドから取得されます。

**構造：**```html
<!-- Section 1: Header + Comparison Cards -->
<div class="header">
  <div class="report-badge">INCREMENTAL THREAT MODEL COMPARISON</div>
  <h1>{{repo_name}}</h1>
</div>
<div class="comparison-cards">
  <div class="compare-card baseline">
    <div class="card-label">BASELINE</div>
    <div class="card-hash">{{baseline_sha}}</div>
    <div class="card-date">{{baseline_commit_date from git log}}</div>
    <div class="risk-badge">{{old_risk_rating}}</div>
  </div>
  <div class="compare-arrow">→</div>
  <div class="compare-card target">
    <div class="card-label">TARGET</div>
    <div class="card-hash">{{target_sha}}</div>
    <div class="card-date">{{target_commit_date from git log}}</div>
    <div class="risk-badge">{{new_risk_rating}}</div>
  </div>
  <div class="compare-card trend">
    <div class="card-label">TREND</div>
    <div class="trend-direction">{{Improving|Worsening|Stable}}</div>
    <div class="trend-duration">{{N months}}</div>
  </div>
</div>

<!-- Section 2: Metrics Bar (5 boxes — NO Time Between, use Code Changes) -->
<div class="metrics-bar">
  Components: {{old_count}} → {{new_count}} (±N)
  Trust Boundaries: {{old_boundaries}} → {{new_boundaries}} (±N)
  Threats: {{old_count}} → {{new_count}} (±N)  
  Findings: {{old_count}} → {{new_count}} (±N)
  Code Changes: {{COMMIT_COUNT}} commits, {{PR_COUNT}} PRs
</div>

<!-- Section 3: Status Summary Cards (colored cards — primary visualization) -->
<div class="status-cards">
  <!-- Green card: Fixed (count + list of fixed items) -->
  <!-- Red card: New (code + modified) (count + list of new items) -->
  <!-- Amber card: Previously Unidentified (count + list) -->
  <!-- Gray card: Still Present (count) -->
</div>

<!-- Section 4: Component Status Grid -->
<table class="component-grid">
  <!-- Row per component: ID | Type | Status (color-coded) | Source Files -->
</table>

<!-- Section 5: Threat/Finding Status Breakdown -->
<div class="status-breakdown">
  <!-- Grouped by status: Fixed items, New items, etc. -->
  <!-- Each item: ID | Title | Component | Status -->
</div>

<!-- Section 6: STRIDE Heatmap with Deltas -->
<!-- ⛔ MANDATORY: Heatmap MUST have 13 columns including T1/T2/T3 after a divider -->
<table class="stride-heatmap">
  <thead>
    <tr>
      <th>Component</th>
      <th>S</th><th>T</th><th>R</th><th>I</th><th>D</th><th>E</th><th>A</th>
      <th>Total</th>
      <th class="divider"></th>
      <th>T1</th><th>T2</th><th>T3</th>
    </tr>
  </thead>
  <tbody>
    <!-- Row per component. Each STRIDE cell: value (▲+N or ▼-N delta from baseline) -->
    <!-- The divider column is a thin visual separator between STRIDE totals and tier breakdown -->
  </tbody>
</table>

<!-- Section 7: Needs Verification -->
<div class="needs-verification">
  <!-- Items where analysis disagrees with old report -->
</div>

<!-- Section 8: Footer -->
<div class="footer">
  Model: {{model}} | Duration: {{duration}}
  Baseline: {{baseline_folder}} at {{baseline_sha}}
  Generated: {{timestamp}}
</div>
```**スタイリングのルール:**
- 自己完結型: インライン `<style>` ブロック内のすべての CSS。 CDN リンクはありません。
- 色の規則: 緑 (#28a745) = 修正、赤 (#dc3545) = 新しい脆弱性、琥珀 (#fd7e14) = 以前は特定されていなかった、灰色 (#6c757d) = 現在も存在、青 (#2171b5) = 修正
- 印刷用: `@media print` スタイルを含める
- 視覚的な一貫性を保つために、上で定義したものと同じ CSS 色の規則を使用します。

---

## フェーズ 5: 検証

＃＃＃５ａ．標準検証

新しいレポートに対して標準の `verification-checklist.md` (フェーズ 0 ～ 9) を実行します。増分レポートはスタンドアロン レポートであるため、すべての標準品質チェックに合格する必要があります。 **出力フォルダーの絶対パス**を使用してサブエージェントに委任し、レポート ファイルを読み取れるようにします。

＃＃＃５ｂ．増分検証

標準検証に合格したら、`experiment-history/mode-c-verification-suite.md` から増分固有のチェックを実行します (フェーズ 1 ～ 9、33 チェック)。これらは以下を検証します。
- 構造の連続性 (古い項目はすべて考慮されています)
- コードで検証されたステータスの正確さ (例: コードの差分に対して実際に検証された「修正済み」)
- 以前は識別されていなかった分類 (ベースライン ワークツリーに対して検証済み)
- DFD の一貫性 (古いノードが存在し、新しいノードが区別される)
- スタンドアロンの品質 (古いレポートへのぶら下がり参照なし)
- 比較概要の正確性 (カウントと在庫の一致)
- 完全な検証が必要
- エッジケース (マージ、分割、リライト)
- メトリクス/JSON の整合性

＃＃＃５ｃ．修正ワークフロー

1. すべての合否結果を収集する
2. FAIL ごとに → チェックの「Fail remediation」アクションを適用します。
3. 失敗したチェックを再実行して、合格することを確認します。
4. 2 回修正を試みた後、残りの失敗を「要確認」にエスカレーションします。
5. 終了時刻を記録し、実行概要を生成する

---

## ⛔ 増分分析に固有のルール

これらのルールは、`orchestrator.md` の 34 個の必須ルールを補足します (置き換えるのではなく)。

### ルール I1: 古いレポート評価の判断は保存される

新しい分析で古いレポートとは異なる TMT カテゴリ、コンポーネント タイプ、層、または脅威の関連性が割り当てられる場合 → 古いレポートの値を保持します。意見の相違を「要確認」に記録します。
- 古い値
- 新しい分析の提案値
- 1～2文の推論
- ユーザーが確認すべきこと

**例外:** 事実の修正 (ファイル パス、git メタデータ、算術演算) はサイレントに修正され、レポート メタデータに記録されます。

### ルール I2: サイレント オーバーライドの禁止レポート本文では評価判定にOLD値を使用しています。意見の相違がある場合は、「要確認」に進みます。ユーザーは再分類を明示的に確認する必要があります。

### ルール I3: 以前は識別されていなかったものは検証される必要がある

すべての `previously_unidentified` 分類には、ベースライン ワークツリーからの証拠が含まれなければなりません。アナリストは、引用されたファイル/行の古いコードを実際に読んで、脆弱性パターンが存在することを確認する必要があります。 「おそらくそこにあっただろう」という推測に基づくものではありません。

### ルール I4: 修正はコード検証する必要がある

すべての `fixed` 分類では、脆弱性に対処した特定のコード変更を引用する必要があります。 「チームがこれを修正した」などの一般的な記述は受け入れられません。差分を表示してください。

### ルール I5: new_in_modified には変更属性が必要です

`new_in_modified` の検出結果はすべて、脆弱性をもたらした特定のコード変更を特定しなければなりません。問題を引き起こした差分ハンク、新しい関数、新しい設定値、または新しい依存関係を引用します。

### ルール I6: ベースライン ワークツリーを削除しない

ベースライン ワークツリーは、将来の増分分析で再利用できます。 `git worktree remove` を実行しないでください。ワークツリーのパスは、参照用にレポート メタデータに記録されます。

### ルール I7: 変更ステータスの一貫性

コンポーネントの `change_status` は、その脅威および検出結果のステータスと一致している必要があります。
- `unchanged` コンポーネント → その脅威は `still_present` (または、変更されていないコードで新しく発見された脅威の場合は `previously_unidentified`) である必要があります。
- `removed` コンポーネント → その脅威/調査結果はすべて `removed_with_component` である必要があります
- `modified` コンポーネント → 少なくとも 1 つの脅威は `modified`、`fixed`、または `new_in_modified` である必要があります
- `new` コンポーネント → その脅威はすべて `new_code` である必要があります

### ルール I8: 繰り越し、コピーしない

「繰り越し」とは、同じ内容の脅威/検出エントリを再生成することを意味し、文字通り古いレポート テキストをコピー＆ペーストすることではありません。再生成されたエントリは次のとおりです。
- 同じIDを使用する
- 現在のファイル パスを参照します (変更されていない場合でも)
- 現在のコードについて現在形で表現する
- `[STILL PRESENT]` アノテーションを含めます

---

## 概要: 段階ごとのチェックリスト|フェーズ |アクション |成功基準 |
|------|--------|------|
| 0 |セットアップ、入力の検証、ワークツリー |すべての入力が存在し、ワークツリーにアクセス可能 |
| 1 |古いインベントリ スケルトンをロードする |すべての配列にデータが入力され、メトリクスが一致します。
| 2 |コンポーネントごとの変更検出 |すべてのコンポーネントには `change_status` |
| 3 |新しいコンポーネントをスキャンする |新しいコンポーネントが特定され、不足しているコンポーネントにフラグが付けられました。
| 4 |すべてのレポート ファイルを生成する | 8 ～ 9 個のファイルが出力フォルダーに書き込まれます。
| 5 |検証 (標準 + 増分) |すべてのチェックは合格するか、「検証が必要」にエスカレーションされます。