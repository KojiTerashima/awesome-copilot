# 検証チェックリスト — 分析後の品質ゲート

このファイルは、脅威モデル レポートが完成する前に合格する必要があるすべての検証ルールの **唯一の信頼できる情報源**です。これは、出力フォルダーのパスとともに検証サブエージェントに渡されるように設計されています。

> **権限階層:** このファイルには、CHECKING ルール (品質ゲートの合否基準) が含まれています。チェック対象のコンテンツを生成する AUTHORING ルールは `orchestrator.md` にあります。一部のルールは、可視性のために両方のファイルに表示されます。矛盾する場合は、`orchestrator.md` がオーサリングの決定 (書き方) に優先され、このファイルが合否基準 (有効な出力を構成するもの) に優先されます。 「重複排除」するためにどちらのファイルからもルールを削除しないでください。重複は可視化のために意図的に行われています。

**使用する場合:** すべての出力ファイル (0.1-architecture.md から 0-assessment.md) が書き込まれた後、このファイル内のすべてのチェックを実行します。いずれかのチェックが失敗した場合は、完了する前に問題を修正してください。

**サブエージェントの委任:** オーケストレーターは、次のプロンプトを使用して、このファイル全体を検証サブエージェントに委任できます。
> 「[verification-checklist.md](./verification-checklist.md) を読みます。チェックごとに、指定された出力ファイルを検査し、証拠とともに PASS/FAIL を報告します。失敗があれば修正してください。」

---

## インライン クイック チェック (各ファイルの書き込み直後に実行)

> **目的:** これらは、WRITING エージェントが各ファイルの作成直後に実行する軽量のセルフチェックです。ステップ 10 には延期されません。エージェントはファイルを書き込んだばかりなので、コンテンツはまだアクティブなコンテキストにあり、これらのチェックは非常に効果的です。
>
> **使用方法:** 各ファイルを書き込む前に、`skeletons/skeleton-*.md` から対応するスケルトンを読み取ります。 `create_file` を呼び出すたびに、これらのパターンについて書き込んだコンテンツをスキャンします。いずれかのチェックが失敗した場合は、次の手順に進む前にすぐにファイルを修正してください。
>
> **スケルトンの準拠規則:** すべての出力ファイルは、スケルトンのセクション順序、テーブルの列ヘッダー、および見出し名に従わなければなりません。スケルトンにないセクションやテーブルを追加しないでください。スケルトンの見出しの名前は変更しないでください。### `3-findings.md` を書き込んだ後:
- [ ] 最初に見つかった見出しは `### FIND-01:` で始まります (`F01`、`F-01`、または `Finding 1` ではありません)
- [ ] すべての検出結果には次の正確な行ラベルが付いています: `SDL Bugbar Severity`、`Remediation Effort`、`Mitigation Type`、`Exploitability Tier`、`Exploitation Prerequisites`、`Component`
- [ ] すべての CVSS 値には `CVSS:4.0/` プレフィックスが含まれます
- [ ] すべての `Related Threats` セルには `](2-stride-analysis.md#` (プレーン テキストではなくハイパーリンク) が含まれます
- [ ] すべての結果には `#### Description`、`#### Evidence`、`#### Remediation`、および `#### Verification` の小見出しがあります (`Recommendation`、`Impact`、`Mitigation`、太字の `**Description:**` 段落ではありません) — 正確に 4 つの小見出し、余分なものはありません
- [ ] 各 `#### Description` セクションには、技術的な詳細が少なくとも 2 文含まれています (単一文のスタブではありません)。
- [ ] 各 `#### Evidence` セクションでは、特定のファイル パス、行番号、または設定キーが引用されています (「コードベースで見つかりました」のような一般的なステートメントではありません)。
- [ ] すべての検出結果には 10 個の必須属性行がすべて含まれています: `SDL Bugbar Severity`、`CVSS 4.0`、`CWE`、`OWASP`、`Exploitation Prerequisites`、`Exploitability Tier`、`Remediation Effort`、`Mitigation Type`、`Component`、`Related Threats`
- [ ] すべての CWE 値はハイパーリンクです。`](https://cwe.mitre.org/` が含まれます (`CWE-79` のようなプレーン テキストではありません)。
- [ ] すべての OWASP 値は `:2025` サフィックスを使用します (`:2021` ではありません)。
- [ ] 重大度別 (`## Critical Findings` なし) ではなく、TIER (Tier 1/2/3 の見出し) 別に整理された調査結果
- [ ] **階層と前提条件の一貫性 (インライン)**: 各検出結果に対して、正規マッピングを使用します: `None`→T1; `Authenticated User`/`Privileged User`/`Internal Network`/`Local Process Access`→T2; `Host/OS Access`/`Admin Credentials`/`Physical Access`/`{Component} Compromise`/コンボ→T3。 ⛔ `Application Access` および `Host Access` は禁止されています。
- [ ] 検索見出しをカウントします。これらは連続している必要があります: FIND-01、FIND-02、FIND-03...
- [ ] 時間の見積もりなし: `~`、`Sprint`、`Phase`、`hour`、`day`、`week` を検索 — 表示されてはなりません
- [ ] **脅威カバレッジ検証テーブル** 列 `Threat ID | Finding ID | Status` がファイルの最後にあります
- [ ] **カバレッジ テーブルのステータス値** には絵文字プレフィックスを使用します: `✅ Covered (FIND-XX)`、`✅ Mitigated (FIND-XX)`、`🔄 Mitigated by Platform` — 「調査中」、「緩和済み」、「カバー済み」などのプレーン テキストではありません
- [ ] **カバレッジ テーブルの列名** は正確に `Threat ID | Finding ID | Status` です — `Threat | Finding | Status` ではありません### `0-assessment.md` を書き込んだ後:
- [ ] 最初の `## ` の見出しは `## Report Files` です
- [ ] `## ` 見出しをカウントします。正確に 7 つの名前が付けられます: レポート ファイル、エグゼクティブ サマリー、アクション サマリー、分析コンテキストと前提条件、参照された参考文献、レポート メタデータ、分類リファレンス
- [ ] 見出しに `and` ではなく `&` が含まれています: `Analysis Context & Assumptions` を検索します
- [ ] `---` 区切り線をカウントします — 少なくとも 5 行
- [ ] `### Quick Wins` 見出しが存在します
- [ ] `### Priority by Tier and CVSS Score` 見出しは、クイック ウィンの前に、アクションの概要の下に存在します。
- [ ] **優先度テーブルの最大行数は 10 行**: 階層別優先度テーブルおよび CVSS スコア テーブルのデータ行数は 10 ≤ である必要があります
- [ ] **優先順位テーブルの並べ替え順序**: すべての Tier 1 の結果が最初に表示され、次に Tier 2、次に Tier 3 の順に表示されます。各層内では、CVSS スコアが高いものが最初に表示されます。 ❌ T1 所見の前に T2 所見が出現 → 不合格
- [ ] **優先順位表の検索ハイパーリンク**: すべての検索セルはハイパーリンク `[FIND-XX](3-findings.md#find-xx-title-slug)` です。すべての行で `](3-findings.md#` を検索します — 存在する必要があります。 ❌ リンクなしのプレーンテキスト `FIND-XX` → FAIL
- [ ] **優先度テーブルのアンカー解決**: 各ハイパーリンクについて、アンカーのスラッグが 3-findings.md の実際の `### FIND-XX:` の見出しと一致することを確認します。見出しテキストからアンカーを計算します (小文字、スペースからハイフン、特殊文字を削除)。 ❌ 見出しに `[STILL PRESENT]` や `[NEW]` などのステータス タグが含まれている場合、それは不合格です。ステータス タグは見出しに表示されてはなりません (フェーズ 2 チェックを参照)。アンカーは、タグのないクリーンな見出しテキストから計算される必要があります。
- [ ] **アクション概要層のハイパーリンク**: アクション概要テーブルの階層 1、階層 2、階層 3 のセルは、`3-findings.md#tier-N` アンカーへのハイパーリンクです。
- [ ] `### Needs Verification` 見出しが存在します
- [ ] `### Finding Overrides` 見出しが存在します
- [ ] **アクションサマリーには正確に 4 つのデータ行**があります: Tier 1、Tier 2、Tier 3、Total。 「アクション概要」テーブルで `| Mitigated |`、`| Platform |`、または `| Fixed |` を検索します。見つかった場合は失敗します。これらは別個の層ではありません。
- [ ] **Git コミットには日付が含まれます**: `| Git Commit |` 行には、SHA とコミット日付の両方が含まれている必要があります (例: `f49298ff` (`2026-03-04`))。日付なしでハッシュのみが表示される場合 → FAIL。
- [ ] **ベースライン/ターゲット コミットには日付が含まれます** (増分モード): `| Baseline Commit |` 行と `| Target Commit |` 行には、それぞれ SHA の横に日付が含まれている必要があります。
- [ ] `### Security Standards` および `### Component Documentation` 見出しが存在します (2 つの参照サブセクション)
- [ ] `| Model |` 行がレポート メタデータ テーブルに存在します
- [ ] `| Analysis Started |` 行がレポート メタデータ テーブルに存在します
- [ ] `| Analysis Completed |` 行がレポート メタデータ テーブルに存在します
- [ ] `| Duration |` 行がレポート メタデータ テーブルに存在します
- [ ] バックティックで囲まれたメタデータ値: メタデータ値セル内の `` ` ` をチェックします
- [ ] **レポート ファイル テーブルの最初の行**: `0-assessment.md` は最初のデータ行です (`0.1-architecture.md` ではありません)
- [ ] **レポート ファイルの完全性**: 出力フォルダー内に生成されたすべての `.md` および `.mmd` ファイルには、レポート ファイル テーブルに対応する行があります (`threat-inventory.json` は意図的に除外されています)
- [ ] **レポート ファイルの条件付き行**: `1.2-threatmodel-summary.mmd` 行と `incremental-comparison.html` 行は、これらのファイルが実際に生成された場合にのみ存在します。
- [ ] **脅威数のブロック引用に関する注意**: エグゼクティブサマリーには `> **Note on threat counts:**` 段落が含まれています
- [ ] **境界数**: エグゼクティブ サマリーの境界数は、`1-threatmodel.md` の実際の信頼境界テーブルの行数と一致します。
- [ ] **アクションサマリー階層の優先順位**: 階層 1 = 🔴 重大リスク、階層 2 = 🟠 高リスク、階層 3 = 🟡 中リスク。これらは修正済みであり、カウントに基づいて変更されることはありません。
- [ ] **リスク評価の見出し**には絵文字がありません: `### Risk Rating: 🟠 Elevated` ではなく `### Risk Rating: Elevated`### `0.1-architecture.md` を書き込んだ後:
- [ ] `sequenceDiagram` の出現数をカウントします — 少なくとも 3 回
- [ ] 最初の 3 つのシーケンス図には `participant` 行と `->>` メッセージ矢印があります (空の図ブロックではありません)。
- [ ] キー コンポーネント テーブルの行数がコンポーネント ダイアグラムのノード数と一致します
- [ ] キー コンポーネント テーブルのすべての行で PascalCase 名が使用されます (kebab-case `my-component` や Snake_case `my_component` ではありません)
- [ ] すべてのキー コンポーネント タイプ セルは次のいずれかです: `Process`、`Data Store`、`External Service`、`External Interactor` — `Role`、`Function` のようなアドホック タイプはありません
- [ ] テクノロジー スタック テーブルには、言語、フレームワーク、データ ストア、インフラストラクチャ、セキュリティの 5 つの行がすべて入力されています。
- [ ] `## Security Infrastructure Inventory` セクションが存在します (欠落していません)
- [ ] `## Repository Structure` セクションが存在します (欠落していません)

