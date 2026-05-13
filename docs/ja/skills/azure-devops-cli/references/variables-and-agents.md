# Pipeline 変数、変数グループ、エージェント

## 目次
- [Pipeline 変数](#pipeline-variables)
- [変数グループ](#variable-groups)
- [Pipeline フォルダー](#pipeline-folders)
- [エージェント プール](#agent-pools)
- [エージェント キュー](#agent-queues)
- [エージェント](#agents)

---

## Pipeline 変数

### 変数を一覧表示

```bash
az pipelines variable list --pipeline-id {pipeline-id}
```

### 変数を作成

```bash
# 非シークレット変数
az pipelines variable create \
  --name {var-name} \
  --value {var-value} \
  --pipeline-id {pipeline-id}

# シークレット変数
az pipelines variable create \
  --name {var-name} \
  --secret true \
  --pipeline-id {pipeline-id}

# プロンプト付きシークレット
az pipelines variable create \
  --name {var-name} \
  --secret true \
  --prompt true \
  --pipeline-id {pipeline-id}
```

### 変数を更新

```bash
az pipelines variable update \
  --name {var-name} \
  --value {new-value} \
  --pipeline-id {pipeline-id}

# シークレット変数を更新
az pipelines variable update \
  --name {var-name} \
  --secret true \
  --value "{new-secret-value}" \
  --pipeline-id {pipeline-id}
```

### 変数を削除

```bash
az pipelines variable delete --name {var-name} --pipeline-id {pipeline-id} --yes
```

## 変数グループ

### 変数グループを一覧表示

```bash
az pipelines variable-group list
az pipelines variable-group list --output table
```

### 変数グループを表示

```bash
az pipelines variable-group show --id {group-id}
```

### 変数グループを作成

```bash
az pipelines variable-group create \
  --name {group-name} \
  --variables key1=value1 key2=value2 \
  --authorize true
```

### 変数グループを更新

```bash
az pipelines variable-group update \
  --id {group-id} \
  --name {new-name} \
  --description "更新済みの説明"
```

### 変数グループを削除

```bash
az pipelines variable-group delete --id {group-id} --yes
```

### 変数グループ内の変数

```bash
# 変数を一覧表示
az pipelines variable-group variable list --group-id {group-id}

# 非シークレット変数を作成
az pipelines variable-group variable create \
  --group-id {group-id} \
  --name {var-name} \
  --value {var-value}

# シークレット変数を作成（値を指定しない場合は入力を求められます）
az pipelines variable-group variable create \
  --group-id {group-id} \
  --name {var-name} \
  --secret true

# 環境変数を使ってシークレットを作成
export AZURE_DEVOPS_EXT_PIPELINE_VAR_MySecret=secretvalue
az pipelines variable-group variable create \
  --group-id {group-id} \
  --name MySecret \
  --secret true

# 変数を更新
az pipelines variable-group variable update \
  --group-id {group-id} \
  --name {var-name} \
  --value {new-value} \
  --secret false

# 変数を削除
az pipelines variable-group variable delete \
  --group-id {group-id} \
  --name {var-name}
```

## Pipeline フォルダー

### フォルダーを一覧表示

```bash
az pipelines folder list
```

### フォルダーを作成

```bash
az pipelines folder create --path 'folder/subfolder' --description "My folder"
```

### フォルダーを削除

```bash
az pipelines folder delete --path 'folder/subfolder'
```

### フォルダーを更新

```bash
az pipelines folder update --path 'old-folder' --new-path 'new-folder'
```

## エージェント プール

### エージェント プールを一覧表示

```bash
az pipelines pool list
az pipelines pool list --pool-type automation
az pipelines pool list --pool-type deployment
```

### エージェント プールを表示

```bash
az pipelines pool show --pool-id {pool-id}
```

## エージェント キュー

### エージェント キューを一覧表示

```bash
az pipelines queue list
az pipelines queue list --pool-name {pool-name}
```

### エージェント キューを表示

```bash
az pipelines queue show --id {queue-id}
```

## エージェント

### プール内のエージェントを一覧表示

```bash
az pipelines agent list --pool-id {pool-id}
```

### エージェントの詳細を表示

```bash
az pipelines agent show --agent-id {agent-id} --pool-id {pool-id}
```

