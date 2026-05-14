---
name: azure-devops-cli
description: プロジェクト、リポジトリ、パイプライン、ビルド、プルリクエスト、作業項目、アーティファクト、サービス エンドポイントなどの Azure DevOps リソースを CLI 経由で管理します。Azure DevOps、az コマンド、devops 自動化、CI/CD を扱う場合、またはユーザーが Azure DevOps CLI に言及した場合に使用してください。
---

# Azure DevOps CLI

Azure DevOps 拡張機能を備えた Azure CLI を使用して Azure DevOps リソースを管理します。

**CLI バージョン:** 2.81.0（2025年時点の最新）

## 前提条件

```bash
# Azure CLI をインストール
brew install azure-cli  # macOS
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash  # Linux

# Azure DevOps 拡張機能をインストール
az extension add --name azure-devops
```

## 認証

```bash
# PAT トークンでログイン
az devops login --organization https://dev.azure.com/{org} --token YOUR_PAT_TOKEN

# 既定の organization と project を設定（--org/--project の繰り返し指定を回避）
# 注: 旧 URL https://{org}.visualstudio.com は https://dev.azure.com/{org} に置き換える必要があります
az devops configure --defaults organization=https://dev.azure.com/{org} project={project}

# 現在の構成を一覧表示
az devops configure --list
```

## CLI 構成

```
az devops          # メインの DevOps コマンド
├── admin          # 管理（バナー）
├── extension      # 拡張機能管理
├── project        # チーム プロジェクト
├── security       # セキュリティ操作
│   ├── group      # セキュリティ グループ
│   └── permission # セキュリティ権限
├── service-endpoint # サービス接続
├── team           # チーム
├── user           # ユーザー
├── wiki           # Wiki
├── configure      # 既定値を設定
├── invoke         # REST API を呼び出す
├── login          # 認証
└── logout         # 認証情報をクリア

az pipelines       # Azure Pipelines
├── agent          # エージェント
├── build          # ビルド
├── folder         # パイプライン フォルダー
├── pool           # エージェント プール
├── queue          # エージェント キュー
├── release        # リリース
├── runs           # パイプライン実行
├── variable       # パイプライン変数
└── variable-group # 変数グループ

az boards          # Azure Boards
├── area           # エリア パス
├── iteration      # イテレーション
└── work-item      # 作業項目

az repos           # Azure Repos
├── import         # Git インポート
├── policy         # ブランチ ポリシー
├── pr             # プルリクエスト
└── ref            # Git 参照

az artifacts       # Azure Artifacts
└── universal      # Universal Packages
```

## 参照ファイル

ユーザーのタスクに応じて該当する参照ファイルを読んでください。各ファイルには、その領域に関する完全なコマンド構文と例が含まれています。

| File | When to read | Covers |
|---|---|---|
| `references/repos-and-prs.md` | リポジトリ、ブランチ、プルリクエスト、ブランチ ポリシー | リポジトリ、インポート、PR（作成/一覧/投票/レビュー担当者/ポリシー）、Git 参照、ブランチ ポリシー |
| `references/pipelines-and-builds.md` | パイプライン、ビルド、リリース、アーティファクト | パイプライン CRUD、実行、ビルド、リリース、アーティファクトのダウンロード/アップロード |
| `references/boards-and-iterations.md` | 作業項目、スプリント、エリア パス | 作業項目（WIQL/作成/更新/関連付け）、エリア パス、イテレーション、チーム イテレーション |
| `references/variables-and-agents.md` | パイプライン変数、エージェント プール | パイプライン変数、変数グループ、パイプライン フォルダー、エージェント プール/キュー |
| `references/org-and-security.md` | プロジェクト、チーム、ユーザー、権限、Wiki | プロジェクト、拡張機能、チーム、ユーザー、セキュリティ グループ/権限、サービス エンドポイント、Wiki、管理 |
| `references/advanced-usage.md` | 出力フォーマット、JMESPath クエリ | 出力形式、JMESPath クエリ（基本 + 高度）、グローバル引数、共通パラメーター、Git エイリアス |
| `references/workflows-and-patterns.md` | 自動化スクリプト、ベスト プラクティス、エラーハンドリング | 一般的なワークフロー、ベスト プラクティス、エラーハンドリング、スクリプト パターン、実践的な例 |

