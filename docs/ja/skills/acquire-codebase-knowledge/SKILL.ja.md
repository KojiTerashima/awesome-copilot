---
name: acquire-codebase-knowledge
description: 'ユーザーが既存コードベースのマッピング、ドキュメント化、オンボーディングを明示的に求めたときにこのスキルを使用します。"map this codebase"、"document this architecture"、"onboard me to this repo"、"create codebase docs" のようなプロンプトで起動します。ユーザーがリポジトリ全体レベルの調査を求めていない限り、通常の機能実装・バグ修正・限定的なコード編集では起動しないでください。'
license: MIT
compatibility: 'クロスプラットフォーム。Python 3.8+ と git が必要です。対象プロジェクトのルートで scripts/scan.py を実行してください。'
metadata:
  version: "1.3"
  enhancements:
    - 多言語マニフェスト検出（25 以上の言語をサポート）
    - CI/CD パイプライン検出（10 以上のプラットフォーム）
    - コンテナおよびオーケストレーション検出
    - 言語別コードメトリクス
    - セキュリティおよびコンプライアンス設定検出
    - パフォーマンステストのマーカー検出
argument-hint: '任意: 注力する領域（例: "architecture only"、"testing and concerns"）'
---

# Acquire Codebase Knowledge

`docs/codebase/` に、プロジェクトで効果的に作業するために必要な内容を網羅した 7 つのドキュメントを生成します。ファイルまたはターミナル出力で検証可能な内容のみを記述し、推測や仮定は絶対に行わないでください。

## Output Contract (Required)

完了前に、以下がすべて真である必要があります。

1. `docs/codebase/` に次のファイルが**正確に**存在すること: `STACK.md`, `STRUCTURE.md`, `ARCHITECTURE.md`, `CONVENTIONS.md`, `INTEGRATIONS.md`, `TESTING.md`, `CONCERNS.md`。
2. すべての主張が、ソースファイル・設定・またはターミナル出力に追跡可能であること。
3. 不明点は `[TODO]` で示し、意図依存の判断は `[ASK USER]` で示すこと。
4. すべてのドキュメントに、具体的なファイルパスを含む短い「evidence」リストがあること。
5. 最終応答に、番号付きの `[ASK USER]` 質問と、intent と reality の乖離を含めること。

## Workflow

このチェックリストをコピーして進捗管理してください:

```
- [ ] フェーズ1: スキャン実行、意図ドキュメントの読解
- [ ] フェーズ2: 各ドキュメント領域を調査
- [ ] フェーズ3: docs/codebase/ の7ドキュメントをすべて作成
- [ ] フェーズ4: ドキュメントを検証し、所見を提示し、すべての [ASK USER] 項目を解消
```

## Focus Area Mode

ユーザーが注力領域（例: "architecture only" や "testing and concerns"）を指定した場合:

1. フェーズ1は常に完全実行する。
2. 注力領域のドキュメントを最優先で完全に仕上げる。
3. まだ分析していない非注力ドキュメントは、必須セクションを残したうえで不明点を `[TODO]` として記載する。
4. 最終出力前に、7 ドキュメントすべてに対してフェーズ4の検証ループを実行する。

### Phase 1: Scan and Read Intent

1. 対象プロジェクトのルートでスキャンスクリプトを実行します:
   ```bash
   python3 "$SKILL_ROOT/scripts/scan.py" --output docs/codebase/.codebase-scan.txt
   ```
   ここで `$SKILL_ROOT` はスキルフォルダの絶対パスです。Windows、macOS、Linux で動作します。

   **クイックスタート:** パスを直接書ける場合:
   ```bash
   python3 /absolute/path/to/skills/acquire-codebase-knowledge/scripts/scan.py --output docs/codebase/.codebase-scan.txt
   ```

2. `PRD`, `TRD`, `README`, `ROADMAP`, `SPEC`, `DESIGN` ファイルを検索して読みます。
3. ソースコードを読む前に、明示されたプロジェクト意図を要約します。

### Phase 2: Investigate

スキャン出力を使って、7 つのテンプレートそれぞれの問いに答えてください。テンプレートごとの質問一覧は [`references/inquiry-checkpoints.md`](references/inquiry-checkpoints.md) を読み込んで確認します。

スタックが曖昧な場合（マニフェストファイルが複数ある、見慣れないファイル種別、`package.json` がない等）は、[`references/stack-detection.md`](references/stack-detection.md) を読み込んでください。

### Phase 3: Populate Templates

`assets/templates/` から各テンプレートを `docs/codebase/` にコピーし、次の順で埋めます:

1. [STACK.md](assets/templates/STACK.md) — 言語、ランタイム、フレームワーク、すべての依存関係
2. [STRUCTURE.md](assets/templates/STRUCTURE.md) — ディレクトリ構成、エントリーポイント、主要ファイル
3. [ARCHITECTURE.md](assets/templates/ARCHITECTURE.md) — レイヤー、パターン、データフロー
4. [CONVENTIONS.md](assets/templates/CONVENTIONS.md) — 命名、フォーマット、エラーハンドリング、import
5. [INTEGRATIONS.md](assets/templates/INTEGRATIONS.md) — 外部 API、データベース、認証、監視
6. [TESTING.md](assets/templates/TESTING.md) — フレームワーク、ファイル構成、モック戦略
7. [CONCERNS.md](assets/templates/CONCERNS.md) — 技術的負債、バグ、セキュリティリスク、性能ボトルネック

コードから判断できない内容には `[TODO]` を使います。正しい答えにチーム意図が必要な箇所には `[ASK USER]` を使います。

### Phase 4: Validate, Repair, Verify

最終化前に、以下の必須検証ループを実行します:

