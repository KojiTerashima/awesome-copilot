# Phoenix トレース: カスタム メタデータ (Python)

より豊かな可観測性を実現するために、カスタム属性をスパンに追加します。

＃＃ インストール「」バッシュ
pip install openinference-instrumentation
「」## セッション「」パイソン
openinference.instrumentation からのインポート using_session

using_session(session_id="my-session-id") を使用:
    # スパン取得: "session.id" = "my-session-id"
    ...
「」## ユーザー「」パイソン
openinference.instrumentation インポート using_user から

using_user("my-user-id") を使用:
    # スパン取得: "user.id" = "my-user-id"
    ...
「」## メタデータ「」パイソン
openinference.instrumentation からのインポート using_metadata

using_metadata({"key": "value", "experiment_id": "exp_123"}) を使用:
    # スパン get: "metadata" = '{"key": "value", "experiment_id": "exp_123"}'
    ...
「」## タグ「」パイソン
openinference.instrumentation インポート using_tags から

using_tags(["tag_1", "tag_2"]) を使用:
    # スパン取得: "tag.tags" = '["tag_1", "tag_2"]'
    ...
「」## 結合 (using_attributes)「」パイソン
openinference.instrumentation から import using_attributes

using_attributes(
    session_id="私のセッションID",
    user_id="私のユーザーID",
    メタデータ={"環境": "本番環境"},
    タグ=["製品", "v2"],
    プロンプト_テンプレート="回答: {質問}",
    プロンプト_テンプレート_バージョン = "v1.0",
    prompt_template_variables={"質問": "フェニックスとは何ですか?"},
):
    # このコンテキストでスパンに適用されるすべての属性
    ...
「」## 単一スパン上「」パイソン
span.set_attribute("メタデータ", json.dumps({"キー": "値"}))
span.set_attribute("user.id", "user_123")
span.set_attribute("session.id", "session_456")
「」## デコレータとして

すべてのコンテキスト マネージャーはデコレータとして使用できます。「」パイソン
@using_session(session_id="my-session-id")
@using_user("私のユーザーID")
@using_metadata({"env": "prod"})
def my_function():
    ...
「」
