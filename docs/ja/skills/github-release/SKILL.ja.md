---
name: github-release
description: >
  Guides IA through releasing a new version of a GitHub library end-to-end. 
  Handles SemVer versioning and Keep a Changelog formatting automatically.
compatibility: "requires: gh CLI and git"
---

# GitHubリリーススキル

このスキルは、単一パッケージの GitHub リポジトリの完全なリリース ワークフローを自動化します。
分析から変更ログの作成、PR の作成まで。のみに依存します
`gh` (GitHub CLI) および `git` 他のツールは必要ありません。

ステップ 14 は **読み取り専用の偵察** です。
ステップ 5、バージョン番号を確認したら。

## このスキルをいつ使用するか

ユーザーが新しいリリースをカットしたり、新しいバージョンを公開したりする場合は常にこのスキルを使用します。
バージョンを変更する、リリース ブランチを作成する、変更ログを生成する、またはリリース PR を開く
GitHub リポジトリ上にあります。ユーザーが「しましょう」のようなカジュアルな発言をした場合でもトリガーされます。
新しいバージョンを出荷する」または「リリースまでの時間」。

---

## 前提条件

以下の例には、Bash と PowerShell の両方のバリアントが含まれています。 Windows ユーザーはこちらを選択する必要があります
PowerShell ブロッ​​ク。

開始する前に、環境を確認してください。
```bash
gh auth status                        # must be authenticated
gh repo view --json nameWithOwner     # must be inside a GitHub repo
git status                            # working tree should be clean
```

いずれかのチェックが失敗した場合は、続行する前に停止し、修正すべき内容をユーザーに伝えます。

次に、ユーザーに 1 つの質問をします。

> *「ライブラリの公開ソース コードはどのディレクトリに含まれていますか?
> (例: `src/`、`lib/`、`pkg/` は、どのコンシューマの差分に焦点を当てるために使用されます
> 実際に見てください。 Enter キーを押してリポジトリ全体をスキャンします。)"*

答えを `PUBLIC_PATH` として保存します。空の場合、`PUBLIC_PATH` は `.` (リポジトリのルート) になります。
次のパスをすべての差分から除外します: `tests/`、`test/`、`spec/`、
`__tests__/`、`docs/`、`*.lock`、`*-lock.json`、`*.sum`、生成されたファイル
(「編集禁止」ヘッダー コメントを持つファイル)、アーティファクトをビルドします。

---

## 9 ステップのリリース ワークフロー

すべてのステップを順番に実行してください。これから実行しようとしているコマンドをユーザーに示し、
その出力。明示的に指摘された場合にのみ、一時停止して確認を求めます。

---

### ステップ 1 メインが最新であることを確認します
```bash
git checkout main
git pull origin main
```

今のところは `main` のままにしてください。リリース ブランチは、バージョンの後にステップ 5 で作成されます。
が確認されています。

---

### ステップ 2 最新バージョンのタグを取得します

> **なぜ `gh release list` ではないのでしょうか?** GitHub リリースは Git 上のオプションのレイヤーです
> タグ。多くのリポジトリは、GitHub リリースを作成せずに `git tag` でリリースをタグ付けします。
> したがって、`gh release list` は、バージョン タグが存在する場合でも空を返すことができます。タグの読み取り
> git から直接取得することは、信頼できる信頼できる情報源です。
```bash
# Fetch all tags from remote to ensure local view is current
git fetch --tags

# Find the latest version tag, sorted semantically
# --sort=-version:refname handles 1.10.0 > 1.9.0 correctly (unlike alphabetical)
PREV_TAG=$(git tag --sort=-version:refname | grep -E '^v?[0-9]+\.[0-9]+\.[0-9]+' | head -1)
echo "Latest tag: $PREV_TAG"
```
```PowerShell
# Fetch all tags from remote to ensure local view is current
git fetch --tags

# Find the latest version tag, sorted semantically
# --sort=-version:refname handles 1.10.0 > 1.9.0 correctly (unlike alphabetical)
$prevTag = git tag --sort='-version:refname' | `
  Select-String '^[vV]?\d+\.\d+\.\d+' | `
  Select-Object -First 1 -ExpandProperty Line

if ($prevTag) {
  $prevSha = git rev-list -n 1 $prevTag
} else {
  $prevSha = git rev-list --max-parents=0 HEAD
}

Write-Output "Latest tag: $prevTag"
```

次に、タグが (ローカルだけでなく) リモートに存在することを確認します。
```bash
git ls-remote --tags origin | grep "refs/tags/$PREV_TAG$"
```

