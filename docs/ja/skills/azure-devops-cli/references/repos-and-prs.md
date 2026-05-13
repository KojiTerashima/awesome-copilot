# リポジトリ & プルリクエスト

## 目次
- [リポジトリ](#リポジトリ)
- [リポジトリのインポート](#リポジトリのインポート)
- [プルリクエスト](#プルリクエスト)
- [Git リファレンス](#git-リファレンス)
- [リポジトリ ポリシー](#リポジトリ-ポリシー)

---

## リポジトリ

### リポジトリを一覧表示

```bash
az repos list --org https://dev.azure.com/{org} --project {project}
az repos list --output table
```

### リポジトリの詳細を表示

```bash
az repos show --repository {repo-name} --project {project}
```

### リポジトリを作成

```bash
az repos create --name {repo-name} --project {project}
```

### リポジトリを削除

```bash
az repos delete --id {repo-id} --project {project} --yes
```

### リポジトリを更新

```bash
az repos update --id {repo-id} --name {new-name} --project {project}
```

## リポジトリのインポート

### Git リポジトリをインポート

```bash
# 公開 Git リポジトリからインポート
az repos import create \
  --git-source-url https://github.com/user/repo \
  --repository {repo-name}

# 認証付きでインポート
az repos import create \
  --git-source-url https://github.com/user/private-repo \
  --repository {repo-name} \
  --user {username} \
  --password {password-or-pat}
```

## プルリクエスト

### プルリクエストを作成

```bash
# 基本的な PR 作成
az repos pr create \
  --repository {repo} \
  --source-branch {source-branch} \
  --target-branch {target-branch} \
  --title "PR タイトル" \
  --description "PR の説明" \
  --open

# 作業項目付き PR
az repos pr create \
  --repository {repo} \
  --source-branch {source-branch} \
  --work-items 63 64

# レビュアー付きドラフト PR
az repos pr create \
  --repository {repo} \
  --source-branch feature/new-feature \
  --target-branch main \
  --title "機能: 新しい機能" \
  --draft true \
  --reviewers user1@example.com user2@example.com \
  --required-reviewers lead@example.com \
  --labels "enhancement" "backlog"
```

### プルリクエストを一覧表示

```bash
# すべての PR
az repos pr list --repository {repo}

# ステータスで絞り込み
az repos pr list --repository {repo} --status active

# 作成者で絞り込み
az repos pr list --repository {repo} --creator {email}

# テーブル形式で出力
az repos pr list --repository {repo} --output table
```

### PR の詳細を表示

```bash
az repos pr show --id {pr-id}
az repos pr show --id {pr-id} --open  # ブラウザーで開く
```

### PR を更新（完了/放棄/ドラフト）

```bash
# PR を完了
az repos pr update --id {pr-id} --status completed

# PR を放棄
az repos pr update --id {pr-id} --status abandoned

# ドラフトに設定
az repos pr update --id {pr-id} --draft true

# ドラフト PR を公開
az repos pr update --id {pr-id} --draft false

# ポリシー通過時に自動完了
az repos pr update --id {pr-id} --auto-complete true

# タイトルと説明を設定
az repos pr update --id {pr-id} --title "新しいタイトル" --description "新しい説明"
```

### PR をローカルでチェックアウト

```bash
# PR ブランチをチェックアウト
az repos pr checkout --id {pr-id}

# 特定のリモートでチェックアウト
az repos pr checkout --id {pr-id} --remote-name upstream
```

### PR に投票

```bash
az repos pr set-vote --id {pr-id} --vote approve
az repos pr set-vote --id {pr-id} --vote approve-with-suggestions
az repos pr set-vote --id {pr-id} --vote reject
az repos pr set-vote --id {pr-id} --vote wait-for-author
az repos pr set-vote --id {pr-id} --vote reset
```

### PR レビュアー

```bash
# レビュアーを追加
az repos pr reviewer add --id {pr-id} --reviewers user1@example.com user2@example.com

# レビュアーを一覧表示
az repos pr reviewer list --id {pr-id}

# レビュアーを削除
az repos pr reviewer remove --id {pr-id} --reviewers user1@example.com
```

### PR 作業項目

```bash
# PR に作業項目を追加
az repos pr work-item add --id {pr-id} --work-items {id1} {id2}

# PR の作業項目を一覧表示
az repos pr work-item list --id {pr-id}

# PR から作業項目を削除
az repos pr work-item remove --id {pr-id} --work-items {id1}
```

### PR ポリシー

```bash
# PR のポリシーを一覧表示
az repos pr policy list --id {pr-id}

# PR のポリシー評価をキューに追加
az repos pr policy queue --id {pr-id} --evaluation-id {evaluation-id}
```

## Git リファレンス

### リファレンス（ブランチ）を一覧表示

```bash
az repos ref list --repository {repo}
az repos ref list --repository {repo} --query "[?name=='refs/heads/main']"
```

### リファレンス（ブランチ）を作成

```bash
az repos ref create --name refs/heads/new-branch --object-type commit --object {commit-sha}
```

### リファレンス（ブランチ）を削除

```bash
az repos ref delete --name refs/heads/old-branch --repository {repo} --project {project}
```

### ブランチをロック/ロック解除

```bash
az repos ref lock --name refs/heads/main --repository {repo} --project {project}
az repos ref unlock --name refs/heads/main --repository {repo} --project {project}
```

## リポジトリ ポリシー

### すべてのポリシーを一覧表示

```bash
az repos policy list --repository {repo-id} --branch main
```

### ポリシーを作成/更新/削除

```bash
# 設定ファイルから作成
az repos policy create --config policy.json

# 更新
az repos policy update --id {policy-id} --config updated-policy.json

# 削除
az repos policy delete --id {policy-id} --yes
```

### 承認者数ポリシー

```bash
az repos policy approver-count create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id} \
  --minimum-approver-count 2 \
  --creator-vote-counts true
```

### ビルド ポリシー

```bash
az repos policy build create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id} \
  --build-definition-id {definition-id} \
  --queue-on-source-update-only true \
  --valid-duration 720
```

### 作業項目リンク ポリシー

```bash
az repos policy work-item-linking create \
  --blocking true \
  --branch main \
  --enabled true \
  --repository-id {repo-id}
```

### 必須レビュアー ポリシー

```bash
az repos policy required-reviewer create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id} \
  --required-reviewers user@example.com
```

### マージ戦略ポリシー

```bash
az repos policy merge-strategy create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id} \
  --allow-squash true \
  --allow-rebase true \
  --allow-no-fast-forward true
```

### 大文字小文字強制ポリシー

```bash
az repos policy case-enforcement create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id}
```

### コメント必須ポリシー

```bash
az repos policy comment-required create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id}
```

### ファイルサイズ ポリシー

```bash
az repos policy file-size create \
  --blocking true \
  --enabled true \
  --branch main \
  --repository-id {repo-id} \
  --maximum-file-size 10485760  # バイト単位で 10MB
```

