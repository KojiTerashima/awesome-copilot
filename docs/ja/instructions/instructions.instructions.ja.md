---
description: 'GitHub Copilot 向けの高品質なカスタム instruction ファイルを作成するためのガイドライン'
applyTo: '**/*.instructions.md'
---

# Custom Instructions ファイル ガイドライン

GitHub Copilot がドメイン固有のコード生成やプロジェクト規約への追従を行えるように導く、効果的で保守しやすい custom instruction ファイルを作成するための指針です。

## プロジェクト コンテキスト

- 対象読者: ドメイン固有コードを扱う開発者と GitHub Copilot
- ファイル形式: YAML frontmatter を含む Markdown
- ファイル命名規則: 小文字とハイフン (例: `react-best-practices.instructions.md`)
- 配置場所: `.github/instructions/` ディレクトリ
- 目的: コード生成、レビュー、ドキュメント作成のためのコンテキスト認識型ガイダンスを提供する

## 必須 frontmatter

すべての instruction ファイルは、次のフィールドを持つ YAML frontmatter を含める必要があります。

```yaml
---
description: 'instruction の目的と適用範囲の簡潔な説明'
applyTo: '対象ファイル向け glob pattern (例: **/*.ts, **/*.py)'
---
```

### frontmatter のガイドライン

- **description**: 単一引用符付き文字列、1〜500 文字、目的を明確に示すこと
- **applyTo**: どのファイルにこれらの instruction を適用するかを示す glob pattern
  - 単一パターン: `'**/*.ts'`
  - 複数パターン: `'**/*.ts, **/*.tsx, **/*.js'`
  - 特定ファイル: `'src/**/*.py'`
  - 全ファイル: `'**'`

## ファイル構成

よく構成された instruction ファイルには、次のセクションを含めるべきです。

### 1. タイトルと概要

- `#` 見出しを使った明確で説明的なタイトル
- 目的と範囲を説明する簡潔な導入
- 任意: 主要技術やバージョンを記したプロジェクト コンテキスト セクション

### 2. 中核セクション

ドメインに応じて、内容を論理的なセクションへ整理します。

- **一般的な instruction**: 高レベルなガイドラインと原則
- **ベストプラクティス**: 推奨パターンとアプローチ
- **コード標準**: 命名規則、フォーマット、スタイル ルール
- **アーキテクチャ/構造**: プロジェクト構成とデザイン パターン
- **一般的なパターン**: 頻出の実装
- **セキュリティ**: セキュリティ上の考慮事項 (該当する場合)
- **パフォーマンス**: 最適化ガイドライン (該当する場合)
- **テスト**: テスト標準とアプローチ (該当する場合)

### 3. 例とコード スニペット

明確なラベル付きで具体例を示します。

```markdown
### Good Example
\`\`\`language
// 推奨アプローチ
code example here
\`\`\`

### Bad Example
\`\`\`language
// このパターンは避ける
code example here
\`\`\`
```

### 4. 検証と確認 (任意だが推奨)

- コードを検証する build コマンド
- Lint と format ツール
- テスト要件
- 確認手順

## コンテンツ ガイドライン

### 文体

- 明確で簡潔な言葉を使う
- 命令形で書く ("Use", "Implement", "Avoid")
- 具体的かつ実行可能にする
- "should"、"might"、"possibly" のような曖昧な語は避ける
- 可読性のために箇条書きやリストを使う
- セクションは焦点を絞り、流し読みしやすく保つ

### ベストプラクティス

- **具体的に書く**: 抽象概念より具体例を示す
- **理由を示す**: 価値がある場合は、推奨の背景を説明する
- **表を使う**: 選択肢比較、ルール列挙、パターン提示に使う
- **例を含める**: 実際のコード スニペットは説明だけより有効
- **最新性を保つ**: 現在のバージョンとベストプラクティスを参照する
- **リソースへリンクする**: 公式ドキュメントや信頼できるソースを含める

### instruction の粒度 (ちょうど良いレベル)

- 期待する結果を完全に定義できる最小限のルール セットから始める
- 制約は仮定上のエッジケースではなく、実際に観測した失敗のあとで追加する
- 網羅的な判断表より、高密度の良い例を優先する

