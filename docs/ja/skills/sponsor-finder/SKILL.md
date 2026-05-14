---
name: sponsor-finder
description: Find which of a GitHub repository's dependencies are sponsorable via GitHub Sponsors. Uses deps.dev API for dependency resolution across npm, PyPI, Cargo, Go, RubyGems, Maven, and NuGet. Checks npm funding metadata, FUNDING.yml files, and web search. Verifies every link. Shows direct and transitive dependencies with OSSF Scorecard health data. Invoke with /sponsor followed by a GitHub owner/repo (e.g. "/sponsor expressjs/express").
---
# スポンサーファインダー

プロジェクトの依存関係の背後にあるオープンソースのメンテナーをサポートする機会を見つけてください。 GitHub `owner/repo` (例: `/sponsor expressjs/express`) を受け入れ、依存関係の解決とプロジェクトの健全性データに deps.dev API を使用し、直接的依存関係と推移的依存関係の両方をカバーするフレンドリーなスポンサーシップ レポートを作成します。

## あなたのワークフロー

ユーザーが `/sponsor {owner/repo}` と入力するか、`owner/repo` 形式でリポジトリを指定すると、次のようになります。

1. **入力を解析します** — `owner` と `repo` を抽出します。
2. **エコシステムの検出** — マニフェストを取得してパッケージ名とバージョンを特定します。
3. **完全な依存関係ツリーを取得** — deps.dev `GetDependencies` (1 回の呼び出し)。
4. **リポジトリを解決** — 各depのdeps.dev `GetVersion` → `relatedProjects`はGitHubリポジトリを提供します。
5. **プロジェクトの健全性を取得** — 固有のリポジ​​トリの deps.dev `GetProject` → OSSF スコアカード。
6. **ファンディング リンクの検索** — npm `funding` フィールド、FUNDING.yml、Web 検索フォールバック。
7. **すべてのリンクを確認する** — 各 URL を取得して、ライブであることを確認します。
8. **グループ化してレポート** — 資金提供先ごとに、影響度別に分類します。

---

## ステップ 1: エコシステムとパッケージを検出する

`get_file_contents` を使用して、ターゲット リポジトリからマニフェストを取得します。エコシステムを特定し、パッケージ名と最新バージョンを抽出します。

|ファイル |エコシステム | | からのパッケージ名| からのバージョン
|------|-----------|---------------------|--------------|
| `package.json` |故宮 | `name` フィールド | `version` フィールド |
| `requirements.txt` |ぴぴ |パッケージ名のリスト |最新を使用 (deps.dev 呼び出しでバージョンを省略) |
| `pyproject.toml` |ぴぴ | `[project.dependencies]` |最新のものを使用する |
| `Cargo.toml` |貨物 | `[package] name` | `[package] version` |
| `go.mod` |行く | `module` パス | go.mod から抽出 |
| `Gemfile` |ルビージェムズ |宝石の名前 |最新のものを使用する |
| `pom.xml` |メイブン | `groupId:artifactId` | `version` |

---

## ステップ 2: 完全な依存関係ツリーを取得する (deps.dev)

**これは重要な手順です。** `web_fetch` を使用して deps.dev API を呼び出します。```
https://api.deps.dev/v3/systems/{ECOSYSTEM}/packages/{PACKAGE}/versions/{VERSION}:dependencies
```例えば：```
https://api.deps.dev/v3/systems/npm/packages/express/versions/5.2.1:dependencies
```これにより、各ノードに次の内容が含まれる `nodes` 配列が返されます。
- `versionKey.name` — パッケージ名
- `versionKey.version` — 解決済みバージョン
- `relation` — `"SELF"`、`"DIRECT"`、または `"INDIRECT"`

**この 1 回の呼び出しで、依存関係ツリー全体** (直接的および推移的両方) が正確に解決されたバージョンで得られます。ロックファイルを解析する必要はありません。

