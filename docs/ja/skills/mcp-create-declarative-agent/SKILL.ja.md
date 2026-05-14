---
name: mcp-create-declarative-agent
description: 'mcp-create-declarative-agent.prompt.md から変換されたスキル'
---

````prompt
---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: '認証、ツール選択、設定を備えたMCPサーバーと統合し、Microsoft 365 Copilot用の宣言型エージェントを作成する'
model: 'gpt-4.1'
tags: [mcp, m365-copilot, declarative-agent, model-context-protocol, api-plugin]
---

# Microsoft 365 Copilot向けMCPベース宣言型エージェントの作成

外部システムやデータにアクセスするためにModel Context Protocol（MCP）サーバーと統合されたMicrosoft 365 Copilot用の完全な宣言型エージェントを作成します。

## 要件

Microsoft 365 Agents Toolkitを使用して以下のプロジェクト構造を生成してください。

### プロジェクトセットアップ
1. Agents Toolkitで**宣言型エージェントをスキャフォールド**する
2. MCPサーバーを指す**MCPアクションを追加**する
3. MCPサーバーからインポートする**ツールを選択**する
4. **認証を設定**する（OAuth 2.0またはSSO）
5. 生成されたファイル（manifest.json、ai-plugin.json、declarativeAgent.json）を**確認**する

### 生成される主なファイル