リモート チェックで何も返されない場合は、タグがローカル専用であるように見えることをユーザーに警告します
まだプッシュされていないため、続行する前にプッシュする必要があるかもしれません。

- `PREV_TAG` は、見つかったとおりのタグ名です (例: `v1.4.2`)。先頭の `v` を削除します
算数をするとき。物に名前を付けるときにそれを保持します。
- **タグがまったく存在しない**場合は、`PREV_TAG` を `(none)` として扱い、`PREV_SHA` を
最初にコミットし、新しいバージョンをデフォルトで `1.0.0` に設定します (ステップ 4 のバージョン管理ロジックをスキップします。
そのままステップ 5) に進みます。
- タグが実際のコミット (孤立したタグ) を指していない場合は、フォールバックします。
`git rev-list --max-parents=0 HEAD` とユーザーに警告します。
```bash
PREV_SHA=$(git rev-list -n 1 "$PREV_TAG" 2>/dev/null || git rev-list --max-parents=0 HEAD)
```

---

### ステップ 3 前回のリリース以降に何が変更されたかを分析する

このステップでは **2 つの相補信号**を使用します。コードの差分は主なソースです。
真実;コミットメッセージは、意図に関するサポートコンテキストを提供します。

#### 3a コード差分 (一次信号)
```bash
# Focused diff on the public source path, excluding noise
git diff "$PREV_SHA"..HEAD -- "$PUBLIC_PATH" \
  ':(exclude)tests/' ':(exclude)test/' ':(exclude)spec/' \
  ':(exclude)__tests__/' ':(exclude)docs/' \
  ':(exclude)*.lock' ':(exclude)*-lock.json' ':(exclude)*.sum'
```
```PowerShell
# Focused diff on the public source path, excluding noise
git diff "$($prevSha)..HEAD" -- $publicPath `
  ':(exclude)tests/' ':(exclude)test/' ':(exclude)spec/' `
  ':(exclude)__tests__/' ':(exclude)docs/' `
  ':(exclude)*.lock' ':(exclude)*-lock.json' ':(exclude)*.sum'
