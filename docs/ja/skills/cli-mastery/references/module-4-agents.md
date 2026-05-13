# モジュール 4: エージェントシステム

## 組み込みエージェント

| Agent | Model | 最適な用途 | 主な特性 |
|-------|-------|----------|-----------|
| `explore` | Haiku | 高速なコードベース Q&A | 読み取り専用、300 語未満、並列化しても安全 |
| `task` | Haiku | コマンド実行（テスト、ビルド、lint） | 成功時は簡潔、失敗時は詳しい |
| `general-purpose` | Sonnet | 複雑な多段階タスク | フルツールセット、独立したコンテキストウィンドウ |
| `code-review` | Sonnet | コード変更の分析 | コードは絶対に変更しない、高い S/N 比 |

## カスタムエージェント: Markdown で自分で定義する

| Level | Location | Scope |
|-------|----------|-------|
| Personal | `~/.copilot/agents/*.md` | 自分の全プロジェクト |
| Project | `.github/agents/*.md` | このリポジトリの全員 |
| Organization | `.github-private/agents/` in org repo | 組織全体 |

## エージェントファイルの構成

```markdown
---
name: my-agent
description: What this agent does
tools:
  - bash
  - edit
  - view
---

# Agent Instructions
Your detailed behavior instructions here.
```

## エージェントのオーケストレーションパターン

1. **ファンアウト探索** — 複数の `explore` エージェントを並列起動し、別々の質問に同時に答えさせる
2. **パイプライン** — `explore` → 理解 → `general-purpose` → 実装 → `code-review` → 検証
3. **専門家への引き継ぎ** — タスクを特定 → `/agent` で専門家を選択 → `/fleet` または `/tasks` でレビュー

重要な洞察: AI は適切な場合、自動的にサブエージェントへ委譲する。
