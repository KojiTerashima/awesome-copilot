# Terraform AzureRM セット差分アナライザー スクリプト

Terraform プラン JSON を分析し、AzureRM セット タイプ属性の "偽陽性の差分" を識別する Python スクリプト。

## 概要

AzureRM プロバイダーの Set タイプ属性 (`backend_address_pool`、`security_rule` など) は順序を保証しないため、要素を追加または削除すると、すべての要素が「変更された」ように表示されます。このスクリプトは、そのような「誤検知の差分」を実際の変更から区別します。

### 使用例

- **エージェント スキル**として (推奨)
- 手動実行用の **CLI ツール**として
- **CI/CD パイプライン**の自動分析用

## 前提条件

- Python 3.8以降
- 追加のパッケージは必要ありません (標準ライブラリのみを使用します)

## 使用法

### 基本的な使い方```bash
# Read from file
python analyze_plan.py plan.json

# Read from stdin
terraform show -json plan.tfplan | python analyze_plan.py
```### オプション

|オプション |短い |説明 |デフォルト |
|----------|----------|---------------|----------|
| `--format` | `-f` |出力形式 (markdown/json/summary) |値下げ |
| `--exit-code` | `-e` |変更に基づいて終了コードを返す |偽 |
| `--quiet` | `-q` |警告を抑制する |偽 |
| `--verbose` | `-v` |詳細な警告を表示 |偽 |
| `--ignore-case` | - |大文字と小文字を区別せずに値を比較します。偽 |
| `--attributes` | - |カスタム属性定義ファイルへのパス | (内蔵) |
| `--include` | - |分析するリソースをフィルタリングします (複数指定可能) | (すべて) |
| `--exclude` | - |除外するリソースをフィルターします (複数指定可能) | (なし) |

### 終了コード (`--exit-code` を使用)

|コード |意味 |
|-----|----------|
| 0 |変更なし、または注文のみの変更 |
| 1 |実際のセット属性の変更 |
| 2 |リソースの置き換え（削除+作成） |
| 3 |エラー |

## 出力形式

### マークダウン (デフォルト)

PR コメントおよびレポート用の人が判読可能な形式。```bash
python analyze_plan.py plan.json --format markdown
```### JSON

プログラムによる処理のための構造化データ。```bash
python analyze_plan.py plan.json --format json
```出力例:```json
{
  "summary": {
    "order_only_count": 3,
    "actual_set_changes_count": 1,
    "replace_count": 0
  },
  "has_real_changes": true,
  "resources": [...],
  "warnings": []
}
```### 概要

CI/CD ログの 1 行の概要。```bash
python analyze_plan.py plan.json --format summary
```出力例:```
🟢 3 order-only | 🟡 1 set changes
```## CI/CD パイプラインの使用法

### GitHub アクション```yaml
name: Terraform Plan Analysis

on:
  pull_request:
    paths:
      - '**.tf'

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        
      - name: Terraform Init & Plan
        run: |
          terraform init
          terraform plan -out=plan.tfplan
          terraform show -json plan.tfplan > plan.json
          
      - name: Analyze Set Diff
        run: |
          python path/to/analyze_plan.py plan.json --format markdown > analysis.md
          
      - name: Comment PR
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          path: analysis.md
```### GitHub アクション (終了コードを含むゲート)```yaml
      - name: Analyze and Gate
        run: |
          python path/to/analyze_plan.py plan.json --exit-code --format summary
        # Fail on exit code 2 (resource replacement)
        continue-on-error: false
```### Azure パイプライン```yaml
- task: TerraformCLI@0
  inputs:
    command: 'plan'
    commandOptions: '-out=plan.tfplan'

- script: |
    terraform show -json plan.tfplan > plan.json
    python scripts/analyze_plan.py plan.json --format markdown > $(Build.ArtifactStagingDirectory)/analysis.md
  displayName: 'Analyze Plan'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)/analysis.md'
    artifactName: 'plan-analysis'
```### フィルタリングの例

特定のリソースのみを分析します。```bash
python analyze_plan.py plan.json --include application_gateway --include load_balancer
```特定のリソースを除外します。```bash
python analyze_plan.py plan.json --exclude virtual_network
```## 結果の解釈

|カテゴリー |意味 |推奨されるアクション |
|----------|----------|----------|
| 🟢 注文のみ |偽陽性の差分、実際の変化なし |無視しても安全です |
| 🟡 実際の変化 |セット要素の追加/削除/変更 |コンテンツを確認します (通常はインプレース更新)。
| 🔴 リソースの置き換え |削除 + 作成 |ダウンタイムの影響を確認する |

## カスタム属性の定義

デフォルトでは `references/azurerm_set_attributes.json` が使用されますが、カスタム定義ファイルを指定することもできます。```bash
python analyze_plan.py plan.json --attributes /path/to/custom_attributes.json
```定義ファイルの形式については、`references/azurerm_set_attributes.md`を参照してください。

## 制限事項

- AzureRM リソース (`azurerm_*`) のみがサポートされています
- 一部のリソース/属性はサポートされていない場合があります
- `after_unknown` (適用後に決定される値) を含む属性の比較は不完全である可能性があります。
- 機密属性の比較は不完全である可能性があります (マスクされています)

## 関連ドキュメント

- [SKILL.md](../SKILL.md) - エージェントスキルとしての利用
- [azurerm_set_attributes.md](../references/azurerm_set_attributes.md) - 属性定義リファレンス