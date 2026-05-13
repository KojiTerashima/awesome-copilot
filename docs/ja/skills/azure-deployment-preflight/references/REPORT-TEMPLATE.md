# 事前チェックレポート テンプレート

プロジェクトルートで `preflight-report.md` を生成する際は、このテンプレート構造を使用してください。

---

## テンプレート

```markdown
# Azure Deployment 事前チェックレポート

**生成日時:** {timestamp}
**ステータス:** {overall-status}

---

## 概要

| 項目 | 値 |
|----------|-------|
| **テンプレートファイル** | {bicep-files} |
| **パラメータファイル** | {param-files-or-none} |
| **プロジェクト種別** | {azd-project | standalone-bicep} |
| **デプロイ スコープ** | {resourceGroup | subscription | managementGroup | tenant} |
| **対象** | {resource-group-name | subscription-name | mg-id} |
| **検証レベル** | {Provider | ProviderNoRbac} |

### 検証結果

| チェック | ステータス | 詳細 |
|-------|--------|---------|
| Bicep 構文 | {✅ Pass | ❌ Fail | ⚠️ Warnings | ⏭️ Skipped} | {details} |
| What-If 分析 | {✅ Pass | ❌ Fail | ⏭️ Skipped} | {details} |
| 権限チェック | {✅ Pass | ⚠️ Limited | ❌ Fail} | {details} |

---

## 実行したツール

### 実行コマンド

| 手順 | コマンド | 終了コード | 所要時間 |
|------|---------|-----------|----------|
| 1 | `{command}` | {0 | non-zero} | {duration} |
| 2 | `{command}` | {0 | non-zero} | {duration} |

### ツール バージョン

| ツール | バージョン |
|------|---------|
| Azure CLI | {version} |
| Bicep CLI | {version} |
| Azure Developer CLI | {version-or-n/a} |

---

## 問題

{if-no-issues}
✅ **問題は見つかりませんでした。** デプロイを進める準備ができています。
{end-if}

{if-issues-exist}
### エラー

{for-each-error}
#### ❌ {error-title}

- **重大度:** Error
- **ソース:** {bicep-build | what-if | permissions}
- **場所:** {file-path}:{line}:{column} (該当する場合)
- **メッセージ:** {error-message}
- **対処方法:** {suggested-fix}
- **ドキュメント:** {link-if-available}

{end-for-each}

### 警告

{for-each-warning}
#### ⚠️ {warning-title}

- **重大度:** Warning
- **ソース:** {source}
- **メッセージ:** {warning-message}
- **推奨対応:** {suggested-action}

{end-for-each}
{end-if}

---

## What-If 結果

{if-what-if-succeeded}

### 変更サマリー

| 変更種別 | 件数 |
|-------------|-------|
| 🆕 作成 | {count} |
| 📝 変更 | {count} |
| 🗑️ 削除 | {count} |
| ✓ 変更なし | {count} |
| ⚠️ 無視 | {count} |

### 作成されるリソース

{if-resources-to-create}
| リソース種別 | リソース名 |
|---------------|---------------|
| {type} | {name} |
{end-if}

{if-no-resources-to-create}
*作成されるリソースはありません。*
{end-if}

### 変更されるリソース

{if-resources-to-modify}
#### {resource-type}/{resource-name}

| プロパティ | 現在の値 | 新しい値 |
|----------|---------------|-----------|
| {property-path} | {current} | {new} |

{end-if}

{if-no-resources-to-modify}
*変更されるリソースはありません。*
{end-if}

### 削除されるリソース

{if-resources-to-delete}
| リソース種別 | リソース名 |
|---------------|---------------|
| {type} | {name} |

> ⚠️ **警告:** 削除対象のリソースは完全に削除されます。
{end-if}

{if-no-resources-to-delete}
*削除されるリソースはありません。*
{end-if}

{end-if-what-if-succeeded}

{if-what-if-failed}
### What-If 分析に失敗しました

what-if 操作を完了できませんでした。詳細は「問題」セクションを参照してください。
{end-if}

---

## 推奨事項

{generate-based-on-findings}

1. {recommendation-1}
2. {recommendation-2}
3. {recommendation-3}

---

## 次のステップ

{if-all-passed}
事前チェックの検証に合格しました。デプロイを進められます。

**azd プロジェクトの場合:**
```bash
azd provision
# or
azd up
```

**スタンドアロン Bicep の場合:**
```bash
az deployment group create \
  --resource-group {rg-name} \
  --template-file {bicep-file} \
  --parameters {param-file}
