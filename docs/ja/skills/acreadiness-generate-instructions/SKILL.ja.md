---
name: acreadiness-generate-instructions
description: 'AgentRC 指示コマンドを使用して、カスタマイズされた AI エージェント指示ファイルを生成します。 .github/copilot-instructions.md (デフォルト、VS Code の Copilot に推奨) と、モノリポジトリの applyTo グロブを含むオプションのエリアごとの .instructions.md ファイルを生成します。 /acreadiness-assess の実行後に使用して、AI ツールの柱のギャップを埋めます。'
argument-hint: "[--output .github/copilot-instructions.md|AGENTS.md] [--strategy flat|nested] [--areas | --area <name>] [--apply-to <glob>] [--claude-md] [--dry-run]"
---

# /acreadiness-generate-instructions — AI エージェントの命令を作成します

ユーザーが AI コーディング エージェント (Copilot、Claude など) のカスタム命令を**作成**、**再生成**、**更新**したいときは常に、このスキルを使用します。これは、AgentRC の **測定 → 生成 → 維持** ループの *生成* ステップであり、**AI ツール** の柱にとって最も活用度の高い単一のアクションです。

## 出力オプション

VS Code はいくつかの命令ファイル タイプを認識します。AgentRC は最も一般的なものを生成します。

|ファイル |範囲 |いつ使用するか |
|---|---|---|
| `.github/copilot-instructions.md` |ワークスペース全体で常時稼働 | **デフォルト** — VS Code Copilot のネイティブ命令ファイル |
| `AGENTS.md` |ワークスペース全体で常時稼働 |マルチエージェント リポジトリ (Copilot + Claude + その他) |
| `.github/instructions/*.instructions.md` | `applyTo` glob によってスコープ指定される |モノリポジトリのエリアごと/言語ごとのルール |
| `CLAUDE.md` |クロード固有 | `--claude-md` 経由で追加 (ネストされた場合のみ) |

## 戦略

- **`flat`** *(デフォルト)* — 選択したパスに単一の `.github/copilot-instructions.md`。シンプルでレビューしやすい。
- **`nested`** — `.github/copilot-instructions.md` のハブ + `.github/instructions/<topic>.instructions.md` のトピックごとの詳細ファイル。それぞれに `applyTo` グロブがあるため、VS Code は関連する場合にのみトピックを読み込みます。大規模なリポジトリまたは複数スタックのリポジトリに適しています。

> **なぜ `.agents/` ではなく `.github/instructions/` なのでしょうか?** AgentRC のデフォルトのネストされたレイアウトは、*エージェントに依存しない* リポジトリ (`AGENTS.md` を読み取るコパイロット + クロード + カーソル) の正しいホームである `.agents/` に書き込みます。特に VS Code Copilot の場合、ネイティブの場所は `.github/instructions/` と `applyTo` 前置要素です。これが Copilot によって自動検出されます。このスキルは、メイン出力が `.github/copilot-instructions.md` である場合は常に、AgentRC のネストされた出力を VS Code ネイティブの場所に書き換えます。代わりに `--output AGENTS.md` を選択した場合、ネストされた場合は AgentRC のデフォルトの `.agents/` レイアウトが維持されます。

モノリポジトリの場合は、`--areas`、`--area <name>`、または `--areas-only` を使用して **エリア スコープ** 命令を生成します。エリアは`agentrc.config.json`で定義されます。エリアごとの出力は、`applyTo` グロブを持つ VS Code `.instructions.md` ファイルとして書き込まれます (以下を参照)。

### トピックとエリア `.instructions.md` ファイル

どちらも最終的には `.github/instructions/` になりますが、答えは異なります。

|種類 |ファイル名の例 | `applyTo` 例 |どこから来たのか |
|---|---|---|---|
| **トピック** (ネスト) | `testing.instructions.md` | `**/*.{test,spec}.{ts,tsx,js}` | AgentRC `--strategy nested` トピック分割 |
| **エリア** (モノレポ) | `frontend.instructions.md` | `apps/frontend/**` | `agentrc.config.json` エリア + `--areas` |

ネストされたトピック ファイルのセットとモノリポジトリのエリアごとのファイルの両方を同時に持つことができます。

## `applyTo` を含むエリアごとのファイル

ユーザーがエリアを選択すると、`.github/instructions/<area>.instructions.md` のエリアごとに 1 つの VS Code ネイティブ `.instructions.md` ファイルが出力されます。各ファイルは、ルールが適用されるグロブを宣言するfrontmatterで始まらなければなりません:
```markdown
---
applyTo: "apps/frontend/**"
---

# Frontend area instructions

…AgentRC-generated content for this area…
```

ワークフロー:

1. **`agentrc.config.json`** を読んで、宣言された領域とその `paths` / グロブを確認します。 `paths` が欠落している場合は、グロブ (例: `src/api/**`) をユーザーに尋ねます。
2. **`agentrc instructions --areas`** (または `--area <name>`) を実行して、領域ごとの本文コンテンツを生成します。
3. **各エリアのコンテンツ**を、エリアの `paths` から取得した `applyTo` フロントマターを使用して `.github/instructions/<area>.instructions.md` でラップします。ユーザーが単一エリア呼び出しで `--apply-to <glob>` を渡した場合は、その glob をそのまま使用します。
4. **メイン ファイルはそのままにしておきます** — ルート `.github/copilot-instructions.md` は常時オンの命令として残ります。 `.instructions.md` ファイルは、パスが一致する場合にのみ有効になります。