| 粒度 | 失敗モード | 結果 |
| --- | --- | --- |
| 過剰指定 | 壊れやすい if-else 的な文章 | 列挙外のケースで破綻する |
| 指定不足 | 共有コンテキストを前提にする | 一般論的な出力になる |
| 適切な粒度 | ヒューリスティック + 例 | 安定して再利用しやすい品質 |

### 含めるべき一般的なパターン

1. **命名規則**: 変数、関数、クラス、ファイルの命名方法
2. **コード構成**: ファイル構造、モジュール構成、import 順序
3. **エラー処理**: 推奨されるエラー処理パターン
4. **依存関係**: 依存関係の管理方法と文書化方法
5. **コメントとドキュメント**: コードをいつ、どのように文書化するか
6. **バージョン情報**: 対象言語/フレームワークのバージョン

## 従うべきパターン

### 箇条書きとリスト

```markdown
## Security Best Practices

- 常に処理前にユーザー入力を検証する
- SQL インジェクションを防ぐため、パラメーター化クエリを使う
- シークレットはコードではなく環境変数に保存する
- 適切な認証と認可を実装する
- すべての本番エンドポイントで HTTPS を有効にする
```

### 構造化情報には表を使う

```markdown
## Common Issues

| Issue            | Solution            | Example                       |
| ---------------- | ------------------- | ----------------------------- |
| Magic numbers    | Use named constants | `const MAX_RETRIES = 3`       |
| Deep nesting     | Extract functions   | Refactor nested if statements |
| Hardcoded values | Use configuration   | Store API URLs in config      |
```

### コード比較

```markdown
### Good Example - TypeScript interface を使う
\`\`\`typescript
interface User {
  id: string;
  name: string;
  email: string;
}

function getUser(id: string): User {
  // Implementation
}
\`\`\`

### Bad Example - any 型を使う
\`\`\`typescript
function getUser(id: any): any {
  // 型安全性を失う
}
\`\`\`
```

### 条件付きガイダンス

```markdown
## Framework Selection

- **小規模プロジェクト向け**: Minimal API アプローチを使う
- **大規模プロジェクト向け**: 明確な分離を持つ controller ベース アーキテクチャを使う
- **マイクロサービス向け**: domain-driven design パターンを検討する
```

## 避けるべきパターン

- **冗長すぎる説明**: 簡潔で流し読みしやすく保つ
- **古い情報**: 常に最新のバージョンと実践を参照する
- **曖昧なガイドライン**: 何をするか、何を避けるかを具体的に示す
- **例の欠如**: 具体的なコードのない抽象ルール
- **矛盾した助言**: ファイル全体で一貫性を保つ
- **ドキュメントの単なるコピペ**: 要点を絞り、文脈化して価値を加える
- **仮想失敗に基づくルール膨張**: 実際に起きていない失敗のためにルールを足さない

## instruction をテストする

instruction ファイルを確定する前に:

1. **Copilot で試す**: VS Code で実際のプロンプトに対して instruction を試す
2. **例を検証する**: コード例が正しく、エラーなく動作することを確認する
3. **Glob パターンを確認する**: `applyTo` パターンが意図したファイルに一致することを確かめる

## 構成例

新しい instruction ファイルの最小構成例を示します。

```markdown
---
description: '目的の簡潔な説明'
applyTo: '**/*.ext'
---

# Technology Name Development

簡潔な導入とコンテキスト。

## General Instructions

- 高レベル ガイドライン 1
- 高レベル ガイドライン 2

## Best Practices

- 具体的な実践 1
- 具体的な実践 2

## Code Standards

### Naming Conventions
- ルール 1
- ルール 2

### File Organization
- 構成 1
- 構成 2

## Common Patterns

### Pattern 1
説明と例

\`\`\`language
code example
\`\`\`

### Pattern 2
説明と例

## Validation

- Build command: `command to verify`
- Linting: `command to lint`
- Testing: `command to test`
```

## メンテナンス

- 依存関係やフレームワークが更新されたら instruction を見直す
- 例を現在のベストプラクティスに合わせて更新する
- 古いパターンや非推奨機能を削除する
- コミュニティで現れてきた新しいパターンを追加する
- プロジェクト構造の変化に合わせて glob パターンを正確に保つ

## 追加リソース

- [Custom Instructions Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Awesome Copilot Instructions](https://github.com/github/awesome-copilot/tree/main/instructions)
- [System Prompt Altitude — Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents#the-anatomy-of-effective-context)
