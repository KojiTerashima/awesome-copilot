---
description: 'Microsoft 365 Copilot 向けに、TypeSpec ベースの declarative agent と API plugin を構築するためのガイドラインとベストプラクティス'
applyTo: '**/*.tsp'
---

# Microsoft 365 Copilot 向け TypeSpec 開発ガイドライン

## 中核原則

Microsoft 365 Copilot 向けに TypeSpec を扱うときは、次を守る:

1. **型安全性を最優先**: すべての model と operation で TypeSpec の強い型付けを活用する
2. **宣言的アプローチ**: 実装ではなく意図を表すために decorator を使う
3. **スコープ付き capability**: 可能な限り capability は特定 resource に限定する
4. **明確な instructions**: 明示的で詳細な agent instruction を書く
5. **ユーザー中心**: Microsoft 365 Copilot における end-user 体験を意識して設計する

## ファイル構成

### 標準構成
```
project/
├── appPackage/
│   ├── cards/              # Adaptive Card template
│   │   └── *.json
│   ├── .generated/         # 生成 manifest (自動生成)
│   └── manifest.json       # Teams app manifest
├── src/
│   ├── main.tsp           # Agent 定義
│   └── actions.tsp        # API operation (plugin 用)
├── m365agents.yml         # Agents Toolkit 設定
└── package.json
```

### Import Statement
TypeSpec file の先頭には必ず必要な import を含める:

```typescript
import "@typespec/http";
import "@typespec/openapi3";
import "@microsoft/typespec-m365-copilot";

using TypeSpec.Http;
using TypeSpec.M365.Copilot.Agents;  // agent 用
using TypeSpec.M365.Copilot.Actions; // API plugin 用
```

## Agent 開発のベストプラクティス

### Agent 宣言
```typescript
@agent({
  name: "Role-Based Name",  // 例: "Customer Support Assistant"
  description: "1,000 文字未満の明確で簡潔な説明"
})
```

- agent が何をするか伝わる role-based name を使う
- description は情報量を保ちつつ簡潔にする
- "Helper" や "Bot" のような generic な名前は避ける

### Instructions
```typescript
@instructions("""
  You are a [specific role] specialized in [domain].

  Your responsibilities include:
  - [Key responsibility 1]
  - [Key responsibility 2]

  When helping users:
  - [Behavioral guideline 1]
  - [Behavioral guideline 2]

  You should NOT:
  - [Constraint 1]
  - [Constraint 2]
""")
```

- 二人称で書く ("You are...")
- agent の役割と専門性を具体的に書く
- 何をすべきかと、何をすべきでないかの両方を定義する
- 8,000 文字未満に保つ
- 明確で構造化された形式を使う

### Conversation Starter
```typescript
@conversationStarter(#{
  title: "Action-Oriented Title",  // 例: "Check Status"
  text: "Specific example query"   // 例: "What's the status of my ticket?"
})
```

- 多様な starter を 2〜4 個用意する
- それぞれで異なる capability を見せる
- action-oriented な title を使う
- 現実的な example query を書く

### Capabilities - Knowledge Source

**Web Search** - 可能な限り特定 site に scope を絞る:
```typescript
op webSearch is AgentCapabilities.WebSearch<Sites = [
  { url: "https://learn.microsoft.com" },
  { url: "https://docs.microsoft.com" }
]>;
```

**OneDrive and SharePoint** - URL または ID を使う:
```typescript
op oneDriveAndSharePoint is AgentCapabilities.OneDriveAndSharePoint<
  ItemsByUrl = [
    { url: "https://contoso.sharepoint.com/sites/Engineering" }
  ]
>;
```

**Teams Messages** - channel / chat を指定する:
```typescript
op teamsMessages is AgentCapabilities.TeamsMessages<Urls = [
  { url: "https://teams.microsoft.com/l/channel/..." }
]>;
```

**Email** - 特定 folder に scope を絞る:
```typescript
op email is AgentCapabilities.Email<
  Folders = [
    { folderId: "Inbox" },
    { folderId: "SentItems" }
  ],
  SharedMailbox = "support@contoso.com"  // Optional
>;
```

