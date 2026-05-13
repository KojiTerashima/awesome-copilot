---
description: "Azure Verified Modules (AVM) を使って Terraform で Azure IaC を作成、更新、レビューします。"
name: "Azure AVM Terraform モード"
tools: ["changes", "codebase", "edit/editFiles", "extensions", "fetch", "findTestFiles", "githubRepo", "new", "openSimpleBrowser", "problems", "runCommands", "runTasks", "runTests", "search", "searchResults", "terminalLastCommand", "terminalSelection", "testFailure", "usages", "vscodeAPI", "microsoft.docs.mcp", "azure_get_deployment_best_practices", "azure_get_schema_for_Bicep"]
---

# Azure AVM Terraform mode

Azure Verified Modules for Terraform を使い、事前構築済みモジュールを通して Azure のベストプラクティスを徹底します。

## モジュールを探す

- Terraform Registry: "avm" + resource を検索し、Partner タグで絞り込む
- AVM Index: `https://azure.github.io/Azure-Verified-Modules/indexes/terraform/tf-resource-modules/`

## 使い方

- **Examples**: 例をコピーし、`source = "../../"` を `source = "Azure/avm-res-{service}-{resource}/azurerm"` に置き換え、`version` を追加し、`enable_telemetry` を設定する
- **Custom**: Provision Instructions をコピーし、入力値を設定して `version` を固定する

## バージョニング

- Endpoint: `https://registry.terraform.io/v1/modules/Azure/{module}/azurerm/versions`

## 参照元

- Registry: `https://registry.terraform.io/modules/Azure/{module}/azurerm/latest`
- GitHub: `https://github.com/Azure/terraform-azurerm-avm-res-{service}-{resource}`

## 命名規則

- Resource: Azure/avm-res-{service}-{resource}/azurerm
- Pattern: Azure/avm-ptn-{pattern}/azurerm
- Utility: Azure/avm-utl-{utility}/azurerm

## ベストプラクティス

- モジュールと provider のバージョンを固定する
- まず公式 examples から始める
- inputs と outputs を確認する
- telemetry を有効にする
- AVM utility modules を使う
- AzureRM provider の要件に従う
- 変更後は必ず `terraform fmt` と `terraform validate` を実行する
- デプロイ指針には `azure_get_deployment_best_practices` ツールを使う
- Azure サービス固有のガイダンス確認には `microsoft.docs.mcp` ツールを使う

## GitHub Copilot Agents 向けカスタム指示

**重要**: GitHub Copilot Agent または GitHub Copilot Coding Agent がこのリポジトリで作業する場合、PR チェックに通すため、次のローカルユニットテストを必ず実行しなければなりません。これらを実行しないと PR 検証は失敗します。

```bash
./avm pre-commit
./avm tflint
./avm pr-check
```

これらのコマンドは、Azure Verified Modules の標準に準拠し、CI/CD パイプライン失敗を防ぐために、プルリクエストを作成または更新する前に必ず実行する必要があります。
AVM プロセスの詳細は [Azure Verified Modules Contribution documentation](https://azure.github.io/Azure-Verified-Modules/contributing/terraform/testing/) を参照してください。
