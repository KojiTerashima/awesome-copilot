# フェーズ 1: アーキテクチャ アドバイザー

このファイルにはフェーズ 1 の詳細な手順が記載されています。SKILL.md からフェーズ 1 に入る際は、このファイルを読み、その指示に従ってください。  
Path A（新規設計）と Path B（フェーズ 0 スキャン後の修正）の両方で使用されます。

---

## Path B から入る場合（既存リソース分析後）

フェーズ 0 でスキャンした現行アーキテクチャ図（00_arch_current.html）はすでに存在します。  
この場合、1-1 のプロジェクト名/サービス一覧確認はスキップし、直接修正の対話に入ります。

1. 「ここをどう変更したいですか？」— ユーザーの自然言語リクエスト
2. Delta Confirmation Rule を適用 — 変更に必要な未確定必須項目を確認
3. ファクトチェック — MS Docs と照合確認
4. 更新図を生成（01_arch_diagram_draft.html）
5. 確認後、Phase 2 へ進む

---

**このフェーズの目標**: ユーザーが望む内容を正確に把握し、アーキテクチャを一緒に確定すること。

### 1-1. 図の準備 — 必須情報の収集

図を描く前に、以下のすべての項目が確定するまでユーザーに質問してください。  
**図の生成は、すべての項目が確定した後にのみ行ってください。**

**最初に、プロジェクト名を確認します:**

`ask_user` でデフォルト値を選択肢として提示します。ユーザーが Enter のみ押した場合はデフォルトが適用され、カスタム名を入力することもできます。  
デフォルトはユーザーの要望から推定します（例: RAG chatbot → `rag-chatbot`、data platform → `data-platform`）。

```
ask_user({
  question: "Please choose a project name. It will be used for the Bicep folder name, diagram path, and deployment name.",
  choices: ["<inferred-default>", "azure-project"]
})
```
プロジェクト名は Bicep 出力フォルダー名、図の保存パス、デプロイ名などに使用されます。

**🔹 プロジェクト名質問と同時の並列プリロード（必須）:**

`ask_user` でプロジェクト名を質問している間、ユーザーの応答待ちにアイドル時間が発生します。  
この時間を使って、**後続の質問と Bicep 生成に必要な情報を並列でプリロード**してください。

**ask_user と同時に呼び出すツール:**

```
// Call ask_user + the tools below simultaneously in a single response
[1] ask_user — Project name question

[2] view — Load reference files (pre-acquire Stable information)
    - references/service-gotchas.md
    - references/ai-data.md
    - references/azure-dynamic-sources.md
    - references/architecture-guidance-sources.md

[3] web_fetch — Pre-fetch architecture guidance (when workload type is identified)
    - Up to 2 targeted fetches based on decision rules in architecture-guidance-sources.md

[4] web_fetch — Fetch MS Docs for services mentioned by the user (pre-acquire Dynamic information)
    - e.g., Foundry → API version, model availability page
    - e.g., AI Search → SKU list page
    - Use URL patterns from azure-dynamic-sources.md
```

**メリット**: ユーザーがプロジェクト名を入力している間に、すべての情報を読み込めるため、  
プロジェクト名確定直後に SKU/リージョン質問を正確な選択肢で即時提示できます。  
逐次実行より待ち時間を大幅に短縮できます。

**注意:**
- プリロード対象はプロジェクト名に依存しない情報のみ（名前依存のものは含めない）
- web_fetch はユーザーの初回リクエストで言及されたサービスに対してのみ実施（推測しない）
- Azure CLI チェック（`az account show`）はこの時点では実行しない — アーキテクチャ確定時にプリロード

**🔹 アーキテクチャガイダンスの活用（質問深度の調整）:**

プリロード時に取得したアーキテクチャガイダンス文書から、**設計上の意思決定ポイント**を抽出し、  
後続のユーザー質問に自然に織り込んでください。

**目的**: SKU/リージョンのような仕様確認だけでなく、  
公式アーキテクチャガイダンスが推奨する**設計判断ポイント**も質問に反映することです。

**例 —「RAG chatbot」が要望された場合:**
- Baseline Foundry Chat Architecture（A6）を取得
- 文書から推奨設計判断ポイントを抽出:
  → ネットワーク分離レベル（完全プライベートかハイブリッドか？）  
  → 認証方式（マネージド ID か API キーか？）  
  → データ取り込み戦略（push か pull indexing か？）  
  → 監視範囲（Application Insights は必要か？）
- これらをユーザー質問に自然に含める

**注意:**
- アーキテクチャガイダンスから抽出するのは**「聞くべきポイント」**であり、「回答」ではない
- SKU/API version/region などのデプロイ仕様は引き続き `azure-dynamic-sources.md` によってのみ決定
- 取得予算: 最大 2 ドキュメント。全件走査はしない

**必須確認項目:**
- [ ] Project name（default: `azure-project`）
- [ ] Service list（どの Azure サービスを使うか）
- [ ] 各サービスの SKU/tier
- [ ] ネットワーク方式（Private Endpoint 利用）
- [ ] デプロイ先リージョン（region）

**質問原則:**
- ユーザーがすでに言及した情報は再質問しない
- 図に直接表れない実装詳細（indexing method、query volume など）は聞かない
- 一度に質問しすぎず、未確定の重要項目のみを簡潔に聞く
- 明確なデフォルトがある項目（例: PE 有効）は仮定して確認だけでよい。ただし location は必ずユーザー確認すること
- **SKU、モデル、サービスオプションを聞く際は、MS Docs で確認した利用可能な選択肢をすべて提示し、MS Docs URL も必ず示すこと。** ユーザーが参照して自分で判断できるようにする。部分提示や恣意的な除外は禁止

**🔹 VM/リソース SKU 選定 — リージョン可用性の事前確認が必須:**

VM やその他リソースの SKU をユーザーに尋ねる**前に**、対象リージョンで実際に利用可能な SKU を必ず確認してください。  
特定リージョンで容量制約により SKU がブロックされていると、デプロイは失敗します。