ネーミング: 小文字のケバブケースのエリア名。例: `.github/instructions/frontend.instructions.md`、`.github/instructions/api.instructions.md`、`.github/instructions/infra.instructions.md`。

## ステップ

1. **ターゲット ファイルを選択します**。 **デフォルトは `.github/copilot-instructions.md` です。** ユーザーがマルチエージェント / クロード / カーソルのサポートに言及した場合にのみ `AGENTS.md` に切り替えます。
2. **使用する戦略を必ず尋ねてください** — `flat` または `nested` — ユーザーがメッセージ内または `--strategy` 経由で既に戦略を指定している場合を除きます。トレードオフを簡単に示します。
   - **フラット** *(デフォルト)* — 1 つの `.github/copilot-instructions.md`。シンプルで、1 つの PR で簡単に確認できます。スタックが 1 つある小規模/中規模のリポジトリに最適です。
   - **ネストされた** — ハブ `.github/copilot-instructions.md` + トピックごとの `.github/instructions/<topic>.instructions.md` ファイル (それぞれに `applyTo` グロブがあるため、VS Code は関連する場合にのみそれらをロードします)。大規模なリポジトリまたは複数スタックのリポジトリに最適です。 `--claude-md` を追加すると、`CLAUDE.md` も出力されます。
リポジトリに 5 つを超えるトップレベル ディレクトリがある場合、複数のスタックがある場合、またはすでにモノリポジトリ ツール (turbo/nx/pnpm ワークスペース) を使用している場合は、`nested` を積極的に推奨します。
3. **`agentrc.config.json` を読み取ることで、モノリポジトリ領域を検出します**。エリアが存在する場合は、ルート ファイルに加えて `applyTo`** を含む **エリアごとの `.instructions.md` ファイルが必要かどうかをユーザーに尋ねます。 `agentrc.config.json` が領域を宣言する場合のデフォルトは「yes」です。
4. **ユーザーがプレビューできるように、最初に予行演習を実行します**。
   ```bash
   npx -y github:microsoft/agentrc instructions --output <file> --strategy <flat|nested> [--areas|--area <name>] [--claude-md] --dry-run
   ```
5. **変更内容の短い概要を表示**します。作成または上書きされるファイル、領域数 + それらの `applyTo` グロブ、使用されるモデル (デフォルト `claude-sonnet-4.6`)。
6. **確認時に、`--dry-run`** を付けずに同じコマンドを実行します (ファイルが既に存在する場合は、オプションで `--force` も実行します)。
7. **Copilot 出力の後処理レイアウト**:
   - **`--output` が `copilot-instructions.md` で終わり、戦略が `nested`** の場合: AgentRC の `.agents/<topic>.md` ファイルを `.github/instructions/<topic>.instructions.md` に移動/書き換えます。適切な `applyTo` glob を使用してフロントマターを各ファイルに追加します (下記の「トピック applyTo デフォルト」を参照)。空になった `.agents/` ディレクトリを削除します。
   - **`--areas` が使用されている場合**: `agentrc.config.json` からの各エリアの `paths` を `applyTo` グロブとして使用して、各エリアに `.github/instructions/<area>.instructions.md` も書き込みます (単一エリア呼び出しの場合は `--apply-to` でオーバーライドします)。
   - **`--output AGENTS.md`** が選択された場合: AgentRC のネイティブ `.agents/` レイアウトをネスト用に維持します。エージェントに依存しないリーダーはそこにあることを期待します。
`.github/instructions/` ディレクトリがない場合は作成します。

### トピック `applyTo` のデフォルト

AgentRC のネストされたトピック ファイルを `.instructions.md` にプロモートする場合は、ユーザーが別途指定しない限り、次のデフォルトを使用します。

|トピック |デフォルト `applyTo` |
|---|---|
| `testing` | `**/*.{test,spec}.{ts,tsx,js,jsx,mjs,cjs}` |
| `style` / `code-quality` / `formatting` | `**/*.{ts,tsx,js,jsx,mjs,cjs,py,go,rs,java,kt,cs}` |
| `build` / `ci` | `**/{package.json,turbo.json,nx.json,.github/workflows/**}` |
| `docs` | `**/*.md` |
| `security` | `**` |
|その他 / ハブレベル | `**` |
8. **検証**するには、生成されたファイルを読み戻し、ユーザーに 1 段落の概要 (検出されたスタック、キャプチャされた規約、長さ、グロブを含む `.instructions.md` ファイルのリスト) を表示します。
9. **次のステップを提案します**:
   - `assess` スキルを再実行して、AI Tooling ピラー スコアが改善されたことを確認します。
   - ユーザーがすでに `copilot-instructions.md` と `AGENTS.md` の両方を持っている場合は、単一の信頼できる情報源に統合することをお勧めします (AgentRC は成熟度レベル 2+ でこれにフラグを立てます)。

## 注意事項

- AgentRC は、**実際のコード**を読み取ります。テンプレートは使用しません。出力には、検出された言語、フレームワーク、および規則が反映されます。
- `--claude-md` (ネストされた戦略のみ) `CLAUDE.md` も出力されます。
- VS Code は、アクティブなファイルが `applyTo` と一致する場合、`.instructions.md` ファイルを自動的に適用します。ルート `.github/copilot-instructions.md` は常にロードされます。
- このスキルを CI で非対話的に実行しないでください。手順はリポジトリの一部であり、PR 経由で表示されるはずです。