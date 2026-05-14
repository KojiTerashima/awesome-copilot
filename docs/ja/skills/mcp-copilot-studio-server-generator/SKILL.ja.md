---
name: mcp-copilot-studio-server-generator
description: 'Copilot Studio統合に最適化された完全なMCPサーバー実装を、適切なスキーマ制約とストリーム可能なHTTPサポート付きで生成します'
---

# Power Platform MCPコネクタージェネレーター

Microsoft Copilot Studio向けにModel Context Protocol（MCP）統合を備えた完全なPower Platformカスタムコネクターを生成します。このプロンプトは、Power Platformコネクター標準に従い、MCPストリーム可能HTTPサポートを含むすべての必要なファイルを作成します。

## 手順

以下を満たす完全なMCPサーバー実装を作成してください：

1. **Copilot Studio MCPパターンの使用：**
   - `x-ms-agentic-protocol: mcp-streamable-1.0`を実装
   - JSON-RPC 2.0通信プロトコルをサポート
   - `/mcp`でストリーム可能なHTTPエンドポイントを提供
   - Power Platformコネクター構造に従う

2. **スキーマ準拠要件：**
   - ツールの入力/出力に**参照型を使用しない**（Copilot Studioでフィルタリングされる）
   - **単一タイプの値のみ**（複数タイプの配列は不可）
   - **enum入力は避ける**（enumではなく文字列として解釈される）
   - プリミティブ型のみ使用：string、number、integer、boolean、array、object
   - すべてのエンドポイントは完全なURIを返すこと

3. **含めるMCPコンポーネント：**
   - **ツール**：言語モデルが呼び出す関数（✅ Copilot Studioでサポート）
   - **リソース**：ツールからのファイルのようなデータ出力（✅ Copilot Studioでサポート - アクセス可能にするにはツール出力である必要あり）
   - **プロンプト**：特定タスク用の事前定義テンプレート（❌ Copilot Studioでは未対応）

4. **実装構造：**
   ```
   /apiDefinition.swagger.json  (Power Platformコネクタースキーマ)
   /apiProperties.json         (コネクターメタデータと設定)
   /script.csx                 (カスタムコード変換とロジック)
   /server/                    (MCPサーバー実装)
   /tools/                     (個別MCPツール)
   /resources/                 (MCPリソースハンドラー)
   ```

## コンテキスト変数

- **サーバーの目的**： [MCPサーバーが達成すべき内容を記述]
- **必要なツール**： [実装する特定ツールのリスト]  
- **リソース**： [提供するリソースの種類]
- **認証方式**： [認証方法：なし、api-key、oauth2]
- **ホスト環境**： [Azure Function、Express.js、FastAPIなど]
- **対象API**： [統合する外部API]

## 期待される出力

生成するもの：

1. **apiDefinition.swagger.json**：
   - 適切な`x-ms-agentic-protocol: mcp-streamable-1.0`
   - POST `/mcp`のMCPエンドポイント
   - 参照型なしの準拠スキーマ定義
   - McpResponseおよびMcpErrorResponse定義

2. **apiProperties.json**：
   - コネクターメタデータとブランディング
   - 認証設定
   - 必要に応じたポリシーテンプレート

3. **script.csx**：
   - リクエスト/レスポンス変換のカスタムC#コード
   - MCP JSON-RPCメッセージ処理ロジック
   - データ検証および処理関数
   - エラー処理とログ機能

4. **MCPサーバーコード**：
   - JSON-RPC 2.0リクエストハンドラー
   - ツール登録と実行
   - リソース管理（ツール出力として）
   - 適切なエラー処理
   - Copilot Studio互換性チェック

5. **個別ツール**：
   - プリミティブ型入力のみ受け付ける
   - 構造化された出力を返す
   - 必要に応じてリソースを出力に含める
   - Copilot Studio向けに明確な説明を提供

6. **デプロイ構成**：
   - Power Platform環境用
   - Copilot Studioエージェント統合用
   - テストおよび検証用

## 検証チェックリスト

生成コードが以下を満たすことを確認してください：
- [ ] スキーマに参照型なし
- [ ] すべての型フィールドは単一タイプ
- [ ] enumは文字列として扱い検証
- [ ] リソースはツール出力経由で利用可能
- [ ] 完全なURIエンドポイント
- [ ] JSON-RPC 2.0準拠
- [ ] 適切なx-ms-agentic-protocolヘッダー
- [ ] McpResponse/McpErrorResponseスキーマ
- [ ] Copilot Studio向けに明確なツール説明
- [ ] Generative Orchestration対応

## 使用例

```yaml
Server Purpose: 顧客データ管理と分析
Tools Needed: 
  - searchCustomers
  - getCustomerDetails
  - analyzeCustomerTrends
Resources:
  - 顧客プロファイル
  - 分析レポート
Authentication: oauth2
Host Environment: Azure Function
Target APIs: CRMシステムREST API
```
