# 実際のコードツアーの例

実際のリポジトリが CodeTour 機能をどのように使用するかを確認する場合は、このファイルを参照してください。
各サンプルは、`.tour` ファイルへの直接リンクを持つパブリック GitHub リポジトリから取得されています。

---

##microsoft/codetour — 貢献者志向

**ツアー ファイル:** https://github.com/microsoft/codetour/blob/main/.tours/intro.tour
**ペルソナ:** 新しい寄稿者
**ステップ:** ~5 · **深さ:** 標準

**良い点:**
- SVG アーキテクチャ図が埋め込まれた導入ステップ (説明内の生の GitHub URL)
- 絵文字セクションヘッダーを使用したステップごとの豊富なマークダウン (`### 🎥 Tour Player`)
- 説明内のインライン ファイル間リンク: `[Gutter decorator](./src/player/decorator.ts)`
- 最上位の `description` フィールドをツアー自体のサブタイトルとして使用します

**コピーするテクニック:** 説明に画像と相互リンクを埋め込み、自己完結型にします。```json
{
  "file": "src/player/index.ts",
  "line": 436,
  "description": "### 🎥 Tour Player\n\nThe CodeTour player ...\n\n![Architecture](https://raw.githubusercontent.com/.../overview.svg)\n\nSee also: [Gutter decorator](./src/player/decorator.ts)"
}
```---

## a11yproject/a11yproject.com — 新しい貢献者のオンボーディング

**ツアー ファイル:** https://github.com/a11yproject/a11yproject.com/blob/main/.tours/code-tour.tour
**ペルソナ:** 外部寄稿者
**ステップ:** 26 · **深さ:** 深い

**良い点:**
- ほぼ完全に `directory` ステップ — ファイル内で迷うことなく、すべての `src/` サブディレクトリに方向付けします
- 全体を通して会話的で初心者に優しい口調
- 開始ステップの `selection` は、`package.json` の正確なエントリを強調表示します。
- 心からの感謝と行動喚起で締めくくります

**コピーするテクニック:** ディレクトリ ステップをオンボーディング ツアーの骨組みとして使用します。作成者がすべてのファイルを説明する必要なく、構造を教えます。```json
{
  "directory": "src/_data",
  "description": "This folder contains the **data files** for the site. Think of them as a lightweight database — YAML files that power the resource listings, posts index, and nav."
}
```---

## github/codespaces-codeql — 技術的に最も完全な例

**ツアー ファイル:** https://github.com/github/codespaces-codeql/blob/main/.tours/codeql-tutorial.tour
**ペルソナ:** セキュリティ エンジニア / コンセプト学習者
**ステップ:** 12 · **深さ:** 標準

**良い点:**
- `isPrimary: true` — コードスペースが開くと自動起動します
- ツアー中に実際の VS Code コマンドを実行するための `commands` 配列: リーダーがそのステップに到着すると、ツアーは文字通り `codeQL.runQuery` を実行します
- サイドバーパネルを切り替える `view` プロパティ (`"view": "codeQLDatabases"`)
- 回復力のあるマッチングの場合、`line` の代わりに `pattern` を使用します: `"pattern": "import tutorial.*"`
- `selection` : クエリ ファイル内の正確な `select` 句を強調表示します。

**これは、`commands`、`view`、および `pattern` の正規の参照です。**```json
{
  "file": "tutorial.ql",
  "pattern": "import tutorial.*",
  "view": "codeQLDatabases",
  "commands": ["codeQL.setDefaultTourDatabase", "codeQL.runQuery"],
  "title": "Run your first query",
  "description": "Click the **▶ Run** button above. The results appear in the CodeQL Query Results panel."
}
```---

## github/codespaces-learn-with-me — 最小限の対話型チュートリアル

**ツアー ファイル:** https://github.com/github/codespaces-learn-with-me/blob/main/.tours/main.tour
**ペルソナ:** まったくの初心者
**ステップ:** 4 · **深さ:** クイック

**良い点:**
- たった 4 つのステップ — クイック/バイブコーダーのペルソナにとっては少ない方が良いことを証明します
- `isPrimary: true` 自動起動の場合
- 各ステップでは、ただ読むだけではなく、**何かをする** (文字列を編集したり、色を変更したり) ように読者に指示します。
- 「ページが公開される」という具体的な結果で終了します。

**コピーするテクニック:** クイック/バイブコーダー ツアーの場合は、容赦なくカットしてください。行動を促す 4 つのステップは、すべてを説明する 12 のステップに勝ります。

---

## blackgirlbytes/copilot-todo-list — 28 ステップの対話型チュートリアル

**ツアー ファイル:** https://github.com/blackgirlbytes/copilot-todo-list/blob/main/.tours/main.tour
**ペルソナ:** 概念学習 / 実践チュートリアル
**ステップ数:** 28 · **深さ:** 深い

**良い点:**
- **コンテンツのみのチェックポイント ステップ** (`file` キーなし) を進捗マイルストーンとして使用します: 「ページをチェックしてください! 🎉」と「試してみてください!」コーディングタスクの合間に
- 説明内のターミナル インライン コマンド: `>> npm install uuid; npm install styled-components`
- 各ファイルステップには、ユーザーが受け入れる必要がある正確なコードがマークダウンコードフェンス内に表示されるため、期待される出力がわかります。

**模倣するテクニック:** チェックポイントのステップ (コンテンツのみ、マイルストーン タイトル) は長いツアーを分割し、読者に進歩の感覚を与えます。```json
{
  "title": "Check out your page! 🎉",
  "description": "Open the **Simple Browser** tab to see your to-do list. You should see all three tasks rendering from your data array.\n\nOnce you're happy with it, continue to add interactivity."
}
```---

