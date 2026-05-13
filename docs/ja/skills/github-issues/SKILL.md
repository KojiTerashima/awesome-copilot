---
name: github-issues
description: 'MCP ツールを使って GitHub issue を作成・更新・管理します。ユーザーがバグ報告、機能要望、タスク issue の作成、既存 issue の更新、ラベル/担当者/マイルストーンの追加、issue フィールド（日付、優先度、カスタムフィールド）の設定、issue タイプの設定、issue ワークフロー管理、issue のリンク、依存関係の追加、blocked-by/blocking 関係の追跡を行いたい場合にこのスキルを使用します。「create an issue」「file a bug」「request a feature」「update issue X」「set the priority」「set the start date」「link issues」「add dependency」「blocked by」「blocking」など、GitHub issue 管理に関するあらゆる依頼でトリガーされます。'
---

# GitHub Issues

`@modelcontextprotocol/server-github` MCP サーバーを使って GitHub issue を管理します。

## 利用可能なツール

### MCP ツール（読み取り操作）

| Tool | Purpose |
|------|---------|
| `mcp__github__issue_read` | issue の詳細、サブ issue、コメント、ラベルを取得（メソッド: get, get_comments, get_sub_issues, get_labels） |
| `mcp__github__list_issues` | 状態、ラベル、日付でリポジトリ issue を一覧・フィルタ |
| `mcp__github__search_issues` | GitHub の検索構文でリポジトリ横断の issue 検索 |
| `mcp__github__projects_list` | プロジェクト、プロジェクトフィールド、プロジェクトアイテム、ステータス更新を一覧 |
| `mcp__github__projects_get` | プロジェクト、フィールド、アイテム、またはステータス更新の詳細取得 |
| `mcp__github__projects_write` | プロジェクトアイテムの追加/更新/削除、ステータス更新の作成 |

### CLI / REST API（書き込み操作）

MCP サーバーは現在、issue の作成・更新・コメント追加をサポートしていません。これらの操作には `gh api` を使用してください。

| Operation | Command |
|-----------|---------|
| issue 作成 | `gh api repos/{owner}/{repo}/issues -X POST -f title=... -f body=...` |
| issue 更新 | `gh api repos/{owner}/{repo}/issues/{number} -X PATCH -f title=... -f state=...` |
| コメント追加 | `gh api repos/{owner}/{repo}/issues/{number}/comments -X POST -f body=...` |
| issue クローズ | `gh api repos/{owner}/{repo}/issues/{number} -X PATCH -f state=closed` |
| issue タイプ設定 | 作成時の呼び出しに `-f type=Bug` を含める（REST API のみ。`gh issue create` CLI では未対応） |

**Note:** `gh issue create` は基本的な issue 作成には使えますが、`--type` フラグは **サポートしていません**。issue タイプを設定する必要がある場合は `gh api` を使ってください。

## ワークフロー

1. **Determine action**: 作成、更新、または照会のどれか？
2. **Gather context**: 必要に応じてリポジトリ情報、既存ラベル、マイルストーンを取得
3. **Structure content**: [references/templates.md](references/templates.md) の適切なテンプレートを使用
4. **Execute**: 読み取りは MCP ツール、書き込みは `gh api` を使用
5. **Confirm**: issue URL をユーザーに報告

## issue の作成

issue の作成には `gh api` を使用します。これにより issue タイプを含むすべてのパラメータを指定できます。

```bash
gh api repos/{owner}/{repo}/issues \
  -X POST \
  -f title="Issue title" \
  -f body="Issue body in markdown" \
  -f type="Bug" \
  --jq '{number, html_url}'
```

### オプションパラメータ

`gh api` 呼び出しに以下のフラグを追加できます:

```
-f type="Bug"                    # Issue type (Bug, Feature, Task, Epic, etc.)
-f labels[]="bug"                # Labels (repeat for multiple)
-f assignees[]="username"        # Assignees (repeat for multiple)
-f milestone=1                   # Milestone number
```

**Issue types** は組織レベルのメタデータです。利用可能なタイプを確認するには以下を使います:
```bash
gh api graphql -f query='{ organization(login: "ORG") { issueTypes(first: 10) { nodes { name } } } }' --jq '.data.organization.issueTypes.nodes[].name'
```

**分類にはラベルより issue type を優先してください。** issue type が利用可能な場合（例: Bug, Feature, Task）は、`bug` や `enhancement` などの同等ラベルではなく `type` パラメータを使ってください。GitHub で issue を分類する正規の方法は issue type です。組織で issue type が設定されていない場合のみラベルにフォールバックしてください。

### タイトルのガイドライン

