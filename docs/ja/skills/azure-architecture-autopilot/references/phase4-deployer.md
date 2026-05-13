# フェーズ 4: デプロイエージェント

このファイルにはフェーズ 4 の詳細な手順が記載されています。フェーズ 3（コードレビュー）完了後、ユーザーがデプロイを承認した際はこのファイルを読み、従ってください。

---

**🚨🚨🚨 フェーズ 4 必須実行順序 — どの手順も絶対にスキップしないこと 🚨🚨🚨**

以下の 5 ステップは**厳密に順番どおり**に実行する必要があります。いかなるステップも省略・スキップしてはいけません。  
ユーザーが「deploy it」「go ahead」「do it」などでデプロイを依頼した場合でも、必ずステップ 1 から順番に進めてください。

```
Step 1: Verify prerequisites (az login, subscription, resource group)
    ↓
Step 2: What-if validation (az deployment group what-if) ← Must execute
    ↓
Step 3: Generate preview diagram (02_arch_diagram_preview.html) ← Must generate
    ↓
Step 4: Actual deployment after user final confirmation (az deployment group create)
    ↓
Step 5: Generate deployment result diagram (03_arch_diagram_result.html)
```

**以下は絶対に行わないこと:**
- What-if を行わずに `az deployment group create` を直接実行する
- プレビューダイアグラム（`02_arch_diagram_preview.html`）の生成をスキップする
- What-if 結果をユーザーに提示せずにデプロイを進める
- ユーザーに手動実行させるため `az` コマンドだけを提示する

---

### ステップ 1: 前提条件の確認

```powershell
# az CLI のインストールとログイン状態を確認
az account show 2>&1
```

ログインしていない場合は、ユーザーに `az login` の実行を依頼してください。  
エージェントが資格情報を直接入力・保存してはいけません。

リソースグループを作成:
```powershell
az group create --name "<RG_NAME>" --location "<LOCATION>"  # Location confirmed in Phase 1
```
→ 成功を確認したら次のステップへ進む

### ステップ 2: 検証 → What-if 検証 — 🚨 必須

**このステップはスキップしないこと。ユーザーがどれだけ急いでデプロイを求めても、必ず実行してください。**

**ステップ 2-A: まず Validate を実行（高速な事前検証）**

Azure ポリシー違反やリソース参照エラーなどがあると、`what-if` は**エラーメッセージなしで無期限にハングする**ことがあります。  
これを防ぐため、**必ず先に `validate` を実行**してください。Validate はエラーを素早く返します。

```powershell
# validate — ポリシー違反、スキーマエラー、パラメーター問題を素早く検出
az deployment group validate `
  --resource-group "<RG_NAME>" `
  --parameters main.bicepparam
```

- **Validate 成功** → ステップ 2-B（what-if）へ進む
- **Validate 失敗** → エラーメッセージを分析し、Bicep を修正、再コンパイル、再検証
  - Azure Policy 違反（`RequestDisallowedByPolicy`）→ Bicep にポリシー要件を反映（例: `azureADOnlyAuthentication: true`）
  - スキーマエラー → API バージョン/プロパティを修正
  - パラメーターエラー → パラメーターファイルを修正

**ステップ 2-B: What-if を実行**

Validate 通過後に what-if を実行します。

**パラメーター渡し方式の選択:**
- すべての `@secure()` パラメーターにデフォルト値がある場合 → `.bicepparam` を使用
- `@secure()` パラメーターでユーザー入力が必要な場合 → `--template-file` + JSON パラメーターファイルを使用

```powershell
# Method 1: Use .bicepparam (when all @secure() parameters have defaults)
az deployment group what-if `
  --resource-group "<RG_NAME>" `
  --parameters main.bicepparam

# Method 2: Use JSON parameter file (when @secure() parameters require user input)
az deployment group what-if `
  --resource-group "<RG_NAME>" `
  --template-file main.bicep `
  --parameters main.parameters.json `
  --parameters secureParam='value'
```
→ What-if 結果を要約してユーザーに提示する。

**⏱️ What-if 実行方法とタイムアウト対応:**

What-if は Azure サーバー側でリソース検証を行うため、サービスやリージョンによって時間がかかる場合があります。  
**必ず `initial_wait: 300`（5 分）で実行してください。** 5 分以内に完了しない場合は自動的にタイムアウトします。