### `1.1-threatmodel.mmd` を書き込んだ後:
- [ ] 1 行目は `%%{init:` で始まります
- [ ] `classDef process`、`classDef external`、`classDef datastore`が含まれます
- [ ] チャクラ UI カラーなし (`#4299E1`、`#48BB78`、`#E53E3E`)
- [ ] `linkStyle default stroke:#666666,stroke-width:2px` 現在
- [ ] DFD は `flowchart LR` を使用します (`flowchart TB` ではありません) — `flowchart` を検索し、方向が `LR` であることを確認します
- [ ] **インクリメンタル DFD スタイル (インクリメンタル モードのみ)**: 新しいコンポーネントが存在する場合は、`classDef newComponent fill:#d4edda,stroke:#28a745` が存在し、新しいコンポーネント ノードが `:::newComponent` (`:::process` ではない) を使用していることを確認します。削除されたコンポーネントが存在する場合は、`classDef removedComponent` を灰色の破線のスタイルで確認してください。 ❌ `newComponent fill:#6baed6` (プロセスと同じ青色) → FAIL (視覚的には見えない)。### `2-stride-analysis.md` を書き込んだ後:
- [ ] `## Summary` は `## ComponentName` セクションの前に表示されます (行番号を確認してください)
- [ ] 概要テーブルには次の列があります: `| Component | Link | S | T | R | I | D | E | A | Total | T1 | T2 | T3 | Risk |` — `| S | T | R | I | D | E | A |` を検索して確認します
- [ ] サマリー テーブルの S/T/R/I/D/E/A 列には数値 (0、1、2、3...) が含まれますが、すべてのコンポーネントですべて同じ 1 が含まれるわけではありません。
- [ ] 各コンポーネントには `#### Tier 1`、`#### Tier 2`、`#### Tier 3` の小見出しがあります。
- [ ] いいえ `&`、`/`、`(`、`)`、`:` の `## ` 見出し
- [ ] **見出しにステータス タグなし (任意のファイル)**: すべての `.md` ファイルで `^##.+\[Existing\]`、`^##.+\[Fixed\]`、`^##.+\[Partial\]`、`^##.+\[New\]`、`^##.+\[Removed\]` を検索し、`###` 見出しも同様に検索します。古いスタイルも確認してください: `^##.+\[STILL`、`^##.+\[NEW`、`^###.+\[STILL`、`^###.+\[NEW CODE`。 ❌ 見出し内のタグはアンカーリンクを破壊し、目次を汚します。ステータスは、見出しではなく、ブロック引用符 (`> **[Tag]**`) としてセクション本文の最初の行にある必要があります。
- [ ] **重大 — A = 不正使用、決して許可しない**: ファイル内で `| Authorization |` を検索します。一致するものが STRIDE カテゴリ ラベルである場合 (脅威の説明文内ではない) → `| Abuse |` に置き換えて直ちに修正します。 STRIDE-A の「A」は「Abuse」（ビジネス ロジックの悪用、ワークフローの操作、機能の誤用）を表します。これは、観察される最も一般的なエラーです。
- [ ] **N/A エントリはカウントされません**: コンポーネントの STRIDE カテゴリに `N/A — {justification}` がある場合、サマリー テーブルでカテゴリに `0` (`1` ではない) が表示されていることを確認します。
- [ ] **STRIDE ステータス値**: すべての脅威行のステータス列は、`Open`、`Mitigated`、`Platform` のいずれかを使用します。 `Partial`、`N/A`、`Accepted`、またはアドホック値はありません。
- [ ] **プラットフォーム比率**: `Platform` ステータスの脅威の数と合計の脅威の数。 >20% (スタンドアロン) または >35% (K8s オペレーター) の場合 → 各プラットフォームのエントリを再検査します。
- [ ] **STRIDE 列の演算**: サマリー テーブルの行ごとに、S+T+R+I+D+E+A = 合計 AND T1+T2+T3 = 合計 を確認します。
- [ ] **脅威テーブルの完全なカテゴリ名**: カテゴリ列には完全名 (`Spoofing`、`Tampering`、`Information Disclosure`、`Denial of Service`、`Elevation of Privilege`、`Abuse`) が使用されます。省略形 (`S`、`T`、 `DoS`、`EoP`)
- [ ] **N/A テーブルが存在します**: すべてのコンポーネント セクションには、脅威のない STRIDE カテゴリをリストした `| Category | Justification |` テーブルがあります — 散文/箇条書き形式ではありません
- [ ] **リンク列は別です**: 概要テーブルの 2 番目の列は `[Link](#anchor)` 値を持つ `Link` です — コンポーネント名には埋め込みハイパーリンクは含まれません
- [ ] **悪用可能性層の 4 番目の列**: 層定義テーブルには `Assignment Rule` という名前の 4 番目の列が必要です (`Example`、`Description`、`Criteria` ではありません)### `incremental-comparison.html` を書き込んだ後 (インクリメンタル モードのみ):
- [ ] HTML のメトリクス バーに `Trust Boundaries` または `Boundaries` が含まれています - テキスト「Boundaries」を検索します
- [ ] STRIDE ヒートマップには 13 列があります: コンポーネント、S、T、R、I、D、E、A、合計、ディバイダー、T1、T2、T3 — HTML で `T1`、`T2`、および `T3` を検索します
- [ ] 修正済み/新規/以前は未確認のステータス情報は、色付きのステータス カードにのみ表示され、メトリクス バーの小さなインライン バッジとしても表示されません。
- [ ] ヒートマップの STRIDE カテゴリ ラベルとして `| Authorization |` を使用しません — ヒートマップの行で「Authorization」を検索します
- [ ] **HTML カウントはマークダウン カウントと一致します**: HTML ヒートマップ内の脅威の合計は、`2-stride-analysis.md` の [合計] 行と一致する必要があります。異なる場合は、STRIDE サマリー データから HTML ヒートマップを再生成します。 HTML 内の T1+T2+T3 の合計も一致する必要があります。
- [ ] **比較カードが存在します**: HTML には、ベースライン (ハッシュ + 日付 + 評価)、ターゲット (ハッシュ + 日付 + 評価)、トレンド (方向 + 期間) の 3 つのカードを含む `comparison-cards` div が含まれています。
- [ ] **git ログからのコミット日**: 比較カードのベースラインとターゲットの日付は、実際のコミット日と一致する必要があります (今日の日付や分析実行日ではありません)。
- [ ] **コード変更ボックス**: 5 番目のメトリクス ボックスには、コミット数と PR 数が表示されます (「間隔」ではありません)。
- [ ] **[間隔なし] ボックス**: 「間隔」を検索します。メトリック バーに表示されてはなりません。
- [ ] **ステータス カードは簡潔である**: 各ステータス カードの `card-items` div には、短い概要文のみを含める必要があります。 ❌ カードにリストされている脅威 ID (T06.S、T02.E)、検出 ID (FIND-14)、またはコンポーネント名 → 失敗。 `card-items` div 内で `T\d+\.` と `FIND-\d+` を検索します。項目の詳細な内訳は、概要カードではなく、「脅威/調査結果の内訳」セクションに属します。### 増分レポート ファイルを書き込んだ後 (増分モード - インライン チェック):
- [ ] **簡易表示タグのみ**: すべての `.md` ファイルで古い形式のタグを検索します: `[STILL PRESENT]`、`[NEW CODE]`、`[NEW IN MODIFIED]`、`[PREVIOUSLY UNIDENTIFIED]`、`[PARTIALLY MITIGATED]`、`[REMOVED WITH COMPONENT]`、`[MODIFIED]`。 ❌ 任意の一致 → 失敗。簡略化されたタグに置き換えます: `[Existing]`、`[Fixed]`、`[Partial]`、`[New]`、`[Removed]`。
- [ ] **有効な表示タグ**: すべての検出/脅威の注釈では、5 つの簡略化されたタグのいずれか 1 つが使用されます: `[Existing]`、`[Fixed]`、`[Partial]`、`[New]`、`[Removed]`。タグは本文の最初の行にブロック引用符として指定する必要があります: `> **[Tag]**`。
- [ ] **コンポーネント ステータスの簡略化**: コンポーネント ステータス列では、`Unchanged`、`Modified`、`New`、`Removed` のみを使用します。 ❌ `Restructured` → 失敗 (代わりに `Modified` を使用してください)。
- [ ] **変更概要テーブルは簡略化されたタグを使用します**: 脅威ステータス テーブルには 4 行 (既存/修正/新規/削除) があります。検索ステータス テーブルには 5 行 (既存/修正/部分/新規/削除) があります。 ❌ `Still Present`、`New (Code)`、`Partially Mitigated` などの古いスタイルの行 → FAIL。

### `threat-inventory.json` を書き込んだ後 (インライン チェック):
- [ ] **JSON 脅威数は STRIDE ファイルと一致します**: `2-stride-analysis.md` 内の固有の脅威 ID をカウントします (grep `^\| T\d+\.`)。このカウントは、JSON の `threats` 配列の長さと等しくなければなりません。 STRIDE に JSON よりも多くの脅威がある場合 → シリアル化中に脅威がドロップされました。 JSONを再構築します。
- [ ] **内部的に一貫した JSON メトリクス**: `metrics.total_threats` は `threats` 配列の長さと等しくなければなりません。 `metrics.total_findings` 配列の長さは `findings` と等しくなければなりません。

### `0-assessment.md` (カウント検証) を書き込んだ後:
- [ ] エグゼクティブ サマリーの要素数は実際の要素テーブルの行数と一致します (必要に応じて `1-threatmodel.md` を再読み込みします)
- [ ] 検索カウントが `3-findings.md` の実際の `### FIND-` 見出しカウントと一致します
- [ ] 脅威数は `2-stride-analysis.md` の概要テーブルの合計と一致します

