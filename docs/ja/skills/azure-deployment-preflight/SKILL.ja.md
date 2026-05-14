---
name: azure-deployment-preflight
description: 'Azure への Bicep デプロイメントに対して、テンプレート構文検証、what-if 分析、権限チェックを含む包括的な事前検証を実行します。Azure へデプロイする前にこのスキルを使うことで、変更内容の事前確認、潜在的な問題の特定、デプロイ成功の確実化ができます。ユーザーが Azure へのデプロイ、Bicep ファイルの検証、デプロイ権限の確認、インフラ変更のプレビュー、what-if の実行、または azd provision の準備に言及したときに有効化してください。'
---

# Azure デプロイ事前検証

このスキルは、実行前に Bicep デプロイメントを検証し、Azure CLI (`az`) と Azure Developer CLI (`azd`) の両ワークフローをサポートします。

## このスキルを使うタイミング

- Azure にインフラをデプロイする前
- Bicep ファイルを準備またはレビューするとき
- デプロイによってどのような変更が行われるかを事前確認したいとき
- デプロイに必要な権限が十分か確認したいとき
- `azd up`、`azd provision`、`az deployment` コマンドを実行する前

## 検証プロセス

以下の手順を順番に実行してください。前の手順が失敗しても次の手順へ進み、最終レポートですべての問題を記録します。

### Step 1: プロジェクト種別を判定

プロジェクトの指標を確認して、デプロイワークフローを判定します。

1. **azd プロジェクトか確認**: プロジェクトルートに `azure.yaml` があるか確認
   - 見つかった場合 → **azd ワークフロー**を使用
   - 見つからない場合 → **az CLI ワークフロー**を使用

2. **Bicep ファイルを特定**: 検証対象の `.bicep` ファイルをすべて探す
   - azd プロジェクト: まず `infra/` ディレクトリ、次にプロジェクトルートを確認
   - スタンドアロン: ユーザー指定ファイルを使用、または一般的な場所（`infra/`、`deploy/`、プロジェクトルート）を探索

3. **パラメーターファイルを自動検出**: 各 Bicep ファイルに対して対応するパラメーターファイルを探す
   - `<filename>.bicepparam`（Bicep パラメーター - 推奨）
   - `<filename>.parameters.json`（JSON パラメーター）
   - 同じディレクトリ内の `parameters.json` または `parameters/<env>.json`

### Step 2: Bicep 構文を検証

デプロイ検証を試みる前に、Bicep CLI でテンプレート構文を確認します。

```bash
bicep build <bicep-file> --stdout
```

**記録すべき内容:**
- 行/列番号付きの構文エラー
- 警告メッセージ
- ビルド成功/失敗ステータス

**Bicep CLI がインストールされていない場合:**
- レポートに問題を記載
- Step 3 へ進む（what-if 中に Azure が構文を検証）

### Step 3: 事前検証を実行

Step 1 で判定したプロジェクト種別に応じて適切な検証を選択します。

#### azd プロジェクトの場合（azure.yaml が存在）

`azd provision --preview` を使ってデプロイを検証します。

```bash
azd provision --preview
```

環境が指定されている、または複数環境がある場合:
```bash
azd provision --preview --environment <env-name>
```

#### スタンドアロン Bicep の場合（azure.yaml なし）

Bicep ファイルの `targetScope` 宣言からデプロイスコープを判定します。

| Target Scope | Command |
|--------------|---------|
| `resourceGroup`（デフォルト） | `az deployment group what-if` |
| `subscription` | `az deployment sub what-if` |
| `managementGroup` | `az deployment mg what-if` |
| `tenant` | `az deployment tenant what-if` |

**まず Provider 検証レベルで実行:**

```bash
# リソースグループ スコープ（最も一般的）
az deployment group what-if \
  --resource-group <rg-name> \
  --template-file <bicep-file> \
  --parameters <param-file> \
  --validation-level Provider

# サブスクリプション スコープ
az deployment sub what-if \
  --location <location> \
  --template-file <bicep-file> \
  --parameters <param-file> \
  --validation-level Provider

# 管理グループ スコープ
az deployment mg what-if \
  --location <location> \
  --management-group-id <mg-id> \
  --template-file <bicep-file> \
  --parameters <param-file> \
  --validation-level Provider

# テナント スコープ
az deployment tenant what-if \
  --location <location> \
  --template-file <bicep-file> \
  --parameters <param-file> \
  --validation-level Provider
```

