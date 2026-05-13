---
description: 'GitHub Copilot のカスタムエージェント ファイルを作成するためのガイドライン'
applyTo: '**/*.agent.md'
---

# カスタムエージェント ファイルのガイドライン

GitHub Copilot の特定の開発タスクに特化した専門知識を提供する、効果的で保守可能なカスタムエージェント ファイルを作成する手順。

## プロジェクトのコンテキスト

- 対象者: GitHub Copilot のカスタムエージェントを作成する開発者
- ファイル形式: YAML フロントマターを使用したマークダウン
- ファイルの命名規則: 小文字とハイフン (例: `test-specialist.agent.md`)
- 場所: `.github/agents/` ディレクトリ (リポジトリレベル) または `agents/` ディレクトリ (組織/エンタープライズレベル)
- 目的: 特定のタスクに合わせた専門知識、ツール、指示を備えた専門エージェントを定義する
- 公式ドキュメント: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents

## 必須のフロントマター

すべてのエージェントファイルには、次のフィールドを含む YAML フロントマッターが含まれている必要があります。

```yaml
---
description: 'Brief description of the agent purpose and capabilities'
name: 'Agent Display Name'
tools: ['read', 'edit', 'search']
model: 'Claude Sonnet 4.5'
target: 'vscode'
---
```

### 主要なフロントマターのプロパティ

#### **説明** (必須)
- エージェントの目的と専門分野を明確に示す単一引用符で囲まれた文字列
- 簡潔 (50 ～ 150 文字) で実用的である必要があります
- 例: `'Focuses on test coverage, quality, and testing best practices'`

#### **名前** (オプション)
- UI でのエージェントの表示名
- 省略した場合、デフォルトのファイル名 (`.md` または `.agent.md` なし)
- タイトルを大文字にして説明的にしてください
- 例: `'Testing Specialist'`

#### **ツール** (オプション)
- エージェントが使用できるツール名またはエイリアスのリスト
- カンマ区切り文字列または YAML 配列形式をサポート
- 省略した場合、エージェントは利用可能なすべてのツールにアクセスできます。
- 詳細については、以下の「ツール構成」セクションを参照してください。

#### **モデル** (強く推奨)
- エージェントが使用する AI モデルを指定します
- VS Code、JetBrains IDE、Eclipse、および Xcode でサポートされています
- 例: `'Claude Sonnet 4.5'`、`'gpt-4'`、`'gpt-4o'`
- エージェントの複雑さと必要な機能に基づいて選択してください

#### **ターゲット** (オプション)
- ターゲット環境を指定します: `'vscode'` または `'github-copilot'`
- 省略した場合、エージェントは両方の環境で使用できます
- エージェントに環境固有の機能がある場合に使用します

#### **ユーザー呼び出し可能** (オプション)
- チャットのエージェントドロップダウンにエージェントを表示するかどうかを制御するブール値
- デフォルト: `true` (省略した場合)
- サブエージェントとしてまたはプログラム的にのみアクセス可能なエージェントを作成するには、`false` に設定します。

#### **モデル呼び出しを無効にする** (オプション)
- エージェントを他のエージェントによってサブエージェントとして呼び出すことができるかどうかを制御するブール値
- デフォルト: `false` 省略した場合
- `true` に設定すると、ピッカーで利用可能な状態を維持しながらサブエージェントの呼び出しを防止できます。

#### **メタデータ** (オプション、GitHub.com のみ)
- エージェントの注釈用の名前と値のペアを持つオブジェクト
- 例: `metadata: { category: 'testing', version: '1.0' }`
- VS コードではサポートされていません

#### **mcp-servers** (オプション、組織/企業のみ)
- このエージェントのみが使用できる MCP サーバーを構成します
- 組織/エンタープライズレベルのエージェントのみサポートされます
- 以下の「MCP サーバー構成」セクションを参照してください。

#### **ハンドオフ** (オプション、VS Code のみ)
- 推奨される次のステップを使用してエージェント間を移行するガイド付きの順次ワークフローを有効にする
- ハンドオフ構成のリスト。それぞれターゲットエージェントとオプションのプロンプトを指定します。
- チャットの応答が完了すると、ハンドオフボタンが表示され、ユーザーが次のエージェントに移動できるようになります。
- VS Code (バージョン 1.106 以降) でのみサポートされています
- 詳細については、以下の「ハンドオフ構成」セクションを参照してください。

## ハンドオフ構成

ハンドオフを使用すると、カスタムエージェント間をシームレスに移行するガイド付きの順次ワークフローを作成できます。これは、ユーザーが次のステップに進む前に各ステップを確認して承認できる、複数ステップの開発ワークフローを調整する場合に役立ちます。

### 一般的なハンドオフパターン

- **計画 → 実装**: 計画エージェントで計画を生成し、実装エージェントに引き渡してコーディングを開始します。
- **実装 → レビュー**: 実装を完了し、コードレビュー エージェントに切り替えて品質とセキュリティの問題をチェックします。
- **失敗するテストを書く → 合格するテストを書く**: 失敗するテストを生成し、それらのテストを合格させるコードの実装に引き継ぎます。
- **リサーチ → ドキュメント**: トピックをリサーチし、ドキュメントエージェントに移行してガイドを作成します。

