# エラーハンドリングガイド

このリファレンスでは、事前検証（preflight validation）中によく発生するエラーと、その対処方法を説明します。

## 基本原則

**失敗しても継続する。** 最初のエラーで停止するのではなく、最終レポートにすべての問題を記録してください。これにより、ユーザーは修正すべき点を全体像として把握できます。

---

## 認証エラー

### ログインしていない（Azure CLI）

**検出:**
```
ERROR: Please run 'az login' to setup account.
ERROR: AADSTS700082: The refresh token has expired
```

**終了コード:** 0以外

**対応:**
1. レポートにエラーを記録する
2. 修復手順を含める
3. 残りの Azure CLI コマンドをスキップする
4. 可能であれば他の検証ステップを継続する

**レポート記載例:**
```markdown
#### ❌ Azure CLI Authentication Required

- **Severity:** Error
- **Source:** az cli
- **Message:** Azure CLI にログインしていません
- **Remediation:** `az login` を実行して認証し、preflight validation を再実行してください
- **Documentation:** https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli
```

### ログインしていない（azd）

**検出:**
```
ERROR: not logged in, run `azd auth login` to login
```

**対応:**
1. レポートにエラーを記録する
2. azd コマンドをスキップする
3. `azd auth login` を案内する

**レポート記載例:**
```markdown
#### ❌ Azure Developer CLI Authentication Required

- **Severity:** Error
- **Source:** azd
- **Message:** Azure Developer CLI にログインしていません
- **Remediation:** `azd auth login` を実行して認証し、preflight validation を再実行してください
```

### トークン期限切れ

**検出:**
```
AADSTS700024: Client assertion is not within its valid time range
AADSTS50173: The provided grant has expired
```

**対応:**
1. エラーを記録する
2. 再認証を案内する
3. Azure 操作をスキップする

---

## 権限エラー

### RBAC 権限不足

**検出:**
```
AuthorizationFailed: The client '...' with object id '...' does not have authorization 
to perform action '...' over scope '...'
```

**対応:**
1. **最初の試行:** `--validation-level ProviderNoRbac` で再試行する
2. レポートに権限制約を記録する
3. ProviderNoRbac でも失敗する場合は、不足している具体的な権限を報告する

**レポート記載例:**
```markdown
#### ⚠️ Limited Permission Validation

- **Severity:** Warning
- **Source:** what-if
- **Message:** 完全な RBAC 検証に失敗したため、読み取り専用検証を使用しました
- **Detail:** 不足している権限: スコープ `/subscriptions/xxx` に対する `Microsoft.Resources/deployments/write`
- **Recommendation:** 対象リソースグループへの Contributor ロールを申請するか、管理者にデプロイ権限を確認してください
```

### リソースグループが見つからない

**検出:**
```
ResourceGroupNotFound: Resource group 'xxx' could not be found.
```

**対応:**
1. レポートに記録する
2. リソースグループの作成を案内する
3. このスコープの what-if をスキップする

**レポート記載例:**
```markdown
#### ❌ Resource Group Does Not Exist

- **Severity:** Error
- **Source:** what-if
- **Message:** リソースグループ 'my-rg' が存在しません
- **Remediation:** デプロイ前にリソースグループを作成してください:
  ```bash
  az group create --name my-rg --location eastus
  ```
```

### サブスクリプションへのアクセス拒否

**検出:**
```
SubscriptionNotFound: The subscription 'xxx' could not be found.
InvalidSubscriptionId: Subscription '...' is not valid
```

**対応:**
1. レポートに記録する
2. サブスクリプション ID の確認を案内する
3. 利用可能なサブスクリプション一覧を表示する

---

## Bicep 構文エラー

### コンパイルエラー

**検出:**
```
/path/main.bicep(22,51) : Error BCP064: Found unexpected tokens
/path/main.bicep(10,5) : Error BCP018: Expected the "=" character at this location
```

**対応:**
1. エラー出力から行番号・列番号を解析する
2. レポートにはすべてのエラーを含める（最初の1件で止めない）
3. what-if を継続する（追加の文脈が得られる場合がある）

**レポート記載例:**
```markdown
#### ❌ Bicep Syntax Error

- **Severity:** Error
- **Source:** bicep build
- **Location:** `main.bicep:22:51`
- **Code:** BCP064
- **Message:** 補間式内で予期しないトークンが見つかりました
- **Remediation:** 22行目の文字列補間構文を確認してください
- **Documentation:** https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/diagnostics/bcp064
```

### モジュールが見つからない

**検出:**
```
Error BCP091: An error occurred reading file. Could not find file '...'
Error BCP190: The module is not valid
```

**対応:**
1. 不足しているモジュールを記録する
2. `bicep restore` が必要か確認する
3. モジュールパスを検証する

### パラメータファイルの問題

**検出:**
```
Error BCP032: The value must be a compile-time constant
Error BCP035: The specified object is missing required properties
```

