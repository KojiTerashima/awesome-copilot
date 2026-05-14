---
name: azure-static-web-apps
description: SWA CLI を使用して Azure Static Web Apps を作成・設定・デプロイするのに役立ちます。静的サイトを Azure にデプロイするとき、SWA のローカル開発を設定するとき、staticwebapp.config.json を設定するとき、Azure Functions API を SWA に追加するとき、または Static Web Apps の GitHub Actions CI/CD を設定するときに使用します。
---

## 概要

Azure Static Web Apps (SWA) は、静的フロントエンドを、必要に応じてサーバーレス API バックエンド付きでホストします。SWA CLI (`swa`) は、ローカル開発エミュレーションとデプロイ機能を提供します。

**主な機能:**
- API プロキシと認証シミュレーションを備えたローカルエミュレーター
- フレームワークの自動検出と設定
- Azure への直接デプロイ
- データベース接続のサポート

**設定ファイル:**
- `swa-cli.config.json` - CLI 設定、**`swa init` によって作成**（手動で作成しない）
- `staticwebapp.config.json` - ランタイム設定（ルート、認証、ヘッダー、API ランタイム）- 手動作成可能

## 一般的な手順

### インストール

```bash
npm install -D @azure/static-web-apps-cli
```

確認: `npx swa --version`

### クイックスタートのワークフロー

**重要: 設定ファイルの作成には必ず `swa init` を使用してください。`swa-cli.config.json` を手動で作成してはいけません。**

1. `swa init` - **必須の最初の手順** - フレームワークを自動検出し、`swa-cli.config.json` を作成
2. `swa start` - `http://localhost:4280` でローカルエミュレーターを実行
3. `swa login` - Azure で認証
4. `swa deploy` - Azure にデプロイ

### 設定ファイル

**swa-cli.config.json** - `swa init` で作成されるため、手動作成しないでください:
- `swa init` を実行して、フレームワーク検出付きの対話式セットアップを行う
- `swa init --yes` を実行して、自動検出された既定値を受け入れる
- 初期化後に設定をカスタマイズする場合のみ、生成されたファイルを編集する

生成される設定の例（参照用）:
```json
{
  "$schema": "https://aka.ms/azure/static-web-apps-cli/schema",
  "configurations": {
    "app": {
      "appLocation": ".",
      "apiLocation": "api",
      "outputLocation": "dist",
      "appBuildCommand": "npm run build",
      "run": "npm run dev",
      "appDevserverUrl": "http://localhost:3000"
    }
  }
}
```

**staticwebapp.config.json**（アプリのソースまたは出力フォルダー内）- このファイルはランタイム設定のために手動作成できます:
```json
{
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/images/*", "/css/*"]
  },
  "routes": [
    { "route": "/api/*", "allowedRoles": ["authenticated"] }
  ],
  "platform": {
    "apiRuntime": "node:20"
  }
}
```

## コマンドライン リファレンス

### swa login

デプロイのために Azure で認証します。

```bash
swa login                              # 対話式ログイン
swa login --subscription-id <id>       # 特定のサブスクリプション
swa login --clear-credentials          # キャッシュされた資格情報をクリア
```

**フラグ:** `--subscription-id, -S` | `--resource-group, -R` | `--tenant-id, -T` | `--client-id, -C` | `--client-secret, -CS` | `--app-name, -n`

### swa init

既存のフロントエンドと（任意の）API に基づいて新しい SWA プロジェクトを設定します。フレームワークを自動検出します。

```bash
swa init                    # 対話式セットアップ
swa init --yes              # 既定値を受け入れる
```

### swa build

フロントエンドおよび/または API をビルドします。

```bash
swa build                   # 設定を使ってビルド
swa build --auto            # 自動検出してビルド
swa build myApp             # 特定の構成をビルド
```

**フラグ:** `--app-location, -a` | `--api-location, -i` | `--output-location, -O` | `--app-build-command, -A` | `--api-build-command, -I`

### swa start

ローカル開発エミュレーターを起動します。

```bash
swa start                                    # outputLocation から配信
swa start ./dist                             # 特定フォルダーを配信
swa start http://localhost:3000              # 開発サーバーへプロキシ
swa start ./dist --api-location ./api        # API フォルダー付き
swa start http://localhost:3000 --run "npm start"  # 開発サーバーを自動起動
```

**一般的なフレームワークのポート:**
| Framework | Port |
|-----------|------|
| React/Vue/Next.js | 3000 |
| Angular | 4200 |
| Vite | 5173 |

**主要なフラグ:**
- `--port, -p` - エミュレーターのポート（既定: 4280）
- `--api-location, -i` - API フォルダーのパス
- `--api-port, -j` - API ポート（既定: 7071）
- `--run, -r` - 開発サーバー起動コマンド
- `--open, -o` - ブラウザーを自動で開く
- `--ssl, -s` - HTTPS を有効化

### swa deploy

Azure Static Web Apps にデプロイします。