```
{end-if}

{if-issues-exist}
デプロイ前に、上記の問題を解消してください。修正後は次を実施してください。

1. 事前チェック検証を再実行して修正を確認する
2. すべてのチェックに合格したらデプロイを進める
{end-if}

---

*Azure Deployment Preflight Skill により生成されたレポート*
```

---

## ステータス値

### 全体ステータス

| ステータス | 意味 | 表示 |
|--------|---------|--------|
| **Pass** | すべてのチェックが成功し、安全にデプロイ可能 | ✅ |
| **Pass with Warnings** | チェックは成功したが、警告の確認が必要 | ⚠️ |
| **Fail** | 1 つ以上のチェックが失敗 | ❌ |

### 個別チェックのステータス

| ステータス | 意味 |
|--------|---------|
| ✅ Pass | チェックが正常に完了 |
| ❌ Fail | チェックでエラーを検出 |
| ⚠️ Warnings | 警告付きでチェック通過 |
| ⏭️ Skipped | チェックをスキップ（ツール未使用可能など） |

---

## レポート例

```markdown
# Azure Deployment 事前チェックレポート

**生成日時:** 2026-01-16T14:32:00Z
**ステータス:** ⚠️ Pass with Warnings

---

## 概要

| 項目 | 値 |
|----------|-------|
| **テンプレートファイル** | `infra/main.bicep` |
| **パラメータファイル** | `infra/main.bicepparam` |
| **プロジェクト種別** | azd project |
| **デプロイ スコープ** | subscription |
| **対象** | my-subscription |
| **検証レベル** | Provider |

### 検証結果

| チェック | ステータス | 詳細 |
|-------|--------|---------|
| Bicep 構文 | ✅ Pass | エラーは見つかりませんでした |
| What-If 分析 | ⚠️ Warnings | ネストされたテンプレート制限により 1 件のリソースを無視 |
| 権限チェック | ✅ Pass | 完全なデプロイ権限を確認済み |

---

## 実行したツール

### 実行コマンド

| 手順 | コマンド | 終了コード | 所要時間 |
|------|---------|-----------|----------|
| 1 | `bicep build infra/main.bicep --stdout` | 0 | 1.2s |
| 2 | `azd provision --preview --environment dev` | 0 | 8.4s |

### ツール バージョン

| ツール | バージョン |
|------|---------|
| Azure CLI | 2.76.0 |
| Bicep CLI | 0.25.3 |
| Azure Developer CLI | 1.9.0 |

---

## 問題

### 警告

#### ⚠️ ネストされたテンプレート上限に到達

- **重大度:** Warning
- **ソース:** what-if
- **メッセージ:** ネストされたテンプレート展開の上限に達したため、1 件のリソースが無視されました
- **推奨対応:** デプロイ後に無視されたリソースを手動で確認してください

---

## What-If 結果

### 変更サマリー

| 変更種別 | 件数 |
|-------------|-------|
| 🆕 作成 | 3 |
| 📝 変更 | 1 |
| 🗑️ 削除 | 0 |
| ✓ 変更なし | 2 |
| ⚠️ 無視 | 1 |

### 作成されるリソース

| リソース種別 | リソース名 |
|---------------|---------------|
| Microsoft.Resources/resourceGroups | rg-myapp-dev |
| Microsoft.Storage/storageAccounts | stmyappdev |
| Microsoft.Web/sites | app-myapp-dev |

### 変更されるリソース

#### Microsoft.KeyVault/vaults/kv-myapp-dev

| プロパティ | 現在の値 | 新しい値 |
|----------|---------------|-----------|
| properties.sku.name | standard | premium |
| tags.environment | staging | dev |

### 削除されるリソース

*削除されるリソースはありません。*

---

## 推奨事項

1. ストレージ アカウント名 `stmyappdev` が命名要件を満たしていることを確認してください
2. Key Vault SKU の standard から premium へのアップグレードが意図したものか確認してください
3. 無視されたネストされたテンプレート リソースはデプロイ後に確認してください

---

## 次のステップ

事前チェックの検証は警告付きで合格しました。上記の警告を確認してから、次に進んでください。

```bash
azd provision --environment dev
```

---

*Azure Deployment Preflight Skill により生成されたレポート*
```

---

## 書式ガイドライン

1. 視認性を高めるために**絵文字を統一**する
2. Bicep エラーを参照する際は**行番号を含める**
3. 各問題に対して**実行可能な対処方法**を示す
4. 可能な場合は**ドキュメントへのリンク**を付ける
5. **重大度順に問題を並べる**（先にエラー、その後に警告）
6. 「次のステップ」に**コマンド例を含める**

