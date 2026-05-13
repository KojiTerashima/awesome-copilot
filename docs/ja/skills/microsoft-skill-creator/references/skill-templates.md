# スキルテンプレート

さまざまなMicrosoftテクノロジー向けのすぐに使えるテンプレート。

## MCPツールのCLI代替

以下のすべてのテンプレートはMCPツール呼び出し（例：`microsoft_docs_search`、`microsoft_docs_fetch`、`microsoft_code_sample_search`）を使用しています。Learn MCPサーバーが利用できない場合は、CLIの同等コマンドに置き換えてください：

| MCPツール | CLIコマンド |
|----------|-------------|
| `microsoft_docs_search(query: "...")` | `mslearn search "..."` |
| `microsoft_code_sample_search(query: "...", language: "...")` | `mslearn code-search "..." --language ...` |
| `microsoft_docs_fetch(url: "...")` | `mslearn fetch "..."` |

`npx @microsoft/learn-cli <command>`で直接実行するか、`npm install -g @microsoft/learn-cli`でグローバルインストールしてください。

## テンプレート1：SDK/ライブラリスキル

クライアントライブラリ、SDK、プログラミングフレームワーク向け。

```markdown
---
name: {sdk-name}
description: {何をするか}。エージェントが{技術コンテキスト}で{主なタスク}を行う必要がある場合に使用。{対応言語/プラットフォーム}をサポート。
---

# {SDK名}

{1段落：何であるか、存在理由、使用タイミング}

## インストール

{対応言語のパッケージマネージャーコマンド}

## キーコンセプト

{3～5の重要な概念、それぞれ最大1段落}

### {概念1}
{簡潔な説明}

### {概念2}
{簡潔な説明}

## クイックスタート

{最小限の動作例 - 30行未満ならインライン、そうでなければsample_codes/を参照}

## よくあるパターン

### {パターン1：例「基本CRUD」}
```{language}
{コード}
```

### {パターン2：例「エラーハンドリング」}
```{language}
{コード}
```

## APIクイックリファレンス

| クラス/メソッド | 目的 | 例 |
|--------------|---------|---------|
| {name} | {何をするか} | `{usage}` |

完全なAPIドキュメント：
- `microsoft_docs_search(query="{sdk} {class} API reference")`
- `microsoft_docs_fetch(url="{url}")`

## ベストプラクティス

- **推奨**: {推奨事項}
- **推奨**: {推奨事項}
- **回避**: {アンチパターン}

詳細は[best-practices.md](references/best-practices.md)を参照。

## さらに学ぶ

| トピック | 検索方法 |
|-------|-------------|
| {高度なトピック1} | `microsoft_docs_search(query="{sdk} {topic}")` |
| {高度なトピック2} | `microsoft_docs_fetch(url="{url}")` |
| {コード例} | `microsoft_code_sample_search(query="{sdk} {scenario}", language="{lang}")` |
```

---

## テンプレート2：Azureサービススキル

Azureサービスおよびクラウドリソース向け。

```markdown
---
name: {service-name}
description: {Azureサービス}を操作。エージェントが{主な機能}を必要とする場合に使用。プロビジョニング、構成、SDK使用をカバー。
---

# {Azureサービス名}

{1段落：サービスの内容、主なユースケース}

## 概要

- **カテゴリ**: {コンピュート/ストレージ/AI/ネットワーキングなど}
- **主要機能**: {主な価値提案}
- **使用タイミング**: {シナリオ}

## はじめに

### 前提条件
- Azureサブスクリプション
- {その他の要件}

### プロビジョニング
{リソース作成のCLI/ポータル/Bicepスニペット}

## SDK使用（{言語}）

### インストール
```
{パッケージインストールコマンド}
```

### 認証
```{language}
{認証コードパターン}
```

### 基本操作
```{language}
{CRUDまたは主要操作}
```

## 主要設定

| 設定 | 目的 | デフォルト |
|---------|---------|---------|
| {設定} | {制御内容} | {値} |

## 料金と制限

- **料金モデル**: {従量課金/階層制など}
- **主な制限**: {重要なクォータ}

最新料金は：`microsoft_docs_search(query="{service} pricing")`

## よくあるパターン

### {パターン1}
{コードまたは設定}

### {パターン2}
{コードまたは設定}

## トラブルシューティング

| 問題 | 解決策 |
|-------|----------|
| {よくあるエラー} | {修正方法} |

詳細は：`microsoft_docs_search(query="{service} troubleshoot {symptom}")`

## さらに学ぶ

| トピック | 検索方法 |
|-------|-------------|
| REST API | `microsoft_docs_fetch(url="{url}")` |
| ARM/Bicep | `microsoft_docs_search(query="{service} bicep template")` |
| セキュリティ | `microsoft_docs_search(query="{service} security best practices")` |
```