### ハンドオフフロントマター構造

`handoffs` フィールドを使用して、エージェントファイルの YAML フロントマターでハンドオフを定義します。

```yaml
---
description: 'Brief description of the agent'
name: 'Agent Name'
tools: ['search', 'read']
handoffs:
  - label: Start Implementation
    agent: implementation
    prompt: 'Now implement the plan outlined above.'
    send: false
  - label: Code Review
    agent: code-review
    prompt: 'Please review the implementation for quality and security issues.'
    send: false
---
```

### ハンドオフのプロパティ

リスト内の各ハンドオフには、次のプロパティが含まれている必要があります。

| 財産 | タイプ | 必須 | 説明 |
|----------|------|----------|-------------|
| `label` | 弦 | はい | チャットインターフェイスのハンドオフボタンに表示される表示テキスト |
| `agent` | 弦 | はい | 切り替え先のターゲットエージェント ID (`.agent.md` を除いた名前またはファイル名) |
| `prompt` | 弦 | いいえ | ターゲットエージェントのチャット入力に事前に入力するプロンプトテキスト |
| `send` | ブール値 | いいえ | `true` の場合、プロンプトをターゲットエージェントに自動的に送信します (デフォルト: `false`) |

### ハンドオフの動作

- **ボタン表示**: チャットの応答が完了した後、ハンドオフボタンがインタラクティブな提案として表示されます。
- **コンテキストの保持**: ユーザーがハンドオフボタンを選択すると、会話のコンテキストが維持されたままターゲットエージェントに切り替わります。
- **事前入力済みプロンプト**: `prompt` が指定されている場合、ターゲットエージェントのチャット入力に事前入力されて表示されます。
- **手動と自動**: `send: false` の場合、ユーザーは事前に入力されたプロンプトを確認して手動で送信する必要があります。 `send: true` の場合、プロンプトは自動的に送信されます

### ハンドオフ構成のガイドライン

#### ハンドオフを使用する場合

- **複数ステップのワークフロー**: 専門エージェント間で複雑なタスクを細分化する
- **品質ゲート**: 実装フェーズ間のレビュー手順を確実に行う
- **ガイド付きプロセス**: 構造化された開発プロセスを通じてユーザーを指導します
- **スキルの移行**: 計画/設計から実装/テストのスペシャリストへの移行

#### ベストプラクティス

- **明確なラベル**: 次のステップを明確に示すアクション指向のラベルを使用します。
  - ✅ 良い: 「実装の開始」、「セキュリティのレビュー」、「テストの作成」
  - ❌ 避ける: 「次へ」、「エージェントに行く」、「何かをする」

- **関連プロンプト**: 完了した作業を参照するコンテキスト認識型プロンプトを提供します。
  - ✅ 良い: `'Now implement the plan outlined above.'`
  - ❌ 回避: コンテキストのない一般的なプロンプト

- **選択的使用**: 可能なすべてのエージェントへのハンドオフを作成しないでください。論理的なワークフローの移行に焦点を当てる
  - エージェントごとに最も関連性の高い次のステップを 2 ～ 3 つに制限する
  - ワークフローに自然に従うエージェントに対してのみハンドオフを追加します。

- **エージェントの依存関係**: ハンドオフを作成する前にターゲットエージェントが存在することを確認してください
  - 存在しないエージェントへのハンドオフは黙って無視されます
  - ハンドオフをテストして、期待どおりに機能することを確認します

- **プロンプトの内容**: プロンプトは簡潔かつ実用的なものにしてください
  - コンテンツを複製せずに現在のエージェントの作品を参照する
  - ターゲットエージェントが必要とする可能性のある必要なコンテキストを提供します

### 例: 完全なワークフロー

以下は、3 人のエージェントがハンドオフを行って完全なワークフローを作成する例です。

**企画エージェント** (`planner.agent.md`):
```yaml
---
description: 'Generate an implementation plan for new features or refactoring'
name: 'Planner'
tools: ['search', 'read']
handoffs:
  - label: Implement Plan
    agent: implementer
    prompt: 'Implement the plan outlined above.'
    send: false
---
# Planner Agent
You are a planning specialist. Your task is to:
1. Analyze the requirements
2. Break down the work into logical steps
3. Generate a detailed implementation plan
4. Identify testing requirements

Do not write any code - focus only on planning.
```

**実装エージェント** (`implementer.agent.md`):
```yaml
---
description: 'Implement code based on a plan or specification'
name: 'Implementer'
tools: ['read', 'edit', 'search', 'execute']
handoffs:
  - label: Review Implementation
    agent: reviewer
    prompt: 'Please review this implementation for code quality, security, and adherence to best practices.'
    send: false
---
# Implementer Agent
You are an implementation specialist. Your task is to:
1. Follow the provided plan or specification
2. Write clean, maintainable code
3. Include appropriate comments and documentation
4. Follow project coding standards

Implement the solution completely and thoroughly.
```

