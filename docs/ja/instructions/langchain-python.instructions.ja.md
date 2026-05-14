---
description: 'Python で LangChain を使うための指示'
applyTo: "**/*.py"
---

# LangChain Python 指示

これらの指示は、GitHub Copilot が Python における LangChain application の code と documentation を生成する際の指針です。LangChain 固有の pattern、API、ベストプラクティスに重点を置いてください。

## Runnable Interface（LangChain 固有）

LangChain の `Runnable` interface は、chain、chat model、output parser、retriever、LangGraph graph を compose / execute するための基盤です。component を呼び出し、batch 化し、stream し、調査し、組み合わせるための統一 API を提供します。

**LangChain 固有の主な機能:**

- LangChain の主要 component（chat model、output parser、retriever、graph）はすべて Runnable interface を実装しています。
- 同期実行（`invoke`, `batch`, `stream`）と非同期実行（`ainvoke`, `abatch`, `astream`）をサポートします。
- batching（`batch`, `batch_as_completed`）は並列 API call 向けに最適化されています。並列度の制御には `RunnableConfig` の `max_concurrency` を設定してください。
- streaming API（`stream`, `astream`, `astream_events`）は生成されるそばから output を返し、応答性の高い LLM app に重要です。
- input / output type は component ごとに異なります（例: chat model は message を受け取り、retriever は string を受け取り、output parser は model output を受け取る）。
- validation や OpenAPI 生成のため、`get_input_schema`、`get_output_schema` およびその JSONSchema variant で schema を確認してください。
- 複雑な LCEL chain では、推論された input / output type を上書きするために `with_types` を使ってください。
- Runnable は LCEL で宣言的に compose できます: `chain = prompt | chat_model | output_parser`。
- Python 3.11+ では `RunnableConfig`（tag、metadata、callback、concurrency）が自動伝播されます。Python 3.9 / 3.10 の async code では手動で渡してください。
- custom runnable は `RunnableLambda`（単純変換）または `RunnableGenerator`（streaming 変換）で作成し、直接 subclass 化は避けてください。
- 動的 chain や LangServe deployment では、`configurable_fields` と `configurable_alternatives` を使って runtime 属性や代替候補を設定してください。

**LangChain のベストプラクティス:**

- LLM や retriever への並列 API call には batching を使い、rate limit を避けるため `max_concurrency` を設定する。
- chat UI や長い output では streaming API を優先する。
- custom chain や deploy 済み endpoint では、常に input / output schema を検証する。
- LangSmith での tracing や複雑な chain の debugging のため、`RunnableConfig` の tag と metadata を使う。
- custom logic では subclass 化より `RunnableLambda` や `RunnableGenerator` で function を wrap する。
- 高度な設定では、`configurable_fields` と `configurable_alternatives` で field と代替候補を公開する。


- 会話 AI には LangChain の chat model integration を使う:

- `langchain.chat_models` または `langchain_openai`（例: `ChatOpenAI`）から import する。
- message は `SystemMessage`, `HumanMessage`, `AIMessage` で compose する。
- tool calling には `bind_tools(tools)` method を使う。
- structured output には `with_structured_output(schema)` を使う。

例:
```python
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage

chat = ChatOpenAI(model="gpt-4", temperature=0)
messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What is LangChain?")
]
response = chat.invoke(messages)
print(response.content)
```

- message は `SystemMessage`, `HumanMessage`、必要に応じて `AIMessage` object の list として compose する。
- RAG では、context injection のために chat model を retriever / vectorstore と組み合わせる。
- real-time token streaming が必要なら `streaming=True` を使う（サポートされる場合）。
- function / tool calling には `tools` argument を使う（OpenAI、Anthropic など）。
- structured output には `response_format="json"` を使う（OpenAI model）。

ベストプラクティス:

- model output は downstream task で使う前に必ず検証する。
- 明確さと信頼性のため、明示的な message type を優先する。
- Copilot 向けには、明確で実行可能な prompt を与え、期待する output を文書化する。