---

## テンプレート3：フレームワーク/プラットフォームスキル

開発フレームワークやプラットフォーム向け（例：ASP.NET、MAUI、Blazor）。

```markdown
---
name: {framework-name}
description: {フレームワーク}で{アプリの種類}を構築。エージェントが{フレームワーク}アプリの作成、修正、デバッグを行う必要がある場合に使用。
---

# {フレームワーク名}

{1段落：何であるか、何を作るか、なぜ選ぶか}

## プロジェクト構成

```
{typical-project}/
├── {folder}/     # {目的}
├── {file}        # {目的}
└── {file}        # {目的}
```

## はじめに

### 新規プロジェクト作成
```bash
{スキャフォールド用CLIコマンド}
```

### プロジェクト設定
{設定すべき主要ファイルと制御内容}

## コアコンセプト

### {概念1：例「コンポーネント」}
{最小限のコード例を含む説明}

### {概念2：例「ルーティング」}
{最小限のコード例を含む説明}

### {概念3：例「状態管理」}
{最小限のコード例を含む説明}

## よくあるパターン

### {パターン1}
```{language}
{コード}
```

### {パターン2}
```{language}
{コード}
```

## 設定オプション

| 設定 | ファイル | 目的 |
|---------|------|---------|
| {設定} | {ファイル} | {内容} |

## デプロイ

{簡単なデプロイガイダンスまたは参照}

詳細なデプロイは：`microsoft_docs_search(query="{framework} deploy {target}")`

## さらに学ぶ

| トピック | 検索方法 |
|-------|-------------|
| {高度な機能} | `microsoft_docs_search(query="{framework} {feature}")` |
| {統合} | `microsoft_docs_fetch(url="{url}")` |
| {サンプル} | `microsoft_code_sample_search(query="{framework} {scenario}")` |
```

---

## テンプレート4：API/プロトコルスキル

API、プロトコル、仕様向け（例：Microsoft Graph、OOXML）。

```markdown
---
name: {api-name}
description: {API/プロトコル}と対話。エージェントが{主な操作}を行う必要がある場合に使用。認証、エンドポイント、共通操作をカバー。
---

# {API/プロトコル名}

{1段落：アクセス可能な内容、主なユースケース}

## 認証

{認証方法とコードパターン}

## 基本設定

- **ベースURL**: `{url}`
- **バージョン**: `{version}`
- **フォーマット**: {JSON/XMLなど}

## 共通エンドポイント/操作

### {操作1：例「アイテム一覧取得」}
```
{HTTPメソッド} {エンドポイント}
```
```{language}
{SDKコード}
```

### {操作2：例「アイテム作成」}
```
{HTTPメソッド} {エンドポイント}
```
```{language}
{SDKコード}
```

## リクエスト/レスポンスパターン

### ページネーション
{ページネーションの扱い方}

### エラーハンドリング
{エラーフォーマットと一般的なコード}

## クイックリファレンス

| 操作 | エンドポイント/メソッド | 備考 |
|-----------|-----------------|-------|
| {op} | `{endpoint}` | {備考} |

## 権限/スコープ

| 操作 | 必要な権限 |
|-----------|---------------------|
| {op} | `{permission}` |

## さらに学ぶ

| トピック | 検索方法 |
|-------|-------------|
| 完全なエンドポイントリファレンス | `microsoft_docs_fetch(url="{url}")` |
| 権限 | `microsoft_docs_search(query="{api} permissions {resource}")` |
| SDK | `microsoft_docs_search(query="{api} SDK {language}")` |
```

---

## テンプレートの選択

| 技術タイプ | テンプレート | 例 |
|-----------------|----------|----------|
| クライアントライブラリ、NuGet/npmパッケージ | SDK/ライブラリ | Semantic Kernel、Azure SDK、MSAL |
| Azureリソース | Azureサービス | Cosmos DB、Azure Functions、App Service |
| アプリ開発フレームワーク | フレームワーク/プラットフォーム | ASP.NET Core、Blazor、MAUI |
| REST API、プロトコル、仕様 | API/プロトコル | Microsoft Graph、OOXML、FHIR |

## カスタマイズガイドライン

テンプレートは出発点です。以下でカスタマイズしてください：

1. 技術固有の側面に合わせて**セクションを追加**
2. 該当しないセクションは**削除**
3. 複雑さに応じて**深さを調整**（複雑な技術は概念を増やす）
4. SKILL.mdに収まらない詳細は**参照ファイルを追加**
5. インラインスニペット以上の動作例は**sample_codes/**に追加
