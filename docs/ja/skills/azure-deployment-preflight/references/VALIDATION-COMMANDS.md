# 検証コマンド リファレンス

このリファレンスでは、Azure デプロイ前の事前検証で使用するすべてのコマンドを説明します。

## Azure Developer CLI (azd)

### azd provision --preview

デプロイを実行せずに、azd プロジェクトのインフラ変更内容をプレビューします。

```bash
azd provision --preview [options]
```

**オプション:**
| Option | Description |
|--------|-------------|
| `--environment`, `-e` | 使用する環境名 |
| `--no-prompt` | プロンプトを表示せず既定値を受け入れる |
| `--debug` | デバッグログを有効化する |
| `--cwd` | 作業ディレクトリを設定する |

**例:**

```bash
# 既定環境でプレビュー
azd provision --preview

# 特定環境をプレビュー
azd provision --preview --environment dev

# プロンプトなしでプレビュー (CI/CD)
azd provision --preview --no-prompt
```

**出力:** 作成・変更・削除されるリソースを表示します。

### azd auth login

azd 操作のために Azure へ認証します。

```bash
azd auth login [options]
```

**オプション:**
| Option | Description |
|--------|-------------|
| `--check-status` | ログインせずにログイン状態を確認する |
| `--use-device-code` | デバイスコードフローを使用する |
| `--tenant-id` | テナントを指定する |
| `--client-id` | サービス プリンシパルのクライアント ID |

### azd env list

利用可能な環境を一覧表示します。

```bash
azd env list
```

---

## Azure CLI (az)

### az deployment group what-if

リソース グループ スコープのデプロイ変更をプレビューします。

```bash
az deployment group what-if \
  --resource-group <rg-name> \
  --template-file <bicep-file> \
  [options]
```

**必須パラメーター:**
| Parameter | Description |
|-----------|-------------|
| `--resource-group`, `-g` | 対象リソース グループ名 |
| `--template-file`, `-f` | Bicep ファイルのパス |

**任意パラメーター:**
| Parameter | Description |
|-----------|-------------|
| `--parameters`, `-p` | パラメーターファイルまたはインライン値 |
| `--validation-level` | `Provider` (既定), `ProviderNoRbac`, または `Template` |
| `--result-format` | `FullResourcePayloads` (既定) または `ResourceIdOnly` |
| `--no-pretty-print` | 解析用に生の JSON を出力する |
| `--name`, `-n` | デプロイ名 |
| `--exclude-change-types` | 出力から特定の変更種別を除外する |

**検証レベル:**
| Level | Description | Use Case |
|-------|-------------|----------|
| `Provider` | RBAC チェックを含む完全な検証 | 既定、最も網羅的 |
| `ProviderNoRbac` | 読み取り権限のみで完全な検証 | デプロイ権限がない場合 |
| `Template` | 静的な構文検証のみ | 構文の簡易チェック |

**例:**

```bash
# 基本的な what-if
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep

# パラメーター指定 + 完全検証
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --parameters main.bicepparam \
  --validation-level Provider

# RBAC チェックなしのフォールバック
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --validation-level ProviderNoRbac

# 解析用の JSON 出力
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --no-pretty-print
```

### az deployment sub what-if

サブスクリプション スコープのデプロイ変更をプレビューします。

```bash
az deployment sub what-if \
  --location <location> \
  --template-file <bicep-file> \
  [options]
```

**必須パラメーター:**
| Parameter | Description |
|-----------|-------------|
| `--location`, `-l` | デプロイ メタデータ用のロケーション |
| `--template-file`, `-f` | Bicep ファイルのパス |

**例:**

```bash
az deployment sub what-if \
  --location eastus \
  --template-file main.bicep \
  --parameters main.bicepparam \
  --validation-level Provider
```

### az deployment mg what-if

管理グループ スコープのデプロイ変更をプレビューします。

```bash
az deployment mg what-if \
  --location <location> \
  --management-group-id <mg-id> \
  --template-file <bicep-file> \
  [options]
```

**必須パラメーター:**
| Parameter | Description |
|-----------|-------------|
| `--location`, `-l` | デプロイ メタデータ用のロケーション |
| `--management-group-id`, `-m` | 対象管理グループ ID |
| `--template-file`, `-f` | Bicep ファイルのパス |

### az deployment tenant what-if

