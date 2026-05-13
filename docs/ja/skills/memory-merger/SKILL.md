---
name: memory-merger
description: 'ドメインのメモリーファイルから成熟したレッスンを指示ファイルに統合します。構文: `/memory-merger >domain [scope]` ここで scope は `global`（デフォルト）、`user`、`workspace`、または `ws` です。'
---

# Memory Merger

ドメインのメモリーファイルから成熟した学習内容を指示ファイルに統合し、知識を保持しつつ冗長性を最小限に抑えます。

**ToDoリスト**を使ってプロセスの進行状況を管理し、ユーザーに情報を提供してください。

## スコープ

メモリー指示は2つのスコープに保存できます：

- **グローバル** (`global` または `user`) - `<global-prompts>`（`vscode-userdata:/User/prompts/`）に保存され、すべてのVS Codeプロジェクトに適用されます
- **ワークスペース** (`workspace` または `ws`) - `<workspace-instructions>`（`<workspace-root>/.github/instructions/`）に保存され、現在のプロジェクトのみに適用されます

デフォルトのスコープは**グローバル**です。

このプロンプト内では、`<global-prompts>` と `<workspace-instructions>` はこれらのディレクトリを指します。

## 構文

```
/memory-merger >domain-name [scope]
```

- `>domain-name` - 必須。統合するドメイン（例：`>clojure`、`>git-workflow`、`>prompt-engineering`）
- `[scope]` - 任意。`global`、`user`（どちらもグローバルを意味）、`workspace`、または `ws` のいずれか。デフォルトは `global`

**例:**
- `/memory-merger >prompt-engineering` - グローバルのプロンプトエンジニアリングのメモリを統合
- `/memory-merger >clojure workspace` - ワークスペースのclojureメモリを統合
- `/memory-merger >git-workflow ws` - ワークスペースのgit-workflowメモリを統合

## プロセス

### 1. 入力解析とファイル読み込み

- ユーザー入力からドメインとスコープを**抽出**
- ファイルパスを**決定**：
  - グローバル：`<global-prompts>/{domain}-memory.instructions.md` → `<global-prompts>/{domain}.instructions.md`
  - ワークスペース：`<workspace-instructions>/{domain}-memory.instructions.md` → `<workspace-instructions>/{domain}.instructions.md`
- ユーザーがドメインを誤入力している可能性があるため、メモリーファイルが見つからない場合はディレクトリをグロブして一致候補を探し、疑わしい場合はユーザーに確認を求める
- 両ファイルを**読み込み**（メモリーファイルは必須、指示ファイルは存在しない場合もあり）

### 2. 分析と提案

すべてのメモリーセクションを確認し、統合候補として提示：

```
## 統合候補のメモリー

### メモリー: [見出し]
**内容:** [要点]
**場所:** [指示ファイル内の適切な位置]

[その他のメモリー]...
```

「これらのメモリーを確認してください。すべて承認する場合は 'go' と入力、スキップするものがあれば指定してください。」

**ここで停止し、ユーザーの入力を待ちます。**

### 3. 品質基準の定義

優れた統合後の指示ファイルの10/10基準を設定：

1. **知識の損失ゼロ** - すべての詳細、例、ニュアンスを保持
2. **冗長性の最小化** - 重複する指示は統合
3. **最大の読みやすさ** - 明確な階層構造、並列構造、戦略的な強調、論理的なグルーピング

### 4. 統合と反復

ファイルをまだ更新せずに最終的な統合指示を作成：

1. 承認されたメモリーを取り入れた統合指示の草案作成
2. 品質基準に照らして評価
3. 構成、表現、整理を改善
4. 10/10基準を満たすまで繰り返す

### 5. ファイル更新

最終的な統合指示が10/10基準を満たしたら：

- 指示ファイルを**新規作成または更新**
  - 新規作成時は適切なフロントマターを含める
  - メモリーと指示ファイル両方に `applyTo` パターンがある場合は重複なく包括的に**マージ**
- メモリーファイルから統合済みセクションを**削除**

## 例

```
ユーザー: "/memory-merger >clojure"

エージェント:
1. clojure-memory.instructions.md と clojure.instructions.md を読み込む
2. 統合候補として3つのメモリーを提案
3. [停止]

ユーザー: "go"

エージェント:
4. 10/10の品質基準を定義
5. 新しい統合指示案を作成し、10/10になるまで反復
6. clojure.instructions.md を更新
7. clojure-memory.instructions.md を整理
```