**People** - scope 指定は不要:
```typescript
op people is AgentCapabilities.People;
```

**Copilot Connectors** - connection ID を指定する:
```typescript
op copilotConnectors is AgentCapabilities.GraphConnectors<
  Connections = [
    { connectionId: "your-connector-id" }
  ]
>;
```

**Dataverse** - 特定 table に scope を絞る:
```typescript
op dataverse is AgentCapabilities.Dataverse<
  KnowledgeSources = [
    {
      hostName: "contoso.crm.dynamics.com";
      tables: [
        { tableName: "account" },
        { tableName: "contact" }
      ];
    }
  ]
>;
```

### Capabilities - Productivity Tool

```typescript
// Python code execution
op codeInterpreter is AgentCapabilities.CodeInterpreter;

// 画像生成
op graphicArt is AgentCapabilities.GraphicArt;

// 会議 content へのアクセス
op meetings is AgentCapabilities.Meetings;

// 専用 AI model
op scenarioModels is AgentCapabilities.ScenarioModels<
  ModelsById = [
    { id: "model-id" }
  ]
>;
```

## API Plugin 開発のベストプラクティス

### Service 定義
```typescript
@service
@actions(#{
  nameForHuman: "User-Friendly API Name",
  descriptionForHuman: "ユーザーに伝わる説明",
  descriptionForModel: "model が理解すべき説明",
  contactEmail: "support@company.com",
  privacyPolicyUrl: "https://company.com/privacy",
  legalInfoUrl: "https://company.com/terms"
})
@server("https://api.example.com", "API Name")
@useAuth([AuthType])  // 認証が必要な場合
namespace APINamespace {
  // Operation をここに置く
}
```

### Operation 定義
```typescript
@route("/resource/{id}")
@get
@action
@card(#{
  dataPath: "$.items",
  title: "$.title",
  file: "cards/card.json"
})
@capabilities(#{
  confirmation: #{
    type: "AdaptiveCard",
    title: "Confirm Action",
    body: "Confirm with {{ function.parameters.param }}"
  }
})
@reasoning("Y のときは X を考慮する")
@responding("結果は Z として提示する")
op getResource(
  @path id: string,
  @query filter?: string
): ResourceResponse;
```

### Model
```typescript
model Resource {
  id: string;
  name: string;
  description?: string;  // Optional field
  status: "active" | "inactive";  // enum 用の union type
  @format("date-time")
  createdAt: utcDateTime;
  @format("uri")
  url?: string;
}

model ResourceList {
  items: Resource[];
  totalCount: int32;
  nextPage?: string;
}
```

### 認証

**API Key**
```typescript
@useAuth(ApiKeyAuth<ApiKeyLocation.header, "X-API-Key">)

// または reference ID つき
@useAuth(Auth)
@authReferenceId("${{ENV_VAR_REFERENCE_ID}}")
model Auth is ApiKeyAuth<ApiKeyLocation.header, "X-API-Key">;
```

**OAuth2**
```typescript
@useAuth(OAuth2Auth<[{
  type: OAuth2FlowType.authorizationCode;
  authorizationUrl: "https://auth.example.com/authorize";
  tokenUrl: "https://auth.example.com/token";
  refreshUrl: "https://auth.example.com/refresh";
  scopes: ["read", "write"];
}]>)

// または reference ID つき
@useAuth(Auth)
@authReferenceId("${{OAUTH_REFERENCE_ID}}")
model Auth is OAuth2Auth<[...]>;
```

## 命名規則

### File
- `main.tsp` - Agent 定義
- `actions.tsp` - API operation
- `[feature].tsp` - 追加 feature file
- `cards/*.json` - Adaptive Card template

### TypeSpec 要素
- **Namespace**: PascalCase (例: `CustomerSupportAgent`)
- **Operation**: camelCase (例: `listProjects`、`createTicket`)
- **Model**: PascalCase (例: `Project`、`TicketResponse`)
- **Model Property**: camelCase (例: `projectId`、`createdDate`)

## よくあるパターン

### 複数 capability を持つ agent
```typescript
@agent("Knowledge Worker", "Description")
@instructions("...")
namespace KnowledgeWorker {
  op webSearch is AgentCapabilities.WebSearch;
  op files is AgentCapabilities.OneDriveAndSharePoint;
  op people is AgentCapabilities.People;
}
```