- LLM client factory: provider 設定（API key）、timeout、retry、telemetry を一元化する。provider や client 設定を切り替える場所を 1 か所に集約する。
- Prompt template: template は `prompts/` 配下に置き、安全な helper で読み込む。template は小さく、test しやすく保つ。
- Chain vs Agent: 決定的な pipeline（RAG、summarization）には Chain を優先する。planning や動的 tool selection が必要な場合に Agent を使う。
- Tool: tool には型付き adapter interface を実装し、input / output を厳密に検証する。
- Memory: デフォルトは stateless design。memory が必要なら最小限の context だけを保持し、retention / erasure policy を文書化する。
- Retriever: retrieval + rerank pipeline を構築する。vectorstore schema（id、text、metadata）は安定させる。

### パターン

- Callback と tracing: LangChain callback を使い、LangSmith や独自 tracing system に統合して request / response lifecycle を記録する。
- 関心の分離: prompt の構築、LLM の接続、business logic を分離し、test を簡単にし、意図しない prompt 変更を減らす。

## Embedding と vectorstore

- chunking と metadata field（source、page、chunk_index）は一貫させる。
- 変更のない document に対する繰り返しコストを避けるため、embedding を cache する。
- local / dev には Chroma または FAISS。本番では規模と SLA に応じて managed vector DB（Pinecone、Qdrant、Milvus、Weaviate）を使う。

## Vector store（LangChain 固有）

- semantic search、retrieval-augmented generation (RAG)、document similarity workflow には LangChain の vectorstore integration を使う。
- vectorstore は必ずサポートされる embedding model（例: OpenAIEmbeddings、HuggingFaceEmbeddings）で初期化する。
- 本番では公式 integration（Chroma、FAISS、Pinecone、Qdrant、Weaviate）を優先し、test や demo には InMemoryVectorStore を使う。
- document は `page_content` と `metadata` を持つ LangChain `Document` object として保存する。
- document の追加 / 更新には `add_documents(documents, ids=...)` を使い、upsert のために常に一意の ID を与える。
- ID 指定で削除するには `delete(ids=...)` を使う。
- 類似 document 取得には `similarity_search(query, k=4, filter={...})` を使う。対象を絞るには metadata filter を使う。
- RAG では vectorstore を retriever に接続し、LLM と chain する（LangChain Retriever と RAGChain の doc を参照）。
- 高度な search には vectorstore 固有の option を使う。Pinecone は hybrid search と metadata filtering、Chroma は filtering と custom distance metric をサポートする。
- LangChain の release 間では breaking change が多いため、環境内の vectorstore integration と API version を必ず確認する。
- 例（InMemoryVectorStore）:

```python
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

embedding_model = OpenAIEmbeddings()
vector_store = InMemoryVectorStore(embedding=embedding_model)

documents = [Document(page_content="LangChain content", metadata={"source": "doc1"})]
vector_store.add_documents(documents=documents, ids=["doc1"])

results = vector_store.similarity_search("What is RAG?", k=2)
for doc in results:
    print(doc.page_content, doc.metadata)
```

- 本番では persistent vectorstore（Chroma、Pinecone、Qdrant、Weaviate）を優先し、provider の doc に従って authentication、scaling、backup を設定する。
- 参考: https://python.langchain.com/docs/integrations/vectorstores/

## Prompt engineering と governance

- canonical prompt は `prompts/` 配下に保存し、code からは file 名で参照する。
- 必須 placeholder が存在すること、render 後の prompt が期待パターン（長さ、variable の存在）に合致することを assert する unit test を書く。
- 挙動に影響する prompt や schema の変更については CHANGELOG を維持する。

## Chat model

LangChain は、monitoring、debugging、optimization の追加機能とともに、一貫した chat model interface を提供します。

### Integration

integration には次の 2 種類があります。

1. Official: LangChain team または provider が保守する `langchain-<provider>` integration。
2. Community: `langchain-community` に含まれる contributor 提供 integration。

chat model は通常 `Chat` prefix を持つ命名規約に従います（例: `ChatOpenAI`, `ChatAnthropic`, `ChatOllama`）。`Chat` prefix を持たない model（または `LLM` suffix を持つもの）は古い string-in / string-out interface を実装していることが多く、現代的な chat workflow では優先度が低くなります。

