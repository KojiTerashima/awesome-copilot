---
name: code-tour
description: >
  このスキルは、CodeTour の .tour ファイル（対象ペルソナ別の段階的ウォークスルー）を作成するために使います。
  実在するファイルと行番号にリンクします。トリガー例: "create a tour", "make a code tour",
  "generate a tour", "onboarding tour", "tour for this PR", "tour for this bug", "RCA tour",
  "architecture tour", "explain how X works", "vibe check", "PR review tour",
  "contributor guide", "help someone ramp up"、またはコードを構造的に案内してほしい任意の依頼。
  20種類の開発者ペルソナ（新規参加者、バグ修正者、アーキテクト、PR レビュアー、
  vibecoder、セキュリティレビュアーなど）に対応し、CodeTour のすべてのステップ種別（file/line、selection、
  pattern、uri、commands、view）と、ツアー全体のフィールド（ref、isPrimary、nextTour）を扱えます。
  言語を問わず、任意のリポジトリで利用できます。
---

# Code Tour Skill

あなたは **CodeTour**（コードベースを対象ペルソナ向けに段階的に案内するウォークスルー）を作成します。
CodeTour はファイルと行番号に直接リンクします。CodeTour ファイルは `.tours/` に配置し、
[VS Code CodeTour extension](https://github.com/microsoft/codetour) で利用します。

`scripts/` には2つのスクリプトが同梱されています:

- **`scripts/validate_tour.py`** — ツアー作成後に必ず実行します。JSON の妥当性、ファイル/ディレクトリの存在、行番号範囲、pattern の一致、nextTour の相互参照、物語構成を検証します。実行コマンド: `python ~/.agents/skills/code-tour/scripts/validate_tour.py .tours/<name>.tour --repo-root .`
- **`scripts/generate_from_docs.py`** — README/docs から生成するよう依頼された場合は、まずこれを実行して骨組みを作り、その後に補完します。実行コマンド: `python ~/.agents/skills/code-tour/scripts/generate_from_docs.py --persona new-joiner --output .tours/skeleton.tour`

2つの参照ファイルも同梱されています:

- **`references/codetour-schema.json`** — 正式な JSON スキーマです。フィールド名や型に迷ったらこれを参照してください。使うすべてのフィールドはこのスキーマに準拠している必要があります。
- **`references/examples.md`** — 本番リポジトリ由来の実例 CodeTour 8件（手法注釈付き）です。特定機能（`commands`、`selection`、`view`、`pattern`、`isPrimary`、複数ツアー連携）を実運用でどう使うか確認したいときに参照します。

### GitHub 上の実運用 `.tour` ファイル

以下はいずれも本番で使われている `.tour` ファイルです。特定のステップ種別、ツアー全体フィールド、物語構造の実例が必要なら取得して確認してください。実例があるのに記憶頼みで書かないでください。

GitHub コード検索で追加の例を探せます: https://github.com/search?q=path%3A**%2F*.tour+&type=code

#### ステップ種別 / 手法ごとの実例

| 学ぶ対象 | ファイル URL |
|---|---|
| `directory` + `file+line`（コントリビューター向けオンボーディング） | https://github.com/coder/code-server/blob/main/.tours/contributing.tour |
| `selection` + `file+line` + 導入用 content ステップ（アクセシビリティプロジェクト） | https://github.com/a11yproject/a11yproject.com/blob/main/.tours/code-tour.tour |
| 最小チュートリアル — インタラクティブ学習向けの簡潔な `file+line` ナラティブ | https://github.com/lostintangent/rock-paper-scissors/blob/master/main.tour |
| `nextTour` 連結を使った複数ツアー構成（クラウドネイティブ OCI ウォークスルー） | https://github.com/lucasjellema/cloudnative-on-oci-2021/blob/main/.tours/introduction.tour |
| `isPrimary: true`（オンボーディングの入口を示す） | https://github.com/nickvdyck/webbundlr/blob/main/.tours/getting-started.tour |
| `line` の代わりに `pattern` を使用（正規表現アンカーのステップ） | https://github.com/nickvdyck/webbundlr/blob/main/.tours/architecture.tour |

**Raw コンテンツのヒント:** 生の JSON が必要なときは、URL を `raw.githubusercontent.com` 形式にし、`/blob/` を外してください。

優れたツアーは単なる注釈付きファイル一覧ではありません。**ナラティブ** です。誰に向けて、何が重要で、なぜ重要で、次に何をすべきかを語るものです。目標は「このリポジトリを初めて開いたときに、当人が本当に欲しかった」と思えるツアーを書くことです。

**CRITICAL: 作成してよいのは `.tour` JSON ファイルのみです。他のファイルを作成・変更・スキャフォールドしてはいけません。**

---

## Step 1: リポジトリを把握する

ユーザーへ質問する前に、まずコードベースを探索します:

- ルートディレクトリを一覧し、README と主要設定ファイルを読む
  （package.json、pyproject.toml、go.mod、Cargo.toml、composer.json など）
- 言語/フレームワークと、プロジェクトの目的を特定する
- フォルダ構造を 1〜2 階層で把握する
- エントリポイントを見つける: main ファイル、index ファイル、アプリ起動処理
- **実在するファイルを記録する** — ツアーに書くすべてのパスは実在していなければなりません

リポジトリが疎または空なら、その旨を明示し、存在するものをもとに進めます。

**ユーザーが「README から生成」「docs を使って」と言った場合:** まずスケルトン生成を実行し、その後で実ファイルを読んで `[TODO: ...]` をすべて埋めます。

```bash
python skills/code-tour/scripts/generate_from_docs.py \
  --persona new-joiner \
  --output .tours/skeleton.tour
```

### 言語/フレームワーク別のエントリポイント

全部を読む必要はありません。まずここから始め、import を辿ってください。

| Stack | 最初に読むエントリポイント |
|-------|---------------------------|
| **Node.js / TS** | `index.js/ts`, `server.js`, `app.js`, `src/main.ts`, `package.json` (scripts) |
| **Python** | `main.py`, `app.py`, `__main__.py`, `manage.py` (Django), `app/__init__.py` (Flask/FastAPI) |
| **Go** | `main.go`, `cmd/<name>/main.go`, `internal/` |
| **Rust** | `src/main.rs`, `src/lib.rs`, `Cargo.toml` |
| **Java / Kotlin** | `*Application.java`, `src/main/java/.../Main.java`, `build.gradle` |
| **Ruby** | `config/application.rb`, `config/routes.rb`, `app/controllers/application_controller.rb` |
| **PHP** | `index.php`, `public/index.php`, `bootstrap/app.php` (Laravel) |

### リポジトリ種別ごとの違い — 重点を調整する

同じペルソナでも、リポジトリ種別によって必要情報は変わります:

| Repo type | 強調すべきこと | 典型的なアンカーファイル |
|-----------|----------------|--------------------------|
| **Service / API** | リクエストのライフサイクル、認証、エラー契約 | router, middleware, handler, schema |
| **Library / SDK** | 公開 API 面、拡張ポイント、バージョニング | index/exports, types, changelog |
| **CLI tool** | コマンド解析、設定読み込み、出力整形 | main, commands/, config |
| **Monorepo** | パッケージ境界、共有契約、ビルドグラフ | root package.json/pnpm-workspace, shared/, packages/ |
| **Framework** | プラグイン機構、ライフサイクルフック、escape hatch | core/, plugins/, lifecycle |
| **Data pipeline** | source → transform → sink、スキーマ責務 | ingest/, transform/, schema/, dbt models |
| **Frontend app** | コンポーネント階層、状態管理、ルーティング | pages/, store/, router, api/ |

**モノレポの場合:** ペルソナの目的に最も関係する 2〜3 パッケージを特定してください。全体を無理に案内せず、まずワークスペースの見方を説明するステップを置き、その後は焦点を絞ります。

### 大規模リポジトリ戦略

100 ファイル超のリポジトリでは、すべてを読もうとしないこと。

1. まずエントリポイントと README を読む
2. 上位 5〜7 モジュールのメンタルモデルを作る
3. 要求されたペルソナ向けに、**最重要の 2〜3 モジュール**を特定して深く読む
4. 対象外モジュールは導入ステップで「このツアーのスコープ外」と明示する
5. 読み込んでいないが把握した領域には `directory` ステップを使う（全読込なしで方向づけできる）

正しいファイルに絞った 10 ステップのツアーは、全体を浅く回る 25 ステップより価値があります。

---

## Step 2: 意図を読む — 推測できることは推測し、どうしても必要なことだけ聞く

**ユーザーの1メッセージで十分であるべきです。** 追加質問の前に、依頼文からペルソナ、深さ、焦点を推測してください。

### 意図マップ

| User says | → Persona | → Depth | → Action |
|-----------|-----------|---------|----------|
| "tour for this PR" / "PR review" / "#123" | pr-reviewer | standard | PR 用 `uri` ステップを追加し、ブランチには `ref` を使う |
| "why did X break" / "RCA" / "incident" | rca-investigator | standard | 障害の因果連鎖を追跡する |
| "debug X" / "bug tour" / "find the bug" | bug-fixer | standard | エントリ → 故障点 → テスト |
| "onboarding" / "new joiner" / "ramp up" | new-joiner | standard | ディレクトリ、セットアップ、業務コンテキスト |
| "quick tour" / "vibe check" / "just the gist" | vibecoder | quick | 5〜8 ステップ、最短ルートのみ |
| "explain how X works" / "feature tour" | feature-explainer | standard | UI → API → backend → storage |
| "architecture" / "tech lead" / "system design" | architect | deep | 境界、意思決定、トレードオフ |
| "security" / "auth review" / "trust boundaries" | security-reviewer | standard | 認証フロー、入力検証、機密シンク |
| "refactor" / "safe to extract?" | refactorer | standard | 継ぎ目、隠れ依存、抽出順序 |
| "performance" / "bottlenecks" / "slow path" | performance-optimizer | standard | ホットパス、N+1、I/O、キャッシュ |
| "contributor" / "open source onboarding" | external-contributor | quick | 安全領域、規約、地雷ポイント |
| "concept" / "explain pattern X" | concept-learner | standard | 概念 → 実装 → 理由 |
| "test coverage" / "where to add tests" | test-writer | standard | 契約、継ぎ目、カバレッジ欠落 |
| "how do I call the API" | api-consumer | standard | 公開 API 面、認証、エラー意味論 |

**暗黙に推測すること:** persona、depth、focus、`uri`/`ref` の要否、`isPrimary` の要否。

**本当に推測できない場合だけ質問する:**
- "bug tour" だがバグ内容が未提示 → バグ内容を尋ねる
- "feature tour" だが機能名が未提示 → 対象機能を尋ねる
- "specific files" が明示された → 必須立ち寄り先として扱う

ユーザーが触れていない限り、`nextTour`、`commands`、`when`、`stepMarker` は質問しないでください。

### PR ツアーの定石

PR ツアーでは、`"ref"` をブランチに設定し、PR への `uri` ステップで開始し、変更ファイルを先に、次に未変更だが重要なファイルを扱い、最後にレビュアーチェックリストで閉じます。

### ユーザー指定のカスタマイズ — 常に優先する

| User says | 対応方法 |
|-----------|---------|
| "cover `src/auth.ts` and `config/db.yml`" | そのファイルを必須ステップにする |
| "pin to the `v2.3.0` tag" / "this commit: abc123" | `"ref": "v2.3.0"` を設定 |
| "link to PR #456" / URL を貼る | 適切な文脈で `uri` ステップを追加 |
| "lead into the security tour when done" | `"nextTour": "Security Review"` を設定 |
| "make this the main onboarding tour" | `"isPrimary": true` を設定 |
| "open a terminal at this step" | `"commands": ["workbench.action.terminal.focus"]` を追加 |
| "deep" / "thorough" / "5 steps" / "quick" | 深さ設定を上書き |

---

## Step 3: 実ファイルを読む — 例外なし

**ツアーに書くすべてのファイルパスと行番号は、実際に読んで確認する必要があります。**
誤ったファイルや存在しない行を指すツアーは、ツアーがないより悪いです。

各ステップ案について:
1. ファイルを読む
2. 強調したいコードの正確な行を見つける
3. 対象ペルソナに説明できるレベルまで理解する

ユーザー指定のファイルが存在しない場合は明示してください。黙って別ファイルに差し替えないでください。

---

## Step 4: ツアーを書く

`.tours/<persona>-<focus>.tour` に保存します。正規フィールド一覧は `references/codetour-schema.json` を参照してください。使うフィールドはすべてそのスキーマに存在していなければなりません。

### ツアールート

```json
{
  "$schema": "https://aka.ms/codetour-schema",
  "title": "Descriptive Title — Persona / Goal",
  "description": "One sentence: who this is for and what they'll understand after.",
  "ref": "main",
  "isPrimary": false,
  "nextTour": "Title of follow-up tour",
  "steps": []
}
```

このツアーに不要なフィールドは省略します。

**`when`** — 条件付き表示。実行時に評価される JavaScript 式です。条件が真のときのみツアーを表示します。ペルソナ別の自動起動や、簡易ツアー完了まで上級ツアーを隠す用途に有効です。
```json
{ "when": "workspaceFolders[0].name === 'api'" }
```

**`stepMarker`** — ステップアンカーをソースコードコメントに埋め込みます。設定すると CodeTour は `// <stepMarker>` コメントを探し、行番号の代わり（または併用）に位置決めします。行番号が頻繁にずれる活発なコードに有効です。例: `"stepMarker": "CT"` を設定し、ソースに `// CT` を置く。ユーザーが求めない限り提案しないでください。これはソース編集を伴い、通常は例外的です。

---

### ステップ種別 — 完全リファレンス

すべてのステップ種別: **content**（導入/締め、最大2）、**directory**、**file+line**（主力）、**selection**（コードブロック）、**pattern**（正規表現一致）、**uri**（外部リンク）、**view**（VS Code パネルへフォーカス）、**commands**（VS Code コマンド実行）。

> **パス規則:** `"file"` と `"directory"` はリポジトリルートからの相対パスである必要があります。絶対パスや先頭 `./` は不可です。

---

### ステップ種別の使い分け

| 状況 | Step type |
|-----------|-----------|
| ツアー導入または締め | content |
| 「このフォルダには何があるか」を示す | directory |
| 1行で核心が伝わる | file + line |
| 関数/クラス本体が要点 | selection |
| 行番号が動きやすくファイルが不安定 | pattern |
| PR / issue / doc が「なぜ」を補う | uri |
| ターミナルやエクスプローラーを開かせたい | view or commands |

---

### ステップ数のキャリブレーション

深さとペルソナに合わせてステップ数を調整します。以下は目安であり厳密上限ではありません。

| Depth | Total steps | Core path steps | Notes |
|-------|-------------|-----------------|-------|
| Quick | 5–8 | 3–5 | Vibecoder、高速探索向け — 容赦なく削る |
| Standard | 9–13 | 6–9 | 大半のペルソナ向け — 広さと必要十分な深さ |
| Deep | 14–18 | 10–13 | Architect、RCA 向け — トレードオフをすべて可視化 |

リポジトリ規模にも合わせてください。3ファイルの CLI に 15 ステップは不要です。200ファイルのモノリスを 5 ステップに詰め込むべきでもありません。

| Repo size | 推奨 standard 深さ |
|-----------|--------------------|
| Tiny (< 20 files) | 5–8 steps |
| Small (20–80 files) | 8–11 steps |
| Medium (80–300 files) | 10–13 steps |
| Large (300+ files) | 12–15 steps（関連サブシステムに限定） |

---

### 優れた description の書き方 — SMIG 公式

各 description は、順に4つの問いへ答える必要があります。4段落にする必要はありませんが、短くても4要素すべてを含めてください。

**S — Situation**: いま読者は何を見ているか。文脈を固定する1文。
**M — Mechanism**: このコードはどう動くか。どのパターン/規則/設計が働いているか。
**I — Implication**: それが *このペルソナの目標* に対してなぜ重要か。
**G — Gotcha**: 優秀な人でも何を誤解しやすいか。何が非自明・壊れやすい・意外か。

description は、読者がファイルを読むだけでは得られない知見を与えるべきです。パターン名を示し、設計判断を説明し、失敗モードを指摘し、関連文脈へつなげてください。

---

## 物語の弧（Narrative arc）— すべてのツアー、すべてのペルソナで共通

1. **Orientation** — **必ず `file` または `directory` ステップにする。content-only は不可。**
   `"file": "README.md", "line": 1` か `"directory": "src"` を使い、歓迎文は description に書く。
   先頭ステップが content-only（`file`・`directory`・`uri` なし）だと VS Code CodeTour で空白ページになります。これは既知の拡張機能挙動で、設定変更できません。

2. **High-level map**（1〜3 の directory または uri ステップ）— 主要モジュールと相互関係を示す。
   すべてのフォルダではなく、このペルソナが知るべき範囲に絞る。

3. **Core path**（file/line、selection、pattern、uri ステップ）— 本当に重要なコードへ導く。
   ここがツアーの中心。実際に読んで語る。流し見しない。

4. **Closing**（content）— 読者が今わかったこと、次にできること、
   2〜3件の推奨フォローアップツアーを示す。`nextTour` を設定した場合はここで名前を明示する。

### Closing ステップ

要約は不要です（読者は今読んだばかり）。代わりに、今すぐ *何ができるか*、何を避けるべきか、2〜3件の次ツアーを提示してください。

---

## 20 のペルソナ

| Persona | Goal | Must cover | Avoid |
|---------|------|------------|-------|
| **Vibecoder** | 雰囲気を素早く掴む | エントリポイント、リクエストフロー、主要モジュール。最大8ステップ。 | 深掘り、エッジケース |
| **New joiner** | 構造化された立ち上がり | ディレクトリ、セットアップ、業務文脈、サービス境界。 | 高度な内部実装 |
| **Bug fixer** | 根本原因を素早く掴む | ユーザー操作 → トリガー → 故障点。再現ヒント + テスト位置。 | アーキテクチャ解説 |
| **RCA investigator** | なぜ失敗したかを解明 | 因果連鎖、副作用、競合条件、可観測性。 | ハッピーパス |
| **Feature explainer** | 1機能を端から端まで説明 | UI → API → backend → storage。feature flag、エッジケース。 | 無関係な機能 |
| **PR reviewer** | 変更を正しくレビュー | 変更のストーリー、不変条件、リスク領域、レビューチェックリスト。PR 用 URI ステップ。 | 無関係な文脈 |
| **Security reviewer** | 信頼境界を評価 | 認証フロー、入力検証、秘密情報処理、機密シンク。 | 無関係な業務ロジック |
| **Refactorer** | 安全に再構成 | 継ぎ目、隠れ依存、結合ホットスポット、安全な抽出順。 | 機能説明 |
| **External contributor** | 壊さず貢献 | 安全領域、コードスタイル、アーキテクチャ上の地雷。 | 深い内部実装 |
| **Tech lead / architect** | 形と根拠を把握 | モジュール境界、設計トレードオフ、リスクホットスポット。 | 行単位ウォークスルー |

---

## ツアーシリーズの設計

コードベースが複雑で1本では不十分な場合、シリーズ設計にします。
`nextTour` フィールドで連結すると、読者が1本終えたときに VS Code が次ツアーの起動を提案します。

**ツアーを書く前に、シリーズ全体を設計してください。** 良いシリーズは次を満たします:
- 明確な段階上昇（広い → 狭い、導入 → 深掘り）
- ツアー間で重複ステップがない
- 各ツアーが単体でも有用

各ツアーの `nextTour` は次ツアーの `title` に設定します（完全一致必須）。各ツアーは単体でも使えるようにしてください。

---

## CodeTour でできないこと

次の要望を受けたら、未対応であることを明確に伝えてください。存在しない回避策を提案してはいけません。

| Request | Reality |
|---|---|
| **X 秒後に自動で次ステップへ進める** | 未対応。遷移は常に手動で、読者が Next を押します。CodeTour にタイマー、遅延、自動再生ステップ機能はありません。 |
| **ステップに動画や GIF を埋め込む** | 未対応。description は Markdown テキストのみです。 |
| **任意のシェルコマンドを実行する** | 未対応。`commands` が実行できるのは VS Code コマンド（例: `workbench.action.terminal.focus`）のみで、シェルコマンドは実行できません。 |
| **分岐 / 条件付きで次ステップを変える** | 未対応。ツアーは線形です。`when` はツアー表示可否の制御であり、ステップ遷移制御ではありません。 |
| **ファイルを開かずにステップ表示する** | 部分的には可能。content-only ステップ自体は動きますが、1ステップ目は `file` か `directory` アンカーが必要で、ないと VS Code は空白ページを表示します。 |

---

## アンチパターン

| Anti-pattern | Fix |
|---|---|
| **ファイル一覧化** — 「このファイルには...」だけで巡回する | 物語として構成する。各ステップが前ステップに依存するようにする |
| **汎用的すぎる description** | *この* コードベース固有のパターン/落とし穴を明示する |
| **行番号の推測** | 実ファイルを読んで検証していない行番号は絶対に書かない |
| **ペルソナ無視** | その目標に寄与しないステップはすべて削る |
| **存在しないファイルの幻覚** | ファイルがなければステップを省く |

---

## 品質チェックリスト — ファイルを書き出す前に検証

- [ ] すべての `file` パスが **リポジトリルート相対**（先頭 `/` や `./` なし）
- [ ] すべての `file` パスを読み、存在確認済み
- [ ] すべての `line` 番号を実ファイル読解で検証済み（推測禁止）
- [ ] すべての `directory` が **リポジトリルート相対** かつ存在確認済み
- [ ] すべての `pattern` 正規表現が実ファイル内の行に一致する
- [ ] すべての `uri` が完全で実在する URL（https://...）
- [ ] `ref` を設定した場合、実在するブランチ/タグ/コミットである
- [ ] `nextTour` を設定した場合、別の `.tour` ファイルの `title` と完全一致する
- [ ] 作成したのは `.tour` JSON ファイルのみ（ソースコード未変更）
- [ ] 先頭ステップに `file` か `directory` アンカーがある（content-only 先頭 = VS Code で空白ページ）
- [ ] ツアー末尾が closing content ステップで、読者が次に *何をできるか* を示している
- [ ] すべての description が SMIG（Situation, Mechanism, Implication, Gotcha）を満たす
- [ ] ペルソナの優先事項でステップ選定している（目標に寄与しないものは削除）
- [ ] ステップ数が要求深さとリポジトリ規模に合っている（キャリブレーション表を参照）
- [ ] content-only ステップは最大2つ（導入 + 締め）
- [ ] すべてのフィールドが `references/codetour-schema.json` に準拠

---

## Step 5: ツアーを検証する

**ツアーファイル作成直後に、必ずバリデーターを実行してください。この手順は省略禁止です。**

```bash
python ~/.agents/skills/code-tour/scripts/validate_tour.py .tours/<name>.tour --repo-root .
```

バリデーターの検証項目:
- JSON の妥当性
- すべての `file` パスの存在、すべての `line` がファイル範囲内であること
- すべての `directory` の存在
- すべての `pattern` 正規表現がコンパイル可能で、対象ファイルの少なくとも1行に一致すること
- すべての `uri` が `https://` で始まること
- `nextTour` が `.tours/` 内の既存ツアータイトルと一致すること
- content-only ステップ数（> 2 の場合は警告）
- ナラティブ構成（orientation または closing がない場合は警告）

**エラーはすべて修正してから先に進んでください。** ✓ になるか、警告のみになるまで再実行します。警告は助言なので最終判断はあなたが行います。検証に通る前にユーザーへツアーを見せないでください。

**VS Code で起きやすい問題:** 先頭が content-only だと空白表示（file/directory でアンカーする）。絶対パスや `./` 付きパスは黙って失敗。範囲外行番号はどこにもスクロールしない。

スクリプトを実行できない場合は手動検証: 先頭が `file`/`directory`、全パスが実在、全行番号が範囲内、`nextTour` が完全一致。

**Autoplay:** `isPrimary: true` + `.vscode/settings.json` の `{ "codetour.promptForPrimaryTour": true }` で、リポジトリを開いたときにプロンプト表示されます。どのブランチでも表示したいツアーは `ref` を省略します。

**Share:** 公開リポジトリなら、`https://vscode.dev/github.com/<owner>/<repo>` でインストールなしにツアーを開けます。

---

## Step 6: 要約する

ツアー作成後、ユーザーへ次を伝えます:
- ファイルパス（`.tours/<name>.tour`）
- そのツアーの対象者とカバー範囲を1段落で要約
- リポジトリが公開なら `vscode.dev` URL（すぐ共有できる）
- 推奨フォローアップツアー 2〜3件（またはシリーズ設計済みなら次ツアー）
- ユーザー指定で存在しなかったファイル（明示する。黙って差し替えない）

---

## ファイル命名

`<persona>-<focus>.tour` — kebab-case で、次の両方が伝わること:
```
onboarding-new-joiner.tour
bug-fixer-payment-flow.tour
architect-overview.tour
vibecoder-quickstart.tour
pr-review-auth-refactor.tour
security-auth-boundaries.tour
concept-dependency-injection.tour
rca-login-outage.tour
```