1. 各ドキュメントを `references/inquiry-checkpoints.md` に照らして検証する。
2. 重要な主張ごとに、少なくとも 1 つの evidence 参照があることを確認する。
3. 必須セクションの欠落や根拠不足がある場合:
  - ドキュメントを修正する。
  - 再検証する。
4. 7 ドキュメントすべてが通過するまで繰り返す。

その後、7 ドキュメント全体の要約を提示し、すべての `[ASK USER]` 項目を番号付き質問として列挙し、フェーズ1での Intent と Reality の乖離を強調してください。

検証の合格基準:

- 根拠のない主張がないこと。
- 必須セクションに空欄がないこと。
- 不明点を仮定で埋めず、`[TODO]` を使っていること。
- チーム意図の不足箇所が `[ASK USER]` で明示されていること。

---

## Gotchas

**モノレポ:** ルートの `package.json` にソースがない場合があります。`workspaces`、`packages/`、`apps/` ディレクトリを確認してください。各ワークスペースは依存関係や規約が独立している可能性があるため、サブパッケージごとに個別にマッピングします。

**古い README:** README には現状ではなく意図されたアーキテクチャが書かれていることがよくあります。README の記述を事実として扱う前に、実際のファイル構造と必ず突き合わせてください。

**TypeScript のパスエイリアス:** `tsconfig.json` の `paths` 設定があると、`@/foo` のような import はファイルシステム上のパスに直接対応しません。構造を記述する前に、エイリアスを実パスへ対応付けてください。

**生成物/ビルド成果物:** `dist/`, `build/`, `generated/`, `.next/`, `out/`, `__pycache__/` からパターンを記述してはいけません。これらは成果物です。ソース側の規約のみを記述してください。

**`.env.example` は必要設定を示す:** シークレットはコミットされません。必要な環境変数を把握するため、`.env.example`、`.env.template`、`.env.sample` を確認してください。

**`devDependencies` ≠ 本番スタック:** 本番で動くのは `dependencies`（または同等のもの、例: `[tool.poetry.dependencies]`）だけです。リンター、フォーマッター、テストフレームワークは開発ツールとして分けて記述してください。

**テスト内 TODO ≠ 本番の負債:** `test/`, `tests/`, `__tests__/`, `spec/` 内の TODO は本番の技術的負債ではなく、カバレッジ不足です。`CONCERNS.md` では分けて扱ってください。

**更新頻度の高いファイル = 壊れやすい領域:** 直近の git 履歴で頻出するファイルは変更率が高く、隠れた複雑性を抱えている可能性があります。`CONCERNS.md` に必ず記載してください。

---

## Anti-Patterns

| ❌ Don't | ✅ Do instead |
|---------|--------------|
| "Uses Clean Architecture with Domain/Data layers." (when no such directories exist) | ディレクトリ構造が実際に示している内容だけを記述する。 |
| "This is a Next.js project." (without checking `package.json`) | まず `dependencies` を確認し、実際に存在するものを記述する。 |
| Guess the database from a variable name like `dbUrl` | `pg`、`mysql2`、`mongoose`、`prisma` などをマニフェストで確認する。 |
| Document `dist/` or `build/` naming patterns as conventions | ソースファイルのみを対象にする。 |

---

## Enhanced Scan Output Sections

`scan.py` スクリプトは、従来の出力に加えて次のセクションも出力するようになりました:

- **CODE METRICS** — 総ファイル数、言語別コード行数、最大ファイル（複雑性シグナル）
- **CI/CD PIPELINES** — GitHub Actions、GitLab CI、Jenkins、CircleCI などの検出結果
- **CONTAINERS & ORCHESTRATION** — Docker、Docker Compose、Kubernetes、Vagrant 設定
- **SECURITY & COMPLIANCE** — Snyk、Dependabot、SECURITY.md、SBOM、セキュリティポリシー
- **PERFORMANCE & TESTING** — ベンチマーク設定、プロファイリングマーカー、負荷テストツール

フェーズ2では、これらのセクションを調査質問の補助として使い、ツール固有のパターンを特定してください。

---

## Bundled Assets

| Asset | When to load |
|-------|-------------|
| [`scripts/scan.py`](scripts/scan.py) | フェーズ1 — コードを読む前に最初に実行（Python 3.8+ 必須） |

| [`references/inquiry-checkpoints.md`](references/inquiry-checkpoints.md) | フェーズ2 — テンプレートごとの調査質問を確認するために読み込む |
| [`references/stack-detection.md`](references/stack-detection.md) | フェーズ2 — スタックが曖昧な場合のみ |
| [`assets/templates/STACK.md`](assets/templates/STACK.md) | フェーズ3 ステップ1 |
| [`assets/templates/STRUCTURE.md`](assets/templates/STRUCTURE.md) | フェーズ3 ステップ2 |
| [`assets/templates/ARCHITECTURE.md`](assets/templates/ARCHITECTURE.md) | フェーズ3 ステップ3 |
| [`assets/templates/CONVENTIONS.md`](assets/templates/CONVENTIONS.md) | フェーズ3 ステップ4 |
| [`assets/templates/INTEGRATIONS.md`](assets/templates/INTEGRATIONS.md) | フェーズ3 ステップ5 |
| [`assets/templates/TESTING.md`](assets/templates/TESTING.md) | フェーズ3 ステップ6 |
| [`assets/templates/CONCERNS.md`](assets/templates/CONCERNS.md) | フェーズ3 ステップ7 |

テンプレート利用モード:

- デフォルトモード: 各テンプレートの「Core Sections (Required)」のみを完成させる。
- 拡張モード: リポジトリの複雑性が妥当な場合にのみ、任意セクションを追加する。