```bash
swa deploy                              # 設定を使ってデプロイ
swa deploy ./dist                       # 特定フォルダーをデプロイ
swa deploy --env production             # 本番環境へデプロイ
swa deploy --deployment-token <TOKEN>   # デプロイトークンを使用
swa deploy --dry-run                    # デプロイせずにプレビュー
```

**デプロイトークンの取得:**
- Azure Portal: Static Web App → Overview → Manage deployment token
- CLI: `swa deploy --print-token`
- 環境変数: `SWA_CLI_DEPLOYMENT_TOKEN`

**主要なフラグ:**
- `--env` - 対象環境（`preview` または `production`）
- `--deployment-token, -d` - デプロイトークン
- `--app-name, -n` - Azure SWA リソース名

### swa db

データベース接続を初期化します。

```bash
swa db init --database-type mssql
swa db init --database-type postgresql
swa db init --database-type cosmosdb_nosql
```

## シナリオ

### 既存のフロントエンドとバックエンドから SWA を作成する

**必ず `swa start` または `swa deploy` の前に `swa init` を実行してください。`swa-cli.config.json` を手動で作成してはいけません。**

```bash
# 1. CLI をインストール
npm install -D @azure/static-web-apps-cli

# 2. 初期化 - 必須: 自動検出された設定で swa-cli.config.json を作成
npx swa init              # 対話モード
# OR
npx swa init --yes        # 自動検出された既定値を受け入れる

# 3. アプリケーションをビルド（必要な場合）
npm run build

# 4. ローカルでテスト（swa-cli.config.json の設定を使用）
npx swa start

# 5. デプロイ
npx swa login
npx swa deploy --env production
```

### Azure Functions バックエンドを追加する

1. **API フォルダーを作成:**
```bash
mkdir api && cd api
func init --worker-runtime node --model V4
func new --name message --template "HTTP trigger"
```

2. **関数の例**（`api/src/functions/message.js`）:
```javascript
const { app } = require('@azure/functions');

app.http('message', {
    methods: ['GET', 'POST'],
    authLevel: 'anonymous',
    handler: async (request) => {
        const name = request.query.get('name') || 'World';
        return { jsonBody: { message: `Hello, ${name}!` } };
    }
});
```

3. `staticwebapp.config.json` で **API ランタイムを設定**:
```json
{
  "platform": { "apiRuntime": "node:20" }
}
```

4. `swa-cli.config.json` で **CLI 設定を更新**:
```json
{
  "configurations": {
    "app": { "apiLocation": "api" }
  }
}
```

5. **ローカルでテスト:**
```bash
npx swa start ./dist --api-location ./api
# API には http://localhost:4280/api/message でアクセス
```

**サポートされる API ランタイム:** `node:18`, `node:20`, `node:22`, `dotnet:8.0`, `dotnet-isolated:8.0`, `python:3.10`, `python:3.11`

### GitHub Actions デプロイを設定する

1. Azure Portal または Azure CLI で **SWA リソースを作成**
2. **GitHub リポジトリを連携** - ワークフローは自動生成されるか、手動で作成:

`.github/workflows/azure-static-web-apps.yml`:
```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches: [main]

jobs:
  build_and_deploy:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build And Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: upload
          app_location: /
          api_location: api
          output_location: dist

  close_pr:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          action: close
```

3. **シークレットを追加:** デプロイトークンをリポジトリシークレット `AZURE_STATIC_WEB_APPS_API_TOKEN` にコピー

**ワークフロー設定:**
- `app_location` - フロントエンドのソースパス
- `api_location` - API のソースパス
- `output_location` - ビルド済み出力フォルダー
- `skip_app_build: true` - 事前ビルド済みならスキップ
- `app_build_command` - カスタムビルドコマンド

## トラブルシューティング

| Issue | Solution |
|-------|----------|
| 404 on client routes | `staticwebapp.config.json` に `rewrite: "/index.html"` を含む `navigationFallback` を追加 |
| API returns 404 | `api` フォルダー構造を確認し、`platform.apiRuntime` が設定されていることを確認し、関数エクスポートをチェック |
| Build output not found | `output_location` が実際のビルド出力ディレクトリと一致しているか確認 |
| Auth not working locally | `/.auth/login/<provider>` を使って認証エミュレーター UI にアクセス |
| CORS errors | `/api/*` 配下の API は同一オリジン。外部 API には CORS ヘッダーが必要 |
| Deployment token expired | Azure Portal → Static Web App → Manage deployment token で再生成 |
| Config not applied | `staticwebapp.config.json` が `app_location` または `output_location` にあることを確認 |
| Local API timeout | 既定は 45 秒。関数を最適化するか、ブロッキング呼び出しを確認 |

**デバッグコマンド:**
```bash
swa start --verbose log        # 詳細出力
swa deploy --dry-run           # デプロイをプレビュー
swa --print-config             # 解決済み設定を表示
```

