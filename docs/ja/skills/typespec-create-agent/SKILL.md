---
name: typespec-create-agent
description: 'Generate a complete TypeSpec declarative agent with instructions, capabilities, and conversation starters for Microsoft 365 Copilot'
---
# TypeSpec 宣言エージェントを作成する

次の構造を持つ Microsoft 365 Copilot 用の完全な TypeSpec 宣言エージェントを作成します。

## 要件

以下を使用して `main.tsp` ファイルを生成します。

1. **エージェントの宣言**
   - `@agent` デコレーターをわかりやすい名前と説明とともに使用します。
   - 名前は100文字以内にしてください
   - 説明は 1,000 文字以内にしてください

2. **指示**
   - 明確な行動ガイドラインを備えた `@instructions` デコレータを使用する
   - エージェントの役割、専門知識、性格を定義する
   - エージェントが行うべきこととすべきでないことを指定する
   - 8,000 文字以内に収めてください

3. **会話のきっかけ**
   - 2 ～ 4 人の `@conversationStarter` デコレータを含める
   - それぞれにタイトルとクエリの例が付いています
   - 多様性を持たせ、さまざまな機能を紹介します

4. **機能** (ユーザーのニーズに基づく)
   - `WebSearch` - オプションのサイトスコープを持つ Web コンテンツの場合
   - `OneDriveAndSharePoint` - URL フィルタリングを使用したドキュメント アクセスの場合
   - `TeamsMessages` - Teams チャネル/チャット アクセス用
   - `Email` - フォルダー フィルタリングを使用した電子メール アクセスの場合
   - `People` - 組織の人物検索用
   - `CodeInterpreter` - Python コード実行用
   - `GraphicArt` - 画像生成用
   - `GraphConnectors` - Copilot コネクタのコンテンツ用
   - `Dataverse` - Dataverse データ アクセス用
   - `Meetings` - 会議コンテンツへのアクセス用

## テンプレート構造```typescript
import "@typespec/http";
import "@typespec/openapi3";
import "@microsoft/typespec-m365-copilot";

using TypeSpec.Http;
using TypeSpec.M365.Copilot.Agents;

@agent({
  name: "[Agent Name]",
  description: "[Agent Description]"
})
@instructions("""
  [Detailed instructions about agent behavior, role, and guidelines]
""")
@conversationStarter(#{
  title: "[Starter Title 1]",
  text: "[Example query 1]"
})
@conversationStarter(#{
  title: "[Starter Title 2]",
  text: "[Example query 2]"
})
namespace [AgentName] {
  // Add capabilities as operations here
  op capabilityName is AgentCapabilities.[CapabilityType]<[Parameters]>;
}
```## ベストプラクティス

- わかりやすい役割ベースのエージェント名を使用します (例: 「カスタマー サポート アシスタント」、「リサーチ ヘルパー」)
- 二人称で指示を書きます (「あなたは...」)
- エージェントの専門知識と制限について具体的にする
- さまざまな機能を紹介するさまざまな会話のきっかけを含めます
- エージェントが実際に必要とする機能のみを含めます
- パフォーマンスを向上させるために可能な場合は、機能 (URL、フォルダーなど) をスコープします。
- 複数行の命令には三重引用符で囲まれた文字列を使用します

## 例

ユーザーに次のように尋ねます。
1. エージェントの目的と役割は何ですか?
2. どのような機能が必要ですか?
3. どのような知識源にアクセスする必要がありますか?
4. 典型的なユーザーインタラクションは何ですか?

次に、完全な TypeSpec エージェント定義を生成します。