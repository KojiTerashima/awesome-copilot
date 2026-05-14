---
description: 'Model Context Protocol 統合を用いて、Microsoft 365 Copilot 向けの MCP ベース declarative agent と API plugin を構築するためのベストプラクティス'
applyTo: '**/{*mcp*,*agent*,*plugin*,declarativeAgent.json,ai-plugin.json,mcp.json,manifest.json}'
---

# MCP ベース M365 Copilot 開発ガイドライン

## コア原則

### Model Context Protocol First
- 外部システム統合には MCP server を活用する
- tool は手動定義ではなく server endpoint から import する
- schema discovery と function generation は MCP に任せる
- Agents Toolkit では point-and-click の tool selection を使う

### 命令型より宣言型
- agent の挙動は code ではなく configuration で定義する
- instruction と capability には declarativeAgent.json を使う
- tool と action は ai-plugin.json で指定する
- MCP server は mcp.json で設定する

### セキュリティとガバナンス
- authentication には常に OAuth 2.0 または SSO を使う
- tool selection には最小権限の原則を適用する
- MCP server endpoint が安全であることを検証する
- deploy 前に compliance requirement を確認する

### ユーザー中心設計
- リッチな視覚応答には adaptive card を作成する
- 明確な conversation starter を提供する
- hub をまたいで responsive な体験になるよう設計する
- 組織展開前に十分にテストする

## MCP Server の設計

### Server の選定
次の条件を満たす MCP server を選んでください。
- user task に関連する tool を公開している
- 安全な authentication（OAuth 2.0、SSO）をサポートしている
- 安定した uptime と performance を提供している
- MCP specification standard に従っている
- 構造化された response data を返す

### Tool Import Strategy
- 必要な tool だけを import する（過剰なスコープを避ける）
- 同じ server からの関連 tool をグループ化する
- tool を組み合わせる前に個別にテストする
- 複数 tool を選ぶときは token limit を考慮する

### Authentication Configuration
**OAuth 2.0 Static Registration:**
```json
{
  "type": "OAuthPluginVault",
  "reference_id": "YOUR_AUTH_ID",
  "client_id": "github_client_id",
  "client_secret": "github_client_secret",
  "authorization_url": "https://github.com/login/oauth/authorize",
  "token_url": "https://github.com/login/oauth/access_token",
  "scope": "repo read:user"
}
```

**SSO (Microsoft Entra ID):**
```json
{
  "type": "OAuthPluginVault",
  "reference_id": "sso_auth",
  "authorization_url": "https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
  "token_url": "https://login.microsoftonline.com/common/oauth2/v2.0/token",
  "scope": "User.Read"
}
```

## ファイル構成

### Project Structure
```
project-root/
├── appPackage/
│   ├── manifest.json           # Teams app manifest
│   ├── declarativeAgent.json   # Agent config (instructions, capabilities)
│   ├── ai-plugin.json          # API plugin definition
│   ├── color.png               # App icon color
│   └── outline.png             # App icon outline
├── .vscode/
│   └── mcp.json               # MCP server configuration
├── .env.local                  # Credentials (NEVER commit)
└── teamsapp.yml               # Teams Toolkit config
```

### 重要ファイル

**declarativeAgent.json:**
- agent 名と description
- 振る舞いに関する instruction
- conversation starter
- capability（plugin 由来の action）

**ai-plugin.json:**
- MCP server tool の import
- response semantics（data_path、properties）
- static adaptive card template
- function definition（自動生成）

**mcp.json:**
- MCP server URL
- server metadata endpoint
- authentication reference

**.env.local:**
- OAuth client credential
- API key と secret
- environment ごとの config
- **重要**: .gitignore に追加する

## Response Semantics のベストプラクティス

### Data Path の設定
関連データ抽出には JSONPath を使ってください。
```json
{
  "data_path": "$.items[*]",
  "properties": {
    "title": "$.name",
    "subtitle": "$.description",
    "url": "$.html_url"
  }
}
```

### Template Selection
動的 template の場合:
```json
{
  "data_path": "$",
  "template_selector": "$.templateType",
  "properties": {
    "title": "$.title",
    "url": "$.url"
  }
}
```

### Static Template
一貫した書式にするには ai-plugin.json に定義してください。
- すべての response が同じ構造に従うときに使う
- 動的 template より高性能
- 保守と version control が容易

## Adaptive Card ガイドライン

### 設計原則
- **Single-column layout**: 要素は縦に積む
- **Flexible widths**: 固定 pixel ではなく `stretch` または `auto` を使う
- **Responsive design**: Chat、Teams、Outlook でテストする
- **Minimal complexity**: card はシンプルでスキャンしやすく保つ

### Template Language のパターン
**Conditionals:**
```json
{
  "type": "TextBlock",
  "text": "${if(status == 'active', '✅ Active', '❌ Inactive')}"
}
```

**Data Binding:**
```json
{
  "type": "TextBlock",
  "text": "${title}",
  "weight": "bolder"
}
```

**Number Formatting:**
```json
{
  "type": "TextBlock",
  "text": "Score: ${formatNumber(score, 0)}"
}
```

**Conditional Rendering:**
```json
{
  "type": "Container",
  "$when": "${count(items) > 0}",
  "items": [ ... ]
}
```

