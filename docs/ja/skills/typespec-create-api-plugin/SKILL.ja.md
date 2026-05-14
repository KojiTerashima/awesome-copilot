---
name: typespec-create-api-plugin
description: 'Generate a TypeSpec API plugin with REST operations, authentication, and Adaptive Cards for Microsoft 365 Copilot'
---
# TypeSpec API プラグインを作成する

外部 REST API と統合する Microsoft 365 Copilot 用の完全な TypeSpec API プラグインを作成します。

## 要件

以下を使用して TypeSpec ファイルを生成します。

### main.tsp - エージェント定義```typescript
import "@typespec/http";
import "@typespec/openapi3";
import "@microsoft/typespec-m365-copilot";
import "./actions.tsp";

using TypeSpec.Http;
using TypeSpec.M365.Copilot.Agents;
using TypeSpec.M365.Copilot.Actions;

@agent({
  name: "[Agent Name]",
  description: "[Description]"
})
@instructions("""
  [Instructions for using the API operations]
""")
namespace [AgentName] {
  // Reference operations from actions.tsp
  op operation1 is [APINamespace].operationName;
}
```### action.tsp - API オペレーション```typescript
import "@typespec/http";
import "@microsoft/typespec-m365-copilot";

using TypeSpec.Http;
using TypeSpec.M365.Copilot.Actions;

@service
@actions(#{
    nameForHuman: "[API Display Name]",
    descriptionForModel: "[Model description]",
    descriptionForHuman: "[User description]"
})
@server("[API_BASE_URL]", "[API Name]")
@useAuth([AuthType]) // Optional
namespace [APINamespace] {
  
  @route("[/path]")
  @get
  @action
  op operationName(
    @path param1: string,
    @query param2?: string
  ): ResponseModel;

  model ResponseModel {
    // Response structure
  }
}
```## 認証オプション

API 要件に基づいて選択します。

1. **認証なし** (パブリック API)```typescript
   // No @useAuth decorator needed
   ```2. **API キー**```typescript
   @useAuth(ApiKeyAuth<ApiKeyLocation.header, "X-API-Key">)
   ```3. **OAuth2**```typescript
   @useAuth(OAuth2Auth<[{
     type: OAuth2FlowType.authorizationCode;
     authorizationUrl: "https://oauth.example.com/authorize";
     tokenUrl: "https://oauth.example.com/token";
     refreshUrl: "https://oauth.example.com/token";
     scopes: ["read", "write"];
   }]>)
   ```4. **登録された認証リファレンス**```typescript
   @useAuth(Auth)
   
   @authReferenceId("registration-id-here")
   model Auth is ApiKeyAuth<ApiKeyLocation.header, "X-API-Key">
   ```## 関数の機能

### 確認ダイアログ```typescript
@capabilities(#{
  confirmation: #{
    type: "AdaptiveCard",
    title: "Confirm Action",
    body: """
    Are you sure you want to perform this action?
      * **Parameter**: {{ function.parameters.paramName }}
    """
  }
})
```### アダプティブカードレスポンス```typescript
@card(#{
  dataPath: "$.items",
  title: "$.title",
  url: "$.link",
  file: "cards/card.json"
})
```### 推論と応答の指示```typescript
@reasoning("""
  Consider user's context when calling this operation.
  Prioritize recent items over older ones.
""")
@responding("""
  Present results in a clear table format with columns: ID, Title, Status.
  Include a summary count at the end.
""")
```## ベストプラクティス

1. **オペレーション名**: 明確なアクション指向の名前を使用します (listProjects、createTicket)
2. **モデル**: リクエストとレスポンス用の TypeScript のようなモデルを定義します。
3. **HTTP メソッド**: 適切な動詞 (@get、@post、@patch、@delete) を使用します。
4. **パス**: @route で RESTful パス規則を使用する
5. **パラメータ**: @path、@query、@header、@bodyを適切に使用します
6. **説明**: モデルを理解するために明確な説明を提供します。
7. **確認**: 破壊的な操作の場合に追加 (重要なデータの削除、更新)
8. **カード**: 複数のデータ項目を含む豊富な視覚的応答に使用します

## ワークフロー

ユーザーに次のように尋ねます。
1. API のベース URL と目的は何ですか?
2. どのような操作が必要ですか (CRUD 操作)?
3. API はどのような認証方法を使用しますか?
4. 操作には確認が必要ですか?
5. 応答にはアダプティブ カードが必要ですか?

次に、以下を生成します。
- `main.tsp` にエージェント定義を入力します
- `actions.tsp` を API 操作とモデルで完了します
- アダプティブ カードが必要な場合は、オプションの `cards/card.json`