## lucasjellema/cloudnative-on-oci-2021 — マルチツアー建築シリーズ

**ツアー ファイル:**
- https://github.com/lucasjellema/cloudnative-on-oci-2021/blob/main/.tours/function-tweet-retriever.tour
- https://github.com/lucasjellema/cloudnative-on-oci-2021/blob/main/.tours/oci-and-infrastructor-as-code.tour
- https://github.com/lucasjellema/cloudnative-on-oci-2021/blob/main/.tours/build-and-deployment-pipeline-function-tweet-retriever.tour

**ペルソナ:** プラットフォーム エンジニア / アーキテクト
**ステップ:** ツアーあたり 12 · **深さ:** 標準

**良い点:**
- 3 つの個別の関心事項 (関数コード、IaC、CI/CD パイプライン) に対する 3 つの個別のツアー — それぞれスタンドアロンですが、`nextTour` 経由でリンクされています
- `selection` 座標は、ブロック (単一行ではない) がポイントとなる Terraform ファイルで頻繁に使用されます。
- ステップには、公式 OCI ドキュメントへのマークダウン リンクがインラインで含まれています
- クローンを作成せずに `vscode.dev/github.com/...` 経由で参照できるように設計されています

**コピーするテクニック:** 複雑なシステムの場合は、レイヤーごとに 1 つのツアーを記述し、`nextTour` でチェーンします。インフラストラクチャ + アプリケーション コード + CI/CD を 1 つのツアーでカバーしようとしないでください。

---

## SeleniumHQ/selenium — Monorepo ビルド システムのオンボーディング

**ツアー ファイル:**
- `.tours/bazel.tour` — Bazel ワークスペースとビルド ターゲットの方向
- `.tours/building-and-testing-the-python-bindings.tour` — Python バインディング BUILD.bazel のウォークスルー

**ペルソナ:** 外部貢献者 (ビルド システム重視)
**ステップ:** ツアーごとに ~10

**良い点:**
- 明白ではないエントリ ポイントをターゲットとしています。製品コードではなく、ビルド システムです。
- 「寄稿者オンボーディング」ツアーは `main()` で始まる必要がないことを証明します。ツアーは、この特定のリポジトリに関して混乱を招くものから始まります。
- 大規模で成熟した OSS プロジェクトで大規模に使用

---

## テクニックのクイックリファレンス

|特集 |いつ使用するか |実際の例 |
|----------|---------------|---------------|
| `isPrimary: true` |リポジトリが開いたときにツアーを自動起動する (Codespace、vscode.dev) | codespaces-learn-with-me、codespaces-codeql |
| `commands: [...]` |リーダーがこのステップに到達したら、VS Code コマンドを実行します。コードスペース-codeql (`codeQL.runQuery`) |
| `view: "terminal"` |このステップで VS Code サイドバー/パネルを切り替えます |コードスペース-codeql (`codeQLDatabases`) |
| `pattern: "regex"` |数値ではなく行の内容で一致します - 揮発性ファイルに使用します |コードスペース-codeql |
| `selection: {start, end}` |ブロック (関数本体、構成セクション、型定義) を強調表示します。 a11yプロジェクト、oci-2021、codespaces-codeql |
| `directory: "path/"` |すべてのファイルを読み取らずにフォルダーを指定する | a11yプロジェクト、コードスペース-codeql |
| `uri: "https://..."` | PR、問題、RFC、ADR、外部ドキュメントへのリンク |あらゆる PR レビュー ツアー |
| `nextTour: "Title"` |シリーズのチェーンツアー | oci-2021 (3 部構成シリーズ) |
|チェックポイントの手順 (コンテンツのみ) |長いインタラクティブなツアーでの進歩のマイルストーン |副操縦士のToDoリスト |
| `>> command` 説明文 | VS Code のターミナル インライン コマンド リンク |副操縦士のToDoリスト |
|説明内の埋め込み画像 |アーキテクチャ図、スクリーンショット |マイクロソフト/コードツアー |

---

## GitHub でさらに実際のツアーを発見

**GitHub 上のすべての `.tour` ファイルを検索します:**
https://github.com/search?q=path%3A**%2F*.tour+&type=codeこの検索で​​は、パブリック GitHub リポジトリにコミットされたすべての `.tour` ファイルが返されます。これを使用して次のことを行います。
- 作業しているものと同じ言語/フレームワークでリポジトリのツアーを検索します
- 他の著者が同じペルソナやステップ タイプをどのように扱うかを研究する
- 特定のフィールド (`commands`、`selection`、`pattern`) が実際にどのように使用されているかを調べる

言語またはキーワードでフィルタリングして結果を絞り込みます — 例: `language:TypeScript` または `fastapi` をクエリに追加します。

---

## さらに読む

- **DEV コミュニティ — 「CodeTour を使用してコードベースをオンボードする」**: https://dev.to/tobiastimm/onboard-your-codebase-with-codetour-2jc8
- **コーダー ブログ — 「CodeTour を使用して新しいプロジェクトに迅速に参加できる」**: https://coder.com/blog/onboard-to-new-projects-faster-with-codetour
- **Microsoft Tech Community — 教育者開発者ブログ**: https://techcommunity.microsoft.com/blog/educatordeveloperblog/codetour-vscode-extension-allows-you-to-Produce-interactive-guides-assessments-a/1274297
- **AMIS テクノロジー ブログ — vscode.dev + CodeTour**: https://technology.amis.nl/software-development/visual-studio-code-the-code-tours-extension-for-in-context-and-interactive-readme/
- **CodeTour GitHub トピック**: https://github.com/topics/codetour