- 具体的で実行可能な内容にする
- 72 文字以内に収める
- issue type を設定する場合、`[Bug]` のような冗長な接頭辞は付けない
- 例:
  - `Login fails with SSO enabled`（type=Bug の場合）
  - `Add dark mode support`（type=Feature の場合）
  - `Add unit tests for auth module`（type=Task の場合）

### 本文構成

常に [references/templates.md](references/templates.md) のテンプレートを使用してください。issue type に応じて選択します:

| User Request | Template |
|--------------|----------|
| Bug, error, broken, not working | Bug Report |
| Feature, enhancement, add, new | Feature Request |
| Task, chore, refactor, update | Task |

## issue の更新

PATCH で `gh api` を使用します:

```bash
gh api repos/{owner}/{repo}/issues/{number} \
  -X PATCH \
  -f state=closed \
  -f title="Updated title" \
  --jq '{number, html_url}'
```

変更したいフィールドだけを含めてください。利用可能なフィールド: `title`, `body`, `state` (open/closed), `labels`, `assignees`, `milestone`。

## 例

### 例 1: バグ報告

**User**: "Create a bug issue - the login page crashes when using SSO"

**Action**: 
```bash
gh api repos/github/awesome-copilot/issues \
  -X POST \
  -f title="Login page crashes when using SSO" \
  -f type="Bug" \
  -f body="## Description
ユーザーが SSO を使って認証しようとすると、ログインページがクラッシュします。

## Steps to Reproduce
1. ログインページに移動する
2. 「Sign in with SSO」をクリックする
3. ページがクラッシュする

## Expected Behavior
SSO 認証が完了し、ダッシュボードにリダイレクトされること。

## Actual Behavior
ページが応答しなくなり、エラーが表示される。" \
  --jq '{number, html_url}'
```

### 例 2: 機能要望

**User**: "Create a feature request for dark mode with high priority"

**Action**:
```bash
gh api repos/github/awesome-copilot/issues \
  -X POST \
  -f title="Add dark mode support" \
  -f type="Feature" \
  -f labels[]="high-priority" \
  -f body="## Summary
ユーザー体験とアクセシビリティ向上のため、ダークモードのテーマオプションを追加する。

## Motivation
- 暗い環境での目の負担を軽減できる
- 多くのユーザーが期待している機能である

## Proposed Solution
システム設定の検出を含むテーマ切り替え機能を実装する。

## Acceptance Criteria
- [ ] 設定画面にトグルスイッチがある
- [ ] ユーザー設定が保持される
- [ ] デフォルトでシステム設定を尊重する" \
  --jq '{number, html_url}'
```

## よく使うラベル

該当する場合は以下の標準ラベルを使用します:

| Label | Use For |
|-------|---------|
| `bug` | 動作していないもの |
| `enhancement` | 新機能または改善 |
| `documentation` | ドキュメント更新 |
| `good first issue` | 初学者向け |
| `help wanted` | 追加の対応が必要 |
| `question` | 追加情報が必要 |
| `wontfix` | 対応しない予定 |
| `duplicate` | 既存 issue と重複 |
| `high-priority` | 緊急度の高い issue |

## ヒント

- issue 作成前に必ずリポジトリのコンテキストを確認する
- 重要な情報が不足している場合は推測せず確認する
- 関連 issue が分かっている場合はリンクする: `Related to #123`
- 更新時は、変更しないフィールドを保持するため先に現在の issue を取得する

## 拡張機能

以下の機能は、基本的な MCP ツールを超える REST または GraphQL API が必要です。エージェントが必要な知識だけを読み込めるよう、それぞれ専用の参照ファイルに記載されています。

| Capability | When to use | Reference |
|------------|-------------|-----------|
| 高度な検索 | ブールロジック、日付範囲、リポジトリ横断検索、issue フィールドフィルタ（`field.name:value`）を含む複雑なクエリ | [references/search.md](references/search.md) |
| サブ issue と親 issue | 作業を階層的なタスクに分割する場合 | [references/sub-issues.md](references/sub-issues.md) |
| issue 依存関係 | blocked-by / blocking 関係を追跡する場合 | [references/dependencies.md](references/dependencies.md) |
| issue タイプ（高度） | MCP の `list_issue_types` / `type` パラメータを超える GraphQL 操作 | [references/issue-types.md](references/issue-types.md) |
| Projects V2 | プロジェクトボード、進捗レポート、フィールド管理 | [references/projects.md](references/projects.md) |
| issue フィールド | カスタムメタデータ: 日付、優先度、テキスト、数値（プライベートプレビュー） | [references/issue-fields.md](references/issue-fields.md) |
| issue 内の画像 | CLI 経由で issue 本文やコメントに画像を埋め込む | [references/images.md](references/images.md) |