```powershell
# powershell ツール呼び出し時は常に initial_wait: 300 を設定
# mode: "sync", initial_wait: 300
az deployment group what-if `
  --resource-group "<RG_NAME>" `
  --parameters main.bicepparam
```

**5 分以内に完了** → 通常どおり進行（結果要約 → プレビューダイアグラム → デプロイ確認）

**5 分以内に完了しない（タイムアウト）** → `stop_powershell` で即時停止し、次の選択肢をユーザーに提示:

```
ask_user({
  question: "What-if validation did not complete within 5 minutes. The Azure server response is delayed. How would you like to proceed?",
  choices: [
    "Retry (Recommended)",
    "Skip What-if and deploy directly"
  ]
})
```

**「Retry」が選択された場合:** 同じコマンドを `initial_wait: 300` で再実行。再試行は最大 2 回まで。  
**「Skip What-if and deploy directly」が選択された場合:**
- Phase 1 のドラフトを基にプレビューダイアグラムを生成
- ユーザーにリスクを通知:
  > **⚠️ What-if 検証なしでデプロイします。** 予期しないリソース変更が発生する可能性があります。デプロイ後に Azure Portal で確認してください。

**以下は絶対に行わないこと:**
- `initial_wait` を設定せずに実行し、無期限待機になること
- エージェントが独断で「what-if は任意」と判断してスキップすること
- タイムアウト時にユーザーへ確認せず自動でデプロイへ切り替えること
- 「デプロイのほうが速い」などの理由で what-if を省略すること

### ステップ 3: What-if 結果に基づくプレビューダイアグラム — 🚨 必須

**このステップはスキップしないこと。What-if が成功したら必ずプレビューダイアグラムを生成してください。**

What-if 結果に含まれる実際にデプロイされるリソース（リソース名、種類、ロケーション、件数）を使ってダイアグラムを再生成します。  
Phase 1 のドラフト（`01_arch_diagram_draft.html`）はそのまま保持し、プレビューを `02_arch_diagram_preview.html` として生成します。  
ドラフトはいつでも再表示できます。

```
## デプロイ予定アーキテクチャ（What-if ベース）

[Interactive diagram link — 02_arch_diagram_preview.html]
(Design draft: 01_arch_diagram_draft.html)

Resources to be created (N items):
[What-if results summary table]

Deploy these resources? (Yes/No)
```

ユーザーが確認したらステップ 4 に進みます。**プレビューダイアグラムなしでデプロイに進んではいけません。**

### ステップ 4: 実デプロイ

ユーザーがプレビューダイアグラムと What-if 結果を確認し、デプロイを承認した場合のみ実行します。  
**What-if で使用したのと同じパラメーター渡し方式を使用してください。**

```powershell
$deployName = "deploy-$(Get-Date -Format 'yyyyMMdd-HHmmss')"

# Method 1: Use .bicepparam
az deployment group create `
  --resource-group "<RG_NAME>" `
  --parameters main.bicepparam `
  --name $deployName `
  2>&1 | Tee-Object -FilePath deployment.log

# Method 2: Use JSON parameter file
az deployment group create `
  --resource-group "<RG_NAME>" `
  --template-file main.bicep `
  --parameters main.parameters.json `
  --name $deployName `
  2>&1 | Tee-Object -FilePath deployment.log
```

デプロイ中は定期的に進捗を監視:
```powershell
az deployment group show `
  --resource-group "<RG_NAME>" `
  --name "<DEPLOYMENT_NAME>" `
  --query "{status:properties.provisioningState, duration:properties.duration}" `
  -o table
```

### デプロイ失敗時の対応

デプロイが失敗すると、一部リソースが `Failed` 状態で残る場合があります。この状態で再デプロイすると `AccountIsNotSucceeded` などのエラーが発生します。

**⚠️ リソース削除は破壊的コマンドです。必ず状況をユーザーに説明し、承認を得てから実行してください。**

```
[Resource name] failed during deployment.
To redeploy, the failed resources must be deleted first.

Delete and redeploy? (Yes/No)
```

ユーザー承認後、失敗リソースを削除して再デプロイします。

**🔹 ソフト削除されたリソースの対応（再デプロイ阻害の防止）:**

失敗デプロイ後にリソースグループを削除しても、Cognitive Services（Foundry）や Key Vault などは**ソフト削除状態**で残ります。  
同名で再デプロイすると `FlagMustBeSetForRestore`、`Conflict` エラーが発生します。

**再デプロイ前に必ず確認:**
```powershell
# ソフト削除された Cognitive Services を確認
az cognitiveservices account list-deleted -o table