### Card Element の使い分け
- **TextBlock**: title、description、metadata
- **FactSet**: key-value pair（status、date、ID）
- **Image**: icon、thumbnail（`size: "small"` を使う）
- **Container**: 関連コンテンツのグルーピング
- **ActionSet**: follow-up action 用の button

## テストとデプロイ

### ローカルテストのワークフロー
1. **Provision**: Teams Toolkit → Provision
2. **Deploy**: Teams Toolkit → Deploy
3. **Sideload**: app を Teams にアップロード
4. **Test**: [m365.cloud.microsoft/chat](https://m365.cloud.microsoft/chat) を開く
5. **Iterate**: 問題を修正して再 deploy

### デプロイ前チェックリスト
- [ ] すべての MCP server tool を個別にテストした
- [ ] authentication flow が end-to-end で動作する
- [ ] adaptive card が各 hub で正しく描画される
- [ ] response semantics が期待通りの data を抽出する
- [ ] error handling が明確な message を返す
- [ ] conversation starter が関連性と明瞭さを備えている
- [ ] agent instruction が適切な挙動を導いている
- [ ] compliance と security を確認した

### デプロイ方法
**Organization Deployment:**
- IT 管理者が全員または特定の user / group に deploy
- Microsoft 365 admin center での承認が必要
- 社内向け business agent に最適

**Agent Store:**
- Partner Center へ提出して validation を受ける
- すべての Copilot user に公開可能
- 厳格な security review が必要

## よくあるパターン

### Multi-Tool Agent
複数の MCP server から tool を import します。
```json
{
  "mcpServers": {
    "github": {
      "url": "https://github-mcp.example.com"
    },
    "jira": {
      "url": "https://jira-mcp.example.com"
    }
  }
}
```

### Search and Display
1. tool が MCP server から data を取得する
2. response semantics が必要な field を抽出する
3. adaptive card が整形済み結果を表示する
4. user は card button から action を実行できる

### Authenticated Actions
1. user が authentication を要する tool を起動する
2. OAuth flow が consent のために redirect する
3. access token が plugin vault に保存される
4. 以降の request では保存済み token を使う

## エラーハンドリング

### MCP Server Error
- agent response に明確な error message を含める
- 利用可能なら代替 tool にフォールバックする
- debugging 用に error を記録する
- user に再試行または代替手段を案内する

### Authentication Failure
- .env.local 内の OAuth credential を確認する
- scope が必要な permission と一致しているか検証する
- auth flow はまず Copilot の外でテストする
- token refresh logic が動作することを確認する

### Response Parsing Failure
- response semantics の JSONPath expression を検証する
- data 欠落や null を適切に扱う
- 必要なら default value を提供する
- さまざまな API response でテストする

## パフォーマンス最適化

### Tool Selection
- 必要な tool だけを import する（token 使用量削減）
- 複数 server にまたがる重複 tool を避ける
- 各 tool が response time に与える影響を測定する

### Response Size
- 不要な data を減らすために data_path を使う
- 可能なら result set を制限する
- 大量データには pagination を検討する
- adaptive card は軽量に保つ

### Caching Strategy
- MCP server 側で、適切な場合は cache を行うべきです
- agent response は M365 によって cache される可能性があります
- 時間に敏感な data には cache invalidation を検討してください

## セキュリティのベストプラクティス

### Credential Management
- `.env.local` を source control に **絶対に** commit しない
- すべての secret は environment variable で扱う
- OAuth credential を定期的に rotation する
- dev / prod で別 credential を使う

### Data Privacy
- 必要最小限の scope だけを要求する
- 機密性の高い user data をログに残さない
- data residency requirement を確認する
- compliance policy（GDPR など）に従う

### Server Validation
- MCP server が信頼でき、安全であることを検証する
- HTTPS endpoint のみを使う
- server の privacy policy を確認する
- injection vulnerability に対するテストを行う

## ガバナンスとコンプライアンス

### Admin Control
Agent は次の状態になり得ます。
- **Blocked**: 利用を禁止
- **Deployed**: 特定 user / group に割り当て
- **Published**: 組織全体に公開

### Monitoring
追跡すべき項目:
- agent の usage と adoption
- error rate と performance
- user feedback と satisfaction
- security incident

### Audit Requirement
維持すべきもの:
- agent configuration の change history
- 機密操作の access log
- deployment の承認記録
- compliance の証跡

## リソースと参考資料

### 公式ドキュメント
- [Build Declarative Agents with MCP (DevBlogs)](https://devblogs.microsoft.com/microsoft365dev/build-declarative-agents-for-microsoft-365-copilot-with-mcp/)
- [Build MCP Plugins (Learn)](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/build-mcp-plugins)
- [API Plugin Adaptive Cards (Learn)](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/api-plugin-adaptive-cards)
- [Manage Copilot Agents (Learn)](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/manage-copilot-agents-integrated-apps)

### Tool と SDK
- Microsoft 365 Agents Toolkit (VS Code extension v6.3.x+)
- Teams Toolkit for agent packaging
- Adaptive Cards Designer
- MCP specification documentation

### Partner Example
- monday.com: task management integration
- Canva: design automation
- Sitecore: content management