**VM SKU 検証方法:**
```powershell
# Query only VM SKUs available without restrictions in the target region
az vm list-skus --location "<LOCATION>" --size Standard_D2 --resource-type virtualMachines `
  --query "[?restrictions==``[]``].name" -o tsv
```

**原則:**
- 未検証 SKU を選択肢に含めない
- 「よく使う SKU」を記憶だけで推奨しない — az cli または MS Docs で必ず検証
- `ask_user` の選択肢には検証済み SKU のみを含める
- ユーザー指定 SKU であっても、続行前に可用性を検証する

**この原則は VM だけでなく、容量制約のあるすべてのリソース（Fabric Capacity など）に同様に適用されます。**

**🔹 サービス選択肢調査の原則 —「記憶ベース列挙」禁止:**

ユーザーがサービスカテゴリ（「Spark の選択肢は？」「メッセージキューの選択肢は？」）を尋ねた場合、または特定機能向けにサービスを探索する必要がある場合:

**絶対にしてはいけないこと:**
- 記憶から 2〜3 サービスだけの URL を直接取得して列挙する
- 「Azure では X は A と B がある」と断定する

**必ず行うこと:**
1. **web_search でカテゴリ全体を探索** — `"Azure managed Spark options site:learn.microsoft.com"` のようにカテゴリレベルで検索し、まず存在するサービスを発見
2. **v1 スコープと照合** — 検索結果に関係なく、v1 対象サービス（Foundry、Fabric、AI Search、ADLS Gen2 など）が該当カテゴリに含まれるか確認。例: 「Spark」→ Microsoft Fabric の Data Engineering も Spark を提供
3. **発見した選択肢を対象に取得** — 検索で見つかった各サービスの MS Docs を取得し、正確な比較情報を収集
4. **全選択肢をユーザーへ提示** — 省略なく包括的に比較提示する

**例 —「使える Spark インスタンスは？」と聞かれた場合:**
```
Wrong approach: Fetch only Databricks URL + Synapse URL → Compare only 2
Correct approach: web_search("Azure managed Spark options") → Discover Databricks, Synapse, Fabric Spark, HDInsight
            → v1 scope check: Fabric is v1 scope and provides Spark → MUST include
            → Targeted fetch of each service's MS Docs → Present full comparison table
```

この原則はサービスカテゴリ探索だけでなく、ユーザーが「代替案」「他の選択肢」「比較」などを求めるすべての場面に適用されます。

**🔹 ask_user ツール — 必須利用:**

選択肢付きの質問では `ask_user` ツールを必ず使ってください。矢印キーで選択でき、カスタム入力も可能です。

**ask_user 利用ルール:**
- 2 つ以上の選択肢がある質問は **必ず** ask_user を使用（テキスト列挙しない）
- **`choices` は必ず string 配列（`["A", "B"]`）で渡すこと** — string（`"A, B"`）はエラー
- 推奨選択肢がある場合は先頭に置き、末尾に `(Recommended)` を付与
- 選択肢に参照情報を含める — 例: `"Standard S1 - Recommended for production. Ref: https://..."`
- **1 回の呼び出しで 1 問のみ** — 複数項目が必要なら項目ごとに順次 ask_user を呼ぶ
- 選択肢は最大 4 件。5 件以上ある場合は一般的な 3〜4 件のみ提示（ユーザーはカスタム入力可能）
- 複数選択が必要な場合は質問を分割する

**ask_user が必要な項目:**
- デプロイ location（region）選択
- SKU/tier 選択
- モデル選択（chat model、embedding model など）
- ネットワーク方式選択
- サブスクリプション選択（Phase 1 Step 2）
- リソースグループ選択（Phase 1 Step 3）
- その他、ユーザー選択が必要な質問すべて

**利用例:**
```
// Project name is free-form input so ask_user is not used (ask as text)
// SKU, region, etc. with defined choices use ask_user:

// 1. SKU question
ask_user({
  question: "Please select the SKU for AI Search. Ref: https://learn.microsoft.com/en-us/azure/search/search-sku-tier",
  choices: [
    "Standard S1 - Recommended for production (Recommended)",
    "Basic - For dev/test, up to 15 indexes",
    "Standard S2 - High-traffic production",
    "Free - Free trial, 50MB storage"
  ]
})

// 2. Region question (separate call — only 1 question per call)
ask_user({
  question: "Please select the Azure region for deployment. Ref: https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models",
  choices: [
    "Korea Central - Korea region, supports most services (Recommended)",
    "East US - US East, supports all AI models",
    "Japan East - Japan East, close to Korea"
  ]
})
```

> **注意**: 上記例の SKU と region は説明用です。実際に質問する際は、web_fetch で MS Docs の最新情報を取得して動的に choices を構成してください。ハードコード禁止。

**例 — ユーザー入力が不十分な場合:**
```
User: "I want to build a RAG chatbot. Using a GPT model in Foundry and AI Search."

→ Confirmed: Microsoft Foundry, Azure AI Search
→ Still undecided: Project name, specific model name, embedding model, networking (PE?), SKU, deployment location

The agent first confirms the project name via ask_user (default: rag-chatbot).
Then provides choices for each undecided item via the ask_user tool.
Include MS Docs URLs in the choices so the user can reference them directly.
```

**🚨🚨🚨 [HARD GATE] 仕様収集完了 → 図生成は必須 🚨🚨🚨**

**すべての確認項目が埋まった直後に、必ず以下の手順を順番どおり実行してください。1 つでも欠けると Phase 1 は未完了です。**

1. 確定したサービス一覧に基づいて **services JSON + connections JSON** を作成
2. 組み込み図エンジンで **`<project-name>/01_arch_diagram_draft.html`** を生成
3. `Start-Process` でブラウザに自動表示
4. 下記 **report format** でユーザーに表示 — **詳細構成テーブル**を必ず含める
5. ユーザーに質問: **"Would you like to change or add anything?"**
6. ユーザーに変更がなければ → Phase 2 へ遷移（ask_user で次ステップ案内）

**絶対にしてはいけないこと:**
- ❌ 図を生成せずに「アーキテクチャは確認できました。次に進みますか？」と聞く
- ❌ 図生成を Phase 2 以降へ先送り
- ❌ 「図は後で作成します」と言う
- ❌ 仕様収集完了だけで「アーキテクチャ確定」と宣言する
- ❌ 図を生成したのに構成テーブルを表示しない
- ❌ 「変更ありますか？」を省略して Phase 2 へ直行する

**検証条件**: `01_arch_diagram_draft.html` が生成されていない場合、Phase 2 への移行は不可。

**図完成後の報告フォーマット（全セクション必須）:**
```
## Architecture Diagram

