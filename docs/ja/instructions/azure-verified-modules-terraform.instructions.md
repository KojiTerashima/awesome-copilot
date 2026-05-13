---
description: 'Azure 検証済みモジュール (AVM) と Terraform'
applyTo: '**/*.terraform, **/*.tf, **/*.tfvars, **/*.tfstate, **/*.tflint.hcl, **/*.tf.json, **/*.tfvars.json'
---

# Azure 検証済みモジュール (AVM) Terraform

## 概要

Azure Verified Modules (AVM) は、Azure のベストプラクティスに従って事前に構築、テスト、検証された Terraform および Bicep モジュールです。これらのモジュールを使用すると、自信を持って Azure Infrastructure as Code (IaC) を作成、更新、またはレビューできます。

## GitHub Copilot エージェントのカスタム手順

**重要**: GitHub Copilot Agent または GitHub Copilotcoding Agent がこのリポジトリで動作している場合、PR チェックに準拠するために次のローカル単体テストを実行する必要があります。これらのテストを実行しないと、PR 検証エラーが発生します。

```bash
./avm pre-commit
./avm tflint
./avm pr-check
```

これらのコマンドは、Azure Verified Modules 標準への準拠を確保し、CI/CD パイプラインの障害を防ぐために、プルリクエストが作成または更新される前に実行する必要があります。
AVM プロセスの詳細については、[Azure 検証済みモジュールのコントリビューションドキュメント](https://azure.github.io/Azure-Verified-Modules/contributing/terraform/testing/) を参照してください。

**これらのテストを実行しないと、PR 検証が失敗し、マージが成功しなくなります。**

## モジュールの検出

### Terraform レジストリ

- 「avm」+リソース名を検索します
- 「パートナー」タグでフィルタリングして公式 AVM モジュールを見つけます
- 例: 「avm storage account」を検索 → パートナーでフィルター

### 公式AVMインデックス

> **注意:** 次のリンクは常に、メインブランチ上の CSV ファイルの最新バージョンを指します。意図したとおり、これはファイルが時間の経過とともに変更される可能性があることを意味します。ポイントインタイムバージョンが必要な場合は、URL で特定のリリースタグを使用することを検討してください。

- **Terraform リソースモジュール**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformResourceModules.csv`
- **Terraform パターンモジュール**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformPatternModules.csv`
- **Terraform ユーティリティモジュール**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformUtilityModules.csv`

## Terraform モジュールの使用法

### 例から

1. モジュールのドキュメントからサンプルコードをコピーします。
2. `source = "../../"` を `source = "Azure/avm-res-{service}-{resource}/azurerm"` に置き換えます
3. `version = "~> 1.0"` を追加 (利用可能な最新のものを使用)
4. `enable_telemetry = true`を設定

### ゼロから

1. モジュールのドキュメントからプロビジョニング手順をコピーします。
2. 必須およびオプションの入力を構成する
3. モジュールのバージョンを固定する
4. テレメトリを有効にする

### 使用例

```hcl
module "storage_account" {
  source  = "Azure/avm-res-storage-storageaccount/azurerm"
  version = "~> 0.1"

  enable_telemetry    = true
  location            = "East US"
  name                = "mystorageaccount"
  resource_group_name = "my-rg"

  # Additional configuration...
}
```

## 命名規則

### モジュールの種類

- **リソースモジュール**: `Azure/avm-res-{service}-{resource}/azurerm`
  - 例: `Azure/avm-res-storage-storageaccount/azurerm`
- **パターンモジュール**: `Azure/avm-ptn-{pattern}/azurerm`
  - 例: `Azure/avm-ptn-aks-enterprise/azurerm`
- **ユーティリティモジュール**: `Azure/avm-utl-{utility}/azurerm`
  - 例: `Azure/avm-utl-regions/azurerm`

### サービスのネーミング

- サービスとリソースに kebab-case を使用する
- Azure サービス名に従います (例: `storage-storageaccount`、`network-virtualnetwork`)

## バージョン管理

### 利用可能なバージョンを確認する

- エンドポイント: `https://registry.terraform.io/v1/modules/Azure/{module}/azurerm/versions`
- 例: `https://registry.terraform.io/v1/modules/Azure/avm-res-storage-storageaccount/azurerm/versions`

### バージョン固定のベストプラクティス

- 悲観的なバージョン制約を使用します: `version = "~> 1.0"`
- 実稼働用に特定のバージョンにピン留めする: `version = "1.2.3"`
- アップグレードする前に必ず変更ログを確認してください

## モジュールソース

### Terraform レジストリ

- **URL パターン**: `https://registry.terraform.io/modules/Azure/{module}/azurerm/latest`
- **例**: `https://registry.terraform.io/modules/Azure/avm-res-storage-storageaccount/azurerm/latest`

### GitHub リポジトリ

- **URL パターン**: `https://github.com/Azure/terraform-azurerm-avm-{type}-{service}-{resource}`
- **例**:
  - リソース: `https://github.com/Azure/terraform-azurerm-avm-res-storage-storageaccount`
  - パターン: `https://github.com/Azure/terraform-azurerm-avm-ptn-aks-enterprise`

## 開発のベストプラクティス

### モジュールの使用法

- ✅ **常に** モジュールとプロバイダーのバージョンを固定する
- ✅ **モジュールドキュメントの公式サンプルから始めます**
- ✅ 導入前にすべての入力と出力を **レビュー**
- ✅ **テレメトリを有効にする**: `enable_telemetry = true`
- ✅ **一般的なパターンには AVM ユーティリティモジュールを使用します**
- ✅ **フォロー** AzureRM プロバイダーの要件と制約

### コードの品質

- ✅ **常に** 変更を加えた後は `terraform fmt` を実行してください
- ✅ **常に** 変更を加えた後は `terraform validate` を実行してください
- ✅ **使用** 意味のある変数名と説明を使用する
- ✅ **適切なタグとメタデータを追加**
- ✅ **文書化** 複雑な構成

### 検証要件

プルリクエストを作成または更新する前に、次のことを行ってください。

```bash
# Format code
terraform fmt -recursive

# Validate syntax
terraform validate

# AVM-specific validation (MANDATORY)
./avm pre-commit
./avm tflint
./avm pr-check
```

## ツールの統合

### 利用可能なツールを使用する

- **導入ガイダンス**: `azure_get_deployment_best_practices` ツールを使用する
- **サービスドキュメント**: Azure サービス固有のガイダンスには `microsoft.docs.mcp` ツールを使用してください
- **スキーマ情報**: Bicep リソースには `azure_get_schema_for_Bicep` を使用します

### GitHub コパイロットの統合

AVM リポジトリを使用する場合:

1. 新しいリソースを作成する前に、必ず既存のモジュールを確認してください
2. 公式の例を出発点として使用する
3. コミットする前にすべての検証テストを実行する
4. カスタマイズまたは例からの逸脱を文書化します。

## よくあるパターン

### リソースグループモジュール

```hcl
module "resource_group" {
  source  = "Azure/avm-res-resources-resourcegroup/azurerm"
  version = "~> 0.1"

  enable_telemetry = true
  location         = var.location
  name            = var.resource_group_name
}
```

### 仮想ネットワークモジュール

```hcl
module "virtual_network" {
  source  = "Azure/avm-res-network-virtualnetwork/azurerm"
  version = "~> 0.1"

  enable_telemetry    = true
  location            = module.resource_group.location
  name                = var.vnet_name
  resource_group_name = module.resource_group.name
  address_space       = ["10.0.0.0/16"]
}
```

## トラブルシューティング

### よくある問題

1. **バージョンの競合**: モジュールとプロバイダーのバージョン間の互換性を常に確認してください。
2. **依存関係が欠落しています**: 必要なリソースがすべて最初に作成されていることを確認してください
3. **検証の失敗**: コミットする前に AVM 検証ツールを実行します。
4. **ドキュメント**: 常に最新のモジュールのドキュメントを参照してください。

### サポートリソース

- **AVM ドキュメント**: `https://azure.github.io/Azure-Verified-Modules/`
- **GitHub の問題**: 特定のモジュールの GitHub リポジトリの問題を報告します。
- **コミュニティ**: Azure Terraform プロバイダー GitHub ディスカッション

## コンプライアンスチェックリスト

AVM 関連のコードを送信する前に、次のことを行ってください。

- [ ] モジュールのバージョンが固定されています
- [ ] テレメトリが有効になっています
- [ ] コードはフォーマットされています (`terraform fmt`)
- [ ] コードは検証されました (`terraform validate`)
- [ ] AVM コミット前チェックに合格 (`./avm pre-commit`)
- [ ] TFLint チェックが成功しました (`./avm tflint`)
- [ ] AVM PR チェックに合格 (`./avm pr-check`)
- [ ] ドキュメントが更新されました
- [ ] 例はテストされ、動作しています
