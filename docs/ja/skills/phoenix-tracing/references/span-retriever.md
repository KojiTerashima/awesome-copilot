# レトリバーのスパン

## 目的

RETRIEVER スパンは、ドキュメント/コンテキストの取得操作 (ベクトル DB クエリ、セマンティック検索、キーワード検索) を表します。

## 必須の属性

|属性 |タイプ |説明 |必須 |
|----------|------|---------------|----------|
| `openinference.span.kind` |文字列 | 「レトリバー」でなければなりません |はい |

## 属性参照

### クエリ

|属性 |タイプ |説明 |
|----------|------|---------------|
| `input.value` |文字列 |検索クエリのテキスト |

### ドキュメントスキーマ

|属性パターン |タイプ |説明 |
|---------------------|------|---------------|
| `retrieval.documents.{i}.document.id` |文字列 |一意の文書識別子 |
| `retrieval.documents.{i}.document.content` |文字列 |文書テキストの内容 |
| `retrieval.documents.{i}.document.score` |フロート |関連性スコア (0-1 または距離) |
| `retrieval.documents.{i}.document.metadata` |文字列 (JSON) |ドキュメントのメタデータ |

### ドキュメントのフラット化パターン

ドキュメントはゼロインデックス表記を使用してフラット化されます。「」
取得.documents.0.document.id
取得.documents.0.document.content
取得.documents.0.document.score
取得.documents.1.document.id
取得.documents.1.document.content
取得.documents.1.document.score
...
「」### ドキュメントのメタデータ

共通のメタデータ フィールド (JSON 文字列として保存):```json
{
  "ソース": "knowledge_base.pdf",
  「ページ」: 42、
  "セクション": "はじめに",
  "作者": "ジェーン・ドウ",
  "作成日": "2024-01-15",
  "url": "https://example.com/doc",
  "chunk_id": "chunk_123"
}
「」**メタデータを使用した例:**```json
{
  "retrieval.documents.0.document.id": "doc_123",
  "retrieval.documents.0.document.content": "機械学習はデータ分析の方法です...",
  "retrieval.documents.0.document.score": 0.92、
  "retrieval.documents.0.document.metadata": "{\"source\": \"ml_textbook.pdf\", \"page\": 15, \"chapter\": \" Introduction\"}"
}
「」### 注文

ドキュメントはインデックス (0、1、2、...) によって並べられます。通常:
- インデックス 0 = 最高スコアのドキュメント
- インデックス 1 = 2 番目に高い
-など

フラット化された属性で取得順序を保持します。

### 大きな文書の処理

非常に長い文書の場合:
- `document.content` を最初の N 文字に切り捨てることを検討してください
- 完全なコンテンツを別のドキュメント ストアに保存する
- 完全なコンテンツを参照するには `document.id` を使用します

## 例

### 基本的なベクトル検索```json
{
  "openinference.span.kind": "レトリーバー",
  "input.value": "機械学習とは何ですか?",
  "retrieval.documents.0.document.id": "doc_123",
  "retrieval.documents.0.document.content": "機械学習は人工知能のサブセットです...",
  "retrieval.documents.0.document.score": 0.92、
  "retrieval.documents.0.document.metadata": "{\"source\": \"textbook.pdf\", \"page\": 42}",
  "retrieval.documents.1.document.id": "doc_456",
  "retrieval.documents.1.document.content": "機械学習アルゴリズムはデータからパターンを学習します...",
  "retrieval.documents.1.document.score": 0.87、
  "retrieval.documents.1.document.metadata": "{\"source\": \"article.html\", \"author\": \"Jane Doe\"}",
  "retrieval.documents.2.document.id": "doc_789",
  "retrieval.documents.2.document.content": "教師あり学習は機械学習の一種です...",
  "retrieval.documents.2.document.score": 0.81、
  "retrieval.documents.2.document.metadata": "{\"source\": \"wiki.org\"}",
  "metadata.retriever_type": "vector_search",
  "metadata.vector_db": "松ぼっくり",
  「メタデータ.top_k」: 3
}
「」
