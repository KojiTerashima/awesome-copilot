---
name: polyglot-test-agent
description: 'Generates comprehensive, workable unit tests for any programming language using a multi-agent pipeline. Use when asked to generate tests, write unit tests, improve test coverage, add test coverage, create test files, or test a codebase. Supports C#, TypeScript, JavaScript, Python, Go, Rust, Java, and more. Orchestrates research, planning, and implementation phases to produce tests that compile, pass, and follow project conventions.'
---
# 多言語テスト生成スキル

調整されたマルチエージェント パイプラインを使用して、あらゆるプログラミング言語に対して包括的で実行可能な単体テストを生成する、AI を活用したスキル。

## このスキルを使用する場合

このスキルは、次の場合に使用します。
- プロジェクト全体または特定のファイルの単体テストを生成
- 既存のコードベースのテスト カバレッジを改善する
- プロジェクトの規則に従ってテスト ファイルを作成する
- 実際にコンパイルして合格するテストを作成する
- 新しい機能または未テストのコードのテストを追加します

## 仕組み

このスキルは、**調査 → 計画 → 実装** パイプラインで複数の専門エージェントを調整します。

### パイプラインの概要「」
┌───────────────────────────┐
│ テストジェネレーター │
│ パイプライン全体を調整し、状態を管理します │
━━━━━━━━━━━━━━━━━━━━━━━┘
                      │
        ┌─────────┼─────────┐
        ▼ ▼ ▼
┌───────┐ ┌───────┐ ┌─────────┐
│ 研究者│ │ プランナー │ │ 実装者 │
│ │ │ │ │ │
│ 分析 │ │ 作成 │ │ テストの作成 │
│ コードベース │→ │ 段階的 │→ │ フェーズごと │
│ │ │ 計画 │ │ │
━━━━━┘ ━━━━━┘ └───────┬───────┘
                                      │
                    ┌─────┬───────┼───────┐
                    ▼ ▼ ▼ ▼
              ┌─────┐ ┌───────┐ ┌───────┐ ┌────────┐
              │ ビルダー │ │テスター │ │ フィクサー │ │リンター │
              │ │ │ │ │ │ │
              │ コンパイル│ │ 実行 │ │ 修正 │ │フォーマット│
              │ コード │ │ テスト │ │ エラー│ │ コード │
              ━━━━┘ ━━━━━┘ ━━━━━┘ ━━━━━┘
「」## 詳しい手順

### ステップ 1: ユーザーリクエストを決定する

ユーザーが何を求めているのか、またその範囲は何かを必ず理解してください。
ユーザーがテスト スタイル、カバレッジ目標、または規約について強い要件を表明していない場合は、[unit-test-generation.prompt.md](unit-test-generation.prompt.md) からガイドラインを参照してください。このプロンプトは、規則、パラメーター化戦略、カバレッジ目標 (80% を目指す)、および言語固有のパターンを見つけるためのベスト プラクティスを提供します。

### ステップ 2: テスト ジェネレーターを起動する

まず、テスト生成リクエストを使用して `polyglot-test-generator` エージェントを呼び出します。「」
[unit-test-generation.prompt.md](unit-test-generation.prompt.md) ガイドラインに従って、[テスト対象のパスまたは説明] の単体テストを生成します。
「」テスト ジェネレーターはパイプライン全体を自動的に管理します。

### ステップ 3: 研究フェーズ (自動)

`polyglot-test-researcher` エージェントは、コードベースを分析して以下を理解します。
- **言語とフレームワーク**: C#、TypeScript、Python、Go、Rust、Java などを検出します。
- **テスト フレームワーク**: MSTest、xUnit、Jest、pytest、go test などを識別します。
- **プロジェクト構造**: ソース ファイル、既存のテスト、依存関係をマップします。
- **ビルド コマンド**: プロジェクトをビルドしてテストする方法を説明します。

出力: `.testagent/research.md`

### ステップ 4: 計画フェーズ (自動)

`polyglot-test-planner` エージェントは、構造化された実装計画を作成します。
- ファイルを論理フェーズにグループ化します (通常 2 ～ 5 フェーズ)
- 複雑さと依存関係による優先順位付け
- 各ファイルのテスト ケースを指定します
- フェーズごとに成功基準を定義する

出力: `.testagent/plan.md`

### ステップ 5: 実装フェーズ (自動)

`polyglot-test-implementer` エージェントは、各フェーズを順番に実行します。

1. API を理解するために **ソース ファイルを読んでください**
2. プロジェクト パターンに従って **テスト ファイルを作成**
3. `polyglot-test-builder` サブエージェントを使用して **ビルド** し、コンパイルを検証します
4. `polyglot-test-tester` サブエージェントを使用して **テスト** し、テストが成功したことを確認します
5. エラーが発生した場合は、`polyglot-test-fixer` サブエージェントを使用して **修正**
6. コードのフォーマットに `polyglot-test-linter` サブエージェントを使用する **Lint**

各フェーズは次のフェーズが開始される前に完了し、確実に段階的に進行します。

### 補償範囲の種類
- **ハッピー パス**: 有効な入力により期待される出力が生成されます
- **エッジケース**: 空の値、境界、特殊文字
- **エラーケース**: 無効な入力、null 処理、例外

## 状態管理

すべてのパイプライン状態は `.testagent/` フォルダーに保存されます。

|ファイル |目的 |
|-----|----------|
| `.testagent/research.md` |コードベース分析結果 |
| `.testagent/plan.md` |段階的な実装計画 |
| `.testagent/status.md` |進捗状況の追跡 (オプション) |

## 例

### 例 1: プロジェクト全体のテスト「」
C:\src\Calculator で Calculator プロジェクトの単体テストを生成します。
「」### 例 2: 特定のファイルのテスト「」
src/services/UserService.ts の単体テストを生成する
「」### 例 3: 対象を絞ったカバレッジ「」
エッジケースに焦点を当てた認証モジュールのテストを追加する
「」## エージェントリファレンス

|エージェント |目的 |ツール |
|------|-------|------|
| `polyglot-test-generator` |座標パイプライン | runCommands、コードベース、editFiles、検索、runSubagent |
| `polyglot-test-researcher` |コードベースを分析する | runCommands、コードベース、editFiles、検索、フェッチ、runSubagent |
| `polyglot-test-planner` |テスト計画を作成します |コードベース、editFiles、検索、runSubagent |
| `polyglot-test-implementer` |テスト ファイルを書き込みます | runCommands、コードベース、editFiles、検索、runSubagent |
| `polyglot-test-builder` |コードをコンパイルします | runCommands、コードベース、検索 |
| `polyglot-test-tester` |テストを実行します | runCommands、コードベース、検索 |
| `polyglot-test-fixer` |エラーを修正します | runCommands、コードベース、editFiles、検索 |
| `polyglot-test-linter` |コードのフォーマット | runCommands、コードベース、検索 |

## 要件

- プロジェクトにはビルド/テスト システムが構成されている必要があります
- テスト フレームワークがインストールされている (またはインストール可能である) 必要があります。
- GitHub Copilot 拡張機能を備えた VS Code

## トラブルシューティング

### テストがコンパイルされない
`polyglot-test-fixer` エージェントはコンパイル エラーの解決を試みます。予想されるテスト構造については `.testagent/plan.md` を確認してください。

### テストが失敗する
テスト出力を確認し、テストの期待値を調整します。一部のテストでは、依存関係のモックが必要になる場合があります。

### 間違ったテスト フレームワークが検出されました
最初のリクエストで希望のフレームワークを指定します:「Generate Jest testing for...」