[Interactive diagram link — auto-opened in browser]

### Confirmed Configuration

| Service | Type | SKU/Tier | Details |
|---------|------|----------|---------|
| [Service name] | [Azure resource type] | [SKU] | [Key config: model, capacity, etc.] |
| ... | ... | ... | ... |

**Networking**: [VNet + Private Endpoint / Public / etc.]
**Location**: [confirmed region]
```

**報告表示後、すぐに `ask_user` を次の選択肢で実行:**
```
ask_user({
  question: "The architecture diagram and configuration are ready. What would you like to do?",
  choices: [
    "Looks good — proceed to Bicep code generation (Recommended)",
    "I want to modify the architecture",
    "Add more services"
  ]
})
```

- 「proceed」の場合 → Phase 2 遷移（subscription/RG 情報収集）
- 「modify」または「add」の場合 → 変更を反映し図を再生成、再度レポート表示

**🚨 構成テーブルは任意ではありません。** ユーザーが先に進む前に確認内容を視覚的に検証するために必要です。テーブルなしでは検証できません。

### 1-2. インタラクティブ HTML 図の生成

組み込みの **diagram engine**（skill に同梱された Python スクリプト）で、インタラクティブ HTML 図を作成します。  
`pip install` は不要です。`scripts/` フォルダー内のスクリプトを直接使えるため、ネットワーク接続もパッケージ導入も不要です。  
公式 Azure アイコン 605+ が組み込まれています。

**図ファイル命名規則:**

すべての図は Bicep プロジェクトフォルダー（`<project-name>/`）内に生成されます。  
ステージごとに番号プレフィックスで管理し、前段階のファイルは上書きしません。

| Stage | File Name | When Generated |
|-------|-----------|----------------|
| Phase 1 design draft | `01_arch_diagram_draft.html` | When architecture design is confirmed |
| Phase 4 What-if preview | `02_arch_diagram_preview.html` | After What-if validation |
| Phase 4 deployment result | `03_arch_diagram_result.html` | After actual deployment completes |

**Built-in module path discovery + Python path discovery:**

**🚨 Python path と built-in module path は Phase 1 のプリロード時に 1 回だけ検証し、以後の図生成で再利用します。毎回再探索してはいけません。**

```powershell
# ─── Step 1: Python Path Discovery ───
# ⚠️ Get-Command python may pick up the Windows Store alias, so filesystem discovery is done first
$PythonCmd = $null

# Priority 1: Direct discovery of actual installation path (most reliable)
$PythonExe = Get-ChildItem -Path "$env:LOCALAPPDATA\Programs\Python" -Filter "python.exe" -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.FullName -notlike '*WindowsApps*' } |
  Select-Object -First 1 -ExpandProperty FullName
if ($PythonExe) { $PythonCmd = $PythonExe }

# Priority 2: Program Files discovery
if (-not $PythonCmd) {
  $PythonExe = Get-ChildItem -Path "$env:ProgramFiles\Python*", "$env:ProgramFiles(x86)\Python*" -Filter "python.exe" -Recurse -ErrorAction SilentlyContinue |
    Select-Object -First 1 -ExpandProperty FullName
  if ($PythonExe) { $PythonCmd = $PythonExe }
}

# Priority 3: Find in PATH (only if not a Windows Store alias)
if (-not $PythonCmd) {
  foreach ($cmd in @('python3', 'py')) {
    $found = Get-Command $cmd -ErrorAction SilentlyContinue
    if ($found -and $found.Source -notlike '*WindowsApps*') { $PythonCmd = $cmd; break }
  }
}

if (-not $PythonCmd) {
  Write-Host ""
  Write-Host "Python is not installed or not found in PATH." -ForegroundColor Red
  Write-Host ""
  Write-Host "Please install using one of the following methods:" -ForegroundColor Yellow
  Write-Host "  1. winget install Python.Python.3.12"
  Write-Host "  2. Download from https://www.python.org/downloads/"
  Write-Host "  3. Search for 'Python 3.12' in the Microsoft Store and install"
  Write-Host ""
  Write-Host "After installation, restart your terminal and try again."
  return
}

# ─── Step 2: Built-in Script Path Discovery (no pip install needed) ───
# Priority 1: Project local skill folder
$ScriptsDir = Get-ChildItem -Path ".github\skills\azure-architecture-autopilot" -Filter "cli.py" -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.Directory.Name -eq 'scripts' } |
  Select-Object -First 1 -ExpandProperty DirectoryName
# Priority 2: Global skill folder
if (-not $ScriptsDir) {
  $ScriptsDir = Get-ChildItem -Path "$env:USERPROFILE\.copilot\skills\azure-architecture-autopilot" -Filter "cli.py" -Recurse -ErrorAction SilentlyContinue |
    Where-Object { $_.Directory.Name -eq 'scripts' } |
    Select-Object -First 1 -ExpandProperty DirectoryName
}

# ─── Step 3: Diagram Generation (CLI method — direct script execution) ───
$OutputFile = "<project-name>\01_arch_diagram_draft.html"

& $PythonCmd "$ScriptsDir\cli.py" `
  --services '<services_JSON>' `
  --connections '<connections_JSON>' `
  --title "Architecture Title" `
  --vnet-info "10.0.0.0/16 | pe-subnet: 10.0.1.0/24" `
  --output $OutputFile

# Automatically open in browser after generation
Start-Process $OutputFile
```

**Python API method is also available (alternative):**

JSON が非常に大きい場合、CLI 引数長制限を避けるため Python API を直接呼び出せます。  
`sys.path` に scripts フォルダーを追加して built-in module を import します。