```

完全な diff 出力を読み取ります。変更されたファイルごとに、以下を特定します。

1. **削除されたシンボル** 関数、クラス、メソッド、定数、エクスポートされた名前
以前は存在していましたが、今はなくなっています。 ? MAJORの強いシグナル。
2. **変更されたシグネチャ** 両方のバージョンに存在するが、異なる機能
パラメータ、戻り値の型、またはスローされたエラー。 ? MAJORの強いシグナル。
3. **新しいエクスポートされたシンボル** 存在しなかったパブリック関数、クラス、定数
前に。 ?マイナーの合図。
4. **内部のみの変更** パブリック インターフェイスには一切触れない変更
(プライベート ヘルパー、エクスポートされていない関数、アルゴリズム内部)。 ?パッチ。
5. **バグ修正** 明らかに間違っていたロジックの修正 (例: off-by-one、
null チェック、間違った条件)、パブリック API を変更せずに。 ?パッチ。

差分が非常に大きい (数千行) 場合は、まず統計概要を実行して、
どのファイルを完全に読み取るかを優先します。
```bash
git diff "$PREV_SHA"..HEAD --stat -- "$PUBLIC_PATH"
```

最も変更が多いファイルとその名前のファイルに焦点を当てて詳細を読み取ります。
パブリック インターフェイスを定義することを提案します (例: `index.*`、`api.*`、`exports.*`、
`public.*`、`mod.*`、`__init__.*`)。

#### 3b コミットログ（二次信号）
```bash
git log "$PREV_SHA"..HEAD --oneline --no-merges
```

これを使用して次のことを行います。
- コード変更の背後にある、自明ではない **意図** を理解する
diff のみ (たとえば、そのようにラベル付けされた 1 行のセキュリティ修正)。
- `PUBLIC_PATH` 以外のパスにある可能性があるが、ユーザーにはまだ表示されている変更をキャッチします
(例: `cmd/` ディレクトリ内の CLI フラグの変更)。
- コードだけでは全体が分からない変更ログ エントリのコンテキストを記入します。
話。

メッセージ パターンと変更タイプのマッピングについては、`references/commit-classification.md` を参照してください。

#### 3c 2 つの信号を調整する

信号が一致するとき?その分類を自信を持って使用してください。

信号が競合する場合? **コードの差分を優先します**。例:
- コミットには `fix: typo` と表示されますが、差分には削除されたパブリック メソッドが表示されます。メジャーとして扱います。
- コミットには `feat: new API` と表示されますが、差分はプライベートな内部のみに影響しますか? PATCHとして扱います。
- Commit には `chore: refactor` と表示されますが、差分により新しいエクスポートされたシンボルが追加されますか?マイナーとして扱います。

気づいた競合を文書化して、変更ログのレビュー中にユーザーにフラグを立てます。
ステップ6で。

---

### ステップ 4 次の SemVer バージョンを決定する

これらのルールをステップ 3 の分析に適用します (完全なルールは `references/semver-rules.md` にあります)。

|状態 |バンプ |
|---|---|
|パブリック API への重大な変更 (削除、署名の変更、動作の変更) |メジャー |
|新しいエクスポートされたシンボルまたは機能、重大な変更はありません |マイナー |
|バグ修正、パフォーマンスの向上、セキュリティ修正、ドキュメント、雑務のみ |パッチ |

リリースにミックスが含まれている場合、**最も高い優先順位が優先されます**。
@@コード0@@。

`NEXT_VERSION`を計算します:
- `PREV_TAG` を `MAJOR.MINOR.PATCH` 整数に分割します。
- 適切なバンプを適用します。
- `vMAJOR.MINOR.PATCH` の形式でフォーマットします。

**提案されたバージョンをユーザーに提示します**。次のような簡単な根拠を示します。
コミットメッセージだけでなく、特定のコードの発見結果。例：

> *「私は v2.1.0 を提案しています。差分には 2 つの新しいエクスポートされた関数 (`NewClient` と
> `WithTimeout`) が `src/client.go` にあり、既存のパブリック シンボルが削除されていないか、
> 変わりました。コミット メッセージはこれを機能追加として裏付けています。」*

質問: *「このバージョンは適切ですか? それとも調整しますか?」*
続行する前に確認を待ってください。

---

### ステップ 5 リリース ブランチを作成する

バージョンが確認されたので、最初から正しい名前でブランチを作成します。
```bash
git checkout -b release/vX.Y.Z
git push -u origin release/vX.Y.Z
```

---

### ステップ 6 CHANGELOG.md を更新する

既存の `CHANGELOG.md` を読み取ります (存在しない場合は作成します)。フォローしてください
[変更ログを保持する](https://keepachangelog.com/en/1.1.0/) 形式で厳密に保存してください。

**先頭に挿入する構造** (`# Changelog` ヘッダーのすぐ下):
```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- ...

### Changed
- ...

### Deprecated
- ...

### Removed
- ...

### Fixed
- ...

### Security
- ...
```

ルール:
- 今日の日付を `YYYY-MM-DD` 形式で使用します。
- エントリのないセクションを省略しても、空の見出しは残りません。
- 主にユーザーの視点から、**平易な英語**でエントリを作成します。
コード diff に示されているものから、コミット メッセージ コンテキストによって補足されます。
良い点: *「HTTP クライアント コンストラクターに `WithTimeout` オプションを追加しました。」*
悪い: *「feat: タイムアウト cfg パラメータを追加」*
- 結果をセクションにマッピングします。
  - 新しいエクスポートされたシンボル ?追加した
  - 破壊除去？削除されました
  - 既存の API に対する重大な変更?変更されました (破損としてフラグを立てます)
  - バグ/ロジック修正、パフォーマンス?修理済み
  - セキュリティ修正?安全
  - 内部リファクタリング、ドキュメント、雑用、テスト ?ユーザーに表示されない限り省略
- コードの差分だけでは伝わらない意図がコミットメッセージで明らかになった場合
(例: 1 行の変更を装ったセキュリティ修正)、そのコンテキストを
変更ログのエントリ。
- ファイルの下部にある diff リンクも更新します。
  ```markdown
  [X.Y.Z]: https://github.com/OWNER/REPO/compare/vPREV...vNEXT
  ```

**ディスクに書き込む前に、提案された変更ログ セクションをユーザーに表示します。**
ステップ 3c で信号の競合が見つかった場合は、ユーザーが確認できるようにここでフラグを立てます。
質問: *「この変更ログは正確ですか? 追加、削除、または書き直すエントリはありますか?」*
フィードバックを組み込み、ディスクに書き込みます。

---

### ステップ 7 コミットしてプッシュする
```bash
git add CHANGELOG.md
git commit -m "chore: release vX.Y.Z"
git push origin release/vX.Y.Z
```

