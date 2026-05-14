---
name: playwright-generate-test
description: 'Generate a Playwright test based on a scenario using Playwright MCP'
---
# Playwright MCP によるテスト生成

目標は、規定の手順をすべて完了した後、提供されたシナリオに基づいて Playwright テストを生成することです。

## 具体的な手順

- シナリオが与えられ、それに対する劇作家テストを生成する必要があります。ユーザーがシナリオを提供しない場合は、シナリオを提供するように依頼します。
- 規定の手順をすべて完了せずに、時期尚早にテスト コードを生成したり、シナリオのみに基づいてテスト コードを生成したりしないでください。
- Playwright MCP が提供するツールを使用して、ステップを 1 つずつ実行してください。
- すべての手順が完了した後でのみ、メッセージ履歴に基づいて `@playwright/test` を使用する Playwright TypeScript テストを発行します。
- 生成されたテストファイルをtestsディレクトリに保存します
- テスト ファイルを実行し、テストに合格するまで繰り返します。