```python
import sys, os
# Add scripts folder to Python path (use built-in module without pip install)
scripts_dir = r"<absolute path to scripts folder>"  # $ScriptsDir value found in Step 2
sys.path.insert(0, scripts_dir)

from generator import generate_diagram

services = [...]   # services JSON
connections = [...] # connections JSON

html = generate_diagram(
    services=services,
    connections=connections,
    title="Architecture Title",
    vnet_info="10.0.0.0/16 | pe-subnet: 10.0.1.0/24",
    hierarchy=None  # Only used for multiple subscriptions/RGs
)

with open("<project-name>/01_arch_diagram_draft.html", "w", encoding="utf-8") as f:
    f.write(html)
```

**🔹 CLI vs Python API の選択基準:**

| Scenario | Method | Reason |
|----------|--------|--------|
| 10 or fewer services | CLI (`python scripts/cli.py`) | シンプルで高速 |
| More than 10 services or using hierarchy | Python API (sys.path addition) | CLI 引数長制限を回避 |
| Multi-subscription/RG diagrams | Python API + `hierarchy` parameter | 階層構造表現に対応 |

**サポート対象サービスタイプ一覧:**

skill 内の `references/` 配下の built-in reference files で確認できます。  
services JSON format セクションに、サポートされる service type 値を記載しています。

> **図生成順序**: (1) Python path 検証 → (2) built-in module path 検証 → (3) services/connections JSON 作成 → (4) 実行。Python が未導入なら JSON 作成前にインストール案内を行ってください。これにより、Python 不足で失敗する無駄を防げます。

> **🚨 図の自動表示（例外なし）**: built-in diagram engine で HTML を生成した場合、状況に関係なく **必ず** ブラウザで開いてください。例外はありません。図を（再）生成するたびに `Start-Process` を実行します。図生成とブラウザ表示は常に単一の PowerShell コマンドブロックで実行します。
>
> **適用場面（以下に限らず、HTML 図を生成するすべての場面）:**
> - Phase 1 design draft（`01_arch_diagram_draft.html`）
> - Delta Confirmation 後の再生成
> - Phase 4 What-if preview（`02_arch_diagram_preview.html`）
> - Phase 4 deployment result（`03_arch_diagram_result.html`）
> - デプロイ後のアーキテクチャ変更（`04_arch_diagram_update_draft.html`）
> - その他、任意理由で図を再生成するすべての場合

**services JSON format:**

ユーザーが確定したサービス一覧に基づき動的に構成します。以下は JSON 構造です。

```json
[
  {"id": "uniqueID", "name": "Service Display Name", "type": "iconType", "sku": "SKU", "private": true/false,
   "details": ["Detail line 1", "Detail line 2"]}
]
```

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `id` | Yes | string | 一意識別子（kebab-case） |
| `name` | Yes | string | 図上に表示される名称 |
| `type` | Yes | string | サービスタイプ（下記一覧から選択） |
| `sku` | | string | SKU/tier 情報 |
| `private` | | boolean | Private Endpoint 接続（default: false） |
| `details` | | string[] | サイドバーに表示する追加情報 |
| `subscription` | | string | Subscription 名（hierarchy 利用時は必須） |
| `resourceGroup` | | string | Resource group 名（hierarchy 利用時は必須） |

**Service Type — Canonical Reference:**

> ⚠️ **CRITICAL**: 以下テーブルの **canonical type** を必ず使用してください。Azure ARM resource 名（例: `private_endpoints`, `storage_accounts`, `data_factories`）は使わないでください。generator は一般的な揺れを正規化できますが、canonical type を使うことで正しいアイコン描画、PE 検出、色分けが保証されます。

| Category | Canonical Type | Azure Resource | Icon |
|----------|---------------|----------------|------|
| **AI** | `ai_foundry` | Microsoft.CognitiveServices/accounts (kind: AIServices) | AI Foundry |
| | `openai` | Microsoft.CognitiveServices/accounts (kind: OpenAI) | Azure OpenAI |
| | `ai_hub` | Foundry Project | AI Studio |
| | `search` | Microsoft.Search/searchServices | Cognitive Search |
| | `document_intelligence` | Microsoft.CognitiveServices/accounts (kind: FormRecognizer) | Form Recognizer |
| | `aml` | Microsoft.MachineLearningServices/workspaces | Machine Learning |
| **Data** | `fabric` | Microsoft.Fabric/capacities | Microsoft Fabric |
| | `adf` | Microsoft.DataFactory/factories | Data Factory |
| | `storage` | Microsoft.Storage/storageAccounts | Storage Account |
| | `adls` | ADLS Gen2 (Storage with HNS) | Data Lake |
| | `cosmos_db` | Microsoft.DocumentDB/databaseAccounts | Cosmos DB |
| | `sql_database` | Microsoft.Sql/servers/databases | SQL Database |
| | `sql_server` | Microsoft.Sql/servers | SQL Server |
| | `databricks` | Microsoft.Databricks/workspaces | Databricks |
| | `synapse` | Microsoft.Synapse/workspaces | Synapse Analytics |
| | `redis` | Microsoft.Cache/redis | Redis Cache |
| | `stream_analytics` | Microsoft.StreamAnalytics/streamingjobs | Stream Analytics |
| | `postgresql` | Microsoft.DBforPostgreSQL/flexibleServers | PostgreSQL |
| | `mysql` | Microsoft.DBforMySQL/flexibleServers | MySQL |
| **Security** | `keyvault` | Microsoft.KeyVault/vaults | Key Vault |
| | `sentinel` | Microsoft.SecurityInsights | Sentinel |
| **Compute** | `appservice` | Microsoft.Web/sites | App Service |
| | `function_app` | Microsoft.Web/sites (kind: functionapp) | Function App |
| | `vm` | Microsoft.Compute/virtualMachines | Virtual Machine |
| | `aks` | Microsoft.ContainerService/managedClusters | AKS |
| | `acr` | Microsoft.ContainerRegistry/registries | Container Registry |
| | `container_apps` | Microsoft.App/containerApps | Container Apps |
| | `static_web_app` | Microsoft.Web/staticSites | Static Web App |
| | `spring_apps` | Microsoft.AppPlatform/Spring | Spring Apps |
| **Network** | `pe` | Microsoft.Network/privateEndpoints | Private Endpoint |
| | `vnet` | Microsoft.Network/virtualNetworks | VNet |
| | `nsg` | Microsoft.Network/networkSecurityGroups | NSG |
| | `firewall` | Microsoft.Network/azureFirewalls | Firewall |
| | `bastion` | Microsoft.Network/bastionHosts | Bastion |
| | `app_gateway` | Microsoft.Network/applicationGateways | App Gateway |
| | `front_door` | Microsoft.Cdn/profiles (Front Door) | Front Door |
| | `vpn` | Microsoft.Network/virtualNetworkGateways | VPN Gateway |
| | `load_balancer` | Microsoft.Network/loadBalancers | Load Balancer |
| | `nat_gateway` | Microsoft.Network/natGateways | NAT Gateway |
| | `cdn` | Microsoft.Cdn/profiles | CDN |
| **IoT** | `iot_hub` | Microsoft.Devices/IotHubs | IoT Hub |
| | `digital_twins` | Microsoft.DigitalTwins/digitalTwinsInstances | Digital Twins |
| **Integration** | `event_hub` | Microsoft.EventHub/namespaces | Event Hub |
| | `event_grid` | Microsoft.EventGrid/topics | Event Grid |
| | `apim` | Microsoft.ApiManagement/service | API Management |
| | `service_bus` | Microsoft.ServiceBus/namespaces | Service Bus |
| | `logic_apps` | Microsoft.Logic/workflows | Logic Apps |
| **Monitoring** | `log_analytics` | Microsoft.OperationalInsights/workspaces | Log Analytics |
| | `appinsights` | Microsoft.Insights/components | App Insights |
| | `monitor` | Azure Monitor | Monitor |
| **Other** | `jumpbox`, `user`, `devops` | — | Special |