### URLエンコード
特殊文字を含むパッケージ名はパーセントでエンコードする必要があります。
- `@colors/colors` → `%40colors%2Fcolors`
- `@` を `%40`、`/` を `%2F` としてエンコードします

### 単一のルートパッケージを持たないリポジトリの場合
リポジトリがパッケージを公開しない場合 (ライブラリではなくアプリなど)、`package.json` 依存関係を直接読み取り、それぞれに対して deps.dev `GetVersion` を呼び出すことにフォールバックします。

---

## ステップ 3: GitHub リポジトリ (deps.dev) への各依存関係を解決する

ツリーの依存関係ごとに、deps.dev `GetVersion` を呼び出します。```
https://api.deps.dev/v3/systems/{ECOSYSTEM}/packages/{NAME}/versions/{VERSION}
```応答から次を抽出します。
- **`relatedProjects`** → `relationType: "SOURCE_REPO"` を探す → `projectKey.id` は `github.com/{owner}/{repo}` を返します
- **`links`** → `label: "SOURCE_REPO"` を検索 → `url` フィールド

これは、同じフィールド構造を持つ **すべてのエコシステム** (npm、PyPI、Cargo、Go、RubyGems、Maven、NuGet) で機能します。

### 効率ルール
- **一度に 10 件**のバッチで処理します。
- 重複排除 — 複数のパッケージが同じリポジトリにマップされる場合があります。
- GitHub プロジェクトが見つからない場合は deps をスキップします (「解決不能」としてカウントされます)。

---

## ステップ 4: プロジェクトの健全性データを取得する (deps.dev)

一意の GitHub リポジトリごとに、deps.dev `GetProject` を呼び出します。```
https://api.deps.dev/v3/projects/github.com%2F{owner}%2F{repo}
```応答から次を抽出します。
- **`scorecard.checks`** → `"Maintained"` チェックを検索 → `score` (0–10)
- **`starsCount`** — 人気指標
- **`license`** — プロジェクト ライセンス
- **`openIssuesCount`** — アクティビティインジケーター

Maintained スコアを使用してプロジェクトの健全性をラベル付けします。
- スコア 7–10 → ⭐ 積極的に維持
- スコア 4–6 → ⚠️部分的に維持
- スコア 0–3 → 💤 メンテナンスされていない可能性があります

### 効率ルール
- **固有のリポジトリ**のみを取得します (パッケージごとではありません)。
- **一度に 10 件**のバッチで処理します。
- このステップはオプションです。レートが制限されている場合はスキップし、出力にメモしてください。

---

## ステップ 5: 資金リンクを見つける

固有の GitHub リポジトリごとに、次の 3 つのソースを順番に使用して資金調達情報を確認します。

### 5a: npm `funding` フィールド (npm エコシステムのみ)
`https://registry.npmjs.org/{package-name}/latest` で `web_fetch` を使用し、`funding` フィールドを確認します。
- **文字列:** `"https://github.com/sponsors/sindresorhus"` → URLとして使用
- **オブジェクト:** `{"type": "opencollective", "url": "https://opencollective.com/express"}` → `url` を使用
- **配列:** すべての URL を収集します

### 5b: `.github/FUNDING.yml` (リポジトリレベル、次に組織レベルのフォールバック)

**ステップ 5b-i — リポジトリごとのチェック:**
`get_file_contents` を使用して `{owner}/{repo}` パス `.github/FUNDING.yml` を取得します。

**ステップ 5b-ii — 組織/ユーザーレベルのフォールバック:**
5b-i が 404 を返した場合 (リポジ自体に FUNDING.yml がない)、所有者のデフォルトのコミュニティ ヘルス リポジトリを確認します。
`get_file_contents` を使用して `{owner}/.github` パス `FUNDING.yml` を取得します。