**レビューエージェント** (`reviewer.agent.md`):
```yaml
---
description: 'Review code for quality, security, and best practices'
name: 'Reviewer'
tools: ['read', 'search']
handoffs:
  - label: Back to Planning
    agent: planner
    prompt: 'Review the feedback above and determine if a new plan is needed.'
    send: false
---
# Code Review Agent
You are a code review specialist. Your task is to:
1. Check code quality and maintainability
2. Identify security issues and vulnerabilities
3. Verify adherence to project standards
4. Suggest improvements

Provide constructive feedback on the implementation.
```

このワークフローにより、開発者は次のことが可能になります。
1. Planner エージェントを使用して詳細な計画を作成します。
2. 計画に基づいてコードを作成するよう実装者エージェントに引き継ぎます。
3. 実装をチェックするためにレビュー担当者に引き渡します。
4. 重大な問題が見つかった場合は、オプションで計画に引き戻します。

### バージョンの互換性

- **VS Code**: ハンドオフは VS Code 1.106 以降でサポートされています
- **GitHub.com**: 現在サポートされていません。エージェント移行ワークフローはさまざまなメカニズムを使用します
- **その他の IDE**: サポートが制限されているか、サポートされていません。互換性を最大限に高めるために VS Code の実装に重点を置く

## ツール構成

### ツール仕様戦略

**すべてのツールを有効にする** (デフォルト):
```yaml
# Omit tools property entirely, or use:
tools: ['*']
```

**特定のツールを有効にする**:
```yaml
tools: ['read', 'edit', 'search', 'execute']
```

**MCP サーバーツールを有効にする**:
```yaml
tools: ['read', 'edit', 'github/*', 'playwright/navigate']
```

**すべてのツールを無効にする**:
```yaml
tools: []
```

### 標準ツールのエイリアス

すべてのエイリアスでは大文字と小文字が区別されません。

| エイリアス | 別名 | カテゴリ | 説明 |
|-------|------------------|----------|-------------|
| `execute` | シェル、Bash、パワーシェル | シェルの実行 | 適切なシェルでコマンドを実行する |
| `read` | 読み取り、ノートブック読み取り、表示 | ファイルの読み込み | ファイルの内容を読み取る |
| `edit` | 編集、マルチ編集、書き込み、ノートブック編集 | ファイル編集 | ファイルの編集と変更 |
| `search` | Grep、Glob、検索 | コード検索 | ファイルまたはファイル内のテキストを検索する |
| `agent` | カスタムエージェント、タスク | エージェントの呼び出し | 他のカスタムエージェントを呼び出す |
| `web` | Web検索、Webフェッチ | ウェブアクセス | Web コンテンツを取得して検索する |
| `todo` | TodoWrite | タスク管理 | タスクリストの作成と管理 (VS Code のみ) |

### 組み込みの MCP サーバーツール

**GitHub MCP サーバー**:
```yaml
tools: ['github/*']  # All GitHub tools
tools: ['github/get_file_contents', 'github/search_repositories']  # Specific tools
```
- すべての読み取り専用ツールがデフォルトで利用可能
- ソースリポジトリをスコープとするトークン

**劇作家 MCP サーバー**:
```yaml
tools: ['playwright/*']  # All Playwright tools
tools: ['playwright/navigate', 'playwright/screenshot']  # Specific tools
```
- localhostのみにアクセスするように構成されています
- ブラウザの自動化とテストに役立ちます

### ツール選択のベストプラクティス

- **最小特権の原則**: エージェントの目的に必要なツールのみを有効にします
- **セキュリティ**: 明示的に要求されない限り、`execute` へのアクセスを制限します
- **焦点**: ツールが少ない = エージェントの目的が明確になり、パフォーマンスが向上します
- **ドキュメント**: 複雑な構成に特定のツールが必要な理由をコメントします。

## サブエージェントの呼び出し (エージェントオーケストレーション)

エージェントは、**エージェント呼び出しツール** (`agent` ツール) を使用して他のエージェントを呼び出し、複数ステップのワークフローを調整できます。

推奨されるアプローチは **プロンプトベースのオーケストレーション**です。
- オーケストレーターは、自然言語で段階的なワークフローを定義します。
- 各ステップは専門のエージェントに委任されます。
- オーケストレーターは重要なコンテキスト (ベースパス、識別子など) のみを渡し、各サブエージェントにツール/制約の独自の `.agent.md` 仕様を読み取るように要求します。

### 仕組み

1) オーケストレーターのツールリストに `agent` を含めて、エージェントの呼び出しを有効にします。

```yaml
tools: ['read', 'edit', 'search', 'agent']
```

2) 各ステップで、以下を指定してサブエージェントを呼び出します。
- **エージェント名** (ユーザーが選択/呼び出す識別子)
- **エージェント仕様パス** (読み取りおよびフォローする `.agent.md` ファイル)
- **最小限の共有コンテキスト** (例: `basePath`、`projectName`、`logFile`)

### プロンプトパターン（推奨）

すべてのステップで一貫した「ラッパープロンプト」を使用して、サブエージェントが予測どおりに動作するようにします。

```text
This phase must be performed as the agent "<AGENT_NAME>" defined in "<AGENT_SPEC_PATH>".

IMPORTANT:
- Read and apply the entire .agent.md spec (tools, constraints, quality standards).
- Work on "<WORK_UNIT_NAME>" with base path: "<BASE_PATH>".
- Perform the necessary reads/writes under this base path.
- Return a clear summary (actions taken + files produced/modified + issues).
```

