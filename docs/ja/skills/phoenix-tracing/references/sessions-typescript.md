セッション数 (TypeScript)

トレースをセッション ID でグループ化することで、複数ターンの会話を追跡します。 **`withSpan` を `@arizeai/openinference-core`** から直接使用します。ラッパーやカスタム ユーティリティは必要ありません。

## コアコンセプト

**セッション パターン:**
1. アプリケーションの起動時に一意の `session.id` を 1 回生成します
2. SESSION_ID をエクスポートし、必要に応じて `withSpan` をインポートします
3. `withSpan` を使用して、インタラクションごとに `session.id` を持つ親 CHAIN スパンを作成します
4. すべての子スパン (LLM、TOOL、AGENT など) は自動的に親の下にグループ化されます。
5. Phoenix で `session.id` によってトレースをクエリして、すべてのインタラクションを確認します

## 実装 (ベスト プラクティス)

### 1. セットアップ (instrumentation.ts)```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
「node:crypto」から {randomUUID} をインポートします。

// フェニックスを初期化する
登録({
  プロジェクト名: "あなたのアプリ",
  URL: process.env.PHOENIX_COLLECTOR_ENDPOINT || "http://localhost:6006",
  apiKey: process.env.PHOENIX_API_KEY、
  バッチ: true、
});

// セッション ID を生成してエクスポートする
エクスポート const SESSION_ID = ランダムUUID();
「」### 2. 使い方（アプリコード）```タイプスクリプト
import { withSpan } から "@arizeai/openinference-core";
import { SESSION_ID } から "./instrumentation";

// withSpan を直接使用します - ラッパーは必要ありません
const handleInteraction = withSpan(
  非同期() => {
    const result = await Agent.generate({ プロンプト: userInput });
    結果を返します。
  }、
  {
    名前: "cli.interaction"、
    種類：「チェーン」、
    属性: { "session.id": SESSION_ID },
  }
);

// それを呼び出します
const result = await handleInteraction();
「」### 入力パラメータあり```タイプスクリプト
const processQuery = withSpan(
  async (クエリ: 文字列) => {
    return await Agent.generate({ プロンプト: クエリ });
  }、
  {
    名前: "プロセス.クエリ"、
    種類：「チェーン」、
    属性: { "session.id": SESSION_ID },
  }
);

await processQuery("2+2 とは何ですか?");
「」## 重要なポイント

### セッション ID のスコープ
- **CLI/デスクトップ アプリ**: プロセスの起動時に 1 回生成します
- **Web サーバー**: ユーザーごとのセッションを生成します (例: ログイン時、セッション ストレージに保存)
- **ステートレス API**: クライアントからのパラメータとして session.id を受け入れます

### スパン階層「」
cli.interaction (CHAIN) ← ここに session.id
§── ai.generateText (AGENT)
│ §── ai.generateText.doGenerate (LLM)
│ └── ai.toolCall (TOOL)
━── ai.generateText.doGenerate (LLM)
「」`session.id` は **ルート スパン** にのみ設定されます。子スパンはトレース階層ごとに自動的にグループ化されます。

### セッションのクエリ「」バッシュ
# セッションのすべてのトレースを取得する
npx @arizeai/phoenix-cli トレース \
  --エンドポイント http://localhost:6006 \
  --アプリをプロジェクト \
  --raw 形式でフォーマットする \
  --進捗なし | \
  jq '.[] | select(.spans[0].attributes["session.id"] == "あなたのセッションID")'
「」## 依存関係```json
{
  "依存関係": {
    "@arizeai/openinference-core": "^2.0.5",
    "@arizeai/phoenix-otel": "^0.4.1"
  }
}
「」**注意:** `@opentelemetry/api` は必要ありません。手動でのスパン管理のみに使用されます。

## なぜこのパターンなのか?

1. **シンプル**: SESSION_ID をエクスポートするだけで、withSpan を直接使用します - ラッパーは使用しません
2. **組み込み**: `@arizeai/openinference-core` から `withSpan` がすべてを処理します
3. **タイプセーフ**: 関数の署名と型情報を保持します。
4. **自動ライフサイクル**: スパンの作成、エラー追跡、クリーンアップを処理します。
5. **フレームワークに依存しない**: あらゆる LLM フレームワーク (AI SDK、LangChain など) で動作します。
6. **追加の deps は不要**: `@opentelemetry/api` やカスタム ユーティリティは必要ありません

## さらに属性を追加する```タイプスクリプト
import { withSpan } から "@arizeai/openinference-core";
import { SESSION_ID } から "./instrumentation";

const handleWithContext = withSpan(
  async (userInput: 文字列) => {
    return await Agent.generate({ プロンプト: userInput });
  }、
  {
    名前: "cli.interaction"、
    種類：「チェーン」、
    属性: {
      "session.id": SESSION_ID、
      "user.id": userId, // ユーザーを追跡する
      "metadata.environment": "prod", // カスタムメタデータ
    }、
  }
);
「」## アンチパターン: ラッパーを作成しない

❌ **これは行わないでください:**```タイプスクリプト
// 不要なラッパー
エクスポート関数 withSessionTracking(fn) {
  return withSpan(fn, { 属性: { "session.id": SESSION_ID } });
}
「」✅ **代わりにこれを実行してください:**```タイプスクリプト
// withSpan を直接使用する
import { withSpan } から "@arizeai/openinference-core";
import { SESSION_ID } から "./instrumentation";

const ハンドラー = withSpan(fn, {
  属性: { "session.id": SESSION_ID }
});
「」## 代替: コンテキスト API パターン

ミドルウェアを通じてセッション ID を伝達する必要がある Web サーバーまたは複雑な非同期フローの場合は、Context API を使用できます。```タイプスクリプト
import { context } から "@opentelemetry/api";
import { setSession } から "@arizeai/openinference-core";

コンテキストを待ちます。with(
  setSession(context.active(), { sessionId: "user_123_conv_456" }),
  非同期() => {
    const 応答 = await llm.invoke(prompt);
  }
);
「」**次の場合にコンテキスト API を使用します。**
- ミドルウェア チェーンを使用した Web サーバーの構築
- セッション ID は多くの非同期境界を通過する必要がある
- 呼び出しスタック (フレームワークが提供するハンドラーなど) を制御しない

**次の場合に withSpan を使用します。**
- CLI アプリまたはスクリプトの構築
- 関数呼び出しポイントを制御します
- より単純で、より明示的なコードが好まれます

## 関連

- `fundamentals-universal-attributes.md` - その他の汎用属性 (user.id、メタデータ)
- `span-chain.md` - CHAINスパン指定
- `sessions-python.md` - Python セッション追跡パターン