### CRUD API Plugin
```typescript
namespace ProjectAPI {
  @route("/projects") @get @action
  op list(): Project[];

  @route("/projects/{id}") @get @action
  op get(@path id: string): Project;

  @route("/projects") @post @action
  @capabilities(#{confirmation: ...})
  op create(@body project: CreateProject): Project;

  @route("/projects/{id}") @patch @action
  @capabilities(#{confirmation: ...})
  op update(@path id: string, @body project: UpdateProject): Project;

  @route("/projects/{id}") @delete @action
  @capabilities(#{confirmation: ...})
  op delete(@path id: string): void;
}
```

### Adaptive Card Data Binding
```json
{
  "type": "AdaptiveCard",
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.5",
  "body": [
    {
      "type": "Container",
      "$data": "${$root}",
      "items": [
        {
          "type": "TextBlock",
          "text": "Title: ${if(title, title, 'N/A')}",
          "wrap": true
        }
      ]
    }
  ]
}
```

## Validation とテスト

### Provisioning 前
1. TypeSpec validation を実行する: `npm run build` または Agents Toolkit を使う
2. `@card` decorator 内のすべての file path が存在するか確認する
3. authentication reference が設定と一致しているか検証する
4. capability scoping が適切か確認する
5. instruction の明確さと長さを見直す

### テスト戦略
1. **Provision**: development environment に deploy する
2. **Test**: Microsoft 365 Copilot (https://m365.cloud.microsoft/chat) を使う
3. **Debug**: orchestrator insight のため Copilot developer mode を有効化する
4. **Iterate**: 実際の挙動に基づいて改善する
5. **Validate**: すべての conversation starter と capability をテストする

## パフォーマンス最適化

1. **Capability を絞る**: 必要なのが一部 data だけなら、全 data への access を与えない
2. **Operation を絞る**: agent が実際に使う API operation だけを公開する
3. **効率的な model**: response model は必要な data に集中させる
4. **Card 最適化**: Adaptive Card では条件付き描画 (`$when`) を使う
5. **Caching**: API は適切な caching header を考慮して設計する

## セキュリティのベストプラクティス

1. **Authentication**: 非公開 API には必ず認証を使う
2. **Scoping**: capability access は最小限必要な resource に限定する
3. **Validation**: API operation のすべての input を検証する
4. **Secret**: 機密 data には environment variable を使う
5. **Reference**: 本番 credential には `@authReferenceId` を使う
6. **Permission**: 必要最小限の OAuth scope を要求する

## Error Handling

```typescript
model ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: ErrorDetail[];
  };
}

model ErrorDetail {
  field?: string;
  message: string;
}
```

## ドキュメント

複雑な operation には TypeSpec comment を含める:

```typescript
/**
 * 関連する task と team member を含む project 詳細を取得する。
 *
 * @param id - 一意な project identifier
 * @param includeArchived - archived task を含めるかどうか
 * @returns 完全な project 情報
 */
@route("/projects/{id}")
@get
@action
op getProjectDetails(
  @path id: string,
  @query includeArchived?: boolean
): ProjectDetails;
```

## 避けるべきよくある落とし穴

1. ❌ generic な agent 名 ("Helper Bot")
2. ❌ 曖昧な instruction ("Help users with things")
3. ❌ capability の scope 指定なし (全 data にアクセス)
4. ❌ 破壊的 operation に confirmation がない
5. ❌ 複雑すぎる Adaptive Card
6. ❌ TypeSpec file に credential をハードコードする
7. ❌ error response model がない
8. ❌ 一貫しない命名規則
9. ❌ capability が多すぎる (必要なものだけ使う)
10. ❌ 8,000 文字を超える instruction

## リソース

- [TypeSpec Official Docs](https://typespec.io/)
- [Microsoft 365 Copilot Extensibility](https://learn.microsoft.com/microsoft-365-copilot/extensibility/)
- [Agents Toolkit](https://aka.ms/M365AgentsToolkit)
- [Adaptive Cards Designer](https://adaptivecards.io/designer/)
