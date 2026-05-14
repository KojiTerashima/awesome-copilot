---
name: aws-cdk-python-setup
description: Python で AWS CDK（Cloud Development Kit）アプリケーションを開発するためのセットアップおよび初期化ガイド。このスキルにより、環境の前提条件の設定、新しい CDK プロジェクトの作成、依存関係の管理、AWS へのデプロイが可能になります。
---
# AWS CDK Python セットアップ手順

このスキルは、**Python** を使った **AWS CDK（Cloud Development Kit）** プロジェクト作業のためのセットアップ手順を提供します。

---

## 前提条件

開始前に、次のツールがインストールされていることを確認してください。

- **Node.js** ≥ 14.15.0 — AWS CDK CLI に必要
- **Python** ≥ 3.7 — CDK コードの記述に使用
- **AWS CLI** — 認証情報とリソースを管理
- **Git** — バージョン管理とプロジェクト管理

---

## インストール手順

### 1. AWS CDK CLI をインストールする
```bash
npm install -g aws-cdk
cdk --version
```

### 2. AWS 認証情報を設定する
```bash
# AWS CLI をインストール（未インストールの場合）
brew install awscli

# 認証情報を設定
aws configure
```
プロンプトが表示されたら、AWS Access Key、Secret Access Key、デフォルトリージョン、出力形式を入力してください。

### 3. 新しい CDK プロジェクトを作成する
```bash
mkdir my-cdk-project
cd my-cdk-project
cdk init app --language python
```

プロジェクトには以下が含まれます。
- `app.py` — メインアプリケーションのエントリーポイント
- `my_cdk_project/` — CDK スタック定義
- `requirements.txt` — Python 依存関係
- `cdk.json` — 設定ファイル

### 4. Python 仮想環境をセットアップする
```bash
# macOS/Linux
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

### 5. Python 依存関係をインストールする
```bash
pip install -r requirements.txt
```
主な依存関係:
- `aws-cdk-lib` — コア CDK コンストラクト
- `constructs` — 基本コンストラクトライブラリ

---

## 開発ワークフロー

### CloudFormation テンプレートを合成する
```bash
cdk synth
```
CloudFormation テンプレートを含む `cdk.out/` を生成します。

### スタックを AWS にデプロイする
```bash
cdk deploy
```
設定済みの AWS アカウントへのデプロイ内容を確認し、実行を確定します。

### ブートストラップ（初回デプロイ時のみ）
```bash
cdk bootstrap
```
アセット保存用の S3 バケットなど、環境リソースを準備します。

---

## ベストプラクティス

- 作業前には必ず仮想環境を有効化してください。
- デプロイ前に `cdk diff` を実行して変更内容を確認してください。
- テストには開発用アカウントを使用してください。
- Python らしい命名規則とディレクトリ規約に従ってください。
- 一貫したビルドのために `requirements.txt` のバージョンを固定してください。

---

## トラブルシューティングのヒント

問題が発生した場合は、次を確認してください。

- AWS 認証情報が正しく設定されている。
- デフォルトリージョンが適切に設定されている。
- Node.js と Python のバージョンが最小要件を満たしている。
- `cdk doctor` を実行して環境の問題を診断する。

