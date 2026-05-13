---
name: copilot-spaces
description: 'Copilot Spaces を使って会話に project 固有の context を持ち込みます。ユーザーが「Copilot space」に言及したとき、共有 knowledge base から context を読み込みたいとき、利用可能な space を見つけたいとき、または整理された project documentation、code、instruction に基づく質問をしたいときに使います。'
---

# Copilot Spaces

Copilot Spaces を使って、整理済みの project 固有 context を会話に取り込みます。Space は、repository、file、documentation、instruction を共有の形でまとめたもので、Copilot の応答を team の実際の code と knowledge に基づかせます。

## 利用可能なツール

### MCP ツール（読み取り専用）

| Tool | Purpose |
|------|---------|
| `mcp__github__list_copilot_spaces` | 現在の user がアクセスできるすべての space を一覧表示する |
| `mcp__github__get_copilot_space` | owner と name を指定して space の完全な context を読み込む |

### `gh api` による REST API（完全 CRUD）

Spaces REST API は、space の作成、更新、削除、および collaborator 管理をサポートします。MCP server が expose しているのは read 操作だけなので、write 操作には `gh api` を使います。

**User Spaces:**

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/users/{username}/copilot-spaces` | space を作成する |
| `GET` | `/users/{username}/copilot-spaces` | space を一覧表示する |
| `GET` | `/users/{username}/copilot-spaces/{number}` | space を取得する |
| `PUT` | `/users/{username}/copilot-spaces/{number}` | space を更新する |
| `DELETE` | `/users/{username}/copilot-spaces/{number}` | space を削除する |

**Organization Spaces:** 同じ pattern が `/orgs/{org}/copilot-spaces/...` 配下にあります。

**Collaborators:** `.../collaborators` で collaborator の追加、一覧、更新、削除を行えます。

**Scope requirements:** PAT には read 操作用に `read:user`、write 操作用に `user` が必要です。`gh auth refresh -h github.com -s user` で追加します。

**Note:** この API は動作していますが、まだ public REST API documentation には載っていません。`copilot_spaces_api` feature flag が必要な場合があります。

## Spaces を使うタイミング

- user が「Copilot space」に言及した、または「space を読み込んで」と頼んだとき
- 特定の project docs、code、standard に基づく回答が必要なとき
- 「どんな spaces が使える？」や「X に合う space を探して」と聞かれたとき
- onboarding context、architecture docs、team 固有 guidance が必要なとき
- Space に定義された structured workflow（template、checklist、複数 step の process）に従いたいとき

## ワークフロー

### 1. Space を見つける

利用可能な space を知りたいとき、または適切な space を探す必要があるとき:

```
Call mcp__github__list_copilot_spaces
```

これにより、user がアクセスできるすべての space が返ります。各項目には `name` と `owner_login` が含まれます。関連する候補を user に提示します。

特定 user の space だけを見たい場合は、`owner_login` を username と照合して絞り込みます（例: 「自分の spaces を見せて」）。

### 2. Space を読み込む

user が特定の space を指定した、または適切な space を特定できたとき:

```
Call mcp__github__get_copilot_space with:
  owner: "org-or-user"    (list の owner_login)
  name: "Space Name"      (space 名を正確に。大文字小文字を区別)
```

これにより、その space の full content が返ります。添付された documentation、code context、custom instruction、その他の整理済み資料が含まれます。この context を使って応答を組み立てます。

### 3. 参照先をたどる

Space の内容には、GitHub issue、dashboard、repo、discussion、その他の tool など、外部 resource への参照が含まれていることがよくあります。完全な context を得るために、他の MCP tool で能動的に取得してください。例:
- space が initiative tracking issue を参照している。`issue_read` を使って最新 comment を確認する
- space が project board へ link している。project tool を使って現在の status を確認する
- space が repo の masterplan に言及している。`get_file_contents` を使って読む

### 4. 回答または実行する

読み込み後、その space に含まれているものに応じて使い分けます。

**space に reference material**（docs、code、standard）が含まれている場合:
- project の architecture、pattern、standard に関する質問へ回答する
- team の規約に従った code を生成する
- project 固有 knowledge を使って issue を debug する

**space に workflow instruction**（template、step-by-step process）が含まれている場合:
- 定義された workflow に従って、順番に進める
- workflow が指定する source から data を集める
- workflow が定義する format で output を出す
- 各 step の後に progress を示し、user が方向修正できるようにする

### 5. Space を管理する（`gh api` 経由）

user が space の作成、更新、削除を求めた場合は `gh api` を使います。まず list endpoint から space number を確認します。

**space の instruction を更新する:**
```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f general_instructions="New instructions here"
```

**name、description、instruction をまとめて更新する:**
```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f name="Updated Name" \
  -f description="Updated description" \
  -f general_instructions="Updated instructions"