テナント スコープのデプロイ変更をプレビューします。

```bash
az deployment tenant what-if \
  --location <location> \
  --template-file <bicep-file> \
  [options]
```

**必須パラメーター:**
| Parameter | Description |
|-----------|-------------|
| `--location`, `-l` | デプロイ メタデータ用のロケーション |
| `--template-file`, `-f` | Bicep ファイルのパス |

### az login

Azure CLI へ認証します。

```bash
az login [options]
```

**オプション:**
| Option | Description |
|--------|-------------|
| `--tenant`, `-t` | テナント ID またはドメイン |
| `--use-device-code` | デバイスコードフローを使用する |
| `--service-principal` | サービス プリンシパルとしてログインする |

### az account show

現在のサブスクリプション コンテキストを表示します。

```bash
az account show
```

### az group exists

リソース グループが存在するか確認します。

```bash
az group exists --name <rg-name>
```

---

## Bicep CLI

### bicep build

Bicep を ARM JSON にコンパイルし、構文を検証します。

```bash
bicep build <bicep-file> [options]
```

**オプション:**
| Option | Description |
|--------|-------------|
| `--stdout` | ファイルではなく stdout に出力する |
| `--outdir` | 出力ディレクトリ |
| `--outfile` | 出力ファイルパス |
| `--no-restore` | モジュールの復元をスキップする |

**例:**

```bash
# 構文検証 (stdout に出力、ファイルは作成しない)
bicep build main.bicep --stdout > /dev/null

# 特定ディレクトリにビルド
bicep build main.bicep --outdir ./build

# 複数ファイルを検証
for f in *.bicep; do bicep build "$f" --stdout; done
```

**エラー出力形式:**
```
/path/to/file.bicep(22,51) : Error BCP064: Found unexpected tokens in interpolated expression.
/path/to/file.bicep(22,51) : Error BCP004: The string at this location is not terminated.
```

形式: `<file>(<line>,<column>) : <severity> <code>: <message>`

### bicep --version

Bicep CLI のバージョンを確認します。

```bash
bicep --version
```

---

## パラメーターファイルの検出

### Bicep Parameters (.bicepparam)

モダンな Bicep パラメーターファイル (推奨):

```bicep
using './main.bicep'

param location = 'eastus'
param environment = 'dev'
param tags = {
  environment: 'dev'
  project: 'myapp'
}
```

**検出パターン:** `<template-name>.bicepparam`

### JSON Parameters (.parameters.json)

従来の ARM パラメーターファイル:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": { "value": "eastus" },
    "environment": { "value": "dev" }
  }
}
```

**検出パターン:**
- `<template-name>.parameters.json`
- `parameters.json`
- `parameters/<env>.json`

### コマンドでのパラメーター利用

```bash
# Bicep パラメーターファイル
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --parameters main.bicepparam

# JSON パラメーターファイル
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --parameters @parameters.json

# インラインでパラメーターを上書き
az deployment group what-if \
  --resource-group my-rg \
  --template-file main.bicep \
  --parameters main.bicepparam \
  --parameters location=westus
```

---

## デプロイ スコープの判定

Bicep ファイルの `targetScope` 宣言を確認します:

```bicep
// Resource Group (未指定時の既定)
targetScope = 'resourceGroup'

// Subscription
targetScope = 'subscription'

// Management Group
targetScope = 'managementGroup'

// Tenant
targetScope = 'tenant'
```

**スコープとコマンドの対応:**

| targetScope | Command | Required Parameters |
|-------------|---------|---------------------|
| `resourceGroup` | `az deployment group what-if` | `--resource-group` |
| `subscription` | `az deployment sub what-if` | `--location` |
| `managementGroup` | `az deployment mg what-if` | `--location`, `--management-group-id` |
| `tenant` | `az deployment tenant what-if` | `--location` |

---

## バージョン要件

| Tool | Minimum Version | Recommended Version | Key Features |
|------|-----------------|---------------------|--------------|
| Azure CLI | 2.14.0 | 2.76.0+ | `--validation-level` スイッチ |
| Azure Developer CLI | 1.0.0 | Latest | `--preview` フラグ |
| Bicep CLI | 0.4.0 | Latest | より分かりやすいエラーメッセージ |

**バージョン確認:**
```bash
az --version
azd version
bicep --version
```

