# Labels APIリファレンス

ラベル移行フローで使用されるGitHub Labels REST APIエンドポイントのリファレンス。

## リポジトリ内のラベル一覧取得

```
GET /repos/{owner}/{repo}/labels
```

リポジトリに定義されているすべてのラベルを返します。ページネーション対応（1ページ最大100件）。

**CLIショートカット:**

```bash
gh label list -R {owner}/{repo} --limit 1000 --json name,color,description
```

**レスポンスフィールド:** `id`, `node_id`, `url`, `name`, `description`, `color`, `default`

## ラベルで絞ったイシュー一覧取得

```
GET /repos/{owner}/{repo}/issues?labels={label_name}&state=all&per_page=100
```

指定したラベルに一致するイシュー（およびプルリクエスト）を返します。プルリクエストは`pull_request`フィールドが存在しないことで除外可能です。

**CLIショートカット:**

```bash
gh issue list -R {owner}/{repo} --label "{label_name}" --state all \
  --json number,title,labels --limit 1000
```

`gh issue list`コマンドは自動的にプルリクエストを除外します。

**ページネーション:** CLIでは`--limit`、REST APIでは`page`クエリパラメータを使用。マッチするイシューが1000件を超えるリポジトリでは、Linkヘッダーによるカーソルベースのページネーションを利用してください。

## イシューからラベルを削除

```
DELETE /repos/{owner}/{repo}/issues/{issue_number}/labels/{label_name}
```

イシューから単一のラベルを削除します。削除後のイシューのラベル一覧を含む`200 OK`を返します。

**重要:** スペースや特殊文字を含むラベル名はURLエンコードしてください：
- `good first issue` → `good%20first%20issue`
- `bug/critical` → `bug%2Fcritical`

**CLIショートカット:**

```bash
gh api /repos/{owner}/{repo}/issues/{number}/labels/{label_name} -X DELETE
```

## イシューにラベルを追加

```
POST /repos/{owner}/{repo}/issues/{issue_number}/labels
```

ボディ: `{"labels": ["label1", "label2"]}`

通常の移行ではあまり使いませんが、ロールバック時などに便利です。

## 注意事項

- ラベルはリポジトリ単位で管理されます。同じラベル名でも別リポジトリで独立して存在可能です。
- リポジトリのラベル一覧を取得するMCPツールはありません。`gh label list`またはREST APIを使用してください。
- MCPツール`mcp__github__list_issues`はラベルでのフィルターをサポートしており、ラベルでイシューを取得可能です。
- ラベル名のマッチングは大文字小文字を区別しませんが、APIは元の大文字小文字を保持します。
- イシューあたりのラベル数に厳密な上限はありませんが、実際には数十個程度です。