```

**新しい space を作成する:**
```bash
gh api users/{username}/copilot-spaces \
  -X POST \
  -f name="My New Space" \
  -f general_instructions="Help me with..." \
  -f visibility="private"
```

**resource を添付する（resource list 全体を置き換える）:**
```json
{
  "resources_attributes": [
    { "resource_type": "free_text", "metadata": { "name": "Notes", "text": "Content here" } },
    { "resource_type": "github_issue", "metadata": { "repository_id": 12345, "number": 42 } },
    { "resource_type": "github_file", "metadata": { "repository_id": 12345, "file_path": "docs/guide.md" } }
  ]
}
```

**space を削除する:**
```bash
gh api users/{username}/copilot-spaces/{number} -X DELETE
```

**更新可能 field:** `name`、`description`、`general_instructions`、`icon_type`、`icon_color`、`visibility`（`"private"` / `"public"`）、`base_role`（`"no_access"` / `"reader"`）、`resources_attributes`

## 例

### Example 1: User が Space を指定する

**User**: "Load the Accessibility copilot space"

**Action**:
1. `mcp__github__get_copilot_space` を owner `"github"`、name `"Accessibility"` で呼び出す
2. 返ってきた context を使って、accessibility standard、MAS grade、compliance process などに関する質問に答える

### Example 2: User が Space を探したい

**User**: "What copilot spaces are available for our team?"

**Action**:
1. `mcp__github__list_copilot_spaces` を呼び出す
2. user の org や関心に合う space を絞り込んで提示する
3. 興味のある space を読み込むか提案する

### Example 3: Context に基づく質問

**User**: "Using the security space, what's our policy on secret scanning?"

**Action**:
1. 適切な owner と name で `mcp__github__get_copilot_space` を呼び出す
2. space content から relevant policy を見つける
3. 実際の内部 documentation に基づいて回答する

### Example 4: Workflow Engine としての Space

**User**: "Write my weekly update using the PM Weekly Updates space"

**Action**:
1. `mcp__github__get_copilot_space` を呼び出して space を読み込む。そこには template format と step-by-step instruction が入っている
2. space の workflow に従う。添付された initiative issue から data を引き、metric を集め、各 section を起草する
3. space が参照している外部 resource（tracking issue、dashboard）を他の MCP tool で取得する
4. 各 section の draft を user に見せ、review と不足情報の補完を受けられるようにする
5. 最終 output は、その space が定義した format で作成する

### Example 5: Space の instruction を programmatically 更新する

**User**: "Update my PM Weekly Updates space to include a new writing guideline"

**Action**:
1. `mcp__github__list_copilot_spaces` を呼び出し、space number を確認する（例: 19）
2. `mcp__github__get_copilot_space` を呼び出し、現在の instruction を読む
3. 要求どおりに instruction text を更新する
4. 変更を push する:
```bash
gh api users/labudis/copilot-spaces/19 -X PUT -f general_instructions="updated instructions..."
```

## Tips

- space 名は **case-sensitive** です。`list_copilot_spaces` に出た名前をそのまま使ってください
- space は user または organization が owner になります。必ず `owner` と `name` の両方を渡します
- space content は大きいことがあります（20KB+）。temp file として返ってきた場合は、全体を読むより grep や view_range で relevant section を探すほうが効率的です
- space が見つからない場合は、まず利用可能な space 一覧を出して正しい名前を探すよう提案します
- 基になる repo が更新されると space も自動更新されるため、context は常に最新です
- 一部の space には custom instruction が含まれており、それが振る舞い（coding standard、preferred pattern、workflow）を導きます。suggestion ではなく directive として扱ってください
- **Write 操作**（`gh api` による create/update/delete）には `user` PAT scope が必要です。write 操作で 404 が返る場合は、`gh auth refresh -h github.com -s user` を実行してください
- resource 更新は **配列全体を置き換えます**。resource を追加するには既存 resource 全部と新しい resource を含めます。削除するには配列内に `{ "id": 123, "_destroy": true }` を含めます
