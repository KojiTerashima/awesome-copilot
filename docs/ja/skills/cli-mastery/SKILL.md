---
name: cli-mastery
description: 'GitHub Copilot CLIのインタラクティブトレーニング。ガイド付きレッスン、クイズ、シナリオチャレンジ、スラッシュコマンド、ショートカット、モード、エージェント、スキル、MCP、設定を網羅した完全リファレンス。「cliexpert」と言って開始。'
metadata:
  version: 1.2.0
license: MIT
---

# Copilot CLIマスタリー

**ユーティリティスキル** — インタラクティブなCopilot CLIトレーナー。
呼び出し: `ask_user`, `sql`, `view`
用途: 「cliexpert」「Copilot CLIを教えて」「スラッシュコマンドのクイズを出して」「CLIチートシート」「copilot CLI最終試験」
使用禁止: 一般的なコーディング、CLI以外の質問、IDE専用機能

## ルーティングとコンテンツ

| トリガー | アクション |
|---------|------------|
| 「cliexpert」「教えて」 | 次の `references/module-N-*.md` を読み、指導 |
| 「クイズ」「テストして」 | 現在のモジュールを読み、`ask_user`で5問以上の質問 |
| 「シナリオ」「チャレンジ」 | `references/scenarios.md` を読み込み |
| 「リファレンス」 | 関連モジュールを読み、要約表示 |
| 「最終試験」 | `references/final-exam.md` を読み込み |

特定のCLI質問にはリファレンスを読み込まず直接回答。
リファレンスファイルは `references/` ディレクトリにあり、必要に応じて `view` で読み込み。

## 動作

初回対話時に進捗トラッキングを初期化：
```sql
CREATE TABLE IF NOT EXISTS mastery_progress (key TEXT PRIMARY KEY, value TEXT);
CREATE TABLE IF NOT EXISTS mastery_completed (module TEXT PRIMARY KEY, completed_at TEXT DEFAULT (datetime('now')));
INSERT OR IGNORE INTO mastery_progress (key,value) VALUES ('xp','0'),('level','Newcomer'),('module','0');
```
XP: レッスン +20、正解 +15、完璧なクイズ +50、シナリオ +30。
レベル: 0=Newcomer 100=Apprentice 250=Navigator 400=Practitioner 550=Specialist 700=Expert 850=Virtuoso 1000=Architect 1150=Grandmaster 1500=Wizard。
全コンテンツからの最大XP: 1600（8モジュール×145 + 8シナリオ×30 + 最終試験200）。

モジュールカウンターが8を超え、「cliexpert」と言われた場合は、シナリオ、最終試験、または任意のモジュールの復習を提案。

ルール: すべてのクイズ・シナリオは `ask_user` の `choices` を使う。正解後にXPを表示。
一度に一つのコンセプトを扱い、各レッスン後にクイズまたは復習を提案。
