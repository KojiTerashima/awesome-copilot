# パイプライン、ビルド＆リリース

## 目次
- [パイプライン](#pipelines)
- [パイプライン実行](#pipeline-runs)
- [ビルド](#builds)
- [ビルド定義](#build-definitions)
- [リリース](#releases)
- [リリース定義](#release-definitions)
- [ユニバーサル パッケージ（Artifacts）](#universal-packages-artifacts)

---

## パイプライン

### パイプライン一覧

```bash
az pipelines list --output table
az pipelines list --query "[?name=='myPipeline']"
az pipelines list --folder-path 'folder/subfolder'
```

### パイプライン作成

```bash
# ローカル リポジトリのコンテキストから（設定を自動検出）
az pipelines create --name 'ContosoBuild' --description 'contoso プロジェクト用パイプライン'

# ブランチと YAML パスを指定
az pipelines create \
  --name {pipeline-name} \
  --repository {repo} \
  --branch main \
  --yaml-path azure-pipelines.yml \
  --description "My CI/CD pipeline"

# GitHub リポジトリ向け
az pipelines create \
  --name 'GitHubPipeline' \
  --repository https://github.com/Org/Repo \
  --branch main \
  --repository-type github

# 初回実行をスキップ
az pipelines create --name 'MyPipeline' --skip-run true
```

### パイプライン表示

```bash
az pipelines show --id {pipeline-id}
az pipelines show --name {pipeline-name}
```

### パイプライン更新

```bash
az pipelines update --id {pipeline-id} --name "New name" --description "Updated description"
```

### パイプライン削除

```bash
az pipelines delete --id {pipeline-id} --yes
```

### パイプライン実行

```bash
# 名前で実行
az pipelines run --name {pipeline-name} --branch main

# IDで実行
az pipelines run --id {pipeline-id} --branch refs/heads/main

# パラメーター付き
az pipelines run --name {pipeline-name} --parameters version=1.0.0 environment=prod

# 変数付き
az pipelines run --name {pipeline-name} --variables buildId=123 configuration=release

# 結果をブラウザーで開く
az pipelines run --name {pipeline-name} --open
```

## パイプライン実行

### 実行一覧

```bash
az pipelines runs list --pipeline {pipeline-id}
az pipelines runs list --name {pipeline-name} --top 10
az pipelines runs list --branch main --status completed
```

### 実行詳細の表示

```bash
az pipelines runs show --run-id {run-id}
az pipelines runs show --run-id {run-id} --open
```

### パイプライン アーティファクト

```bash
# 実行のアーティファクト一覧
az pipelines runs artifact list --run-id {run-id}

# アーティファクトをダウンロード
az pipelines runs artifact download \
  --artifact-name '{artifact-name}' \
  --path {local-path} \
  --run-id {run-id}

# アーティファクトをアップロード
az pipelines runs artifact upload \
  --artifact-name '{artifact-name}' \
  --path {local-path} \
  --run-id {run-id}
```

### パイプライン実行タグ

```bash
# 実行にタグを追加
az pipelines runs tag add --run-id {run-id} --tags production v1.0

# 実行タグ一覧
az pipelines runs tag list --run-id {run-id} --output table
```

## ビルド

### ビルド一覧

```bash
az pipelines build list
az pipelines build list --definition {build-definition-id}
az pipelines build list --status completed --result succeeded
```

### ビルドをキュー登録

```bash
az pipelines build queue --definition {build-definition-id} --branch main
az pipelines build queue --definition {build-definition-id} --parameters version=1.0.0
```

### ビルド詳細の表示

```bash
az pipelines build show --id {build-id}
```

### ビルドのキャンセル

```bash
az pipelines build cancel --id {build-id}
```

### ビルドタグ

```bash
# ビルドにタグを追加
az pipelines build tag add --build-id {build-id} --tags prod release

# ビルドからタグを削除
az pipelines build tag delete --build-id {build-id} --tag prod
```

## ビルド定義

### ビルド定義一覧

```bash
az pipelines build definition list
az pipelines build definition list --name {definition-name}
```

### ビルド定義の表示

```bash
az pipelines build definition show --id {definition-id}
```

## リリース

### リリース一覧

```bash
az pipelines release list
az pipelines release list --definition {release-definition-id}
```

### リリース作成

```bash
az pipelines release create --definition {release-definition-id}
az pipelines release create --definition {release-definition-id} --description "Release v1.0"
```

### リリース表示

```bash
az pipelines release show --id {release-id}
```

## リリース定義

### リリース定義一覧

```bash
az pipelines release definition list
```

### リリース定義の表示

```bash
az pipelines release definition show --id {definition-id}
```

## ユニバーサル パッケージ（Artifacts）

### パッケージ公開

```bash
az artifacts universal publish \
  --feed {feed-name} \
  --name {package-name} \
  --version {version} \
  --path {package-path} \
  --project {project}
```

### パッケージのダウンロード

```bash
az artifacts universal download \
  --feed {feed-name} \
  --name {package-name} \
  --version {version} \
  --path {download-path} \
  --project {project}
```

