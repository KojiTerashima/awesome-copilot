---
name: microsoft-skill-creator
description: Learn MCPツールを使用してMicrosoftテクノロジーのエージェントスキルを作成します。ユーザーがAzure、.NET、M365、VS Code、BicepなどのMicrosoftの技術、ライブラリ、フレームワーク、サービスについてエージェントに教えるスキルを作成したい場合に使用します。トピックを深く調査し、重要な知識をローカルに保存しつつ動的な詳細調査を可能にするハイブリッドスキルを生成します。
context: fork
compatibility: Microsoft Learn MCP Server (https://learn.microsoft.com/api/mcp)と最も相性が良いです。mslearn CLIもフォールバックとして使用可能です。
---

# Microsoft Skill Creator

Microsoftテクノロジー向けのハイブリッドスキルを作成し、重要な知識をローカルに保存しながら、Learn MCPによる動的な詳細検索を可能にします。

## スキルについて

スキルは、エージェントの能力を専門知識やワークフローで拡張するモジュール式パッケージです。スキルは汎用エージェントを特定のドメインに特化したものに変えます。

### スキル構造

```
skill-name/
├── SKILL.md (必須)          # フロントマター（name、description）＋指示
├── references/              # 必要に応じてコンテキストに読み込むドキュメント
├── sample_codes/            # 動作するコード例
└── assets/                  # 出力で使用するファイル（テンプレートなど）
```

### 重要な原則

- **フロントマターは重要**：`name`と`description`がスキルのトリガー条件を決めるため、明確かつ包括的に記述すること
- **簡潔さが鍵**：エージェントが既に知っていることは含めず、コンテキストウィンドウは共有される
- **重複禁止**：情報はSKILL.mdか参照ファイルのどちらかにのみ存在させる

## Learn MCPツール

| ツール | 目的 | 使用タイミング |
|--------|-------|---------------|
| `microsoft_docs_search` | 公式ドキュメント検索 | 初期調査、トピック発見 |
| `microsoft_docs_fetch` | ページ全文取得 | 重要ページの詳細調査 |
| `microsoft_code_sample_search` | コード例検索 | 実装パターン取得 |

### CLI代替手段

Learn MCPサーバーが利用できない場合は、ターミナルやシェル（Bash、PowerShell、cmdなど）から`mslearn` CLIを使用してください：

```bash
# 直接実行（インストール不要）
npx @microsoft/learn-cli search "semantic kernel overview"

# またはグローバルインストール後に実行
npm install -g @microsoft/learn-cli
mslearn search "semantic kernel overview"
```

| MCPツール | CLIコマンド |
|----------|-------------|
| `microsoft_docs_search(query: "...")` | `mslearn search "..."` |
| `microsoft_code_sample_search(query: "...", language: "...")` | `mslearn code-search "..." --language ...` |
| `microsoft_docs_fetch(url: "...")` | `mslearn fetch "..."` |

生成されるスキルには、このCLIフォールバック表も含めて、エージェントがどちらの方法でも利用できるようにしてください。

## 作成プロセス

### ステップ1：トピックの調査

Learn MCPツールを使い、3段階で深く理解を構築します：

**フェーズ1 - 範囲の発見：**
```
microsoft_docs_search(query="{technology} overview what is")
microsoft_docs_search(query="{technology} concepts architecture")
microsoft_docs_search(query="{technology} getting started tutorial")
```

**フェーズ2 - コアコンテンツ：**
```
microsoft_docs_fetch(url="...")  # フェーズ1で見つけたページを取得
microsoft_code_sample_search(query="{technology}", language="{lang}")
```

**フェーズ3 - 深掘り：**
```
microsoft_docs_search(query="{technology} best practices")
microsoft_docs_search(query="{technology} troubleshooting errors")
```

#### 調査チェックリスト

調査後に確認：
- [ ] 技術の概要を1段落で説明できる
- [ ] 3～5の主要概念を特定した
- [ ] 基本的な使用例の動作するコードがある
- [ ] 最も一般的なAPIパターンを把握している
- [ ] より深いトピックの検索クエリを用意している

### ステップ2：ユーザーと確認

調査結果を提示し、以下を質問：
1. 「これらの主要分野を見つけました：[リスト]。どれが最も重要ですか？」
2. 「エージェントは主にどのようなタスクをこのスキルで行いますか？」
3. 「コード例はどのプログラミング言語を優先すべきですか？」

### ステップ3：スキルの生成

[skill-templates.md](references/skill-templates.md)から適切なテンプレートを使用：

| 技術タイプ | テンプレート |
|------------|--------------|
| クライアントライブラリ、NuGet/npmパッケージ | SDK/Library |
| Azureリソース | Azure Service |
| アプリ開発フレームワーク | Framework/Platform |
| REST API、プロトコル | API/Protocol |

#### 生成されるスキル構造

```
{skill-name}/
├── SKILL.md                    # コア知識＋Learn MCPガイダンス
├── references/                 # 詳細なローカルドキュメント（必要に応じて）
└── sample_codes/               # 動作するコード例
    ├── getting-started/
    └── common-patterns/
```

### ステップ4：ローカルと動的コンテンツのバランス調整

**ローカル保存すべき場合：**
- 基礎的（どのタスクでも必要）
- 頻繁にアクセスされる
- 安定している（変わらない）
- 検索で見つけにくい

**動的に保持すべき場合：**
- 網羅的なリファレンス（大規模すぎる）
- バージョン依存
- 状況依存（特定タスクのみ）
- 良くインデックスされている（検索しやすい）

#### コンテンツガイドライン

| コンテンツタイプ | ローカル | 動的 |
|------------------|----------|-------|
| コア概念（3～5） | ✅ 完全 |  |
| Hello worldコード | ✅ 完全 |  |
| 共通パターン（3～5） | ✅ 完全 |  |
| 主要APIメソッド | シグネチャ＋例 | フルドキュメント(fetch経由) |
| ベストプラクティス | 上位5つの箇条書き | 追加は検索で |
| トラブルシューティング |  | 検索クエリ |
| フルAPIリファレンス |  | ドキュメントリンク |

### ステップ5：検証

1. レビュー：ローカルコンテンツは一般的なタスクに十分か？
2. テスト：提案した検索クエリは有用な結果を返すか？
3. 確認：コード例はエラーなく実行できるか？

## よく使う調査パターン

### SDK/ライブラリ向け
```
"{name} overview" → 目的、アーキテクチャ
"{name} getting started quickstart" → セットアップ手順
"{name} API reference" → コアクラス・メソッド
"{name} samples examples" → コードパターン
"{name} best practices performance" → 最適化
```

### Azureサービス向け
```
"{service} overview features" → 機能
"{service} quickstart {language}" → セットアップコード
"{service} REST API reference" → エンドポイント
"{service} SDK {language}" → クライアントライブラリ
"{service} pricing limits quotas" → 制限事項
```

### フレームワーク/プラットフォーム向け
```
"{framework} architecture concepts" → メンタルモデル
"{framework} project structure" → 構成規約
"{framework} tutorial walkthrough" → エンドツーエンドの流れ
"{framework} configuration options" → カスタマイズ
```

## 例：「Semantic Kernel」スキルの作成

### 調査

```
microsoft_docs_search(query="semantic kernel overview")
microsoft_docs_search(query="semantic kernel plugins functions")
microsoft_code_sample_search(query="semantic kernel", language="csharp")
microsoft_docs_fetch(url="https://learn.microsoft.com/semantic-kernel/overview/")
```

### 生成されたスキル

```
semantic-kernel/
├── SKILL.md
└── sample_codes/
    ├── getting-started/
    │   └── hello-kernel.cs
    └── common-patterns/
        ├── chat-completion.cs
        └── function-calling.cs
```

### 生成されたSKILL.md

```markdown
---
name: semantic-kernel
description: Microsoft Semantic KernelでAIエージェントを構築します。プラグイン、プランナー、メモリを備えたLLM対応アプリを.NETまたはPythonで作成する際に使用します。
---

# Semantic Kernel

プラグイン、プランナー、メモリを用いてLLMをアプリケーションに統合するオーケストレーションSDK。

## 主要概念

- **Kernel**：AIサービスとプラグインを管理する中央オーケストレーター
- **Plugins**：AIが呼び出せる関数の集合
- **Planner**：目標達成のためにプラグイン関数を順序付ける
- **Memory**：RAGパターンのためのベクターストア統合

## クイックスタート

[getting-started/hello-kernel.cs](sample_codes/getting-started/hello-kernel.cs)を参照

## 詳細情報

| トピック | 検索方法 |
|----------|----------|
| プラグイン開発 | `microsoft_docs_search(query="semantic kernel plugins custom functions")` |
| プランナー | `microsoft_docs_search(query="semantic kernel planner")` |
| メモリ | `microsoft_docs_fetch(url="https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/agent-memory")` |

## CLI代替手段

Learn MCPサーバーが利用できない場合は、`mslearn` CLIを使用してください：

| MCPツール | CLIコマンド |
|----------|-------------|
| `microsoft_docs_search(query: "...")` | `mslearn search "..."` |
| `microsoft_code_sample_search(query: "...", language: "...")` | `mslearn code-search "..." --language ...` |
| `microsoft_docs_fetch(url: "...")` | `mslearn fetch "..."` |

`npx @microsoft/learn-cli <command>`で直接実行するか、`npm install -g @microsoft/learn-cli`でグローバルインストールしてください。
```