**appPackage/manifest.json** - プラグイン参照付きTeamsアプリマニフェスト:
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/vDevPreview/MicrosoftTeams.schema.json",
  "manifestVersion": "devPreview",
  "version": "1.0.0",
  "id": "...",
  "developer": {
    "name": "...",
    "websiteUrl": "...",
    "privacyUrl": "...",
    "termsOfUseUrl": "..."
  },
  "name": {
    "short": "Agent Name",
    "full": "Full Agent Name"
  },
  "description": {
    "short": "Short description",
    "full": "Full description"
  },
  "copilotAgents": {
    "declarativeAgents": [
      {
        "id": "declarativeAgent",
        "file": "declarativeAgent.json"
      }
    ]
  }
}
```

**appPackage/declarativeAgent.json** - エージェント定義:
```json
{
  "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "v1.0",
  "name": "Agent Name",
  "description": "Agent description",
  "instructions": "あなたは[specific domain]を支援するアシスタントです。利用可能なツールを使って[capabilities]を実行してください。",
  "capabilities": [
    {
      "name": "WebSearch",
      "websites": [
        {
          "url": "https://learn.microsoft.com"
        }
      ]
    },
    {
      "name": "MCP",
      "file": "ai-plugin.json"
    }
  ]
}
```

**appPackage/ai-plugin.json** - MCPプラグインマニフェスト:
```json
{
  "schema_version": "v2.1",
  "name_for_human": "Service Name",
  "description_for_human": "ユーザー向け説明",
  "description_for_model": "AIモデル向け説明",
  "contact_email": "support@company.com",
  "namespace": "serviceName",
  "capabilities": {
    "conversation_starters": [
      {
        "text": "例のクエリ1"
      }
    ]
  },
  "functions": [
    {
      "name": "functionName",
      "description": "関数の説明",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.title",
            "subtitle": "$.description"
          }
        }
      }
    }
  ],
  "runtimes": [
    {
      "type": "MCP",
      "spec": {
        "url": "https://api.service.com/mcp/"
      },
      "run_for_functions": ["functionName"],
      "auth": {
        "type": "OAuthPluginVault",
        "reference_id": "${{OAUTH_REFERENCE_ID}}"
      }
    }
  ]
}
```

**/.vscode/mcp.json** - MCPサーバー設定:
```json
{
  "serverUrl": "https://api.service.com/mcp/",
  "pluginFilePath": "appPackage/ai-plugin.json"
}
```

## MCPサーバー統合

### 対応するMCPエンドポイント
MCPサーバーは以下を提供する必要があります：
- **サーバーメタデータ**エンドポイント
- **ツール一覧**エンドポイント（利用可能な関数を公開）
- **ツール実行**エンドポイント（関数呼び出しを処理）

### ツール選択
MCPからインポートする際：
1. サーバーから利用可能なツールを取得
2. セキュリティや簡潔さのために含めるツールを選択
3. ツール定義はai-plugin.jsonに自動生成される

### 認証タイプ

**OAuth 2.0（静的登録）**
```json
"auth": {
  "type": "OAuthPluginVault",
  "reference_id": "${{OAUTH_REFERENCE_ID}}",
  "authorization_url": "https://auth.service.com/authorize",
  "client_id": "${{CLIENT_ID}}",
  "client_secret": "${{CLIENT_SECRET}}",
  "scope": "read write"
}
```

**シングルサインオン（SSO）**
```json
"auth": {
  "type": "SSO"
}
```

## レスポンスセマンティクス

### データマッピングの定義
APIレスポンスから関連フィールドを抽出するために`response_semantics`を使用：

```json
"capabilities": {
  "response_semantics": {
    "data_path": "$.results",
    "properties": {
      "title": "$.name",
      "subtitle": "$.description",
      "url": "$.link"
    }
  }
}
```

### アダプティブカードの追加（任意）
視覚的なカードテンプレートを追加するには、`mcp-create-adaptive-cards`プロンプトを参照してください。

## 環境設定

認証情報用に`.env.local`または`.env.dev`を作成：

```env
OAUTH_REFERENCE_ID=your-oauth-reference-id
CLIENT_ID=your-client-id
CLIENT_SECRET=your-client-secret
```

## テストとデプロイ

### ローカルテスト
1. Agents Toolkitでエージェントを**プロビジョニング**
2. **デバッグ開始**してTeamsにサイドロード
3. https://m365.cloud.microsoft/chat でMicrosoft 365 Copilot内でテスト
4. プロンプトに従い認証
5. 自然言語でエージェントに問い合わせ

### 検証
- ai-plugin.jsonでツールのインポートを確認
- 認証設定をチェック
- 公開された各関数をテスト
- レスポンスデータマッピングを検証

## ベストプラクティス

### ツール設計
- **焦点を絞った関数**：各ツールは一つのことを得意にする
- **明確な説明**：モデルがいつツールを使うか理解しやすくする
- **最小限のスコープ**：エージェントに必要なツールのみをインポート
- **説明的な名前**：アクション指向の関数名を使用

### セキュリティ
- **本番環境ではOAuth 2.0を使用**
- **シークレットは環境変数に保存**
- **MCPサーバー側で入力を検証**
- **必要最小限の権限にスコープを制限**
- **OAuth登録にはリファレンスIDを使用**

### インストラクション
- **エージェントの目的と能力を具体的に記述**
- **成功時とエラー時の動作を定義**
- **該当する場合はツールを明示的に参照**
- **ユーザーにエージェントの可能・不可能を伝える**

### パフォーマンス
- **MCPサーバーで適切にレスポンスをキャッシュ**
- **可能な限りバッチ処理を活用**
- **長時間処理にはタイムアウトを設定**
- **大規模データはページネーションを実装**

## よくあるMCPサーバー例

### GitHub MCPサーバー
```
URL: https://api.githubcopilot.com/mcp/
Tools: search_repositories, search_users, get_repository
Auth: OAuth 2.0
```

### Jira MCPサーバー
```
URL: https://your-domain.atlassian.net/mcp/
Tools: search_issues, create_issue, update_issue
Auth: OAuth 2.0
```

### カスタムサービス
```
URL: https://api.your-service.com/mcp/
Tools: サービスが公開するカスタムツール
Auth: OAuth 2.0 または SSO
```

## ワークフロー

ユーザーに質問：
1. 統合するMCPサーバーのURLは？
2. Copilotに公開するツールは？
3. サーバーがサポートする認証方式は？
4. エージェントの主な目的は？
5. レスポンスセマンティクスやアダプティブカードは必要か？

その後、以下を生成：
- 完全なappPackage/構造（manifest.json、declarativeAgent.json、ai-plugin.json）
- mcp.json設定ファイル
- .env.localテンプレート
- プロビジョニングとテスト手順

## トラブルシューティング

### MCPサーバーが応答しない
- サーバーURLが正しいか確認
- ネットワーク接続をチェック
- MCPサーバーが必要なエンドポイントを実装しているか検証

### 認証に失敗する
- OAuth認証情報が正しいか確認
- リファレンスIDが登録情報と一致しているかチェック
- スコープが正しく要求されているか確認
- OAuthフローを単独でテスト

### ツールが表示されない
- mcp.jsonが正しいサーバーを指しているか確認
- インポート時にツールを選択したか確認
- ai-plugin.jsonに正しい関数定義があるかチェック
- サーバーが変更された場合はアクションを再取得

### エージェントがクエリを理解しない
- declarativeAgent.jsonのインストラクションを見直す
- 関数説明が明確か確認
- response_semanticsが正しくデータを抽出しているか検証
- より具体的なクエリでテスト

````