**対応:**
1. パラメータの問題を記録する
2. 問題のあるパラメータを明示する
3. 修正案を提示する

---

## ツール未インストール

### Azure CLI が見つからない

**検出:**
```
'az' is not recognized as an internal or external command
az: command not found
```

**対応:**
1. レポートに記録する
2. インストール手順を提示する。
  - 利用可能な場合は Azure MCP の `extension_cli_install` ツールを使ってインストール手順を取得する。
  - それ以外の場合は https://learn.microsoft.com/en-us/cli/azure/install-azure-cli の手順を参照する。
3. az コマンドをスキップする

**レポート記載例:**
```markdown
#### ⏭️ Azure CLI Not Installed

- **Severity:** Warning
- **Source:** environment
- **Message:** Azure CLI (az) がインストールされていないか、PATH にありません
- **Remediation:** Azure CLI をインストールしてください <ADD INSTALLATION INSTRUCTIONS HERE>
- **Impact:** az コマンドを使う what-if 検証はスキップされました
```

### Bicep CLI が見つからない

**検出:**
```
'bicep' is not recognized as an internal or external command
bicep: command not found
```

**対応:**
1. レポートに記録する
2. Azure CLI に Bicep が内蔵されている可能性があるため、`az bicep build` を試す
3. インストールリンクを提示する

**レポート記載例:**
```markdown
#### ⏭️ Bicep CLI Not Installed

- **Severity:** Warning
- **Source:** environment
- **Message:** Bicep CLI がインストールされていません
- **Remediation:** Bicep CLI をインストールしてください: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install
- **Impact:** 構文検証はスキップされました。what-if 実行時に Azure 側で検証されます
```

### Azure Developer CLI が見つからない

**検出:**
```
'azd' is not recognized as an internal or external command
azd: command not found
```

**対応:**
1. `azure.yaml` が存在する場合、これは必須
2. 可能であれば az CLI コマンドにフォールバックする
3. レポートに記録する

---

## What-If 固有のエラー

### ネストされたテンプレート上限

**検出:**
```
The deployment exceeded the nested template limit of 500
```

**対応:**
1. 警告として記録する（エラーではない）
2. 影響を受けるリソースは "Ignore" と表示されることを説明する
3. 手動レビューを案内する

### Template Link 未対応

**検出:**
```
templateLink references in nested deployments won't be visible in what-if
```

**対応:**
1. 警告として記録する
2. 制限事項を説明する
3. 実際のデプロイ時にリソースが検証されることを明記する

### 未評価の式

**検出:** 値ではなく `[utcNow()]` のような関数名がプロパティに表示される

**対応:**
1. 情報として記録する
2. これらはデプロイ時に評価されることを説明する
3. エラーではない

---

## ネットワークエラー

### タイムアウト

**検出:**
```
Connection timed out
Request timed out
```

**対応:**
1. 再試行を案内する
2. ネットワーク接続を確認する
3. Azure サービス側の問題の可能性もある

### SSL/TLS エラー

**検出:**
```
SSL: CERTIFICATE_VERIFY_FAILED
unable to get local issuer certificate
```

**対応:**
1. レポートに記録する
2. プロキシまたは企業ファイアウォールが原因の可能性がある
3. SSL 設定の確認を案内する

---

## フォールバック戦略

主要な検証が失敗した場合、以下の順でフォールバックを試行します。

```
Provider (完全 RBAC 検証)
    ↓ 権限エラーで失敗
ProviderNoRbac (書き込み権限チェックなしの検証)
    ↓ 失敗
Template (静的構文のみ)
    ↓ 失敗
すべての失敗を報告し、what-if 分析をスキップ
```

**すべての検証ステップが失敗した場合でも、必ずレポートを生成してください。**

---

## エラーレポートの集約

複数のエラーが発生した場合は、論理的に集約します。

1. **発生元でグループ化**（bicep, what-if, permissions）
2. **重大度順に並べる**（warning より error を先）
3. 類似エラーを**重複排除**する
4. 先頭に**件数サマリー**を示す

例:
```markdown
## Issues

**3件のエラー** と **2件の警告** が見つかりました

### Errors (3)

1. [Bicep Syntax Error - main.bicep:22:51](#error-1)
2. [Bicep Syntax Error - main.bicep:45:10](#error-2)
3. [Resource Group Not Found](#error-3)

### Warnings (2)

1. [Limited Permission Validation](#warning-1)
2. [Nested Template Limit Reached](#warning-2)
```

---

## 終了コードリファレンス

| Tool | Exit Code | Meaning |
|------|-----------|---------|
| az | 0 | 成功 |
| az | 1 | 一般エラー |
| az | 2 | コマンドが見つからない |
| az | 3 | 必須引数不足 |
| azd | 0 | 成功 |
| azd | 1 | エラー |
| bicep | 0 | ビルド成功 |
| bicep | 1 | ビルド失敗（エラーあり） |
| bicep | 2 | 警告ありでビルド成功 |