オプション: トレーサビリティのために軽量で構造化されたラッパーが必要な場合は、プロンプトに小さな JSON ブロックを埋め込みます (人間が判読可能でツールに依存しない)。

```text
{
  "step": "<STEP_ID>",
  "agent": "<AGENT_NAME>",
  "spec": "<AGENT_SPEC_PATH>",
  "basePath": "<BASE_PATH>"
}
```

### オーケストレーターの構造 (汎用的なものにする)

保守可能なオーケストレーターについては、次の構造要素を文書化します。

- **動的パラメータ**: ユーザーから抽出される値 (例: `projectName`、`fileName`、`basePath`)。
- **サブエージェントレジストリ**: 各ステップを `agentName` + `agentSpecPath` にマッピングするリスト/テーブル。
- **ステップの順序付け**: 明示的なシーケンス (ステップ 1 → ステップ N)。
- **トリガー条件** (オプションですが推奨): ステップが実行されるときとスキップされるときを定義します。
- **ログ戦略** (オプションですが推奨): 各ステップの後に更新される単一のログ/レポートファイル。

オーケストレーションの「コード」 (JavaScript、Python など) をオーケストレータープロンプト内に埋め込むことは避けてください。決定論的でツール主導の調整を好みます。

### 基本パターン

各ステップの呼び出しを次のように構造化します。

1. **ステップの説明**: 明確な 1 行の目的 (ログとトレーサビリティに使用)
2. **エージェント ID**: `agentName` + `agentSpecPath`
3. **コンテキスト**: 小規模で明示的な変数セット (パス、ID、環境名)
4. **予想される出力**: 作成/更新するファイルとそのファイルを書き込む場所
5. **要約を返す**: サブエージェントに、短く構造化された要約を返すように依頼します。

### 例: 複数ステップの処理

```text
Step 1: Transform raw input data
Agent: data-processor
Spec: .github/agents/data-processor.agent.md
Context: projectName=${projectName}, basePath=${basePath}
Input: ${basePath}/raw/
Output: ${basePath}/processed/
Expected: write ${basePath}/processed/summary.md

Step 2: Analyze processed data (depends on Step 1 output)
Agent: data-analyst
Spec: .github/agents/data-analyst.agent.md
Context: projectName=${projectName}, basePath=${basePath}
Input: ${basePath}/processed/
Output: ${basePath}/analysis/
Expected: write ${basePath}/analysis/report.md
```

### 重要なポイント

- **プロンプトで変数を渡す**: すべての動的値には `${variableName}` を使用します
- **プロンプトに焦点を当て続ける**: 各サブエージェントの明確な具体的なタスク
- **要約を返す**: 各サブエージェントは自分が達成した内容を報告する必要があります
- **順次実行**: 出力/入力間に依存関係が存在する場合、ステップを順番に実行します。
- **エラー処理**: 依存する手順に進む前に結果を確認してください

### ⚠️ ツールの可用性要件

**重要**: サブエージェントが特定のツール (`edit`、`execute`、`search` など) を必要とする場合、オーケストレーターはそれらのツールを独自の `tools` リストに含める必要があります。サブエージェントは、親オーケストレーターが利用できないツールにはアクセスできません。

**例**：
```yaml
# If your sub-agents need to edit files, execute commands, or search code
tools: ['read', 'edit', 'search', 'execute', 'agent']
```

オーケストレーターのツール権限は、呼び出されるすべてのサブエージェントの上限として機能します。すべてのサブエージェントが必要なツールを確実に備えられるように、ツールリストを慎重に計画してください。

### ⚠️重要な制限事項

**サブエージェントオーケストレーションは大規模なデータ処理には適していません。** 次の場合は、複数ステップのサブエージェントパイプラインの使用を避けてください。
- 数百または数千のファイルを処理する
- 大規模なデータセットの処理
- 大規模なコードベースでの一括変換の実行
- 5 ～ 10 を超える一連のステップを調整する

サブエージェントの呼び出しごとに、レイテンシーとコンテキストのオーバーヘッドが追加されます。大量の処理の場合は、代わりに単一のエージェントにロジックを直接実装します。オーケストレーションは、焦点を絞った管理可能なデータセットに対する特殊なタスクを調整する場合にのみ使用してください。

## エージェントプロンプトの構造

フロントマターの下のマークダウンコンテンツは、エージェントの動作、専門知識、および指示を定義します。適切に構造化されたプロンプトには通常、次のものが含まれます。

1. **エージェントのアイデンティティと役割**: エージェントとは誰であり、その主な役割
2. **中核的な責任**: エージェントが実行する具体的なタスク
3. **アプローチと方法**: タスクを達成するためにエージェントがどのように機能するか
4. **ガイドラインと制約**: すべきこと/避けるべきこと、および品質基準
5. **期待される出力**: 期待される出力形式と品質

### プロンプトライティングのベストプラクティス