次に進む前に、プッシュが成功したことを確認してください。

---

### ステップ 8 プルリクエストを開く

**??重要:** PR 本文テキストを渡すには常に `--body-file` を使用し、インライン テキストでは `--body` を使用しないでください。
`\n` のようなインライン エスケープ シーケンスは、PowerShell によって改行として解釈されず、表示されます。
PR 内のリテラルテキストとして。ファイルを使用すると、適切なマークダウン形式が保証されます。
```bash
gh pr create \
  --base main \
  --head release/vX.Y.Z \
  --title "Release vX.Y.Z" \
  --body "$(cat <<'EOF'
## Release vX.Y.Z

This PR prepares the **vX.Y.Z** release.

### What's included
<!-- paste the changelog section here -->

### Checklist
- [ ] Changelog reviewed
- [ ] Version bump verified
- [ ] CI passing

After merging, create the tag on the merge commit:
\`\`\`
git tag vX.Y.Z <merge-commit-sha>
git push origin vX.Y.Z
\`\`\`
EOF
)"
```
```PowerShell
# Create PR body using here-string (preserves actual newlines, not escape sequences)
$prBody = @"
## Release vX.Y.Z

This PR prepares the **vX.Y.Z** release.

### What's included
<paste changelog here>

### Checklist
- [ ] Changelog reviewed
- [ ] Version bump verified
- [ ] CI passing

After merging, create the tag on the merge commit:
``````
git tag vX.Y.Z <merge-commit-sha>
git push origin vX.Y.Z
``````
"@

# Write to file and use --body-file (do NOT use inline --body with escape sequences)
$prBody | Out-File -FilePath release_pr_body.md -Encoding utf8 -NoNewline
gh pr create --base main --head release/vX.Y.Z --title "Release vX.Y.Z" --body-file release_pr_body.md
```

変更ログセクションを PR 本文の「含まれるもの」ブロックに貼り付けます (または手動レビュー用にプレースホルダーを残します)。


---

### ステップ 9 ユーザーに引き渡す

ユーザーに次のように伝えます。

> **リリースPR公開中！ ??**
>
> 新しいバージョン: **vX.Y.Z**
>
> PR がレビューされて統合されたら、**自分でタグを作成**する必要があります。
> マージコミット:
>
> ```bash
> git tag vX.Y.Z <merge-commit-sha>
> git push origin vX.Y.Z
> ```
>
> 次に、GitHub Releases に移動し、そのタグからリリースを公開します。コピーできます
> 変更ログセクションをリリースノートに直接追加してください。

---

## エラー処理

|状況 |何をすべきか |
|---|---|
| `gh auth status` は失敗します |停止; `gh auth login` を実行するようにユーザーに指示します。
| git リポジトリ内にない |停止; `cd` をリポジトリに入れるようにユーザーに指示します。
|作業ツリーが汚れています |警告してください。隠したいか中止したいかを尋ねる |
|最後のタグ以降コミットはありません |解放するものは何もないことをユーザーに伝える |
|タグは存在しますが、コミットを指していません |最初のコミットを差分ベースとして使用します。ユーザーに警告する |
|最新のタグはローカルに存在しますが、リモートには存在しません |ユーザーに警告します。最初にタグをプッシュするか、それとも続行するかを尋ねる |
| `PUBLIC_PATH` の差分は空ですが、コミットは存在します。警告してください。すべての変更は内部的なものである可能性があります。まだ続行するかどうかを尋ねる |
| `git push` は失敗します (例: 保護されたブランチ ルール) |エラーをそのまま報告してください。ブランチ保護設定を確認することを提案してください。

---

## PowerShell でのトラブルシューティング

- ローカルで動作するコマンドが gh の使用法を出力するか、サブコマンドを別のトークンとして扱う場合は、次のことを確認してください。
PATH で gh.exe を呼び出し (Get-Command gh)、展開されていないネストされた置換を渡すことを避けます。 PowerShellを使用する
上記のパターン。
- 推奨テスト: gh --version; git fetch --tags; PowerShell スニペットを実行して $prevTag を設定し、 git diff --name-only $prevSha..HEAD -- src/ を実行します。

---

## 制限事項

- `gh` CLI をインストールして認証する必要があります。
- 現在のバージョンを確認するには git タグが必要です。

---

## 参照ファイル

- `references/semver-rules.md` 拡張 SemVer 決定ルールとエッジケース
- `references/commit-classification.md` コミットメッセージを変更タイプに分類するためのヒューリスティック