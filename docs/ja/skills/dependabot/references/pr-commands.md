# Dependabot PR Comment Commands

`@dependabot <command>` とコメントして Dependabot pull request を操作できます。Dependabot はコマンドを受け取ると親指アップのリアクションで応答します。

> **Deprecation Notice (January 27, 2026):** 次のコマンドは削除されました。
> `@dependabot merge`、`@dependabot squash and merge`、`@dependabot cancel merge`、
> `@dependabot close`、`@dependabot reopen`。
> 代わりに GitHub のネイティブ UI、CLI (`gh pr merge`)、API、または auto-merge 機能を使ってください。

## Commands for Individual PRs

| Command | Description |
|---|---|
| `@dependabot rebase` | 対象ブランチに対して PR を rebase する |
| `@dependabot recreate` | 手動編集を上書きして PR を最初から再作成する |
| `@dependabot ignore this dependency` | PR を閉じ、この dependency に対する今後の更新をすべて停止する |
| `@dependabot ignore this major version` | この major version の更新を停止する |
| `@dependabot ignore this minor version` | この minor version の更新を停止する |
| `@dependabot ignore this patch version` | この patch version の更新を停止する |
| `@dependabot show DEPENDENCY_NAME ignore conditions` | その dependency に対して現在設定されている ignore 条件を一覧表示する |

## Commands for Grouped Updates

これらのコマンドは、grouped version update または grouped security update によって作られた Dependabot PR で使えます。

| Command | Description |
|---|---|
| `@dependabot ignore DEPENDENCY_NAME` | PR を閉じ、group 内でこの dependency の更新を停止する |
| `@dependabot ignore DEPENDENCY_NAME major version` | この dependency の major version 更新を停止する |
| `@dependabot ignore DEPENDENCY_NAME minor version` | この dependency の minor version 更新を停止する |
| `@dependabot ignore DEPENDENCY_NAME patch version` | この dependency の patch version 更新を停止する |
| `@dependabot unignore *` | 現在の PR を閉じ、group 内のすべての dependency に対するすべての ignore 条件を解除し、新しい PR を開く |
| `@dependabot unignore DEPENDENCY_NAME` | 現在の PR を閉じ、特定 dependency に対するすべての ignore を解除し、その更新を含む新しい PR を開く |
| `@dependabot unignore DEPENDENCY_NAME IGNORE_CONDITION` | 現在の PR を閉じ、特定の ignore 条件だけを解除し、新しい PR を開く |

## Usage Examples

### Merge After CI (Use Native GitHub Features)

auto-merge は、廃止された `@dependabot merge` コマンドの推奨代替です。

```bash
# GitHub CLI で auto-merge を有効化
gh pr merge <PR_NUMBER> --auto --squash

# または GitHub UI で auto-merge を有効化:
# PR → "Enable auto-merge" → merge method を選択 → confirm
```

必要な CI チェックがすべて通過すると、GitHub が自動的に PR をマージします。

### Ignore a Major Version Bump

```
@dependabot ignore this major version
```

major version に破壊的変更があり、まだ移行予定がない場合に有効です。

### Check Active Ignore Conditions

```
@dependabot show express ignore conditions
```

`express` dependency に現在保存されている ignore 条件を表形式で表示します。

### Unignore a Dependency in a Group

```
@dependabot unignore lodash
```

現在の grouped PR を閉じ、`lodash` に対するすべての ignore 条件を解除し、利用可能な `lodash` 更新を含む新しい PR を開きます。

### Unignore a Specific Condition

```
@dependabot unignore express [< 1.9, > 1.8.0]
```

`express` に対して指定された version range の ignore だけを解除します。

## Tips

- **Rebase vs Recreate**: レビュー状態を保ったまま競合を解消したいなら `rebase` を使います。PR の差分が大きく乖離しているなら `recreate` で作り直します。
- **Force push over extra commits**: Dependabot branch に commit を積んだ上で Dependabot に再 rebase させたい場合は、commit message に `[dependabot skip]` を含めてください。
- **Persistent ignores**: PR コメントによる ignore コマンドは中央管理されます。チーム リポジトリで透明性を高めるには、`dependabot.yml` の `ignore` を優先してください。
- **Merging Dependabot PRs**: GitHub のネイティブ auto-merge、CLI (`gh pr merge`)、または Web UI を使ってください。旧 `@dependabot merge` コマンドは 2026 年 1 月に廃止されました。
- **Closing/Reopening**: GitHub UI または CLI を使ってください。旧 `@dependabot close` と `@dependabot reopen` は 2026 年 1 月に廃止されました。
- **Grouped commands**: `@dependabot unignore` を使うと、Dependabot は現在の PR を閉じ、更新された dependency set を持つ新しい PR を開きます。
