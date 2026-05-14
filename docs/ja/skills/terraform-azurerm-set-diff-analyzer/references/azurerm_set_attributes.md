# AzureRM セットタイプ属性のリファレンス

本書は`azurerm_set_attributes.json`の概要とメンテナンスについて説明します。

> **最終更新日**: 2026 年 1 月 28 日

## 概要

`azurerm_set_attributes.json` は、AzureRM Provider で Set 型として扱われる属性の定義ファイルです。
`analyze_plan.py` スクリプトは、この JSON を読み取り、Terraform プラン内の「誤検知の差分」を識別します。

### Set-Type 属性とは何ですか?

Terraform の Set タイプは、**順序を保証しない** コレクションです。
したがって、要素を追加または削除すると、変更されていない要素が「変更された」ように表示される場合があります。
これは「偽陽性差分」と呼ばれます。

## JSON ファイル構造

### 基本フォーマット```json
{
  "resources": {
    "azurerm_resource_type": {
      "attribute_name": "key_attribute"
    }
  }
}
```- **key_attribute**: Set 要素を一意に識別する属性 (例: `name`、`id`)
- **null**: key属性がない場合(要素全体を比較)

### ネストされた形式

Set 属性に別の Set 属性が含まれる場合:```json
{
  "rewrite_rule_set": {
    "_key": "name",
    "rewrite_rule": {
      "_key": "name",
      "condition": "variable",
      "request_header_configuration": "header_name"
    }
  }
}
```- **`_key`**: そのレベルの Set 要素のキー属性
- **その他のキー**: ネストされた Set 属性の定義

### 例: azurerm_application_gateway```json
"azurerm_application_gateway": {
  "backend_address_pool": "name",           // Simple Set (key is name)
  "rewrite_rule_set": {                     // Nested Set
    "_key": "name",
    "rewrite_rule": {
      "_key": "name",
      "condition": "variable"
    }
  }
}
```## メンテナンス

### 新しい属性の追加

1. **公式ドキュメントを確認してください**
   - [Terraform Registry](https://registry.terraform.io/providers/bashicorp/azurerm/latest/docs) でリソースを検索します。
   - 属性が「Set of ...」としてリストされていることを確認します。
   - `azurerm_application_gateway` のような一部のリソースには、明示的に記載された Set 属性があります

2. **ソースコードを確認する (より信頼性の高い)**
   - [AzureRM Provider GitHub](https://github.com/bashicorp/terraform-provider-azurerm)でリソースを検索
   - スキーマ定義の`Type: pluginsdk.TypeSet`を確認してください
   - `_key` として機能するセットの `Schema` 内の属性を特定します。

3. **JSON に追加**```json
   "azurerm_new_resource": {
     "set_attribute": "key_attribute"
   }
   ```4. **テスト**```bash
   # Verify with an actual plan
   python3 scripts/analyze_plan.py your_plan.json
   ```### 主要な属性の特定

|共通キー属性 |使い方 |
|---------------------|------|
| `name` |名前付きブロック (最も一般的) |
| `id` |リソース ID リファレンス |
| `location` |地理的位置 |
| `address` |ネットワークアドレス |
| `host_name` |ホスト名 |
| `null` |キーが存在しない場合（要素全体を比較） |

## 関連ツール

### 分析プラン.py

Terraform プランの JSON を分析して、誤検知の差分を特定します。```bash
# Basic usage
terraform show -json plan.tfplan | python3 scripts/analyze_plan.py

# Read from file
python3 scripts/analyze_plan.py plan.json

# Use custom attribute file
python3 scripts/analyze_plan.py plan.json --attributes /path/to/custom.json
```## サポートされているリソース

現在サポートされているリソースについては、`azurerm_set_attributes.json` を直接参照してください。```bash
# List resources
jq '.resources | keys' azurerm_set_attributes.json
```主要なリソース:
- `azurerm_application_gateway` - バックエンド プール、リスナー、ルールなど。
- `azurerm_firewall_policy_rule_collection_group` - ルールコレクション
- `azurerm_frontdoor` - バックエンド プール、ルーティング
- `azurerm_network_security_group` - セキュリティルール
- `azurerm_virtual_network_gateway` - IP設定、VPNクライアント設定

## 注意事項

- 属性の動作はプロバイダー/API バージョンによって異なる場合があります
- 新しいリソースと属性が利用可能になったら追加する必要がある
- 深く入れ子になった構造をすべてのレベルで定義すると精度が向上します