**Private Endpoints を使う場合 — PE ノード追加が必須:**

アーキテクチャに Private Endpoints を含める場合、図に表示するために、サービスごとの PE ノードを services JSON に必ず追加し、connections にも PE リンクを含めてください。

```json
// Add PE node corresponding to each service
{"id": "pe_serviceID", "name": "PE: ServiceName", "type": "pe", "details": ["groupId: correspondingGroupID"]}

// Add service → PE connection in connections
{"from": "serviceID", "to": "pe_serviceID", "label": "", "type": "private"}
```

**🚨🚨🚨 PE 接続とビジネスロジック接続は別物 — 両方必須 🚨🚨🚨**

PE 接続（`"type": "private"`）はネットワーク分離を表します。しかしこれだけでは、図上にサービス間の実際の**データフロー/API 呼び出し**は表現されません。

**必ず両方の接続タイプを含めること:**

1. **ビジネスロジック接続** — サービス間の実データフロー（api, data, security types）
2. **PE 接続** — サービス ↔ PE のネットワーク分離（private type）

```json
// ✅ Correct example — Function App → Foundry
// 1) Business logic: Function App calls Foundry for chat/embedding
{"from": "func_app", "to": "foundry", "label": "RAG Chat + Embedding", "type": "api"}
// 2) PE connection: Foundry's Private Endpoint
{"from": "foundry", "to": "pe_foundry", "label": "", "type": "private"}

// ❌ Wrong example — Only PE connection, no business logic connection
{"from": "foundry", "to": "pe_foundry", "label": "", "type": "private"}
// → No connection line between Function App and Foundry in the diagram, so the architecture flow is not visible
```

**絶対にしてはいけないこと:**
- PE 接続だけ作ってビジネスロジック接続を省略する
- ビジネスロジック接続の `from`/`to` を PE ノードに向ける（**実サービス ID を使う**。PE ではない）
- 「PE があるから接続線は表示される」と思い込む

PE の groupId はサービスごとに異なります。`references/service-gotchas.md` の PE groupId & DNS Zone マッピング表を参照してください。

> **サービス命名規則**: 最新の公式 Azure 名称を必ず使用。名称が不確かな場合は MS Docs で確認すること。  
> サービスごとのリソースタイプと主要プロパティは `references/ai-data.md` を参照。

**connections JSON format:**
```json
[
  {"from": "serviceA_ID", "to": "serviceB_ID", "label": "Connection description", "type": "api|data|security|private"}
]
```

**Connection Types:**

| type | Color | Style | Use For |
|------|-------|-------|---------|
| `api` | Blue | Solid | API calls, queries |
| `data` | Green | Solid | Data flow, indexing |
| `security` | Orange | Dashed | Secrets, auth |
| `private` | Purple | Dashed | Private Endpoint connections |
| `network` | Gray | Solid | Network routing |
| `default` | Gray | Solid | Other |

**🔹 図の多言語原則:**
- services の `name`, `details` と connections の `label` は**ユーザー言語**で記述
- 例: `"label": "RAG Search"`, `"label": "Data Ingestion"`
- 公式 Azure サービス名（Microsoft Foundry、AI Search など）は言語に関係なく常に英語

**🔹 VNet ノード — services JSON へ追加しないこと:**
- VNet は図上で **紫色破線の境界**として自動表示される（PE がある場合）
- services JSON に VNet ノードを別追加すると、境界線と重複して混乱を生む
- VNet 情報（CIDR、subnets）はサイドバーの VNet 境界ラベルで十分伝わる

生成された HTML ファイルのフルパスをユーザーに提示してください。

### 1-3. 対話によるアーキテクチャ最終化

アーキテクチャはユーザーとの対話で段階的に確定します。ユーザーが変更を求めた場合、最初からすべて聞き直さず、**現在の確定状態を基準に要求された変更だけ反映**して図を再生成してください。

**⚠️ Delta Confirmation Rule — サービス追加/変更時の必須検証:**

サービス追加/変更は「単純更新」ではなく、そのサービスに対する**未確定必須項目を再オープンするイベント**です。

