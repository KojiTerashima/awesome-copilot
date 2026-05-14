---
description: 'GitHub Copilot 向けの高品質な prompt file を作成するためのガイドライン'
applyTo: '**/*.prompt.md'
---

# Copilot Prompt File ガイドライン

あらゆる repository で GitHub Copilot が一貫して高品質な成果を出せるように導く、効果的で保守しやすい prompt file を作成するための instruction です。

## スコープと原則
- 対象読者: Copilot Chat 向けの再利用可能な prompt を作成する maintainer と contributor。
- 目標: 予測可能な振る舞い、明確な期待値、最小権限、repository 間での可搬性。
- 主な参照先: prompt file に関する VS Code ドキュメントと、組織固有の規約。

## Frontmatter 要件

すべての prompt file には、次の field を持つ YAML frontmatter を含める必要があります。

### 必須 / 推奨 field

| Field | Required | Description |
|-------|----------|-------------|
| `description` | 推奨 | prompt の短い説明 (1 文、実行可能な結果を示す) |
| `name` | 任意 | chat で `/` を入力した後に表示される名前。未指定時は filename が既定 |
| `agent` | 推奨 | 使用する agent: `ask`、`edit`、`agent`、または custom agent 名。既定では現在の agent |
| `model` | 任意 | 使用する language model。既定では現在選択中の model |
| `tools` | 任意 | この prompt で利用可能な tool / tool set 名の一覧 |
| `argument-hint` | 任意 | chat input に表示され、ユーザー操作を案内する hint text |

### ガイドライン

- 一貫した quoting を使い (single quote 推奨)、可読性と version control 上の明確さのため 1 行につき 1 field にする
- `tools` が指定され、現在の agent が `ask` または `edit` の場合、既定 agent は `agent` になる
- 組織で必要な追加 metadata (`language`、`tags`、`visibility` など) は保持する

## ファイル名と配置
- filename は kebab-case で `.prompt.md` で終わるようにし、workspace 標準で別の directory が指定されていない限り `.github/prompts/` 配下に保存する
- 行う操作が伝わる短い filename を付ける (例: `prompt1.prompt.md` ではなく `generate-readme.prompt.md`)

## 本文構成
- Quick Pick search で見つけやすいよう、prompt の意図に一致する `#` レベルの見出しで始める
- 予測可能な section で内容を整理する。推奨される基本構成は `Mission` または `Primary Directive`、`Scope & Preconditions`、`Inputs`、`Workflow` (段階的手順)、`Output Expectations`、`Quality Assurance`
- section 名はドメインに合わせて調整してよいが、論理の流れは維持する: なぜ → 文脈 → 入力 → 操作 → 出力 → 検証
- discoverability を高めるため、関連する prompt や instruction file を相対リンクで参照する

## 入力とコンテキストの扱い
- 必須値には `${input:variableName[:placeholder]}` を使い、ユーザーが入力すべき場面を説明する。可能なら既定値や代替手段も示す
- `${selection}`、`${file}`、`${workspaceFolder}` のような contextual variable は本当に必要な場合のみ明記し、Copilot がそれをどう解釈すべきか説明する
- 必須コンテキストが欠けている場合の進め方を文書化する (例: 「file path を要求し、未定義のままなら停止する」)

## Tool と権限のガイダンス
- `tools` は、タスク達成に必要な最小集合に限定する。順序が重要な場合は、望ましい実行順に列挙する
- prompt が chat mode から tool を継承する場合はその関係を明記し、重要な tool の挙動や副作用を説明する
- 破壊的な操作 (file 作成、編集、terminal command) については警告し、workflow に guard rail や確認手順を含める

## Instruction のトーンとスタイル
- Copilot に向けた直接的な命令文で書く (例: 「Analyze」、「Generate」、「Summarize」)
- localization を支援するため、Google Developer Documentation の翻訳 best practice に従い、文は短く曖昧さをなくす
- 慣用句、ユーモア、文化依存の表現は避け、中立で包括的な言葉を優先する

## 出力定義
- 期待する結果の形式、構造、出力先を明示する (例: 「以下の template を使って `docs/adr/adr-XXXX.md` を作成する」)
- Copilot が停止または再試行すべき条件が分かるよう、成功条件と失敗トリガーを含める
- prompt 実行後に reviewer が実施できる検証手順 (手動確認、automated command、acceptance criteria の一覧) を提供する

## 例と再利用可能な資産
- prompt が生成または従うべき Good / Bad の例や scaffold (Markdown template、JSON stub) を埋め込む
- self-contained に保つため、参照表 (capability、status code、role description) は本文内に保持する。上流のリソースが変わったらこれらの表も更新する
- 長い説明を複製するのではなく、権威あるドキュメントへリンクする

## 品質保証チェックリスト
- [ ] Frontmatter field が完全、正確で、最小権限になっている
- [ ] 入力には placeholder、既定の挙動、fallback が含まれている
- [ ] Workflow が準備、実行、後処理を抜けなくカバーしている
- [ ] 出力要件に書式と保存先の詳細が含まれている
- [ ] 検証手順が実行可能である (command、diff check、review prompt)
- [ ] prompt が参照する security、compliance、privacy policy が最新である
- [ ] prompt が代表的なシナリオで VS Code (`Chat: Run Prompt`) 上で正常に実行できる

## 保守ガイダンス
- prompt は影響を受けるコードと一緒に version control し、依存関係、tooling、review process が変わったら更新する
- tool 一覧、model 要件、リンク先ドキュメントが有効なままであることを確認するため、prompt を定期的に見直す
- 他の repository と連携し、広く有用だと分かった prompt は共通 instruction file や共有 prompt pack に抽出する

## 追加リソース
- [Prompt Files Documentation](https://code.visualstudio.com/docs/copilot/customization/prompt-files#_prompt-file-format)
- [Awesome Copilot Prompt Files](https://github.com/github/awesome-copilot/tree/main/prompts)
- [Tool Configuration](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode#_agent-mode-tools)