**フォールバック戦略:**

`--validation-level Provider` が権限エラー（RBAC）で失敗した場合は、`ProviderNoRbac` で再試行します。

```bash
az deployment group what-if \
  --resource-group <rg-name> \
  --template-file <bicep-file> \
  --validation-level ProviderNoRbac
```

フォールバックをレポートに記載してください。ユーザーに完全なデプロイ権限がない可能性があります。

### Step 4: What-If 結果を記録

what-if の出力を解析し、リソース変更を分類します。

| Change Type | Symbol | Meaning |
|-------------|--------|---------|
| Create | `+` | 新しいリソースが作成される |
| Delete | `-` | リソースが削除される |
| Modify | `~` | リソースプロパティが変更される |
| NoChange | `=` | リソースに変更なし |
| Ignore | `*` | リソースは解析されない（上限到達） |
| Deploy | `!` | リソースはデプロイされる（変更内容不明） |

変更されたリソースについては、具体的なプロパティ変更を記録します。

### Step 5: レポートを生成

**プロジェクトルート**に次の名前で Markdown レポートファイルを作成します。
- `preflight-report.md`

テンプレート構造は [references/REPORT-TEMPLATE.md](references/REPORT-TEMPLATE.md) を使用してください。

**レポートのセクション:**
1. **Summary** - 全体ステータス、タイムスタンプ、検証したファイル、ターゲットスコープ
2. **Tools Executed** - 実行コマンド、バージョン、使用した検証レベル
3. **Issues** - すべてのエラーと警告（重大度・対処方法付き）
4. **What-If Results** - 作成/変更/削除/変更なしのリソース
5. **Recommendations** - 実行可能な次のアクション

## 必要な情報

検証を実行する前に、以下を収集してください。

| Information | Required For | How to Obtain |
|-------------|--------------|---------------|
| Resource Group | `az deployment group` | ユーザーに確認、または既存の `.azure/` 設定を確認 |
| Subscription | すべてのデプロイ | `az account show` またはユーザーに確認 |
| Location | Sub/MG/Tenant スコープ | ユーザーに確認、または設定のデフォルトを使用 |
| Environment | azd プロジェクト | `azd env list` またはユーザーに確認 |

必要な情報が不足している場合は、進行前にユーザーへ確認してください。

## エラーハンドリング

詳細なエラーハンドリング指針は [references/ERROR-HANDLING.md](references/ERROR-HANDLING.md) を参照してください。

**重要原則:** エラーが発生しても検証を継続し、最終レポートにすべての問題を記録します。

| Error Type | Action |
|------------|--------|
| 未ログイン | レポートに記載し、`az login` または `azd auth login` を提案 |
| 権限拒否 | `ProviderNoRbac` にフォールバックし、レポートに記載 |
| Bicep 構文エラー | すべてのエラーを記載し、他ファイルの検証を継続 |
| ツール未インストール | レポートに記載し、その検証ステップをスキップ |
| リソースグループ未検出 | レポートに記載し、作成を提案 |

## ツール要件

このスキルは以下のツールを使用します。

- **Azure CLI** (`az`) - `--validation-level` には Version 2.76.0+ 推奨
- **Azure Developer CLI** (`azd`) - `azure.yaml` を持つプロジェクト用
- **Bicep CLI** (`bicep`) - 構文検証用
- **Azure MCP Tools** - ドキュメント参照とベストプラクティス確認用

開始前にツールの利用可否を確認してください。
```bash
az --version
azd version
bicep --version
```

## ワークフロー例

1. ユーザー: "Validate my Bicep deployment before I run it"
2. エージェントが `azure.yaml` を検出 → azd プロジェクト
3. エージェントが `infra/main.bicep` と `infra/main.bicepparam` を検出
4. エージェントが `bicep build infra/main.bicep --stdout` を実行
5. エージェントが `azd provision --preview` を実行
6. エージェントがプロジェクトルートに `preflight-report.md` を生成
7. エージェントがユーザーに結果を要約

## 参照ドキュメント

- [Validation Commands Reference](references/VALIDATION-COMMANDS.md)
- [Report Template](references/REPORT-TEMPLATE.md)
- [Error Handling Guide](references/ERROR-HANDLING.md)

