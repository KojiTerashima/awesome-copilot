# 高度な Issue 検索

`search_issues` MCP ツールは、クロスリポジトリ検索のために GitHub の Issue 検索クエリ形式を使用し、暗黙 AND クエリ、日付範囲、メタデータフィルターをサポートします（ただし明示的な OR/NOT 演算子は不可）。

## Search と List と Advanced Search の使い分け

Issue を見つける方法は 3 つあり、それぞれ機能が異なります。

| 機能 | `list_issues` (MCP) | `search_issues` (MCP) | Advanced search (`gh api`) |
|-----------|---------------------|----------------------|---------------------------|
| **対象範囲** | 単一リポジトリのみ | クロスリポジトリ・クロス組織 | クロスリポジトリ・クロス組織 |
| **Issue フィールドフィルター** (`field.priority:P0`) | いいえ | いいえ | **はい**（ドット記法） |
| **Issue タイプフィルター** (`type:Bug`) | いいえ | はい | はい |
| **ブール論理** (AND/OR/NOT、ネスト) | いいえ | はい（暗黙 AND のみ） | **はい**（明示 AND/OR/NOT） |
| **ラベル/状態/日付フィルター** | はい | はい | はい |
| **担当者/作成者/メンション** | いいえ | はい | はい |
| **否定** (`-label:x`, `no:label`) | いいえ | はい | はい |
| **テキスト検索** (title/body/comments) | いいえ | はい | はい |
| **`since` フィルター** | はい | いいえ | いいえ |
| **結果上限** | 上限なし（全ページ取得） | 最大 1,000 件 | 最大 1,000 件 |
| **呼び出し方法** | MCP ツールを直接呼び出す | MCP ツールを直接呼び出す | `advanced_search=true` を付けて `gh api` |

**判断ガイド:**
- **単一リポジトリでシンプルなフィルター（state、labels、最近の更新）:** `list_issues` を使う
- **クロスリポジトリ、テキスト検索、author/assignee、issue type:** `search_issues` を使う
- **Issue フィールド値（Priority、日付、カスタムフィールド）や複雑なブール論理:** `advanced_search=true` を付けた `gh api` を使う

## クエリ構文

`query` パラメータは検索語と修飾子の文字列です。語句の間のスペースは暗黙 AND として扱われます。

### スコープ指定

```
repo:owner/repo       # 単一リポジトリ（owner+repo params を渡すと自動追加）
org:github            # 組織内の全リポジトリ
user:octocat          # ユーザー所有の全リポジトリ
in:title              # タイトルのみ検索
in:body               # 本文のみ検索
in:comments           # コメントのみ検索
```

### 状態とクローズ理由

```
is:open               # オープンな Issue（自動追加: is:issue）
is:closed             # クローズ済み Issue
reason:completed      # 完了としてクローズ
reason:"not planned"  # 計画なしとしてクローズ
```

### ユーザー関連

```
author:username       # 作成者
assignee:username     # 担当者
mentions:username     # ユーザーへのメンションあり
commenter:username    # そのユーザーのコメントあり
involves:username     # author OR assignee OR mentioned OR commenter
author:@me            # 現在認証中のユーザー
team:org/team         # チームへのメンション
```

### ラベル、マイルストーン、プロジェクト、タイプ

```
label:"bug"                 # ラベルあり（複数語ラベルは引用符で囲む）
label:bug label:priority    # 両方のラベルを持つ（AND）
label:bug,enhancement       # いずれかのラベルを持つ（OR）
-label:wontfix              # ラベルを持たない
milestone:"v2.0"            # マイルストーン内
project:github/57           # プロジェクトボード内
type:"Bug"                  # Issue タイプ
```

### メタデータ欠落

```
no:label              # ラベル未設定
no:milestone          # マイルストーン未設定
no:assignee           # 担当者未設定
no:project            # どのプロジェクトにも未所属
```

### 日付

すべての日付修飾子は、ISO 8601 形式で `>`, `<`, `>=`, `<=`, 範囲（`..`）演算子をサポートします。

```
created:>2026-01-01              # 1月1日以降に作成
updated:>=2026-03-01             # 3月1日以降に更新
closed:2026-01-01..2026-02-01   # 1月中にクローズ
created:<2026-01-01              # 1月1日より前に作成
```

