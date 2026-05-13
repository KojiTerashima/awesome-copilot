# 組織・セキュリティ・管理

## 目次
- [プロジェクト](#projects)
- [拡張機能管理](#extension-management)
- [サービス エンドポイント](#service-endpoints)
- [チーム](#teams)
- [ユーザー](#users)
- [セキュリティ グループ](#security-groups)
- [セキュリティ権限](#security-permissions)
- [Wiki](#wikis)
- [管理](#administration)
- [DevOps 拡張機能](#devops-extensions)

---

## プロジェクト

### プロジェクトを一覧表示

```bash
az devops project list --organization https://dev.azure.com/{org}
az devops project list --top 10 --output table
```

### プロジェクトを作成

```bash
az devops project create \
  --name myNewProject \
  --organization https://dev.azure.com/{org} \
  --description "My new DevOps project" \
  --source-control git \
  --visibility private
```

### プロジェクト詳細を表示

```bash
az devops project show --project {project-name} --org https://dev.azure.com/{org}
```

### プロジェクトを削除

```bash
az devops project delete --id {project-id} --org https://dev.azure.com/{org} --yes
```

## 拡張機能管理

### 拡張機能を一覧表示

```bash
# 利用可能な拡張機能を一覧表示
az extension list-available --output table

# インストール済み拡張機能を一覧表示
az extension list --output table
```

### Azure DevOps 拡張機能を管理

```bash
# Azure DevOps 拡張機能をインストール
az extension add --name azure-devops

# Azure DevOps 拡張機能を更新
az extension update --name azure-devops

# 拡張機能を削除
az extension remove --name azure-devops

# ローカル パスからインストール
az extension add --source ~/extensions/azure-devops.whl
```

## サービス エンドポイント

### サービス エンドポイントを一覧表示

```bash
az devops service-endpoint list --project {project}
az devops service-endpoint list --project {project} --output table
```

### サービス エンドポイントを表示

```bash
az devops service-endpoint show --id {endpoint-id} --project {project}
```

### サービス エンドポイントを作成

```bash
# 構成ファイルを使用
az devops service-endpoint create --service-endpoint-configuration endpoint.json --project {project}
```

### サービス エンドポイントを削除

```bash
az devops service-endpoint delete --id {endpoint-id} --project {project} --yes
```

## チーム

### チームを一覧表示

```bash
az devops team list --project {project}
```

### チームを表示

```bash
az devops team show --team {team-name} --project {project}
```

### チームを作成

```bash
az devops team create \
  --name {team-name} \
  --description "Team description" \
  --project {project}
```

### チームを更新

```bash
az devops team update \
  --team {team-name} \
  --project {project} \
  --name "{new-team-name}" \
  --description "Updated description"
```

### チームを削除

```bash
az devops team delete --team {team-name} --project {project} --yes
```

### チーム メンバーを表示

```bash
az devops team list-member --team {team-name} --project {project}
```

## ユーザー

### ユーザーを一覧表示

```bash
az devops user list --org https://dev.azure.com/{org}
az devops user list --top 10 --output table
```

### ユーザーを表示

```bash
az devops user show --user {user-id-or-email} --org https://dev.azure.com/{org}
```

### ユーザーを追加

```bash
az devops user add \
  --email user@example.com \
  --license-type express \
  --org https://dev.azure.com/{org}
```

### ユーザーを更新

```bash
az devops user update \
  --user {user-id-or-email} \
  --license-type advanced \
  --org https://dev.azure.com/{org}
```

### ユーザーを削除

```bash
az devops user remove --user {user-id-or-email} --org https://dev.azure.com/{org} --yes
```

## セキュリティ グループ

### グループを一覧表示

```bash
# プロジェクト内のすべてのグループを一覧表示
az devops security group list --project {project}

# 組織内のすべてのグループを一覧表示
az devops security group list --scope organization

# フィルター付きで一覧表示
az devops security group list --project {project} --subject-types vstsgroup
```

### グループ詳細を表示

```bash
az devops security group show --group-id {group-id}
```

### グループを作成

```bash
az devops security group create \
  --name {group-name} \
  --description "Group description" \
  --project {project}
```

### グループを更新

```bash
az devops security group update \
  --group-id {group-id} \
  --name "{new-group-name}" \
  --description "Updated description"
```

### グループを削除

```bash
az devops security group delete --group-id {group-id} --yes
```

### グループ メンバーシップ

```bash
# メンバーシップを一覧表示
az devops security group membership list --id {group-id}

# メンバーを追加
az devops security group membership add \
  --group-id {group-id} \
  --member-id {member-id}

# メンバーを削除
az devops security group membership remove \
  --group-id {group-id} \
  --member-id {member-id} --yes
```

## セキュリティ権限

### 名前空間を一覧表示

```bash
az devops security permission namespace list
```

### 名前空間の詳細を表示

```bash
# 名前空間で利用可能な権限を表示
az devops security permission namespace show --namespace "GitRepositories"
```

### 権限を一覧表示

```bash
# ユーザー/グループと名前空間の権限を一覧表示
az devops security permission list \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project}

# 特定トークン（リポジトリ）の権限を一覧表示
az devops security permission list \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}"
```

### 権限を表示

```bash
az devops security permission show \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}"
```

### 権限を更新

```bash
# 権限を許可
az devops security permission update \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}" \
  --permission-mask "Pull,Contribute"

# 権限を拒否
az devops security permission update \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}" \
  --permission-mask 0
```

### 権限をリセット

```bash
# 特定の権限ビットをリセット
az devops security permission reset \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}" \
  --permission-mask "Pull,Contribute"

# すべての権限をリセット
az devops security permission reset-all \
  --id {user-or-group-id} \
  --namespace "GitRepositories" \
  --project {project} \
  --token "repoV2/{project}/{repository-id}" --yes
```

## Wiki

### Wiki を一覧表示

```bash
# プロジェクト内のすべての Wiki を一覧表示
az devops wiki list --project {project}

# 組織内のすべての Wiki を一覧表示
az devops wiki list
```

### Wiki を表示

```bash
az devops wiki show --wiki {wiki-name} --project {project}
az devops wiki show --wiki {wiki-name} --project {project} --open
```

### Wiki を作成

```bash
# プロジェクト Wiki を作成
az devops wiki create \
  --name {wiki-name} \
  --project {project} \
  --type projectWiki

# リポジトリからコード Wiki を作成
az devops wiki create \
  --name {wiki-name} \
  --project {project} \
  --type codeWiki \
  --repository {repo-name} \
  --mapped-path /wiki
```

### Wiki を削除

```bash
az devops wiki delete --wiki {wiki-id} --project {project} --yes
```

### Wiki ページ

```bash
# ページを一覧表示
az devops wiki page list --wiki {wiki-name} --project {project}

# ページを表示
az devops wiki page show \
  --wiki {wiki-name} \
  --path "/page-name" \
  --project {project}

# ページを作成
az devops wiki page create \
  --wiki {wiki-name} \
  --path "/new-page" \
  --content "# New Page\n\nPage content here..." \
  --project {project}

# ページを更新
az devops wiki page update \
  --wiki {wiki-name} \
  --path "/existing-page" \
  --content "# Updated Page\n\nNew content..." \
  --project {project}

# ページを削除
az devops wiki page delete \
  --wiki {wiki-name} \
  --path "/old-page" \
  --project {project} --yes
```

## 管理

### バナー管理

```bash
# バナーを一覧表示
az devops admin banner list

# バナー詳細を表示
az devops admin banner show --id {banner-id}

# 新しいバナーを追加
az devops admin banner add \
  --message "System maintenance scheduled" \
  --level info  # info, warning, error

# バナーを更新
az devops admin banner update \
  --id {banner-id} \
  --message "Updated message" \
  --level warning \
  --expiration-date "2025-12-31T23:59:59Z"

# バナーを削除
az devops admin banner remove --id {banner-id}
```

## DevOps 拡張機能

Azure DevOps 組織にインストールされた拡張機能を管理します（CLI 拡張機能とは異なります）。

```bash
# インストール済み拡張機能を一覧表示
az devops extension list --org https://dev.azure.com/{org}

# Marketplace 拡張機能を検索
az devops extension search --search-query "docker"

# 拡張機能詳細を表示
az devops extension show --ext-id {extension-id} --org https://dev.azure.com/{org}

# 拡張機能をインストール
az devops extension install \
  --ext-id {extension-id} \
  --org https://dev.azure.com/{org} \
  --publisher {publisher-id}

# 拡張機能を有効化
az devops extension enable \
  --ext-id {extension-id} \
  --org https://dev.azure.com/{org}

# 拡張機能を無効化
az devops extension disable \
  --ext-id {extension-id} \
  --org https://dev.azure.com/{org}

# 拡張機能をアンインストール
az devops extension uninstall \
  --ext-id {extension-id} \
  --org https://dev.azure.com/{org} --yes
```