**プロセス:**
1. 現在の確定状態と新規要求の差分を取る
2. 新規追加サービスの必須項目を特定（`domain-packs` または MS Docs を参照）
3. サービスのリージョン可用性/選択肢を MS Docs から取得
4. 必須項目に未確定があれば、**先に ask_user で確認**
5. **確認完了後にのみ**図を再生成

**絶対にしてはいけないこと:**
- 必須項目が未確定のまま図更新を確定する
- ユーザーが言及していない下位コンポーネント/ワークロードを恣意的に追加する（例: Fabric 要求に対して OneLake と data pipeline を自動追加）
- 「F SKU」など SKU/model を曖昧に仮定する

**既存の確定済みサービス設定は再質問しないこと。** 新規追加/変更サービスの未確定項目だけ確認します。

---

**🚨🚨🚨 [最優先原則] 設計フェーズ中の即時ファクトチェック 🚨🚨🚨**

**Phase 1 の目的は「実現可能なアーキテクチャ」の確認です。**  
**ユーザー要求を図に反映する前に、必ず web_fetch で MS Docs を直接確認し、実現可否をファクトチェックしてください。**

**設計方針とデプロイ仕様 — 情報経路を分離する:**

| Decision Type | Reference Path | Examples |
|--------------|----------------|----------|
| **Design direction**（アーキテクチャパターン、ベストプラクティス、サービス組み合わせ） | `references/architecture-guidance-sources.md` → targeted fetch | 「推奨 RAG 構成は？」「Enterprise baseline は？」 |
| **Deployment specs**（API version、SKU、region、model、PE mapping） | `references/azure-dynamic-sources.md` → MS Docs fetch | 「API version は？」「このモデルは Korea Central で使える？」 |

- **設計方針は architecture guidance、実デプロイ値は dynamic sources から取得。** この 2 経路を混同しないこと。
- SKU/API version/region の決定に Architecture guidance 文書を使わないこと。
- **すべての要求ごとに Architecture Center 配下を総当たりしないこと。** トリガーベースの targeted fetch を最大 2 文書までで実施。
- 質問タイプごとの trigger/fetch budget/判断規則は `architecture-guidance-sources.md` を参照。

**この原則は例外なくすべての要求に適用:**
- モデル追加/変更 → そのモデルが存在し対象リージョンでデプロイ可能かを MS Docs で検証
- サービス追加/変更 → 対象リージョンで利用可能かを MS Docs で検証
- SKU 変更 → SKU が有効で必要機能をサポートするかを MS Docs で検証
- 機能要求 → 実際にサポートされる機能かを MS Docs で検証
- サービス組み合わせ → サービス間連携が可能かを MS Docs で検証
- **その他すべての要求** → MS Docs でファクトチェック

**MS Docs 検証結果:**
- **可能** → 図に反映
- **不可能** → 理由を直ちに説明し、利用可能な代替案を提示

**ファクトチェック手順 — クロス検証必須:**

ユーザー要求に対し、1 回問い合わせて終わりにしてはいけません。  
**別の MS Docs ページ/ソースによるクロス検証を必ず実施**してください。

> **GHCP 環境制約**: Sub-agent（explore/task/general-purpose）には `web_fetch`/`web_search` がありません。  
> そのため、MS Docs クエリが必要な検証は **main agent が直接実施**する必要があります。

```
[1st Verification] Main agent directly queries MS Docs via web_fetch (primary page)
    ↓
[2nd Verification] Main agent additionally fetches other/related MS Docs pages via web_fetch for cross-checking
    - e.g., Model availability → 1st: models page / 2nd: regional availability or pricing page
    - e.g., API version → 1st: Bicep reference page / 2nd: REST API reference page
    - Compare 1st and 2nd results and flag any discrepancies
    ↓
[Consolidate Results] If both verifications match, respond to the user
    - On discrepancy: Resolve with additional queries, or honestly inform the user about the uncertainty
```

**ファクトチェック品質基準 — 表面的でなく徹底的に:**
- MS Docs ページを取得したら、**関連セクション、タブ、条件を漏れなく確認**
- モデル可用性確認時は、Global Standard、Standard、Provisioned、Data Zone など**全デプロイ種別**を確認。1 種別だけ見て「未対応」と結論づけない
- SKU 確認時は、その SKU がサポートする機能一覧を**完全に**検証
- ページが大きい場合は、関連セクションを**複数回**取得して精度を担保
- 不確実な場合は追加ページを照会。**推測で回答しない**

**絶対にしてはいけないこと:**
- 検証せずに図へ追加する
- 「Bicep 生成時に確認します」「デプロイ時に検証されます」と検証を先送りする
- 記憶だけで「動くはず」と回答する — **MS Docs を必ず直接照会**
- MS Docs を取得したのに一部だけ読んで急いで結論を出す
- 単一クエリで確定する — **別ソースで必ずクロス検証**

**🚫 Sub-Agent 利用ルール:**

**GHCP における Sub-agent = `task` ツール:**
- `agent_type: "explore"` — コードベース探索、ファイル検索などの読み取り専用タスク（**web_fetch/web_search は不可**）
- `agent_type: "task"` — az cli、bicep build などのコマンド実行
- `agent_type: "general-purpose"` — 複雑な Bicep 生成などの高レベルタスク

> **⚠️ Sub-agent ツール制約**: すべての sub-agent（explore/task/general-purpose）は `web_fetch`/`web_search` を使えません。  
> MS Docs クエリを要するファクトチェック、API version 検証、モデル可用性確認などは **main agent が直接実施**してください。

**Foreground vs Background の判断基準:**
- **次ステップへ進む前に結果が必要 → `mode: "sync"`（default）**
  - 例: SKU 一覧取得後に選択肢提示、モデル可用性検証後に図反映
  - ここで background にするとユーザーが結果待ちで停止する
- **待機中に並行できる独立作業がある → `mode: "background"`**
  - 例: クロス検証のため複数 MS Docs ページを同時 web_fetch

**多くのファクトチェックは foreground（`mode: "sync"`）で実行すべき**です。結果がないと次の質問に進めないためです。