### Interface

chat model は `BaseChatModel` を実装し、streaming、async、batching などを含む Runnable interface をサポートします。多くの operation は LangChain `messages`（`system`, `user`, `assistant` の role）を受け取り、返します。詳細は BaseChatModel API reference を参照してください。

主な method:

- `invoke(messages, ...)` — message の list を送り、response を受け取る。
- `stream(messages, ...)` — token が届くたびに partial output を stream する。
- `batch(inputs, ...)` — 複数 request をまとめて処理する。
- `bind_tools(tools)` — tool calling 用の adapter を紐付ける。
- `with_structured_output(schema)` — structured response を要求する helper。

### Input と output

- LangChain 独自の message format と OpenAI の message format の両方をサポートします。codebase ではどちらかに統一してください。
- message は `role` と `content` block を持ち、サポートされる場合は structured または multimodal payload を content に含められます。

### 標準 parameter

一般的にサポートされる parameter（provider 依存）:

- `model`: model identifier（例: `gpt-4o`, `gpt-3.5-turbo`）。
- `temperature`: ランダム性の制御（0.0 は決定的、1.0 は創造的）。
- `timeout`: cancel するまでの秒数。
- `max_tokens`: response token 上限。
- `stop`: stop sequence。
- `max_retries`: network / limit failure に対する retry 回数。
- `api_key`, `base_url`: provider 認証と endpoint 設定。
- `rate_limiter`: request 間隔を空けて quota error を避けるための任意の BaseRateLimiter。

> 注: すべての provider がすべての parameter を実装しているわけではありません。必ず provider integration の doc を確認してください。

### Tool calling

chat model は tool（API、DB、system adapter）を呼び出せます。LangChain の tool-calling API を使って次を実現してください。

- strict な input / output typing を備えた tool を登録する。
- tool call の request と result を観測・記録する。
- model に返したり副作用を実行したりする前に tool output を検証する。

例と安全な pattern は LangChain doc の tool-calling guide を参照してください。

### Structured output

JSON または型付き output を model に要求するには `with_structured_output` あるいは schema 強制の method を使ってください。structured output は、抽出、parser、DB 書き込み、analytics などの downstream 処理を信頼できるものにするため不可欠です。

### Multimodality

一部 model は image や audio のような multimodal input をサポートします。サポートされる input type と制約は provider doc を確認してください。multimodal output はまれなので、実験的なものとして慎重に検証してください。

### Context window

model には token 単位で測られる有限の context window があります。会話フローを設計する際は次を守ってください。

- message は簡潔にし、重要な context を優先する。
- window を超える古い context は model 外で要約・退避する。
- 長文 document を chat に貼り付ける代わりに、retriever + RAG pattern で関連 context を提示する。

## 高度なトピック

### Rate-limiting

- chat model 初期化時に `rate_limiter` を使って call 間隔を空ける。
- retry は exponential backoff で実装し、throttled 時には fallback model や degraded mode も検討する。

### Caching

- 会話の完全一致入力に対する caching は効果が薄いことが多いです。意味レベルでの繰り返し問い合わせには semantic caching（embedding ベース）を検討してください。
- semantic caching は embedding への依存を導入するため、すべてのケースに適しているわけではありません。
- cost 削減と正しさの両立ができる場所（例: FAQ bot）だけで cache を使ってください。

## ベストプラクティス

- public API には type hint と dataclass を使う。
- LLM や tool を呼ぶ前に input を検証する。
- secret は secret manager から読み込む。secret や未加工の model output を log に残さない。
- 決定的な test のために LLM と embedding call を mock する。
- embedding と頻出 retrieval 結果は cache する。
- observability のため、request_id、model 名、latency、サニタイズ済み token count を log に残す。
- 外部 call には exponential backoff と idempotency を実装する。

## セキュリティとプライバシー

- model output は信頼できない入力として扱う。生成 code や system command を実行する前にサニタイズする。
- SSRF や injection 攻撃を避けるため、user 提供 URL と input は検証する。
- data retention を文書化し、user data を削除する API を用意する。
- 保存する PII は最小限にし、機密 field は保存時に暗号化する。