- **具体的かつ直接的である**: 命令型のムード (「分析」、「生成」) を使用します。曖昧な用語を避ける
- **境界の定義**: 範囲の制限と制約を明確に示します。
- **コンテキストを含める**: ドメインの専門知識を説明し、関連するフレームワークを参照する
- **行動に焦点を当てる**: エージェントがどのように考え、機能するべきかを説明します。
- **構造化フォーマットを使用**: ヘッダー、箇条書き、リストによりプロンプトをスキャン可能にします

## 変数の定義と抽出

エージェントは動的パラメータを定義して、ユーザー入力から値を抽出し、エージェントの動作やサブエージェントの通信全体でそれらの値を使用できます。これにより、ユーザーが提供したデータに適応する柔軟でコンテキスト認識型のエージェントが可能になります。

### 変数を使用する場合

**次の場合に変数を使用します**:
- エージェントの動作はユーザー入力に依存します
- 動的な値をサブエージェントに渡す必要がある
- さまざまなコンテキスト間でエージェントを再利用できるようにしたい
- パラメータ化されたワークフローが必要
- ユーザー提供のコンテキストを追跡または参照する必要がある

**例**:
- ユーザープロンプトからプロジェクト名を抽出する
- パイプライン処理用の証明書名を取得します
- ファイルパスまたはディレクトリを特定する
- 構成オプションの抽出
- 機能名またはモジュール識別子を解析する

### 変数宣言パターン

エージェントプロンプトの早い段階で変数セクションを定義して、予期されるパラメーターを文書化します。

```markdown
# Agent Name

## Dynamic Parameters

- **Parameter Name**: Description and usage
- **Another Parameter**: How it's extracted and used

## Your Mission

Process [PARAMETER_NAME] to accomplish [task].
```

### 変数の抽出方法

#### 1. **明示的なユーザー入力**
プロンプトで変数が検出されない場合は、ユーザーに変数を指定するように依頼します。

```markdown
## Your Mission

Process the project by analyzing your codebase.

### Step 1: Identify Project
If no project name is provided, **ASK THE USER** for:
- Project name or identifier
- Base path or directory location
- Configuration type (if applicable)

Use this information to contextualize all subsequent tasks.
```

#### 2. **プロンプトからの暗黙的な抽出**
ユーザーの自然言語入力から変数を自動的に抽出します。

```javascript
// Example: Extract certification name from user input
const userInput = "Process My Certification";

// Extract key information
const certificationName = extractCertificationName(userInput);
// Result: "My Certification"

const basePath = `certifications/${certificationName}`;
// Result: "certifications/My Certification"
```

#### 3. **コンテキスト変数の解決**
ファイルコンテキストまたはワークスペース情報を使用して変数を取得します。

```markdown
## Variable Resolution Strategy

1. **From User Prompt**: First, look for explicit mentions in user input
2. **From File Context**: Check current file name or path
3. **From Workspace**: Use workspace folder or active project
4. **From Settings**: Reference configuration files
5. **Ask User**: If all else fails, request missing information
```

### エージェントプロンプトでの変数の使用

#### 命令内の変数置換

エージェントプロンプトでテンプレート変数を使用して、プロンプトを動的にします。

```markdown
# Agent Name

## Dynamic Parameters
- **Project Name**: ${projectName}
- **Base Path**: ${basePath}
- **Output Directory**: ${outputDir}

## Your Mission

Process the **${projectName}** project located at `${basePath}`.

## Process Steps

1. Read input from: `${basePath}/input/`
2. Process files according to project configuration
3. Write results to: `${outputDir}/`
4. Generate summary report

## Quality Standards

- Maintain project-specific coding standards for **${projectName}**
- Follow directory structure: `${basePath}/[structure]`
```

#### サブエージェントに変数を渡す

サブエージェントを呼び出すときは、プロンプト内の置換変数を介してすべてのコンテキストを渡します。ファイルの内容全体ではなく、**パスと識別子**を渡すことを優先します。

例 (プロンプトテンプレート):

```text
This phase must be performed as the agent "documentation-writer" defined in ".github/agents/documentation-writer.agent.md".

IMPORTANT:
- Read and apply the entire .agent.md spec.
- Project: "${projectName}"
- Base path: "projects/${projectName}"
- Input: "projects/${projectName}/src/"
- Output: "projects/${projectName}/docs/"

Task:
1. Read source files under the input path.
2. Generate documentation.
3. Write outputs under the output path.
4. Return a concise summary (files created/updated, key decisions, issues).
```

サブエージェントは、プロンプトに埋め込まれた必要なすべてのコンテキストを受け取ります。変数はプロンプトを送信する前に解決されるため、サブエージェントは変数のプレースホルダーではなく、具体的なパスと値を使用して動作します。

### 実際の例: コードレビュー オーケストレーター

複数の専門エージェントを通じてコードを検証する単純なオーケストレーターの例:

1) 共有コンテキストを決定します。
- `repositoryName`、`prNumber`
- `basePath` (例: `projects/${repositoryName}/pr-${prNumber}`)

2) 特殊なエージェントを順番に呼び出します (各エージェントは独自の `.agent.md` 仕様を読み取ります)。

```text
Step 1: Security Review
Agent: security-reviewer
Spec: .github/agents/security-reviewer.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/security-review.md

Step 2: Test Coverage
Agent: test-coverage
Spec: .github/agents/test-coverage.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/coverage-report.md

Step 3: Aggregate
Agent: review-aggregator
Spec: .github/agents/review-aggregator.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/final-review.md
```