---

## フェーズ 0 — 共通偏差スキャン

これらは、以前のすべての実行で最も頻繁に観察された偏差です。出力が生成された後、すべての出力ファイルをスキャンして、これらの特定のパターンを探します。各チェックには、検索対象の **間違った** パターンと、**正しい** 予想されるパターンがあります。

**使用方法:** チェックごとに、出力ファイルを grep/スキャンして間違ったパターンがないか調べます。見つかった場合→失敗。次に、正しいパターンが存在することを確認します。このフェーズでは、生成モデルが指示にもかかわらず犯しがちな繰り返しの間違いを発見します。

### 0.1 構造の逸脱- [ ] **階層ではなく重大度別に整理された結果** — `## Critical Findings`、`## Important Findings`、`## High Findings` を検索します。これらは存在してはなりません。 ❌ `## Critical Findings` → ✅ `## Tier 1 — Direct Exposure (No Prerequisites)`
- [ ] **フラット STRIDE テーブル (階層サブセクションなし)** — `2-stride-analysis.md` の各コンポーネントには、`#### Tier 1`、`#### Tier 2`、`#### Tier 3` の小見出しが必要です。 ❌ コンポーネントごとに 1 つのフラット テーブル → ✅ 3 つの独立した層のサブセクション
- [ ] **悪用可能性層の欠落または検出結果に対する修復作業** — `3-findings.md` 内のすべての `### FIND-` ブロックには、`Exploitability Tier` 行と `Remediation Effort` 行の両方が含まれている必要があります。 ❌ いずれかのフィールドが欠落している → ✅ 両方とも必須
- [ ] **STRIDE サマリーに階層列がありません** — `2-stride-analysis.md` のサマリー テーブルには `T1`、`T2`、`T3` 列が含まれている必要があります。 ❌ S/T/R/I/D/E/A/合計のみ → ✅ T1/T2/T3/リスク列も必要
- [ ] **STRIDE の概要は下部にあります** — `## Summary` と最初の `## Component` の行番号を検索します。 ❌ コンポーネント後の概要 → ✅ すべてのコンポーネントセクションの前、`## Exploitability Tiers` の直後
- [ ] **悪用可能性層テーブルの列** — `2-stride-analysis.md` の層定義テーブルには、まさに次の 4 つの列が必要です: `Tier | Label | Prerequisites | Assignment Rule`。 ❌ `Example`、`Description`、`Criteria` を 4 列目として → ✅ `Assignment Rule` のみ。 [割り当てルール] セルには、展開固有の例ではなく、厳格なルール テキストを含める必要があります。

### 0.2 ファイル形式の逸脱