### リンク済みコンテンツ

```
linked:pr             # Issue にリンクされた PR がある
-linked:pr            # まだどの PR にもリンクされていない Issue
linked:issue          # PR が Issue にリンクされている
```

### 数値フィルター

```
comments:>10          # コメントが 10 件超
comments:0            # コメントなし
interactions:>100     # リアクション + コメント > 100
reactions:>50         # リアクションが 50 件超
```

### ブール論理とネスト

`AND`、`OR`、括弧を使用できます（最大 5 階層、演算子は最大 5 個）。

```
label:bug AND assignee:octocat
assignee:octocat OR assignee:hubot
(type:"Bug" AND label:P1) OR (type:"Feature" AND label:P1)
-author:app/dependabot          # bot の Issue を除外
```

明示的な演算子なしで語句を空白区切りにした場合は AND として扱われます。

## よく使うクエリパターン

**担当者未設定のバグ:**
```
repo:owner/repo type:"Bug" no:assignee is:open
```

**今週クローズされた Issue:**
```
repo:owner/repo is:closed closed:>=2026-03-01
```

**古いオープン Issue（90 日更新なし）:**
```
repo:owner/repo is:open updated:<2026-01-01
```

**リンクされた PR がないオープン Issue（作業要）:**
```
repo:owner/repo is:open -linked:pr
```

**組織全体で自分が関与している Issue:**
```
org:github involves:@me is:open
```

**アクティビティの高い Issue:**
```
repo:owner/repo is:open comments:>20
```

**タイプと優先度ラベルで絞った Issue:**
```
repo:owner/repo type:"Epic" label:P1 is:open
```

## Issue フィールド検索

> **信頼性の警告:** `field.name:value` 検索修飾子の構文は実験的で、一致する Issue が存在していても 0 件返る場合があります。フィールド値で信頼性高く絞り込むには、[issue-fields.md](issue-fields.md#searching-by-field-values) に記載の GraphQL 一括クエリアプローチを使用してください。

Issue フィールドは理論上、**advanced search mode** で `field.name:value` 修飾子を使って検索できます。Web UI では動作しますが、API の結果は一貫しません。

### REST API

クエリパラメータとして `advanced_search=true` を追加します。

```bash
gh api "search/issues?q=org:github+field.priority:P0+type:Epic+is:open&advanced_search=true" \
  --jq '.items[] | "#\(.number): \(.title)"'
```

### GraphQL

`type: ISSUE` ではなく `type: ISSUE_ADVANCED` を使用します。

```graphql
{
  search(query: "org:github field.priority:P0 type:Epic is:open", type: ISSUE_ADVANCED, first: 10) {
    issueCount
    nodes {
      ... on Issue { number title }
    }
  }
}
```

### Issue フィールド修飾子

構文は、フィールドのスラッグ名（小文字、空白はハイフン）を使う**ドット記法**です。

```
field.priority:P0                  # 単一選択フィールドが指定値と一致
field.priority:P1                  # 別の選択肢の値
field.target-date:>=2026-04-01     # 日付比較
has:field.priority                 # 何らかの値が設定されている
no:field.priority                  # 値が設定されていない
```

**MCP の制限:** `search_issues` MCP ツールは `advanced_search=true` を渡しません。Issue フィールド検索には `gh api` を直接使用する必要があります。

### よく使うフィールド検索パターン

**組織全体の P0 エピック:**
```
org:github field.priority:P0 type:Epic is:open
```

**今四半期に target date がある Issue:**
```
org:github field.target-date:>=2026-04-01 field.target-date:<=2026-06-30 is:open
```

**優先度未設定のオープンバグ:**
```
org:github no:field.priority type:Bug is:open
```

## 制限事項

- クエリテキスト: 最大 **256 文字**（演算子/修飾子を除く）
- ブール演算子: 1 クエリあたり AND/OR/NOT 合計 **5 個**まで
- 結果: 合計最大 **1,000 件**（全 Issue が必要なら `list_issues` を使用）
- リポジトリ走査: 一致する最大 **4,000** リポジトリまで検索
- レート制限: 認証済み検索で **30 リクエスト/分**
- Issue フィールド検索には `advanced_search=true`（REST）または `ISSUE_ADVANCED`（GraphQL）が必要。MCP `search_issues` では利用不可