#### 例: 条件付きステップオーケストレーション (コードレビュー)

この例では、**プリフライトチェック**、**条件付きステップ**、**必須動作とオプション** の動作を使用した、より完全なオーケストレーションを示します。

**動的パラメータ (入力):**
- `repositoryName`、`prNumber`
- `basePath` (例: `projects/${repositoryName}/pr-${prNumber}`)
- `logFile` (例: `${basePath}/.review-log.md`)

**飛行前チェック (推奨):**
- 予期されるフォルダー/ファイルが存在することを確認します (例: `${basePath}/changes/`、`${basePath}/reports/`)。
- ステップトリガーに影響を与える高レベルの特性を検出します (例: リポジトリ言語、`package.json`、`pom.xml`、`requirements.txt`、テストフォルダーの存在)。
- 最初に検出結果を 1 回記録します。

**ステップトリガー条件:**

| ステップ | 状態 | トリガー条件 | 失敗時 |
|------|--------|-------------------|-----------|
| 1: セキュリティのレビュー | **必須** | 常に実行する | パイプラインを停止する |
| 2: 依存関係の監査 | オプション | 依存関係マニフェストが存在する場合 (`package.json`、`pom.xml` など) | 続く |
| 3: テストカバレッジのチェック | オプション | テストプロジェクト/ファイルが存在する場合 | 続く |
| 4: パフォーマンスチェック | オプション | perf に依存するコードが変更された場合、または perf 構成が存在する場合 | 続く |
| 5: 集計と評決 | **必須** | ステップ 1 が完了したら常に実行 | パイプラインを停止する |

**実行フロー (自然言語):**
1. `basePath` を初期化し、`logFile` を作成/更新します。
2. 飛行前チェックを実行し、記録します。
3. ステップ1→Nを順に実行します。
4. 各ステップについて:
  - トリガー条件が false の場合: **SKIPPED** としてマークして続行します。
  - それ以外の場合: ラッパープロンプトを使用してサブエージェントを呼び出し、その概要をキャプチャします。
  - **SUCCESS** または **FAILED** としてマークします。
  - ステップが **必須** で失敗した場合: パイプラインを停止し、失敗の概要を書き込みます。
5. 最後の概要セクション (全体的なステータス、成果物、次のアクション) で終了します。

**サブエージェント呼び出しプロンプト (例):**

```text
This phase must be performed as the agent "security-reviewer" defined in ".github/agents/security-reviewer.agent.md".

IMPORTANT:
- Read and apply the entire .agent.md spec.
- Work on repository "${repositoryName}" PR "${prNumber}".
- Base path: "${basePath}".

Task:
1. Review the changes under "${basePath}/changes/".
2. Write findings to "${basePath}/reports/security-review.md".
3. Return a short summary with: critical findings, recommended fixes, files created/modified.
```

**ロギング形式(例):**

```markdown
## Step 2: Dependency Audit
**Status:** ✅ SUCCESS / ⚠️ SKIPPED / ❌ FAILED
**Trigger:** package.json present
**Started:** 2026-01-16T10:30:15Z
**Completed:** 2026-01-16T10:31:05Z
**Duration:** 00:00:50
**Artifacts:** reports/dependency-audit.md
**Summary:** [brief agent summary]
```

このパターンは、変数を抽出し、明確なコンテキストでサブエージェントを呼び出し、結果を待つというあらゆるオーケストレーションシナリオに当てはまります。

### さまざまなベストプラクティス

#### 1. **明確なドキュメント**
どのような変数が予想されるかを常に文書化してください。

```markdown
## Required Variables
- **projectName**: The name of the project (string, required)
- **basePath**: Root directory for project files (path, required)

## Optional Variables
- **mode**: Processing mode - quick/standard/detailed (enum, default: standard)
- **outputFormat**: Output format - markdown/json/html (enum, default: markdown)

## Derived Variables
- **outputDir**: Automatically set to ${basePath}/output
- **logFile**: Automatically set to ${basePath}/.log.md
```

#### 2. **一貫した命名**
一貫した変数命名規則を使用します。

```javascript
// Good: Clear, descriptive naming
const variables = {
  projectName,          // What project to work on
  basePath,            // Where project files are located
  outputDirectory,     // Where to save results
  processingMode,      // How to process (detail level)
  configurationPath    // Where config files are
};

// Avoid: Ambiguous or inconsistent
const bad_variables = {
  name,     // Too generic
  path,     // Unclear which path
  mode,     // Too short
  config    // Too vague
};
```

#### 3. **検証と制約**
有効な値と制約を文書化します。

```markdown
## Variable Constraints

**projectName**:
- Type: string (alphanumeric, hyphens, underscores allowed)
- Length: 1-100 characters
- Required: yes
- Pattern: `/^[a-zA-Z0-9_-]+$/`

**processingMode**:
- Type: enum
- Valid values: "quick" (< 5min), "standard" (5-15min), "detailed" (15+ min)
- Default: "standard"
- Required: no
```

## MCP サーバー構成 (組織/企業のみ)