**クロス検証を並列で実行する方法:**
```
// Execute 1st and 2nd verification simultaneously (main agent performs directly)
[Simultaneously] Directly query primary MS Docs page via web_fetch (1st)
[Simultaneously] Additionally query related MS Docs page via web_fetch (2nd)
// Compare both results to check for discrepancies
// e.g., Model availability → parallel fetch of models page + regional availability page
```

**絶対にしてはいけないこと:**
- 結果が必要なのに background 実行して、待機中に何もしない
- web_fetch/web_search が必要なタスクを sub-agent に委譲する（main agent が必須実施）
- sub-agent 内部ファイルを直接読もうとする

---

**⚠️ 重要: ユーザーが次ステップ進行を明示承認するまで、shell コマンドは実行しないこと。**  
ただし、上記ファクトチェックのための MS Docs web_fetch は例外的に許可されます。

アーキテクチャが確定したら（ユーザーが図に変更なしと回答）、次ステップへ進むかを確認してください。

**🚨 Phase 2 遷移の前提条件 — 以下すべてを満たしてから質問すること:**

1. 組み込み diagram engine で `01_arch_diagram_draft.html` が**生成済み**
2. 図を**ブラウザで開き**、**構成テーブル付き**の report format でユーザーへ表示済み
3. **"Would you like to change or add anything?"** を質問し、ユーザーが**変更なし**と回答、または修正反映後に**最終確認**済み

**上記のいずれか 1 つでも未達なら、Phase 2 へ進んではいけません。**  
図が未作成なら **今すぐ生成** — 1-2 の手順に従う。  
構成テーブル未表示なら、変更確認の前に **今すぐ表示** してください。

**並列プリロード原則に従い、subscription/RG 選択肢を事前準備するため `az account list` と `az group list` を ask_user と同時実行します。**

```
// Call simultaneously in the same response:
[1] ask_user — "The architecture is confirmed! Shall we proceed to the next step?"
[2] powershell — az account show 2>&1              (pre-check login status)
[3] powershell — az account list --output json      (pre-prepare subscription choices)
[4] powershell — az group list --output json        (pre-prepare resource group choices)
```

ask_user 表示形式:
```
The architecture is confirmed! Shall we proceed to the next step?

✅ Confirmed architecture: [summary]

The following steps will proceed:
1. [Bicep Code Generation] — AI automatically writes IaC code
2. [Code Review] — Automated security/best practice review
3. [Azure Deployment] — Actual resource creation (optional)

Shall we proceed? (If you'd like just the code without deployment, let me know)
```

ユーザー承認後、次の順序で情報を収集します。  
**`az account show` + `az account list` + `az group list` はプリロード時に完了済みのため、subscription/RG の選択肢を即時提示できます。**

**Step 1: Azure Login 検証**

`az account show` の結果はプリロードで取得済みです。追加呼び出し不要。

- ログイン済み → Step 2 へ進む
- 未ログイン → ユーザー案内:
  ```
  Azure CLI login is required. Please run the following command in your terminal:
  az login
  Please let me know once completed.
  ```

**Step 2: Subscription 選択**

`az account list` の結果はプリロードで取得済みです。追加呼び出し不要。

クエリ結果から最大 4 件を `ask_user` の choices として提示。  
5 件以上ある場合は利用頻度の高い 3〜4 件を choices に含める（カスタム入力可）。  
ユーザー選択後、`az account set --subscription "<ID>"` を実行。

**Step 3: Resource Group 確認**

`az group list` の結果はプリロードで取得済みです。追加呼び出し不要。

既存 resource groups を最大 4 件、`ask_user` choices として提示。  
既存を選んだ場合はそのまま使用。カスタム入力で新規名が指定された場合は Phase 4 デプロイ時に作成。

**必須確認項目:**
- [ ] サービス一覧と SKU
- [ ] ネットワーク方式（Private Endpoint 利用）
- [ ] Subscription ID（Step 2 で確認）
- [ ] Resource group 名（Step 3 で確認）
- [ ] Location（ユーザー確認済み — サービス別リージョン可用性は MS Docs で検証済み）

---

## 🚨 Phase 1 完了チェックリスト — Phase 2 移行前の必須確認

Phase 1 を離れる前に、以下を**すべて**確認してください。未完了が 1 つでもあれば Phase 2 へ進んではいけません。

| # | Item | Verification Method |
|---|------|---------------------|
| 1 | 必須仕様の全確認完了 | Project name、services、SKUs、region、networking method がすべて確定 |
| 2 | ファクトチェック完了 | MS Docs のクロス検証を実施済み |
| 3 | **図生成済み** | built-in diagram engine で `01_arch_diagram_draft.html` を生成済み |
| 4 | **構成テーブル表示済み** | Service/Type/SKU/Details の詳細テーブルを report format で表示済み |
| 5 | **ユーザーが図を確認済み** | ブラウザ自動表示 + report format + 「変更ありますか？」質問済み |
| 6 | ユーザー最終承認 | ユーザーが変更なしを確認し、「次へ進む」を選択 |

**⚠️ 3〜5 が未完了の状態で 6 を聞いてはいけません。** フローは必ず: 図 → テーブル → 変更確認 → 確定 → 次ステップ。

---

## Phase 2 Handoff: Bicep Generation Agent

ユーザーが続行に同意したら、`references/bicep-generator.md` の手順を読み、Bicep テンプレートを生成します。  
または別 sub-agent に委譲しても構いません。

**機密情報取り扱い原則（絶対遵守）:**
- VM password、API key などの機密値をチャットで要求してはならず、parameter files に保存してもならない
- コードレビュー時に `main.bicepparam` に平文機密値が見つかった場合は、即時削除

**🔹 VM Password などユーザー入力機密値 — 複雑性検証必須:**

ユーザーが VM admin password などを入力した場合、Azure 送信前に複雑性要件を検証してください。  
Azure VM は次の条件を**すべて**満たす必要があります:
- 12 文字以上
- 大文字・小文字・数字・特殊文字のうち少なくとも 3 種類を含む

