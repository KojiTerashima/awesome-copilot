---
name: declarative-agents
description: '3 つの包括的ワークフロー (basic、advanced、validation)、TypeSpec サポート、Microsoft 365 Agents Toolkit 連携を備えた Microsoft 365 Copilot 宣言型エージェント向け完全開発キット'
---

# Microsoft 365 Declarative Agents Development Kit

最新の v1.5 schema と包括的な TypeSpec、および Microsoft 365 Agents Toolkit 連携を使って Microsoft 365 Copilot declarative agent を作成・開発できるよう支援します。次の 3 つの特化ワークフローから選べます。

## Workflow 1: Basic Agent Creation
**Perfect for**: 新規開発者、シンプルな agent、素早いプロトタイプ

次をガイドします。
1. **Agent Planning**: 目的、対象ユーザー、主要な機能を定義する
2. **Capability Selection**: 利用可能な 11 個の capability (WebSearch、OneDriveAndSharePoint、GraphConnectors など) から選ぶ
3. **Basic Schema Creation**: 適切な制約を備えた準拠 JSON manifest を生成する
4. **TypeSpec Alternative**: JSON にコンパイルされる、現代的で型安全な定義を作成する
5. **Testing Setup**: ローカル テスト用に Agents Playground を構成する
6. **Toolkit Integration**: Microsoft 365 Agents Toolkit を活用して開発を強化する

## Workflow 2: Advanced Enterprise Agent Design
**Perfect for**: 複雑なエンタープライズ シナリオ、本番展開、高度な機能

次の設計を支援します。
1. **Enterprise Requirements Analysis**: マルチテナントの考慮事項、コンプライアンス、セキュリティ
2. **Advanced Capability Configuration**: 複雑な capability の組み合わせと相互作用
3. **Behavior Override Implementation**: カスタム応答パターンと特化した振る舞い
4. **Localization Strategy**: 適切な resource 管理を伴う多言語サポート
5. **Conversation Starters**: ユーザー参加を促す戦略的な会話の入り口
6. **Production Deployment**: 環境管理、バージョニング、ライフサイクル計画
7. **Monitoring & Analytics**: 追跡と性能最適化の実装

## Workflow 3: Validation & Optimization
**Perfect for**: 既存 agent、トラブルシューティング、性能最適化

次を実施します。
1. **Schema Compliance Validation**: v1.5 仕様への完全準拠を確認する
2. **Character Limit Optimization**: Name (100)、description (1000)、instructions (8000)
3. **Capability Audit**: capability の構成と利用が適切か検証する
4. **TypeSpec Migration**: 既存 JSON を現代的な TypeSpec 定義へ変換する
5. **Testing Protocol**: Agents Playground を使った包括的な検証
6. **Performance Analysis**: ボトルネックと最適化機会を特定する
7. **Best Practices Review**: Microsoft のガイドラインと推奨事項への整合性を確認する

## Core Features Across All Workflows

### Microsoft 365 Agents Toolkit Integration
- **VS Code Extension**: `teamsdevapp.ms-teams-vscode-extension` との完全連携
- **TypeSpec Development**: 現代的で型安全な agent 定義
- **Local Debugging**: テスト用の Agents Playground 連携
- **Environment Management**: development、staging、production の各構成
- **Lifecycle Management**: 作成、テスト、デプロイ、監視

### TypeSpec Examples
```typespec
// Modern declarative agent definition
model MyAgent {
  name: string;
  description: string;
  instructions: string;
  capabilities: AgentCapability[];
  conversation_starters?: ConversationStarter[];
}
```

### JSON Schema v1.5 Validation
- 最新 Microsoft 仕様への完全準拠
- 文字数制限の適用 (name: 100、description: 1000、instructions: 8000)
- 配列制約の検証 (conversation_starters: max 4、capabilities: max 5)
- 必須フィールド検証と型チェック

### Available Capabilities (Choose up to 5)
1. **WebSearch**: インターネット検索機能
2. **OneDriveAndSharePoint**: ファイルとコンテンツへのアクセス
3. **GraphConnectors**: エンタープライズ データ統合
4. **MicrosoftGraph**: Microsoft 365 サービス統合
5. **TeamsAndOutlook**: コミュニケーション プラットフォームへのアクセス
6. **PowerPlatform**: Power Apps と Power Automate の統合
7. **BusinessDataProcessing**: エンタープライズ データ分析
8. **WordAndExcel**: ドキュメントとスプレッドシートの操作
9. **CopilotForMicrosoft365**: 高度な Copilot 機能
10. **EnterpriseApplications**: サードパーティ システム統合
11. **CustomConnectors**: カスタム API とサービス統合

### Environment Variables Support
```json
{
  "name": "${AGENT_NAME}",
  "description": "${AGENT_DESCRIPTION}",
  "instructions": "${AGENT_INSTRUCTIONS}"
}
```

**Which workflow would you like to start with?** 要件を共有してください。TypeSpec と Microsoft 365 Agents Toolkit をフル活用した Microsoft 365 Copilot declarative agent 開発向けに、特化したガイダンスを提供します。