MCP サーバーは、追加のツールを使用してエージェントの機能を拡張します。組織およびエンタープライズレベルのエージェントでのみサポートされます。

### 設定フォーマット

```yaml
---
name: my-custom-agent
description: 'Agent with MCP integration'
tools: ['read', 'edit', 'custom-mcp/tool-1']
mcp-servers:
  custom-mcp:
    type: 'local'
    command: 'some-command'
    args: ['--arg1', '--arg2']
    tools: ["*"]
    env:
      ENV_VAR_NAME: ${{ secrets.API_KEY }}
---
```

### MCP サーバーのプロパティ

- **type**: サーバーの種類 (`'local'` または `'stdio'`)
- **コマンド**: MCPサーバーを起動するコマンド
- **args**: コマンド引数の配列
- **ツール**: このサーバーから有効にするツール (すべて `["*"]`)
- **env**: 環境変数 (シークレットをサポート)

### 環境変数とシークレット

シークレットは、「copilot」環境のリポジトリ設定で構成する必要があります。

**サポートされている構文**:
```yaml
env:
  # Environment variable only
  VAR_NAME: COPILOT_MCP_ENV_VAR_VALUE

  # Variable with header
  VAR_NAME: $COPILOT_MCP_ENV_VAR_VALUE
  VAR_NAME: ${COPILOT_MCP_ENV_VAR_VALUE}

  # GitHub Actions-style (YAML only)
  VAR_NAME: ${{ secrets.COPILOT_MCP_ENV_VAR_VALUE }}
  VAR_NAME: ${{ var.COPILOT_MCP_ENV_VAR_VALUE }}
```

## ファイルの構成と命名

### リポジトリレベルのエージェント
- 場所：`.github/agents/`
- 範囲: 特定のリポジトリでのみ使用可能
- アクセス: リポジトリで構成された MCP サーバーを使用します

### 組織/企業レベルのエージェント
- 場所: `.github-private/agents/` (その後、`agents/` ルートに移動)
- 範囲: 組織/企業内のすべてのリポジトリで利用可能
- アクセス: 専用の MCP サーバーを構成できます

### 命名規則
- 小文字とハイフンを使用してください: `test-specialist.agent.md`
- 名前はエージェントの目的を反映する必要があります
- ファイル名はデフォルトのエージェント名になります (`name` が指定されていない場合)
- 使用できる文字: `.`、`-`、`_`、`a-z`、`A-Z`、`0-9`

## エージェントの処理と動作

### バージョン管理
- エージェントファイルの Git コミット SHA に基づく
- さまざまなエージェントバージョンのブランチ/タグを作成する
- リポジトリ/ブランチの最新バージョンを使用してインスタンス化されています
- PR インタラクションでは一貫性を保つために同じエージェントバージョンを使用します

### 名前の競合
優先度 (最高から最低):
1. リポジトリレベルのエージェント
2. 組織レベルのエージェント
3. エンタープライズレベルのエージェント

下位レベルの構成は、同じ名前の上位レベルの構成をオーバーライドします。

### 工具加工
- `tools` 利用可能なツール (組み込みおよび MCP) のフィルタをリストします。
- ツールが指定されていない = すべてのツールが有効になっています
- 空のリスト (`[]`) = すべてのツールが無効になります
- 特定のリスト = 有効なツールのみ
- 認識できないツール名は無視されます (環境固有のツールは許可されます)

### MCP サーバーの処理順序
1. すぐに使える MCP サーバー (GitHub MCP など)
2. カスタムエージェント MCP 構成 (組織/企業のみ)
3. リポジトリレベルのMCP構成

各レベルは、前のレベルの設定をオーバーライドできます。

## エージェント作成チェックリスト

### フロントマター
- [ ] `description` フィールドが存在し、説明的 (50 ～ 150 文字)
- [ ] `description` 一重引用符で囲む
- [ ] `name` を指定 (オプションですが推奨)
- [ ] `tools` は適切に設定されています (または意図的に省略されています)。
- [ ] `model` は最適なパフォーマンスを実現するために指定されています
- [ ] `target` 環境固有の場合に設定
- [ ] `user-invocable: false` を使用して、サブエージェントの呼び出しを許可しながらピッカーから非表示にします
- [ ] `disable-model-invocation: true` を使用して、ピッカーの可視性を維持しながらサブエージェントの呼び出しを防止します

### プロンプトコンテンツ
- [ ] 明確なエージェント ID と役割の定義
- [ ] 明示的にリストされた中核的な責任
- [ ] アプローチと方法論の説明
- [ ] 指定されたガイドラインと制約
- [ ] 期待される出力を文書化
- [ ] 役立つ場合に例を示します
- [ ] 指示は具体的で実行可能です
- [ ] 範囲と境界が明確に定義されている
- [ ] コンテンツの合計が 30,000 文字未満

### ファイル構造
- [ ] ファイル名は小文字とハイフンの規則に従います
- [ ] ファイルは正しいディレクトリ (`.github/agents/` または `agents/`) に配置されました
- [ ] ファイル名には許可された文字のみが使用されます
- [ ] ファイル拡張子は `.agent.md` です

