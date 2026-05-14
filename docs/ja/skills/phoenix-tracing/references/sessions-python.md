セッション数 (Python)

トレースをセッション ID でグループ化することで、複数ターンの会話を追跡します。

＃＃ 設定「」パイソン
openinference.instrumentation からのインポート using_session

using_session(session_id="user_123_conv_456") を使用:
    応答 = llm.invoke(プロンプト)
「」## ベストプラクティス

**悪い: 親スパンのみがセッション ID を取得します**「」パイソン
openinference.semconv.trace から SpanAttributes をインポート
opentelemetry インポート トレースから

スパン = トレース.get_current_span()
span.set_attribute(SpanAttributes.SESSION_ID, session_id)
応答 = client.chat.completions.create(...)
「」**良い: すべての子スパンはセッション ID を継承します**「」パイソン
using_session(session_id) を使用:
    応答 = client.chat.completions.create(...)
    結果 = my_custom_function()
「」**理由:** `using_session()` は、セッション ID をすべてのネストされたスパンに自動的に伝播します。

## セッション ID パターン「」パイソン
UUIDをインポートする

session_id = str(uuid.uuid4())
session_id = f"user_{user_id}_conv_{conversation_id}"
session_id = f"デバッグ_{タイムスタンプ}"
「」良い: `str(uuid.uuid4())`、`"user_123_conv_456"`
悪い例: `"session_1"`、`"test"`、空の文字列

## マルチターンチャットボットの例「」パイソン
UUIDをインポートする
openinference.instrumentation からのインポート using_session

session_id = str(uuid.uuid4())
メッセージ = []

def send_message(user_input: str) -> str:
    messages.append({"役割": "ユーザー", "コンテンツ": user_input})

    using_session(session_id) を使用:
        応答 = client.chat.completions.create(
            モデル = "gpt-4"、
            メッセージ=メッセージ
        ）

    Assistant_message = 応答.選択[0].メッセージ.コンテンツ
    messages.append({"役割": "アシスタント", "コンテンツ": Assistant_message})
    アシスタントメッセージを返す
「」## 追加の属性「」パイソン
openinference.instrumentation から import using_attributes

using_attributes(
    user_id="user_123",
    session_id="conv_456",
    メタデータ={"階層": "プレミアム", "地域": "米国西部"}
):
    応答 = llm.invoke(プロンプト)
「」## LangChain の統合

LangChain スレッドは自動的にセッションとして認識されます。「」パイソン
langchain.chat_models から ChatOpenAI をインポート

応答 = llm.invoke(
    [HumanMessage(content="こんにちは!")]、
    config={"メタデータ": {"スレッドID": "ユーザー_123_スレッド"}}
）
「」Phoenix は、`thread_id`、`session_id`、`conversation_id` を認識します。

## 関連項目

- **TypeScript セッション:** `sessions-typescript.md`
- **セッションドキュメント:** https://docs.arize.com/phoenix/tracing/sessions