GitHub は、[デフォルトのコミュニティ ヘルス ファイル](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) 規則をサポートしています。ユーザー/組織レベルの `.github` リポジトリは、独自のリポジトリが存在しないすべてのリポジトリにデフォルトを提供します。たとえば、`isaacs/.github/FUNDING.yml` はすべての `isaacs/*` リポジトリに適用されます。

それぞれの一意の `{owner}/.github` リポジトリを **1 回**のみ検索します。その所有者の下にあるすべてのリポジトリに対して結果を再利用します。 **一度に 10 人の所有者** のバッチで処理します。

YAML を解析します (5b-i と 5b-ii の両方で同じ)。
- `github: [username]` → `https://github.com/sponsors/{username}`
- `open_collective: slug` → `https://opencollective.com/{slug}`
- `ko_fi: username` → `https://ko-fi.com/{username}`
- `patreon: username` → `https://patreon.com/{username}`
- `tidelift: platform/package` → `https://tidelift.com/subscription/pkg/{platform-package}`
- `custom: [urls]` → そのまま使用

### 5c: Web 検索のフォールバック
**資金のない依存関係の上位 10 位** (推移的な依存関係の数による) については、`web_search` を使用します。```
"{package name}" github sponsors OR open collective OR funding
```企業が管理していることが知られているパッケージ (React/Meta、TypeScript/Microsoft、@types/DefinitelyTyped) をスキップします。

### 効率ルール
- **すべての部門について 5a と 5b を確認してください。** 上位の資金のない部門にのみ 5c を使用してください。
- 非 npm エコシステムの npm レジストリ呼び出しをスキップします。
- リポジトリの重複を排除 — 各リポジトリを 1 回だけチェックします。
- **一意の所有者ごとに 1 つの `{owner}/.github` チェック** - すべてのリポジトリに対して結果を再利用します。
- **一度に 10 人の所有者**のバッチで組織レベルの検索を処理します。

---

## ステップ 6: すべてのリンクを確認する (重要)

**資金リンクを含める前に、そのリンクが存在することを確認してください。**

各ファンディング URL で `web_fetch` を使用します。
- **有効なページ** → ✅ 含める
- **404 / "見つからない" / "登録されていません"** → ❌ 除外
- **有効なページにリダイレクト** → ✅ 最終 URL を含める

**一度に 5 つのバッチ**で確認します。未検証のリンクを決して表示しないでください。

---

## ステップ 7: レポートを出力する

### 出力規律

**データ収集中の中間出力を最小限に抑えます。** 各バッチをアナウンスしないでください (「バッチ 3/7…」、「資金調達を確認中…」)。代わりに:
- 各主要フェーズの開始時に **1 つの短いステータス行**を表示します (例: 「67 の依存関係を解決しています…」、「資金リンクを確認しています…」)
- **レポートを作成する前にすべてのデータを収集してください。** 部分的なテーブルをドリップフィードしないでください。
- 最終レポートを最後に **単一のまとまったブロック**として出力します。

### レポートテンプレート```
## 💜 Sponsor Finder Report

**Repository:** {owner}/{repo} · {ecosystem} · {package}@{version}
**Scanned:** {date} · {total} deps ({direct} direct + {transitive} transitive)

---

### 🎯 Ways to Give Back

Sponsoring just {N} people/orgs supports {sponsorable} of your {total} dependencies — a great way to invest in the open source your project depends on.

1. **💜 @{user}** — {N} direct + {M} transitive deps · ⭐ Maintained
   {dep1}, {dep2}, {dep3}, ...
   https://github.com/sponsors/{user}

2. **🟠 Open Collective: {name}** — {N} direct + {M} transitive deps · ⭐ Maintained
   {dep1}, {dep2}, {dep3}, ...
   https://opencollective.com/{name}

3. **💜 @{user2}** — {N} direct dep · 💤 Low activity
   {dep1}
   https://github.com/sponsors/{user2}

---

### 📊 Coverage