### 品質保証
- [ ] エージェントの目的は一意であり、重複はありません
- [ ] ツールは最小限で必要です
- [ ] 指示は明確で明確です
- [ ] エージェントは代表的なタスクでテストされています
- [ ] ドキュメントの参照は最新のものです
- [ ] セキュリティ上の考慮事項に対処しました (該当する場合)

## 一般的なエージェントのパターン

### テストスペシャリスト
**目的**: テスト範囲と品質に重点を置く
**ツール**: すべてのツール (包括的なテスト作成用)
**アプローチ**: 分析、ギャップの特定、テストの作成、運用コードの変更の回避

### 実装プランナー
**目的**: 詳細な技術計画と仕様を作成する
**ツール**: `['read', 'search', 'edit']` に限定
**アプローチ**: 要件を分析し、ドキュメントを作成し、実装を回避します。

### コードレビューア
**目的**: コードの品質をレビューし、フィードバックを提供します
**ツール**: `['read', 'search']` のみ
**アプローチ**: 分析し、改善を提案します。直接の変更は行いません。

### リファクタリングスペシャリスト
**目的**: コード構造と保守性を向上させる
**ツール**: `['read', 'search', 'edit']`
**アプローチ**: パターンを分析し、リファクタリングを提案し、安全に実装する

### セキュリティ監査人
**目的**: セキュリティの問題と脆弱性を特定する
**ツール**: `['read', 'search', 'web']`
**アプローチ**: コードをスキャンし、OWASP に対してチェックし、結果を報告します

## 避けるべきよくある間違い

### フロントマターエラー
- ❌ `description` フィールドがありません
- ❌ 説明が引用符で囲まれていない
- ❌ ドキュメントを確認しない無効なツール名
- ❌ 不正な YAML 構文 (インデント、引用符)

### ツール構成の問題
- ❌ 不必要に過剰なツールアクセスを許可する
- ❌ エージェントの目的に必要なツールが不足している
- ❌ ツールエイリアスを一貫して使用していない
- ❌ MCP サーバーの名前空間を忘れています (`server-name/tool`)

### プロンプトコンテンツの問題
- ❌ 漠然とした曖昧な指示
- ❌ 矛盾または矛盾するガイドライン
- ❌ 明確な範囲定義の欠如
- ❌ 期待される出力が欠如している
- ❌ 過度に冗長な指示（文字数制限を超える）
- ❌ 複雑なタスクの例やコンテキストがない

### 組織の問題
- ❌ ファイル名がエージェントの目的を反映していない
- ❌ 間違ったディレクトリ (リポジトリと組織レベルの混同)
- ❌ ファイル名にスペースまたは特殊文字を使用する
- ❌ エージェント名が重複すると競合が発生する

## テストと検証

### 手動テスト
1. 適切なフロントマターを使用してエージェントファイルを作成する
2. VS Code をリロードするか、GitHub.com を更新します
3. Copilot チャットのドロップダウンからエージェントを選択します
4. 代表的なユーザークエリでテストする
5. ツールへのアクセスが期待どおりに機能することを確認する
6. 出力が期待どおりであることを確認する

### 統合テスト
- スコープ内の異なるファイルタイプでエージェントをテストする
- MCP サーバーの接続を確認します (構成されている場合)
- コンテキストが欠落しているエージェントの動作を確認する
- テストエラー処理とエッジケース
- エージェントの切り替えとハンドオフを検証する

### 品質チェック
- エージェント作成チェックリストを実行する
- よくある間違いリストと照らし合わせて確認する
- リポジトリ内のサンプルエージェントと比較する
- 複雑なエージェントのピアレビューを取得する
- 特別な構成が必要な場合は文書化する

## 追加リソース

### 公式ドキュメント
- [カスタムエージェントの作成](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- [カスタムエージェント構成](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [VS Code のカスタムエージェント](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [MCPの統合](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)

### コミュニティリソース
- [素晴らしい副操縦士エージェントコレクション](https://github.com/github/awesome-copilot/tree/main/agents)
- [カスタマイズライブラリの例](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents)
- [初めてのカスタムエージェントのチュートリアル](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents/your-first-custom-agent)

### 関連ファイル
- [プロンプトファイルのガイドライン](./prompt.instructions.md) - プロンプトファイルの作成用
- [指示ガイドライン](./instructions.instructions.md) - 指示ファイルの作成用

## バージョン互換性に関する注意事項

### GitHub.com (コーディングエージェント)
- ✅ すべての標準フロントマタープロパティを完全にサポート
- ✅ リポジトリと組織/エンタープライズレベルのエージェント
- ✅ MCP サーバー構成 (組織/企業)
- ❌ `model`、`argument-hint`、`handoffs` プロパティはサポートされません

### VS コード / JetBrains / Eclipse / Xcode
- ✅ AI モデル選択のための `model` プロパティをサポート
- ✅ `argument-hint` および `handoffs` プロパティをサポート
- ✅ ユーザープロファイルとワークスペースレベルのエージェント
- ❌ MCP サーバーをリポジトリレベルで構成できない
- ⚠️ 一部のプロパティは動作が異なる場合があります

複数の環境用にエージェントを作成する場合は、共通のプロパティに焦点を当て、すべてのターゲット環境でテストします。必要に応じて `target` プロパティを使用して環境固有のエージェントを作成します。