# ソフト削除された Key Vault を確認
az keyvault list-deleted -o table
```

**解決オプション（ユーザーに選択肢を提示）:**
```
ask_user({
  question: "Soft-deleted resources from a previous deployment were found. How would you like to handle this?",
  choices: [
    "Purge and redeploy (Recommended) - Clean delete then create new",
    "Redeploy in restore mode - Recover existing resources"
  ]
})
```

**注意 — `enablePurgeProtection: true` の Key Vault:**
- Purge できない（保持期間終了まで待つ必要がある）
- 同名で再作成できない
- **対処法: Key Vault 名を変更**して再デプロイ（例: `uniqueString()` シードにタイムスタンプを追加）
- 状況をユーザーに説明し、名前変更を案内する

### ステップ 5: デプロイ完了 — 実リソースからダイアグラム生成と報告

デプロイ完了後、実際にデプロイされたリソースを照会し、最終アーキテクチャ図を生成します。

**ステップ 1: デプロイ済みリソースの照会**
```powershell
az resource list --resource-group "<RG_NAME>" --output json
```

**ステップ 2: 実リソースからダイアグラムを生成**

照会結果からリソース名、種類、SKU、エンドポイントを抽出し、組み込みダイアグラムエンジンで最終図を生成します。  
既存図を上書きしないようファイル名に注意:
- `01_arch_diagram_draft.html` — 設計ドラフト（保持）
- `02_arch_diagram_preview.html` — What-if プレビュー（保持）
- `03_arch_diagram_result.html` — デプロイ結果の最終版

ダイアグラムの services JSON に、実際にデプロイされたリソース情報を反映:
- `name`: 実際のリソース名（例: `foundry-duru57kxgqzxs`）
- `sku`: 実際の SKU
- `details`: エンドポイント、ロケーションなどの実値

**ステップ 3: 報告**
```
## Deployment Complete!

[Interactive architecture diagram — 03_arch_diagram_result.html]
(Design draft: 01_arch_diagram_draft.html | What-if preview: 02_arch_diagram_preview.html)

Created resources (N items):
[Dynamically extracted resource names, types, and endpoints from actual deployment results]

## Next Steps
1. Verify resources in Azure Portal
2. Check Private Endpoint connection status
3. Additional configuration guidance if needed

## Cleanup Command (If Needed)
az group delete --name <RG_NAME> --yes --no-wait
```

---

### デプロイ後のアーキテクチャ変更要求への対応

**デプロイ完了後にユーザーからリソースの追加・変更・削除要求があった場合、Bicep/デプロイに直接進んではいけません。**  
必ず Phase 1 に戻り、まずアーキテクチャを更新してください。

**プロセス:**

1. **ユーザー意図を確認** — まず、現在デプロイ済みのアーキテクチャに追加したいか確認:
   ```
   Would you like to add a VM to the currently deployed architecture?
   Current configuration: [Deployed services summary]
   ```

2. **Phase 1 に戻る — Delta Confirmation Rule を適用**
   - 既存デプロイ結果（`03_arch_diagram_result.html`）を現行状態のベースラインとして使用
   - 新規サービスの必須項目（SKU、ネットワーク、リージョン可用性など）を確認
   - 未確定事項は ask_user で確認
   - ファクトチェック（MS Docs fetch + クロスバリデーション）

3. **更新後アーキテクチャ図を生成**
   - 既存デプロイ済みリソース + 新規リソースを統合し、`04_arch_diagram_update_draft.html` を生成
   - ユーザーに提示して確認を取得:
   ```
   ## Updated Architecture

   [Interactive diagram — 04_arch_diagram_update_draft.html]
   (Previous deployment result: 03_arch_diagram_result.html)

   **Changes:**
   - Added: [New services list]
   - Removed: [Removed services list] (if any)

   Proceed with this configuration?
   ```

4. **確認後、Phase 2 → 3 → 4 を順に実施**
   - 既存 Bicep に新規リソースモジュールを段階的に追加
   - レビュー → What-if → デプロイ（増分デプロイ）

**以下は絶対に行わないこと:**
- デプロイ後の変更要求時に、アーキテクチャ図を更新せず Bicep 生成へ直接進むこと
- 既存デプロイ状態を無視して、新規リソースを単独で作成すること
- 既存アーキテクチャに追加するかをユーザー確認せずに進めること