**検証失敗時:** デプロイを試行してはいけません。直ちに再入力を求める:
> **⚠️ The password does not meet Azure complexity requirements.** It must be 12 characters or more and contain at least 3 of: uppercase + lowercase + numbers + special characters.

**絶対にしてはいけないこと:**
- 「要件未満かもしれない」と警告しつつデプロイ試行 — **必ずブロック**
- 複雑性検証なしで Azure に送信し、デプロイ失敗を起こす

**🚨 `@secure()` Parameter と `.bicepparam` 互換性原則:**

`.bicepparam` に `using './main.bicep'` がある場合、`az deployment group what-if/create` で追加 `--parameters` フラグは併用できません。  
そのため、`@secure()` parameter の扱いは次ルールに従います:

1. **`@secure()` parameters はデフォルト値を必ず持つこと** — `newGuid()`、`uniqueString()` などの Bicep 関数を使用
   ```bicep
   @secure()
   param sqlAdminPassword string = newGuid()  // Auto-generated at deployment, store in Key Vault if needed
   ```
2. **`@secure()` parameters にユーザー指定値が必要な場合:**
   - `.bicepparam` は使わず、`--template-file` + `--parameters` の組み合わせを使用
   - あるいは別 JSON パラメータファイル（`main.parameters.json`）を生成
   ```powershell
   # When .bicepparam cannot be used — substitute with JSON parameter file
   az deployment group what-if `
     --template-file main.bicep `
     --parameters main.parameters.json `
     --parameters sqlAdminPassword='user-input-value'
   ```
3. **デプロイコマンドで `.bicepparam` と `--parameters` を同時併用しないこと**
   ```
   ❌ az deployment group create --parameters main.bicepparam --parameters key=value
   ✅ az deployment group create --parameters main.bicepparam
   ✅ az deployment group create --template-file main.bicep --parameters main.parameters.json --parameters key=value
   ```

**判断基準:**
- すべての `@secure()` parameters に default 値あり（newGuid など）→ `.bicepparam` を利用可能
- いずれかの `@secure()` parameter がユーザー入力必須 → `.bicepparam` ではなく JSON parameter file を使用

**MS Docs fetch 失敗時:**
- rate limiting などで web_fetch が失敗した場合、必ずユーザーに通知:
  ```
  ⚠️ MS Docs API version lookup failed. Generating with the last known stable version.
  Verifying the actual latest version before deployment is recommended.
  Shall we continue?
  ```
- ユーザー承認なしに、ハードコード版で黙って続行してはいけない

**Bicep 生成前の参照ファイル:**
- `references/service-gotchas.md` — 必須プロパティ、よくあるミス、PE groupId/DNS Zone マッピング
- `references/ai-data.md` — AI/Data サービス構成ガイド（v1 domain）
- `references/azure-common-patterns.md` — PE/security/naming 共通パターン
- `references/azure-dynamic-sources.md` — MS Docs URL レジストリ（API version 取得用）
- 上記未対応サービスは、MS Docs を直接取得して resource types、properties、PE mappings を確認

**出力構造:**
```
<project-name>/
├── main.bicep              # Main orchestration
├── main.bicepparam         # Parameters (environment-specific values)
└── modules/
    ├── network.bicep       # VNet, Subnet (including private endpoint subnet)
    ├── ai.bicep            # AI services (configured per user requirements)
    ├── storage.bicep       # ADLS Gen2 (isHnsEnabled: true)
    ├── fabric.bicep        # Microsoft Fabric (if needed)
    ├── keyvault.bicep      # Key Vault
    └── private-endpoints.bicep  # All PEs + DNS Zones
```

**Bicep 必須原則:**
- すべてのリソース名をパラメータ化 — `param openAiName string = 'oai-${uniqueString(resourceGroup().id)}'`
- private サービスは必ず `publicNetworkAccess: 'Disabled'`
- pe-subnet に `privateEndpointNetworkPolicies: 'Disabled'` を設定
- Private DNS Zone + VNet Link + DNS Zone Group — 3 点すべて必須
- Microsoft Foundry 利用時は **Foundry Project（`accounts/projects`）を必ず同時作成** — ないとポータル利用不可
- ADLS Gen2 は必ず `isHnsEnabled: true`（省略すると通常 Blob Storage が作成される）
- secrets は Key Vault に保存し、`@secure()` parameters で参照
- 各セクションの目的を英語コメントで明記

生成完了後は直ちに Phase 3 へ移行します。

---

## Phase 3 Handoff: Bicep Review Agent

`references/bicep-reviewer.md` の手順に従ってレビューを実施します。

**⚠️ 重要: 見た目確認だけで「pass」としてはいけません。`az bicep build` を実行し、実際のコンパイル結果を必ず確認してください。**

```powershell
az bicep build --file main.bicep 2>&1
```

1. コンパイルエラー/警告 → 修正
2. チェックリストレビュー → 修正
3. 再コンパイルで確認
4. 結果報告（コンパイル結果を含む）

詳細なチェックリストと修正手順は `references/bicep-reviewer.md` を参照。

レビュー完了後は Phase 4 へ移る前に結果をユーザーへ提示し、**次ステップを必ず案内**してください。

**🚨 Phase 3 完了時の必須報告フォーマット:**

```
## Bicep Code Review Complete

[Review result summary — bicep-reviewer.md Step 6 format]

---

**Next Step: Phase 4 (Azure Deployment)**

The review is complete. The following steps will proceed:
1. **What-if Validation** — Preview planned resources without making actual changes
2. **Preview Diagram** — Architecture visualization based on What-if results (02_arch_diagram_preview.html)
3. **Actual Deployment** — Create resources in Azure after user confirmation

Shall we proceed with deployment? (If you'd like just the code without deployment, let me know)
```

**絶対にしてはいけないこと:**
- Phase 3 完了時に、案内なしで `az deployment group create` コマンドだけ提示する
- What-if 検証なしで直接デプロイする、またはユーザーにコマンド実行だけ任せる
- Phase 4 の手順（What-if → Preview Diagram → Deployment）を省略する

