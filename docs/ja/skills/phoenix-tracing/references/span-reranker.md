# RERANKER スパン

## 目的

RERANKER スパンは、取得されたドキュメントの並べ替えを表します (Cohere Rerank、クロスエンコーダー モデル)。

## 必須の属性

|属性 |タイプ |説明 |必須 |
|----------|------|---------------|----------|
| `openinference.span.kind` |文字列 | 「リランカー」である必要があります |はい |

## 属性参照

### リランカーパラメータ

|属性 |タイプ |説明 |
|----------|------|---------------|
| `reranker.model_name` |文字列 |リランカーモデル識別子 |
| `reranker.query` |文字列 |再ランキングに使用されるクエリ |
| `reranker.top_k` |整数 |返す文書の数 |

### 入力ドキュメント

|属性パターン |タイプ |説明 |
|---------------------|------|---------------|
| `reranker.input_documents.{i}.document.id` |文字列 |文書 ID を入力 |
| `reranker.input_documents.{i}.document.content` |文字列 |ドキュメントの内容を入力 |
| `reranker.input_documents.{i}.document.score` |フロート |元の検索スコア |
| `reranker.input_documents.{i}.document.metadata` |文字列 (JSON) |ドキュメントのメタデータ |

### 出力ドキュメント

|属性パターン |タイプ |説明 |
|---------------------|------|---------------|
| `reranker.output_documents.{i}.document.id` |文字列 |出力ドキュメント ID (並べ替え) |
| `reranker.output_documents.{i}.document.content` |文字列 |出力ドキュメントの内容 |
| `reranker.output_documents.{i}.document.score` |フロート |新しいリランカースコア |
| `reranker.output_documents.{i}.document.metadata` |文字列 (JSON) |ドキュメントのメタデータ |

### スコアの比較

入力スコア (レトリーバーから) と出力スコア (リランカーから):```json
{
  "reranker.input_documents.0.document.id": "doc_A",
  "reranker.input_documents.0.document.score": 0.7、
  "reranker.input_documents.1.document.id": "doc_B",
  "reranker.input_documents.1.document.score": 0.9、
  "reranker.output_documents.0.document.id": "doc_B",
  "reranker.output_documents.0.document.score": 0.95、
  "reranker.output_documents.1.document.id": "doc_A",
  「reranker.output_documents.1.document.score」: 0.85
}
「」この例では:
- 入力: doc_A (0.7) よりも上位にランク付けされた doc_B (0.9)
- 出力: doc_B が依然として最高ですが、両方のスコアが増加しました
- リランカーはレトリーバーの順序を確認しましたが、スコアは洗練されました

## 例

### 完全な再ランキングの例```json
{
  "openinference.span.kind": "リランカー",
  "reranker.model_name": "cohere-rerank-v2",
  "reranker.query": "機械学習とは何ですか?",
  "reranker.top_k": 2、
  "reranker.input_documents.0.document.id": "doc_123",
  "reranker.input_documents.0.document.content": "機械学習はサブセットです...",
  "reranker.input_documents.1.document.id": "doc_456",
  "reranker.input_documents.1.document.content": "教師あり学習アルゴリズム...",
  "reranker.input_documents.2.document.id": "doc_789",
  "reranker.input_documents.2.document.content": "ニューラル ネットワークは...",
  "reranker.output_documents.0.document.id": "doc_456",
  "reranker.output_documents.0.document.content": "教師あり学習アルゴリズム...",
  "reranker.output_documents.0.document.score": 0.95、
  "reranker.output_documents.1.document.id": "doc_123",
  "reranker.output_documents.1.document.content": "機械学習はサブセットです...",
  「reranker.output_documents.1.document.score」: 0.88
}
「」
