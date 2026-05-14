---
name: terraform-azurerm-set-diff-analyzer
description: Analyze Terraform plan JSON output for AzureRM Provider to distinguish between false-positive diffs (order-only changes in Set-type attributes) and actual resource changes. Use when reviewing terraform plan output for Azure resources like Application Gateway, Load Balancer, Firewall, Front Door, NSG, and other resources with Set-type attributes that cause spurious diffs due to internal ordering changes.
license: MIT
---
# Terraform AzureRM セット差分アナライザー

AzureRM プロバイダーの Set タイプ属性によって引き起こされる Terraform プラン内の "誤検知の差分" を特定し、実際の変更と区別するスキル。

## いつ使用するか

- `terraform plan` には多くの変更が示されていますが、追加または削除された要素は 1 つだけです
- Application Gateway、Load Balancer、NSG などで「すべての要素が変更されました」と表示される
- CI/CD で誤検知の差分を自動的にフィルタリングしたい

## 背景

Terraform の Set タイプはキーではなく位置によって比較するため、要素を追加または削除すると、すべての要素が「変更された」ように表示されます。これは Terraform の一般的な問題ですが、Application Gateway、Load Balancer、NSG などの Set タイプの属性を頻繁に使用する AzureRM リソースで特に顕著です。

これらの「誤検知の差分」は実際にはリソースに影響しませんが、Terraform プランの出力のレビューを困難にします。

## 前提条件

- Python 3.8以降

Python が利用できない場合は、パッケージ マネージャー (例: `apt install python3`、`brew install python3`) を介して、または [python.org](https://www.python.org/downloads/) からインストールします。

## 基本的な使い方```bash
# 1. Generate plan JSON output
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json

# 2. Analyze
python scripts/analyze_plan.py plan.json
```## トラブルシューティング

- **`python: command not found`**: 代わりに `python3` を使用するか、Python をインストールしてください
- **`ModuleNotFoundError`**: スクリプトは標準ライブラリのみを使用します。 Python 3.8以降を確認してください

## 詳細なドキュメント

- [scripts/README.md](scripts/README.md) - すべてのオプション、出力形式、終了コード、CI/CD の例
- [references/azurerm_set_attributes.md](references/azurerm_set_attributes.md) - サポートされているリソースと属性