- [ ] **`.md` コード フェンスでラップ** — `.md` ファイルが ` で始まるかどうかを確認します```markdown ` or ` ```@@コード0@@。 ❌ ````markdown\n# Title` → ✅ `# Title` on line 1
- [ ] **`.mmd` wrapped in code fences** — Check if `.mmd` file starts with ` ```平文 ` or ````mermaid `. ❌ ` ```人魚\n%%{init:` → ✅ `%%{init:` (1 行目)
- [ ] **出力内のスキル ディレクティブの漏洩** — すべての `.md` ファイルで `⛔`、`RIGID TIER`、`Do NOT use subjective`、`MANDATORY`、`CRITICAL —`、`decision procedure` を検索します。これらは内部スキルの指示であり、レポート出力に表示してはなりません。 ❌ 任意の一致 → ✅ 一致はありません。漏れたディレクティブ行を削除します。
- [ ] **ネストされた重複出力フォルダー** — 出力フォルダーに同じ名前のサブフォルダーが含まれているかどうかを確認します (例: `threat-model-20260307-081613/threat-model-20260307-081613/`)。 ❌ サブフォルダーが存在します → ✅ ネストされた重複を削除します。出力フォルダーにはファイルのみが含まれ、サブフォルダーは含まれません。
- [ ] **STRIDE-A "Abuse" ではなく "Authorization"** — STRIDE カテゴリ名として使用されている `| Authorization |` または `**Authorization**` を `2-stride-analysis.md` で検索します。 STRIDE-A の A は常に「悪用」であり、「承認」ではありません。 ❌ Authorization が STRIDE カテゴリとして使用されている一致 → ✅ 「Abuse」に置き換えます。注: 脅威の説明内に「Authorization」が表示されている場合は、「Authorization」を置き換えないでください (「Authorization ヘッダー」、「認可チェックが欠如している」など)。

### 0.3 評価セクションの逸脱

- [ ] **アクション概要のセクション名が間違っています** — `Priority Remediation Roadmap`、`Top Recommendations`、`Key Recommendations`、`Risk Profile` を検索します。 ❌ これらの名前のいずれか → ✅ `## Action Summary` のみ
- [ ] **別の推奨事項セクション** — `### Key Recommendations` または `### Top Recommendations` をスタンドアロン セクションとして検索します。 ❌ 別のセクション → ✅ アクションの概要は推奨事項です
- [ ] **Quick Win サブセクションがありません** — [アクションの概要] で `### Quick Wins` を検索します。 ❌ 欠落 → ✅ 存在 (ローエフォート T1 所見がない場合はメモあり)
- [ ] **脅威数のコンテキストがありません** — エグゼクティブサマリーで `> **Note on threat counts:**` ブロック引用を検索します。 ❌ 欠落 → ✅ 存在
- [ ] **分析コンテキストと仮定が欠落しています** — `## Analysis Context & Assumptions` を検索します。 ❌ 欠落 → ✅ `### Needs Verification` および `### Finding Overrides` サブセクションが存在します
- [ ] **必須の評価セクションが欠落しています** — 7 つすべてが存在することを確認します: レポート ファイル、エグゼクティブ サマリー、アクション サマリー、分析コンテキストと前提条件、参照した参考文献、レポート メタデータ、分類リファレンス。 ❌ どれかが欠けている → ✅ 7 つすべてが存在する

### 0.4 参照とメタデータの逸脱- [ ] **フラット テーブルとして参照される参照** — `| Reference | Usage |` パターンを検索します。 ❌ 2 列のフラット テーブル → ✅ 2 つのサブセクション: `### Security Standards` と `| Standard | URL | How Used |` および `### Component Documentation` と `| Component | Documentation URL | Relevant Section |`
- [ ] **参考文献に URL がありません** — 参考文献テーブルのすべての行には、完全な `https://` URL が必要です。 ❌ URL 列が欠落しているか空の URL → ✅ すべての行に完全な URL
- [ ] **モデルが欠落しているメタデータをレポート** — `| **Model** |` または `| Model |` 行を検索します。 ❌ 欠品 → ✅ 実際のモデル名とともに存在
- [ ] **タイムスタンプが欠落しているメタデータをレポート** — `Analysis Started`、`Analysis Completed`、`Duration` 行を検索します。 ❌ 欠落しているものがある → ✅ 3 つすべてが計算された値で存在する

### 0.5 品質の偏差を見つける

- [ ] **ベクターまたはプレフィックスが欠落している CVSS スコア** — 各検出結果の CVSS フィールドを Grep します。値はパターン `\d+\.\d+ \(CVSS:4\.0/AV:` と一致する必要があります。特に `CVSS:4.0/` プレフィックスを確認してください。最も一般的な逸脱は、このプレフィックスなしでベクトルを出力することです (裸の `AV:N/AC:L/...`)。 ❌ `9.3` (スコアのみ) → ❌ `9.3 (AV:N/AC:L/...)` (プレフィックスなし) → ✅ `9.3 (CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N)`
- [ ] **ハイパーリンクなしの CWE** — `[` の前に付けずに `CWE-\d+` を Grep します。 ❌ `CWE-78: OS Command Injection` → ✅ `[CWE-78](https://cwe.mitre.org/data/definitions/78.html): OS Command Injection`
- [ ] **OWASP `:2021` サフィックス** — `:2021` の Grep。 ❌ `A01:2021` → ✅ `A01:2025`
- [ ] **プレーン テキストとしての関連脅威** — `](` を含まないパターンの `Related Threats` 行を Grep します。 ❌ `T-02, T-17, T-23` → ✅ `[T02.S](2-stride-analysis.md#component-name), [T17.I](2-stride-analysis.md#other-component)`
- [ ] **ID の順序が間違っている** — FIND-NN ID が連続していることを確認します: FIND-01、FIND-02、FIND-03... ❌ `FIND-06` が `FIND-04` の前に出現 → ✅ 上から下への連続番号
- [ ] **階層 1 の CVSS AV:L または PR:H** — `AV:L` または `PR:H` の各階層 1 所見の CVSS ベクトルを grep します。 ❌ ローカル専用アクセスの Tier 1 → ✅ T2/T3 にダウングレード
- [ ] **Tier 1 でローカルホストのみまたは管理者のみが見つかった** — 導入コンテキストを確認します: エアギャップ、ローカルホスト、単一管理サービスは Tier 1 であってはなりません。 ❌ 管理者専用の Tier 1 → ✅ T2/T3
- [ ] **出力の推定時間** — `~1 hour`、`Sprint`、`Phase 1`、`(hours)`、`(days)`、`(weeks)`、`Immediate` の grep。 ❌ 任意のスケジュール言語 → ✅ `Low`/`Medium`/`High` エフォート ラベルのみ
- [ ] **カバレッジ表の「許容リスク」** — `Accepted Risk` を `3-findings.md` で Grep します。 ❌ 任意の一致 → 失敗。ツールにはリスクを受け入れる権限がありません。すべての `Open` 脅威には調査結果が必要です。すべての `⚠️ Accepted Risk` を `✅ Covered` に置き換え、対応する結果を作成します。

### 0.6 図の逸脱- [ ] **間違ったカラーパレット** — `.mmd` ファイルおよび Mermaid ブロック内のすべての `#[0-9a-fA-F]{6}` を Grep します。 ❌ `#4299E1`、`#48BB78`、`#E53E3E`、`#2B6CB0`、`#2D3748`、`#2F855A`、`#C53030` (Chakra UI) → ✅ のみ許可されます: `#6baed6`、`#2171b5`、`#fdae61`、`#d94701`、 `#74c476`、`#238b45`、`#e31a1c`、`#666666`、`#ffffff`、`#000000`
- [ ] **カスタムテーマ変数の色** — `secondaryColor`、`tertiaryColor`、または `primaryTextColor` の init ブロックを検索します。 ❌ `"primaryColor": "#2D3748", "secondaryColor": "#4299E1"` → ✅ テーマ変数内の `'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666'` のみ
- [ ] **概要 MMD が欠落しています** — `1.1-threatmodel.mmd` のノードとサブグラフをカウントします。要素が 15 を超えるか、サブグラフが 4 を超える場合、`1.2-threatmodel-summary.mmd` が存在しなければなりません。 ❌ しきい値は満たしたがファイルが見つからない → ✅ 概要図で作成されたファイル
- [ ] **スタンドアロン サイドカー ノード (K8 のみ)** — `MISE`、`Dapr`、`Envoy`、`Istio`、`Sidecar` という名前のノードのダイアグラムを個別のエントリとして検索します。 ❌ `MISE(("MISE Sidecar"))` → ✅ `InferencingFlow(("Inferencing Flow<br/>+ MISE"))`
- [ ] **ポッド内ローカルホスト フロー (K8 のみ)** — 共存するコンテナ間で `-->|"localhost"|` 矢印を検索します。 ❌ 存在 → ✅ 存在しない (暗黙的)
- [ ] **シーケンス図が欠落しています** — `0.1-architecture.md` の最初の 3 つのシナリオには、それぞれ `sequenceDiagram` ブロックが必要です。 ❌ 3 つ未満 → ✅ 少なくとも 3 つ
- [ ] **テクノロジー固有のギャップ** — リポジトリ内のすべてのテクノロジー (Redis、PostgreSQL、Docker、K8s、ML/LLM、NFS など) について、少なくとも 1 つの調査結果または文書化された緩和策が存在することを確認します。 ❌ テクノロジーは存在するがカバーされていない → ✅ 各テクノロジーが取り上げられている

### 0.7 正規パターンのチェック

- [ ] **検索見出しパターン** — すべての検索見出しは `^### FIND-\d{2}: ` と一致します (`F01`、`F-01`、`Finding 1` はありません)
- [ ] **CVSS プレフィックス パターン** - すべての CVSS フィールドは `\d+\.\d+ \(CVSS:4\.0/AV:` と一致します (裸の `AV:N/AC:L/...` はありません)
- [ ] **関連脅威リンク パターン** - すべての関連脅威トークンが `\[T\d{2}\.[STRIDEA]\]\(2-stride-analysis\.md#[a-z0-9-]+\)` と一致します
- [ ] **評価セクションの見出しは正確に設定されています** — `0-assessment.md` 内の `##` の見出しとまったく同じです: レポート ファイル、概要、アクションの概要、分析コンテキストと前提条件、参照した参考文献、レポート メタデータ、分類リファレンス
- [ ] **禁止されている見出しがありません** — 次の内容を含む `##` または `###` 見出しはありません: 重大度分布、アーキテクチャ リスク領域、方法論メモ、成果物、優先修復ロードマップ、主要な推奨事項、上位の推奨事項

---

## フェーズ 1 — ファイルごとの構造チェック

これらのチェックでは、各ファイルを個別に検証します。これらは並行して実行できます。

### 1.1 すべての `.md` ファイル- [ ] **コードフェンスのラッピングなし**: ` で始まる `.md` ファイルはありません```markdown ` or ` ```@@コード0@@。すべての `.md` ファイルは、最初の行が `# Heading` で始まる必要があります。ファイルがフェンスで囲まれている場合は、最初と最後の行をすぐに削除します。
- [ ] **`.mmd` コードフェンスラッピングなし**: `.mmd` ファイルは ` で始まってはなりません```plaintext ` or ` ```mermaid `. It must start with `%%{init:` を最初の文字として使用します。巻き付けられている場合は、フェンスのラインを剥がします。
- [ ] **空のファイルはありません**: すべてのファイルには、見出しを超えた実質的なコンテンツが含まれています。

### 1.2 `0.1-architecture.md`

- [ ] **必須セクションが存在します**: システムの目的、主要コンポーネント、コンポーネント図、トップシナリオ、テクノロジースタック、デプロイメントモデル、リポジトリ構造
- [ ] **コンポーネント図は ` 内の Mermaid `flowchart` として存在します**```mermaid ` code fence
- [ ] **Architecture styles used** — NOT DFD circles `(("Name"))`. Must use `["Name"]` or `(["Name"])` with `service`/`external`/`datastore` classDef names
- [ ] **At least 3 scenarios** have Mermaid `sequenceDiagram` blocks
- [ ] **No separate `.mmd` files** were created for 0.1-architecture.md — all diagrams are inline
- [ ] **Component Diagram elements match Key Components table** — every row in the table has a corresponding node in the diagram, and vice versa. Count both and verify counts are equal.
- [ ] **Top Scenarios reflect actual code paths**, not hypothetical use cases
- [ ] **Deployment Model has network details** — must mention at least: port numbers OR bind addresses OR network topology

### 1.3 `1.1-threatmodel.mmd`

- [ ] **File exists** with pure Mermaid code (no markdown wrapper, no ` ```マーメイド`フェンス)
- [ ] **で始まる** `%%{init:` ブロック
- [ ] **含む** `classDef process`、`classDef external`、`classDef datastore`
- [ ] **DFD シェイプを使用**: プロセスには円 `(("Name"))`、外部には長方形 `["Name"]`、データ ストアには円柱 `[("Name")]`

### 1.4 `1-threatmodel.md`

- [ ] **図の内容は `1.1-threatmodel.mmd` と同一** — Mermaid ブロックの内容のバイトごとの比較 (` を除く)```mermaid ` fence wrapper)
- [ ] **Element Table** present with columns: Element, Type, TMT Category, Description, Trust Boundary
- [ ] **Data Flow Table** present with columns: ID, Source, Target, Protocol, Description
- [ ] **Trust Boundary Table** present with columns: Boundary, Description, Contains
- [ ] **TMT Category IDs used** — Element Table's TMT Category column uses specific TMT element IDs from `tmt-element-taxonomy.md` (e.g., `SE.P.TMCore.WebSvc`, `SE.EI.TMCore.Browser`). NOT generic labels like `Process`, `External`.
- [ ] **Flow IDs match DF\d{2} pattern** — Every flow ID in the Data Flow Table uses `DF01`, `DF02`, etc. format. NOT `F1`, `Flow-1`, `DataFlow1`.
- [ ] **If >15 elements or >4 boundaries**: `1.2-threatmodel-summary.mmd` MUST exist AND `1-threatmodel.md` MUST include a "Summary View" section with the summary diagram AND a "Summary to Detailed Mapping" table. **To verify:** count nodes (lines matching `[A-Z]\d+` with shape syntax) and subgraphs in `1.1-threatmodel.mmd`. If count exceeds thresholds but `1.2-threatmodel-summary.mmd` does not exist → **FAIL — create the summary diagram before proceeding**.

### 1.5 `2-stride-analysis.md`

- [ ] **Exploitability Tiers section** present at top with tier definition table
- [ ] **Summary table** appears BEFORE individual component sections (immediately after Exploitability Tiers, NOT at the bottom of the file)
- [ ] **Summary table** includes columns: Component, Link, S, T, R, I, D, E, A, Total, T1, T2, T3, Risk
- [ ] **Every component** has `## Component Name` heading followed by Tier 1, Tier 2, Tier 3 sub-sections (all three present even if empty)
- [ ] **Empty tiers** use "*No Tier N threats identified for this component.*"
- [ ] **Anchor-safe headings**: No `## ` heading in this file contains ANY of these characters: `&`, `/`, `(`, `)`, `.`, `:`, `'`, `"`, `+`, `@`, `!`. Replace: `&` → `and`, `/` → `-`, parentheses → omit, `:` → omit.
- [ ] **Pod Co-location line** present for K8s components listing co-located sidecars
- [ ] **STRIDE Status values** — Every threat row's Status column uses exactly one of: `Open`, `Mitigated`, `Platform`. No `Partial`, `N/A`, or other ad-hoc values.
- [ ] **A category labeled Abuse** — Search `2-stride-analysis.md` for `| Authorization |` as a STRIDE category label. FAIL if found. The "A" in STRIDE-A is always "Abuse" (business logic abuse, workflow manipulation, feature misuse), NEVER "Authorization". Also check N/A entries: `Authorization — N/A` is WRONG, must be `Abuse — N/A`.
- [ ] **STRIDE-Coverage Consistency** — For every threat ID, the STRIDE Status and Coverage table Status must agree:
  - STRIDE `Open` → Coverage `✅ Covered (FIND-XX)` (finding documents vulnerability needing remediation)
  - STRIDE `Mitigated` → Coverage `✅ Mitigated (FIND-XX)` (finding documents existing control the team built)
  - STRIDE `Platform` → Coverage `🔄 Mitigated by Platform`
  - If STRIDE says `Partial` but Coverage says `Mitigated by Platform` → **CONFLICT. Fix it.**
  - If STRIDE says `Open` but Coverage says `⚠️ Needs Review` → only valid if prerequisites ≠ `None`

### 1.6 `3-findings.md`

- [ ] **Organized by tier** using exactly: `## Tier 1 — Direct Exposure (No Prerequisites)`, `## Tier 2 — Conditional Risk (...)`, `## Tier 3 — Defense-in-Depth (...)`
- [ ] **NOT organized by severity** — no `## Critical Findings` or `## Important Findings` headings
- [ ] **Every finding** has ALL mandatory attributes: SDL Bugbar Severity, CVSS 4.0, CWE, OWASP (with `:2025` suffix), Exploitation Prerequisites, Exploitability Tier, Remediation Effort, Mitigation Type, Component, Related Threats
- [ ] **Mitigation Type valid values** — Every finding's `Mitigation Type` row is one of exactly: `Redesign`, `Standard Mitigation`, `Custom Mitigation`, `Existing Control`, `Accept Risk`, `Transfer Risk`. ❌ Abbreviated forms (`Custom`, `Accept`, `Standard`) or invented values → FAIL
- [ ] **SDL Severity valid values** — Every finding's severity is one of: `Critical`, `Important`, `Moderate`, `Low`. ❌ `High`, `Medium`, `Info` → FAIL
- [ ] **Remediation Effort valid values** — Every finding's effort is one of: `Low`, `Medium`, `High`. ❌ Time estimates, sprint labels → FAIL
- [ ] **CVSS 4.0 has full vector**: Every finding's CVSS value includes BOTH the numeric score AND the full vector string (e.g., `9.3 (CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N)`). Score-only is NOT acceptable.
- [ ] **CWE format**: Every CWE uses `CWE-NNN: Name` format (not just number)
- [ ] **OWASP format**: Every OWASP uses `A0N:2025` format (never `:2021`)
- [ ] **Related Threats** use individual links per threat ID: `[T01.S](2-stride-analysis.md#component-name)` — no grouped links like `[T01.S, T01.T](2-stride-analysis.md)`
- [ ] **Exploitation Prerequisites present** — Every `### FIND-` block has a row `| Exploitation Prerequisites |`
- [ ] **Component field present** — Every `### FIND-` block has a row `| Component |`
- [ ] **No Tier 1 with AV:L or PR:H** — For every Tier 1 finding, verify its CVSS vector does NOT contain `AV:L` or `PR:H`. If found → tier must be downgraded to T2/T3.
- [ ] **Tier-Prerequisite Consistency (MANDATORY)** — For EVERY finding and EVERY threat row, the tier MUST follow mechanically from the prerequisite using the canonical mapping:
  - `None` → T1 (only valid if component's Reachability = External AND Auth = No)
  - `Authenticated User`, `Privileged User`, `Internal Network`, `Local Process Access` → T2
  - `Host/OS Access`, `Admin Credentials`, `Physical Access`, `{Component} Compromise`, any `A + B` → T3
  - **⛔ FORBIDDEN values:** `Application Access`, `Host Access` → FAIL. Replace with `Local Process Access` (T2) or `Host/OS Access` (T3).
  - **Deployment context rule (Rule 20):** If Deployment Classification is `LOCALHOST_DESKTOP` or `LOCALHOST_SERVICE`, `None` is FORBIDDEN for all components. Fix prerequisite to `Local Process Access` or `Host/OS Access`, then derive tier.
  - **Exposure table cross-check:** For each finding, look up its Component in the Component Exposure Table. The finding's prerequisite MUST be ≥ the component's `Min Prerequisite`. The finding's tier MUST be ≥ the component's `Derived Tier`.
  - **Mismatch = FAIL.** Fix by adjusting prerequisites to match deployment evidence, then derive tier from prerequisite.
  - **Common violations:** `None` on a localhost-only component; `Application Access` (ambiguous); T1 with `Internal Network` prerequisite; T2 with `None` prerequisite.
- [ ] **Threat Coverage Verification table** present at end of file mapping every threat ID → finding ID with status
- [ ] **Coverage table valid statuses ONLY** — Every row in the Coverage table must use exactly one of these three statuses: `✅ Covered (FIND-XX)`, `✅ Mitigated (FIND-XX)`, or `🔄 Mitigated by Platform`. ❌ `⚠️ Accepted Risk` → FAIL (tool cannot accept risks). ❌ `⚠️ Needs Review` → FAIL (every threat must be resolved). ❌ `—` without a status → FAIL (unaccounted threat).
- [ ] **Mitigated vs Platform distinction** — For every `✅ Mitigated (FIND-XX)` entry: verify the finding documents an existing security control the engineering team built (auth middleware, TLS, input validation, file permissions). For every `🔄 Mitigated by Platform`: verify the mitigation is from a genuinely EXTERNAL system (Azure AD, K8s RBAC, TPM). If "Platform" describes THIS repo's code → reclassify as `✅ Mitigated` and create a finding.
- [ ] **Platform Mitigation Ratio Audit (MANDATORY)** — Count threats marked `🔄 Mitigated by Platform` vs total threats. If Platform > 20% → **WARNING: Likely overuse of Platform status.** For each Platform-mitigated threat, verify ALL three conditions: (1) mitigation is EXTERNAL to this repo's code, (2) managed by a different team, (3) cannot be disabled by modifying this code. Common violations: "auth middleware" (that's THIS code → should be `Mitigated`), "TLS on localhost" (THIS code → should be `Mitigated`), "file permissions" (THIS code → should be `Mitigated`).
- [ ] **Coverage Feedback Loop Verification** — After the Coverage table is written, verify: (1) every threat with STRIDE status `Open` has a corresponding finding in the table. (2) No `—` dashes without a status. (3) If gaps exist, new findings were created to fill them. The Coverage table is a FEEDBACK LOOP — its purpose is to catch missed findings and force their creation. If gaps remain after the table is written, the loop was not executed.
- [ ] **"Accepted Risk" in Coverage table** — Grep `3-findings.md` for `Accepted Risk`. ❌ Any match → FAIL. The tool does NOT have authority to accept risks. Every `Open` threat MUST have a finding. Every `Mitigated` threat MUST have a finding documenting the team's control.
- [ ] **"Needs Review" in Coverage table** — Grep `3-findings.md` for `Needs Review`. ❌ Any match → FAIL. "Needs Review" has been replaced: threats are either Covered (vulnerability), Mitigated (team built a control), or Platform (external system). There is no deferred category.

### 1.7 `0-assessment.md`

- [ ] **Section order**: Report Files → Executive Summary → Action Summary → Analysis Context & Assumptions → References Consulted → Report Metadata → Classification Reference (last)
- [ ] **Report Files section** is the very first section after the title
- [ ] **Risk Rating heading** has NO emojis: `### Risk Rating: Elevated` not `### Risk Rating: 🟠 Elevated`
- [ ] **Threat count context paragraph** present as blockquote at end of Executive Summary
- [ ] **No separate Recommendations section** — Action Summary IS the recommendations
- [ ] **Action Summary table** present with Tier, Description, Threats, Findings, Priority columns
- [ ] **Action Summary is the ONLY name**: No sections titled "Priority Remediation Roadmap", "Top Recommendations", "Key Recommendations", or "Risk Profile"
- [ ] **Quick Wins subsection** present (or explicitly omitted if no low-effort T1 findings)
- [ ] **Needs Verification section** present under Analysis Context & Assumptions
- [ ] **References Consulted** has two subsections: `### Security Standards` and `### Component Documentation`
- [ ] **References Consulted tables** use three columns with full URLs: `| Standard | URL | How Used |` and `| Component | Documentation URL | Relevant Section |` — NOT a flat `| Reference | Usage |` table
- [ ] **Finding Overrides** uses table format even when empty (never plain text)
- [ ] **Report Metadata** is the absolute last section before Classification Reference with all required fields
- [ ] **Metadata timestamps** came from actual command execution (not derived from folder names)
- [ ] **Model** field present — value matches the model being used (e.g., `Claude Opus 4.6`, `GPT-5.3 Codex`, `Gemini 3 Pro`)
- [ ] **Analysis Started** and **Analysis Completed** fields present with UTC timestamps from `Get-Date` commands
- [ ] **Duration** field present — computed from Analysis Started and Analysis Completed timestamps
- [ ] **Metadata values in backticks** — Every value cell in the Report Metadata table must be wrapped in backticks. Spot-check at least 5 rows.
- [ ] **Horizontal rules between sections** — Count lines matching `---` in the file. Must be ≥ 6 (one between each pair of the 7 `## ` sections).
- [ ] **Classification Reference is last section** — `## Classification Reference` present as the final `## ` heading. Contains a single 2-column table (`Classification | Values`) with rows for: Exploitability Tiers, STRIDE + Abuse, SDL Severity, Remediation Effort, Mitigation Type, Threat Status, CVSS, CWE, OWASP. ❌ Missing section or wrong format → FAIL.
- [ ] **Classification Reference is static** — Values in the table must match the skeleton EXACTLY (copied verbatim). No additional rows, no modified descriptions. Compare against `skeleton-assessment.md` Classification Reference section.
- [ ] **No forbidden section headings** — Search for: `Severity Distribution`, `Architecture Risk Areas`, `Methodology Notes`, `Deliverables`, `Priority Remediation Roadmap`, `Key Recommendations`, `Top Recommendations`. Must return 0 matches.
- [ ] **Action Summary tier priorities are FIXED** — In the Action Summary table of `0-assessment.md`, verify the Priority column: Tier 1 = `🔴 Critical Risk`, Tier 2 = `🟠 Elevated Risk`, Tier 3 = `🟡 Moderate Risk`. ❌ Tier 1 with Low/Moderate/Elevated → FAIL. ❌ Tier 2 with Critical/Low → FAIL. These are FIXED labels that never change regardless of threat/finding counts.
- [ ] **Action Summary has all 3 tiers** — The Action Summary table MUST have rows for Tier 1, Tier 2, AND Tier 3, even if a tier has 0 threats and 0 findings. Missing tiers → FAIL.

---

## Phase 2 — Diagram Rendering Checks

Run against ALL Mermaid blocks across all files. Can be delegated as a focused sub-task.

### 2.1 Init Blocks

- [ ] **Every flowchart** has `%%{init}%%` block with `'background': '#ffffff'` as the first line
- [ ] **Every sequence diagram** has the full `%%{init}%%` theme variables block with `'background': '#ffffff'`
- [ ] **NO custom color keys in themeVariables** — init block must NOT contain `primaryColor` (except `#ffffff`), `secondaryColor`, or `tertiaryColor`. All element colors come from classDef only.

### 2.2 Class Definitions & Color Palette

- [ ] **Every `classDef`** includes `color:#000000` (explicit black text)
- [ ] **DFD diagrams** use `process`/`external`/`datastore` class names
- [ ] **Architecture diagrams** use `service`/`external`/`datastore` class names
- [ ] **EXACT hex codes used** — grep all `#[0-9a-fA-F]{6}` values in `.mmd` files. The ONLY allowed fill colors are: `#6baed6`, `#fdae61`, `#74c476`, `#ffffff`, `#000000`. The ONLY allowed stroke colors are: `#2171b5`, `#d94701`, `#238b45`, `#e31a1c`, `#666666`. If ANY other hex color appears (e.g., `#4299E1`, `#48BB78`, `#E53E3E`, `#2B6CB0`), the diagram FAILS this check.

### 2.3 Styling

- [ ] **Every flowchart** has `linkStyle default stroke:#666666,stroke-width:2px`
- [ ] **Trust boundary styles** use `stroke:#e31a1c,stroke-width:3px` (NOT `#ff0000` or `stroke-width:2px`)
- [ ] **Architecture layer styles** use light fills with matching borders (not red dashed trust boundaries)

### 2.4 Syntax Validation

- [ ] **All labels quoted**: `["Name"]`, `(("Name"))`, `[("Name")]`, `-->|"Label"|`, `subgraph ID["Title"]`
- [ ] **Subgraph/end pairs matched**: Every `subgraph` has a closing `end`
- [ ] **No stray characters** or unclosed quotes in any Mermaid block

### 2.5 Kubernetes Sidecar Rules

Skip this section if the target system is NOT deployed on Kubernetes.

- [ ] **Every K8s service node** annotated with sidecars: `<br/>+ SidecarName` in the node label
- [ ] **Zero standalone sidecar nodes**: Search all diagrams for nodes named `MISE`, `Dapr`, `Envoy`, `Istio`, `Sidecar` — these must NOT exist as separate nodes
- [ ] **Zero intra-pod localhost flows**: No arrows between a container and its sidecars (no `-->|"localhost"` patterns)
- [ ] **Cross-boundary sidecar flows originate from host container**: All arrows to external targets (Azure AD, Redis, etc.) come from the host container node, not from a standalone sidecar node
- [ ] **Element Table**: No separate rows for sidecars — described in host container's description column

---

## Phase 3 — Cross-File Consistency Checks

These checks validate relationships between files. They require reading multiple files together.

### 3.1 Component Coverage (Architecture → STRIDE → Findings)

- [ ] **Every component** in `0.1-architecture.md` Key Components table has a corresponding `## Component` section in `2-stride-analysis.md`
- [ ] **Every element** in the `1-threatmodel.md` Element Table that is a Process has a corresponding `## Component` section in `2-stride-analysis.md`
- [ ] **No orphaned components** in `2-stride-analysis.md` that don't appear in the Element Table
- [ ] **Summary table component count** matches the number of `## Component` sections in the file
- [ ] **Component count exact match** — Count rows in `0.1-architecture.md` Key Components table (excluding header/separator). Count `## ` component sections in `2-stride-analysis.md` (excluding `## Exploitability Tiers`, `## Summary`). These counts MUST be equal.

### 3.2 Data Flow Coverage (STRIDE ↔ DFD)

- [ ] **Every Data Flow ID** (`DF01`, `DF02`, ...) from the `1-threatmodel.md` Data Flow Table appears in at least one "Affected Flow" cell in `2-stride-analysis.md`
- [ ] **No orphaned flow IDs** in STRIDE analysis that aren't defined in the Data Flow Table

### 3.3 Threat-to-Finding Traceability (STRIDE ↔ Findings)

This is the most critical cross-file check. It ensures no identified threat is silently dropped.

- [ ] **Every threat ID** in `2-stride-analysis.md` (e.g., T01.S, T01.T1, T02.I) is referenced by at least one finding in `3-findings.md` via its Related Threats field
- [ ] **Collect all threat IDs** from all tier tables in `2-stride-analysis.md`
- [ ] **Collect all threat IDs** referenced in Related Threats fields in `3-findings.md`
- [ ] **Coverage gap report**: List any threat ID present in STRIDE but missing from findings. If gaps exist → either add a finding or group the threat into an existing related finding

### 3.4 Finding-to-STRIDE Anchor Integrity (Findings → STRIDE)

- [ ] **Every Related Threats link** in `3-findings.md` uses format `[ThreatID](2-stride-analysis.md#component-anchor)`
- [ ] **Every `#component-anchor`** resolves to an actual `## Heading` in `2-stride-analysis.md`
- [ ] **Anchor construction verified**: heading → lowercase → spaces to hyphens → strip non-alphanumeric except hyphens
- [ ] **Spot-check at least 3 anchors** by following the link and confirming the threat ID exists under that heading

### 3.5 Count Consistency (Assessment ↔ All Files)

- [ ] **Element count** in Executive Summary matches actual Element Table row count in `1-threatmodel.md`
- [ ] **Finding count** in Executive Summary matches actual finding count in `3-findings.md`
- [ ] **Threat count** in Executive Summary matches Total from summary table in `2-stride-analysis.md`
- [ ] **Tier counts** in threat count context paragraph match actual T1/T2/T3 totals from `2-stride-analysis.md`
- [ ] **Action Summary tier table** counts match actual per-tier counts from `3-findings.md` (findings column) and `2-stride-analysis.md` (threats column)

**Verification methods for count checks:**
- Element count: count `|` rows in Element Table of `1-threatmodel.md`, subtract 2 (header + separator)
- Finding count: count `### FIND-` headings in `3-findings.md`
- Threat count: read the Totals row in `2-stride-analysis.md` Summary table, take the `Total` column value
- Tier counts: from same Totals row, take T1, T2, T3 column values

### 3.6 STRIDE Summary Table Arithmetic

- [ ] **Per-row**: S + T + R + I + D + E + A = Total for every component
- [ ] **Per-row**: T1 + T2 + T3 = Total for every component
- [ ] **Totals row**: Each column sum across all component rows equals the Totals row value
- [ ] **Row count cross-check**: Number of threat rows in each component's detail tables equals its Total in the summary table
- [ ] **No artificial all-1s pattern**: Check the Summary table for the pattern where every STRIDE column (S,T,R,I,D,E,A) is exactly 1 for every component. If ALL components have exactly 1 threat in every STRIDE category → FAIL (indicates formulaic "minimum 1 per category" inflation rather than genuine analysis). A valid analysis should have varying counts per category reflecting actual attack surface: some categories may be 0 (with N/A justification), others 2-3. Uniform 1s across all components is a strong signal of artificial padding.
- [ ] **N/A entries excluded from totals**: If any component has `N/A — {justification}` entries for STRIDE categories, verify those categories show 0 in the Summary table (not 1). N/A entries do NOT count as threats.

### 3.7 Sort Order (Findings)

- [ ] **Within each tier section**: Findings appear in order Critical → Important → Moderate → Low
- [ ] **Within each severity band**: Higher-CVSS findings appear before lower-CVSS findings
- [ ] **No misordering**: Scan sequentially and confirm no reversal

### 3.8 Report Files Table (Assessment ↔ Output Folder)

- [ ] **Every file listed** in the Report Files table of `0-assessment.md` exists in the output folder
- [ ] **`0.1-architecture.md` is listed** in the Report Files table
- [ ] **If `1.2-threatmodel-summary.mmd` was not generated**: it is omitted from the Report Files table (not listed with a "N/A" note)

---

## Phase 4 — Evidence Quality Checks

These checks validate the substance of findings, not just structure. Ideally run by a sub-agent with code access.

### 4.1 Finding Evidence

- [ ] **Every finding** has an Evidence section citing specific files/lines/configs
- [ ] **Evidence is concrete**: Shows actual code or config, not just "absence of config"
- [ ] **For "missing security" claims**: Evidence proves the platform default is insecure (not just that explicit config is absent)

### 4.2 Verify-Before-Flagging Compliance

- [ ] **Security infrastructure inventory** was performed before STRIDE analysis (check for platform security defaults verification in findings)
- [ ] **No false positive patterns**: No finding claims "missing mTLS" when Dapr Sentry is present, or "missing RBAC" on K8s ≥1.6, etc.
- [ ] **Finding classification applied**: Every documented finding is "Confirmed" (not "Needs Verification" — those belong in `0-assessment.md`)

### 4.3 Needs Verification Placement

- [ ] **All "Needs Verification" items** are in `0-assessment.md` under Analysis Context & Assumptions — NOT in `3-findings.md`
- [ ] **No ambiguous findings**: Findings in `3-findings.md` have positive evidence of a vulnerability

---

## Verification Summary Template

After running all checks, produce a summary.

Sub-agent output MUST include:
- Phase name
- Total checks, Passed, Failed
- For each failure: Check ID, file, evidence, exact fix instruction
- Re-run status after fixes

Do not return "looks good" without counts.

```値下げ
## 検証結果

|フェーズ |小切手 |合格 |失敗しました |メモ |
|----------|----------|----------|----------|----------|
| 0 — 共通偏差スキャン | [N] | [N] | [N] | [パターン一致] |
| 1 — ファイルごとの構造 | [N] | [N] | [N] | [問題のあるファイル] |
| 2 — ダイアグラムのレンダリング | [N] | [N] | [N] | [具体的な失敗] |
| 3 — ファイル間の一貫性 | [N] | [N] | [N] | [ギャップが見つかりました] |
| 4 — 証拠の質 | [N] | [N] | [N] | [偽陽性リスク] |
| 5 — JSON スキーマ | [N] | [N] | [N] | [スキーマの問題] |

### 失敗したチェックの詳細
<!-- 失敗したチェックごとに、チェック ID、ファイル、問題点、修正案をリストします -->
「」

---

## フェーズ 5 — Threat-inventory.json スキーマの検証

これらのチェックは、ステップ 8b で生成された JSON インベントリ ファイルを検証します。このファイルは比較モードにとって重要です。

### 5.1 スキーマフィールド

- [ ] **`schema_version` フィールド** — 存在し、`"1.0"` (スタンドアロン) または `"1.1"` (増分) と同等です。レポートに `"incremental": true` が含まれる場合、schema_version は `"1.1"` でなければなりません。それ以外の場合は `"1.0"`。
- [ ] **`commit` フィールド** — 存在します (短い SHA または `"Unknown"`)
- [ ] **`components` 配列** — 空ではなく、少なくとも 1 つのエントリがあります
- [ ] **コンポーネント ID** — すべてのコンポーネントには `id` (PascalCase)、`display`、`type`、`boundary` があります。
- [ ] **コンポーネント フィールド名の準拠** — コンポーネントは `"display"` (`"display_name"` ではありません) を使用します。 Grep: `"display_name"` は 0 件の一致を返す必要があります。
- [ ] **脅威フィールド名の準拠** — 脅威は `"stride_category"` (`"category"` ではありません) を使用します。脅威には `"title"` と `"description"` の両方が含まれます (`description` だけではなく、`"name"` もありません)。脅威→コンポーネントのリンクは `"identity_key"."component_id"` 内にあります (脅威オブジェクトのトップレベル `"component_id"` ではありません)。 Grep:identity_key の外側のトップレベル `"category":` は 0 一致を返す必要があります。 Grep: すべての脅威オブジェクトには `"title":` が含まれている必要があります。
- [ ] **`boundaries` 配列** — 存在します (フラット システムの場合は空にすることができます)
- [ ] **`flows` 配列** — 現在、各フローには正規の ID 形式 `DF_{Source}_to_{Target}` があります
- [ ] **`threats` 配列** — 空ではない
- [ ] **`findings` 配列** — 空ではない
- [ ] **`metrics` オブジェクト** — `total_components`、`total_threats`、`total_findings` とともに存在します

### 5.2 メトリクスの一貫性- [ ] **`metrics.total_components == components.length`** — 配列の長さが count と一致します
- [ ] **`metrics.total_threats == threats.length`** — 配列の長さが count と一致します
- [ ] **`metrics.total_findings == findings.length`** — 配列の長さが count と一致します
- [ ] **メトリクスはマークダウン レポートと一致します** — `total_threats` は STRIDE サマリー テーブルの合計と等しく、`total_findings` は `3-findings.md` の `### FIND-` カウントと等しくなります
- [ ] **切り捨て回復ゲート** — 上記で配列の長さの不一致が検出された場合は、ファイルが再生成された (パッチが適用されていない) ことを確認します。チェック: 脅威が 40 を超えるリポジトリのファイル サイズが 10 KB を超える。脅威配列には、`2-stride-analysis.md` に表示されるすべてのコンポーネントのエントリがあります。
- [ ] **事前書き込み戦略への準拠** — `metrics.total_threats > 50` の場合、JSON が単一の `create_file` 呼び出しではなく、サブエージェント委任、Python スクリプト、またはチャンク追加によって書き込まれたことを確認します。証拠: JSON ファイルに対する `agent` 呼び出し、`_extract.py` スクリプト、または複数の `replace_string_in_file` 操作のログを確認してください。

### 5.3 決定論的なアイデンティティの安定性 (比較の準備のため)

- [ ] **コンポーネントには決定的な ID フィールドが含まれます** - すべてのコンポーネントには `aliases` (配列)、`boundary_kind`、および `fingerprint` があります
- [ ] **`boundary_kind` 有効な値** - すべてのコンポーネントの `boundary_kind` は、`MachineBoundary`、`NetworkBoundary`、`ClusterBoundary`、`ProcessBoundary`、`PrivilegeBoundary`、`SandboxBoundary` のいずれかです。 ❌ その他の値 (例: `DataStorage`、`ApplicationCore`、`deployment`、`trust`) → FAIL
- [ ] **境界には決定論的な ID フィールドが含まれます** - すべての境界には `kind`、`aliases` (配列)、および `contains_fingerprint` があります
- [ ] **境界 `kind` の有効な値** — すべての境界の `kind` は、`boundary_kind` と同じ 6 つの TMT 調整値のいずれかです。 ❌ その他の値 → FAIL
- [ ] **正規コンポーネント ID が重複しない** — `components[].id` 値は正規化後は一意になります
- [ ] **エイリアス マッピングは一貫しています** - 同じインベントリ内の 2 つの無関係なコンポーネント ID の下にエイリアスは表示されません
- [ ] **フィンガープリント証拠フィールドは安定版のみ** — `fingerprint` は、自由形式の散文ではなく、ソース ファイル/トポロジ/タイプ/プロトコルを使用します
- [ ] **決定論的順序付けが適用** — 正規キー (`components.id`、`boundaries.id`、`flows.id`、`threats.id`、`findings.id`) によって並べ替えられた配列

### 5.4 比較ドリフト ガードレール (比較出力を検証する場合)- [ ] **信頼性の高い名前変更候補は追加/削除として残されません** — エイリアス/ソース ファイル/トポロジの強い重複があるコンポーネント ペアは `renamed`/`modified` として分類されます
- [ ] **境界名の変更候補は包含オーバーラップを使用します** — 同じ `kind` + 高い `contains` オーバーラップは、`added` + `removed` ではなく、境界 `renamed` として分類されます。
- [ ] **分割/マージ境界遷移が認識されました** — 1 対多および多対 1 の包含遷移は `split`/`merged` カテゴリにマッピングされます

### 5.5 比較整合性チェック (比較出力を検証する場合)

- [ ] **ベースライン ≠ 現在のコミット** — `metadata.json` → `baseline.commit` は `current.commit` とは異なる必要があります。同一コミットの比較は無効です (比較する実際のコード変更はゼロです)。
- [ ] **変更されたファイル数 > 0** — `metadata.json` → `git_diff_stats.files_changed` は > 0 である必要があります。変更されたファイルが 0 件との比較にはコード デルタがないため、無意味です。
- [ ] **継続時間 > 0** — `metadata.json` → `duration` は `"0m 0s"` または 2 分未満の値であってはなりません。本物の比較には、2 つのインベントリを読み取り、複数信号のマッチングを実行し、ヒートマップを計算し、HTML を生成する必要があり、これにはリアルタイムがかかります。
- [ ] **外部フォルダー参照は禁止** — `metadata.json` およびすべての出力ファイルには、`D:\One\tm` または分析対象のリポジトリ外のフォルダーへの参照が含まれていてはなりません。レポートは、現在のリポジトリ内のフォルダーのみを参照する必要があります。
- [ ] **再利用防止検証** — 比較出力は、以前の `threat-model-compare-*` フォルダーからコピーするのではなく、新たに生成する必要があります。 `metadata.json` タイムスタンプが現在の実行のものであることを確認して検証します。
- [ ] **方法論ドリフト率** — `diff-result.json` → `metrics.methodology_drift_ratio` > 0.50 の場合、HTML レポートに方法論ドリフト警告バナーが含まれていることを確認します。比率が計算されていないが、名前変更されたコンポーネントの >50% が同じエイリアス/フィンガープリントを共有している場合は、検証失敗としてフラグを立てます。

---

## フェーズ 6 — 決定的な ID と名前付けの安定性

これらのチェックでは、コンポーネント/境界/フローの命名が決定論的なルールに従っていることを検証し、同じコードの独立した実行間で出力を再現できるようにします。

### 6.1 コンポーネント ID の決定- [ ] **コード成果物から派生したコンポーネント ID** — `threat-inventory.json` のすべてのコンポーネント ID は、実際のクラス名、ファイル パス、デプロイメント マニフェスト `metadata.name`、または構成キーまで追跡する必要があります。抽象的な概念はありません (`ConfigurationStore`、`DataLayer`、`LocalFileSystem`)。ソース ファイル名およびクラス名に対してコンポーネント ID を grep します。少なくとも 80% が直接一致する必要があります。
- [ ] **コンポーネント アンカーの検証** — `threat-inventory.json` 内のすべてのプロセス タイプ コンポーネントには、空でない `fingerprint.source_files` または `fingerprint.source_directories` が必要です。両方が空の場合 → 失敗します (コンポーネントにコード アンカーがありません)。
- [ ] **Helm/K8s ワークロードの名前付け** — K8s にデプロイされたコンポーネントの場合、コンポーネント ID が Helm テンプレートのファイル名やディレクトリではなく、Deployment/StatefulSet YAML の `metadata.name` と一致することを確認します。例: `templates-knowledge-deployment` (ファイル パスから) ではなく、`DevPortal` (デプロイメント名から)。
- [ ] **外部サービスのアンカー** — 外部サービス (リポジトリにソース コードがない) は、統合ポイント (クライアント クラス名、構成キー、または SDK の依存関係) にアンカーする必要があります。 `fingerprint.config_keys` または `fingerprint.class_names` が入力されていることを確認します。
- [ ] **禁止された命名パターンがありません** — コンポーネント ID は汎用ラベルではありません: `ConfigurationStore`、`DataLayer`、`LocalFileSystem`、`SecurityModule`、`NetworkLayer`、`DatabaseAccess` の場合は grep です。 → 0 件の一致を返す必要があります。
- [ ] **頭字語の一貫性** — PascalCase ID では、既知の頭字語はすべて大文字にする必要があります: `API`、`NFS`、`LLM`、`SQL`、`DB`、`AD`、`UI`。 `Api` (`API` である必要があります)、`Nfs` (`NFS` である必要があります)、`Llm` (`LLM` である必要があります) の grep です。 → 0 件の一致を返す必要があります。
- [ ] **一般的なテクノロジの命名精度** - 該当する場合は、次の正確な ID を確認します: `Redis` (`RedisCache` ではない)、`Milvus` (`MilvusDB` ではない)、`NginxIngress` (`IngressNginx` ではない)、`AzureAD` (`AzureAd` ではない)、 `PostgreSQL` (`Postgres` ではありません)。

### 6.2 境界命名の安定性- [ ] **境界 ID は PascalCase です** — `threat-inventory.json` のすべての境界 ID は、デプロイメント トポロジから派生した PascalCase を使用します (例: `K8sCluster`、`External`、`Application`)。コード アーキテクチャ層 (`PresentationLayer`、`BusinessLogic`) ではありません。
- [ ] **単一プロセス アプリにはコード層境界なし** — システムが単一プロセス (1 つの .exe、1 つのコンテナー) の場合、`Application` 境界は 1 つだけ存在する必要があります。プレゼンテーション/ビジネス/データ レイヤーには 4 つ以上の境界はありません。境界を数えて比率を確認します。
- [ ] **K8s マルチサービスのサブ境界** — 複数のデプロイメントを持つ K8s 名前空間の場合、サブ境界が存在することを確認します: `BackendServices`、`DataStorage`、`MLModels`、`Agentic` (該当する場合)。

### 6.3 データフローの完全性

- [ ] **イングレス/リバース プロキシの双方向フロー** - イングレス コンポーネント (Nginx、Traefik) がバックエンドにルーティングする場合は、両方の方向が存在することを確認します: `DF_Ingress_to_Backend` と `DF_Backend_to_Ingress`。入力を通じて転送フローをカウントし、一致する応答フローを確認します。
- [ ] **データベースの双方向フロー** — `DF_Service_to_Datastore` フローごとに、対応する `DF_Datastore_to_Service` 読み取りフローが存在することを確認します。データストア: Redis、Milvus、PostgreSQL、MongoDB など。
- [ ] **フロー カウントの安定性** — `threat-inventory.json` でフローをカウントします。同じコードに対する 2 つの独立した実行では、同じカウントが生成されるはずです (±3 は許容可能)。変更されていないコンポーネントについて、古い分析と HEAD 分析の間でフロー数が 5 を超えて異なる場合は、名前のドリフトとしてフラグを立てます。

### 6.4 カウントの安定性 (クロスラン決定論)

- [ ] **許容範囲内のコンポーネント数** — 同じコードの 2 つの解析を比較する場合、コンポーネント数は ±1 以内でなければなりません。差 ≥3 = 不合格。
- [ ] **公差内の境界数** — 同じコード → ±1 以内の境界数。
- [ ] **プロセス コンポーネントのフィンガープリントの完全性** — `type: "process"` を持つすべてのコンポーネントには、空でない `fingerprint.source_directories` および `fingerprint.class_names` が必要です。プロセスコンポーネントの空の配列 → 失敗。
- [ ] **STRIDE カテゴリの 1 文字の強制** — JSON 内のすべての `threats[].stride_category` は、S、T、R、I、D、E、または A の 1 文字です。フルネームの Grep (`"Spoofing"`、`"Tampering"`、`"Denial of Service"`) → 0 件の一致を返す必要があります。これにより、ヒートマップの計算エラーが防止されます。

---

## フェーズ 7 — 証拠に基づく前提条件と適用範囲の完全性

これらのチェックは、前提条件、階層、およびカバレッジが決定的な証拠に基づくルールに従っていることを検証します。

### 7.1 前提条件となる決定の証拠- [ ] **デプロイメントの証拠がなければ前提条件なし** — `Exploitation Prerequisites` ≠ `None` のすべての結果について、前提条件が実際のデプロイメント構成 (Helm 値、Dockerfile、サービス タイプ、イングレス ルール) を反映していることを確認します。前提条件に `Internal Network` と表示されているが、ネットワーク制限の証拠が存在しない場合 → 失敗します。
- [ ] **同じコード間の前提条件の一貫性** — 同じコードの 2 つの分析により、同じ脆弱性に対して異なる前提条件が生成された場合、スキル ルールは不十分です。調査のためのフラグ。

### 7.1b デプロイメント分類ゲート (必須)

- [ ] **展開分類が存在します** — `0.1-architecture.md` には、`LOCALHOST_DESKTOP`、`LOCALHOST_SERVICE`、`AIRGAPPED`、`K8S_SERVICE`、`NETWORK_SERVICE` のいずれかを含む `**Deployment Classification:**` 行が含まれている必要があります。 ❌ 欠落 → 失敗。
- [ ] **コンポーネント公開テーブルが存在します** — `0.1-architecture.md` には、コンポーネント、リッスンオン、認証が必要、到達可能性、最小前提条件、派生層の列を持つ `### Component Exposure Table` が含まれている必要があります。 ❌ 欠落 → 失敗。
- [ ] **エクスポージャ テーブルの完全性** - キー コンポーネント テーブルのすべてのコンポーネントには、コンポーネント エクスポージャ テーブルの対応する行があります。 ❌ 行が欠落している → 失敗。
- [ ] **T1 で適用される展開分類** — 展開分類が `LOCALHOST_DESKTOP` または `LOCALHOST_SERVICE` の場合:
  - `Exploitation Prerequisites` = `None` を使用して検出結果をカウントします。 ❌ カウント > 0 → FAIL (`Local Process Access` または `Host/OS Access` でなければなりません)。
  - `## Tier 1` の検出結果をカウントします。 ❌ カウント > 0 → FAIL (ローカルホスト/デスクトップ アプリの場合は T2+ である必要があります)。
  - CVSS の `AV:N` の各検出結果について、コンポーネントの `Reachability` 列を確認します。 ❌ `AV:N` と `Reachability ≠ External` → 失敗。
- [ ] **前提条件の下限適用** — すべての検出結果について、エクスポージャー テーブルで検出結果の `Component` を検索します。検出結果の `Exploitation Prerequisites` は、テーブル内の `Min Prerequisite` 以上である必要があります。検出結果の階層は `Derived Tier` 以上である必要があります。 ❌ 検索には `None` がありますが、テーブルには `Local Process Access` と表示されます → FAIL。
- [ ] **証拠の前提条件** — すべての調査結果の `#### Evidence` セクションには、前提条件を決定する特定のコード/構成を引用する `**Prerequisite basis:**` 行が含まれている必要があります。 ❌ 欠落または汎用 (「コードベースで見つかった」) → 失敗。

### 7.2 対象範囲の完全性- [ ] **テクノロジ カバレッジ チェック** — リポジトリ内の各主要テクノロジ (Redis、PostgreSQL、Docker、K8s、ML/LLM、NFS など) について、少なくとも 1 つの調査結果または文書化された緩和策がそれに対処していることを確認します。 `0.1-architecture.md` テクノロジー スタック テーブルをスキャンし、各テクノロジーについて `3-findings.md` を grep して一致する結果を見つけます。
- [ ] **最小検出しきい値** — 小規模リポジトリ (<20 ファイル): 検出結果が 8 つ以上。中 (20-100): ≥12;大 (100+): ≥18。 `### FIND-` の見出しをカウントし、リポジトリのサイズと照らし合わせて確認します。
- [ ] **コンテキスト認識制限内のプラットフォーム率** — 導入パターンの検出: go.mod に `controller-runtime`/`kubebuilder`/`operator-sdk` が含まれる場合 → K8s オペレーター (制限 ≤35%);それ以外の場合 → スタンドアロン アプリ (制限 ≤20%)。プラットフォームのステータスの脅威 / 脅威の合計をカウントします。制限を超えた場合→FAIL。評価で検出されたパターンを文書化します。
- [ ] **前提条件なしの DoS = 調査結果** — `Prerequisites: None` を伴うすべての DoS 脅威 (`.D`) には、対応する調査結果が必要です。前提条件なしで `.D` 脅威の STRIDE 分析を grep し、それぞれがカバレッジ テーブルの検出結果 ID にマップされていることを確認します。

### 7.3 セキュリティインフラストラクチャの認識

- [ ] **セキュリティ インフラストラクチャのインベントリについて言及** — `0.1-architecture.md` または `2-stride-analysis.md` がコードベースに存在する場合、セキュリティ コンポーネント (サービス メッシュ、証明書管理、認証ミドルウェア) を参照していることを確認します。 Dapr Sentry が展開されている場合、mTLS に「欠落」のフラグを付けることはできません。
- [ ] **セキュリティ不足の主張に対する立証責任** — 「X が不足している」と主張するすべての発見は、明示的な設定が存在しないだけでなく、プラットフォームのデフォルトが安全ではないことを証明する必要があります。最も重要度の高い「欠落」所見をスポットチェックします。

---

## フェーズ 8 — 比較 HTML レポート構造 (比較出力のみ)

これらのチェックにより、HTML 比較レポートの構造が検証されます。

### 8.1 HTML 比較レポートの構造- [ ] **ちょうど 4 つの `<h2>` セクション** - HTML には、`<h2>` の見出しが順番に含まれている必要があります:「概要」、「脅威層の分布」、「STRIDE-A ヒートマップ (デルタ インジケーター付き)」、「比較基準 — コンポーネント マッピング」。 ❌ 「全体的なリスク シフト」、「主要なデルタ メトリクス」、「メトリクスの概要」、「結果の差異」などの追加セクション `<h2>` → FAIL (これらはインライン要素であるか削除されています)。 ❌ 4 つのいずれかが欠けている → 不合格。
- [ ] **調査結果の差分セクションなし** — HTML には、「調査結果の差分」`<h2>` セクションまたは調査結果の差分サブセクション (修正、削除、分析ギャップ、新規、変更、未変更) を含めることはできません。存在する場合→失敗。
- [ ] **デルタ メトリック カードなし** — HTML には `.risk-delta` カード (修正された検出結果、新しい検出結果、実質的な変更、削除、分析ギャップ、コード検証済み) を含めることはできません。存在する場合→失敗。
- [ ] **インライン要素としてのリスク シフトとメトリクス バー** — リスク シフトとメトリクス バー (コンポーネント/脅威/境界/フロー/時間) はインライン カード要素であり、`<h2>` セクションではありません。 `<h2>` と表示される場合 → FAIL。
- [ ] **メトリクス バーには信頼境界が含まれます** — メトリクス バーには信頼境界カウント (例: `2 → 2`) が表示されなければなりません。メトリクスバーに境界がない場合 → 失敗します。コンポーネント、脅威、信頼境界、調査結果、コード変更の 5 つの必須メトリック ボックスです。
- [ ] **メトリクス バーの 5 番目のボックスはコード変更です** — 5 番目のメトリクス ボックスには、コミット数と PR 数が表示されなければなりません (例: `142 commits, 23 PRs`)。 ❌「間隔」→失敗。期間/日付は、指標バーではなく比較カード (セクション 1) に表示されるようになりました。
- [ ] **比較カードの構造** — セクション 1 には、ベースライン (ハッシュ、日付、評価)、ターゲット (ハッシュ、日付、評価)、トレンド (方向、期間) の 3 つのサブカードを含む `comparison-cards` div が含まれている必要があります。 ❌ 古い形式の `subtitle` div と `Baseline: SHA → Target: SHA` → 失敗。 ❌ `risk-shift` div を分離 → FAIL (比較カードにマージ)。
- [ ] **ステータス インジケーターの重複禁止** — ステータス情報 (修正/新規/以前は未確認のカウント) は、色付きのステータス概要カードの 1 か所のみに表示されなければなりません。また、メトリクス バーに小さなインライン バッジやテキストとして表示してはなりません。同じカウントがメトリクス バーと色付きカードの両方に表示される場合 → 失敗 (メトリクス バーから削除し、色付きカードは保持します)。
- [ ] **階層ラベルは分析レポートと一致します** — HTML の脅威階層分布セクションでは、「階層 1 — 直接暴露」、「階層 2 — 条件付きリスク」、「階層 3 — 多層防御」のラベルを正確に使用する必要があります。 ❌ 「曝露の可能性」、「理論的」、「高リスク」、または発明されたバリアント → 不合格。
- [ ] **セクションのタイトルは「アーキテクチャの変更」ではなく「比較の基礎」です** — コンポーネント マッピング セクションのタイトルは、「アーキテクチャの変更」ではなく「比較の基礎 - コンポーネント マッピング」にする必要があります。
- [ ] **ヒートマップには 13 列** — STRIDE-A ヒートマップ グリッドには次のものが必要です。 S |た | R |私 | D | E |あ |合計 |ディバイダー | T1 | T2 | T3。 T1/T2/T3 カラムがない場合 → FAIL。ヒートマップのタイトルには「(デルタ インジケーター付き)」を含める必要があります。### 8.2 ヒートマップの精度 (比較出力)

- [ ] **ヒートマップはすべてゼロではありません** — `stride_heatmap.components` 内の `baseline.Total` と `current.Total` をすべて合計します。どちらかの合計が 0 であるが、対応するインベントリに脅威がある場合 → FAIL (ヒートマップ計算のバグ)。
- [ ] **名前変更されたコンポーネント行が重複していない** — `components_diff.renamed` のすべてのエントリについて、ヒートマップに名前変更されたコンポーネント (現在の名前を使用) が 2 行 (すべてゼロのベースラインが 1 つ、すべてゼロの電流が 1 つ) ではなく、正確に 1 行あることを確認します。
- [ ] **ヒートマップ異常検出が実行されました** — `baseline.Total > 0, current.Total == 0` (消失) を含むすべてのヒートマップ行および `baseline.Total == 0, current.Total > 0` (表示) を含むすべての行: 指紋のクロスチェックが実行されたことを確認します。消えた、または出現したペアがソース ファイル、クラス名、または名前空間を共有している場合 → それは名前変更が失敗しているため、再分類する必要があります。ヒートマップには、共有ソース ファイルと一致するすべてゼロ/すべて新しいペアがあってはなりません。
- [ ] **比較信頼度スコアが存在します** — `diff-result.json` には `comparison_confidence` フィールド (「高」または「低」) が含まれている必要があります。未解決のヒートマップ異常が 3 つ以上存在する場合 → 信頼性は「低」である必要があり、HTML に警告バナーが表示されます。
- [ ] **コンポーネントごとの STRIDE 演算** — 各ヒートマップ行: ベースラインと現在の両方の `S+T+R+I+D+E+A == Total` および `T1+T2+T3 == Total`。不一致がある場合 → 失敗します。
- [ ] **デルタ矢印は JSON データと一致します** — ヒートマップ セルごとに、`delta = current - baseline`。デルタ == 0 の場合、矢印は表示されません。デルタ > 0 の場合、▲。デルタ < 0 の場合、▼。少なくとも 3 つのコンポーネントをスポットチェックします。
- [ ] **コンポーネント削除ソース ファイルの検証** — `components_diff.removed` のすべてのコンポーネントについて、その `source_files` が現在のコミットに本当に存在しないことを確認します。ソース ファイルがまだ存在する場合 → 名前変更または方法論のギャップとして再分類します。