- **{sponsorable}/{total}** dependencies have funding options ({percentage}%)
- **{destinations}** unique funding destinations
- **{unfunded_direct}** direct deps don't have funding set up yet ({top_names}, ...)
- All links verified ✅
```### レポート形式の規則

- **「🎯 恩返しの方法」でリードする** — これが主要な成果です。カバーされる合計 Dep 順 (降順) でソートされた番号付きリスト。
- **単独の行に裸の URL** — マークダウン リンク構文でラップされていません。これにより、どのターミナル エミュレーターでも確実にクリックできるようになります。
- **インライン依存関係名** — 各スポンサーの下に、対象となる依存関係名をカンマ区切りの行にリストします。これにより、ユーザーは自分が何に資金を提供しているのかを正確に確認できます。
- **ヘルス インジケーター インライン** — 表の別の列ではなく、各宛先の横に ⭐/⚠️/💤 を表示します。
- **1 つの「📊 カバレッジ」セクション** — コンパクトな統計。個別の「検証済み資金リンク」テーブルや「資金が見つかりません」テーブルはありません。
- **資金のないdepsについて簡単にメモ** — カウントと上位の名前だけ。ギャップを強調するのではなく、「まだ資金調達が準備されていない」という枠を設定します。資金がないからといってプロジェクトを恥じることはありません。多くのメンテナは他の形式の貢献を好みます。
- 💜 GitHub スポンサー、🟠 Open Collective、☕ Ko-fi、🔗 その他
- 同じメンテナーに複数の資金源が存在する場合、GitHub スポンサーのリンクを優先します。

---

## エラー処理

- deps.dev がパッケージに対して 404 を返した場合 → マニフェストを直接読み取り、レジストリ API を介して解決することにフォールバックします。
- deps.dev がレート制限されている場合 → 部分的な結果に注目し、フェッチされた内容を続行します。
- `get_file_contents` がリポジトリに対して 404 を返した場合 → リポジトリが存在しないかプライベートである可能性があることをユーザーに通知します。
- リンクの検証が失敗した場合 → リンクをサイレントに除外します。
- 部分的であっても常にレポートを作成します。黙って失敗しないでください。

---

## 重要なルール1. **未確認のリンクは決して表示しないでください。** 表示する前にすべての URL を取得してください。 5 つの検証済みリンク > 20 の推測リンク。
2. **トレーニングの知識から決して推測しないでください。** 常に確認してください。資金調達ページは時間の経過とともに変化します。
3. **常に励まし、決して恥じることはありません。** 結果を前向きに捉えてください。IS が資金を提供したものを賞賛し、資金のない開発者を失敗ではなく機会として扱います。すべてのプロジェクトが財政的スポンサーを必要としたり、望んだりするわけではありません。
4. **行動で先導する。** 「🎯 恩返しの方法」セクションが主な出力であり、リンク先ごとにグループ化された、クリック可能な URL です。
5. **deps.dev をプライマリ リゾルバーとして使用します。** deps.dev が使用できない場合にのみ、レジストリ API にフォールバックします。
6. **常に GitHub MCP ツールを使用してください** (`get_file_contents`)、`web_fetch`、および `web_search` — クローンを作成したり、シェルアウトしたりしないでください。
7. **効率的であること。** API 呼び出しをバッチ処理し、リポジトリを重複排除し、各所有者の `.github` リポジトリを 1 回だけチェックします。
8. **GitHub スポンサーに焦点を当てます。** 最も実用的なプラットフォーム — 他のプラットフォームも紹介しますが、GitHub を優先します。
9. **メンテナによる重複排除。** 1 人のスポンサーによる実際の影響を示すグループ。
10. **実行可能な最小限を示します。** 最も多くの Deps をサポートするために、最も少ないスポンサーシップをユーザーに伝えます。
11. **中間出力を最小限に抑えます。** 各バッチを発表しないでください。すべてのデータを収集し、1 つのまとまったレポートを出力します。