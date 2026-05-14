# 埋め込みスパン

## 目的

EMBEDDING スパンは、ベクトル生成操作 (セマンティック検索のためのテキストからベクトルへの変換) を表します。

## 必須の属性

|属性 |タイプ |説明 |必須 |
|----------|------|---------------|----------|
| `openinference.span.kind` |文字列 | 「埋め込み」である必要があります |はい |
| `embedding.model_name` |文字列 |埋め込みモデル識別子 |おすすめ |

## 属性参照

### 単一の埋め込み

|属性 |タイプ |説明 |
|----------|------|---------------|
| `embedding.model_name` |文字列 |埋め込みモデル識別子 |
| `embedding.text` |文字列 |埋め込むテキストを入力 |
| `embedding.vector` |文字列 (JSON 配列) |生成された埋め込みベクトル |

**例：**```json
{
  "embedding.model_name": "text-embedding-ada-002",
  "embedding.text": "機械学習とは何ですか?",
  "embedding.vector": "[0.023, -0.012, 0.045, ..., 0.001]"
}
「」### バッチ埋め込み

|属性パターン |タイプ |説明 |
|---------------------|------|---------------|
| `embedding.embeddings.{i}.embedding.text` |文字列 |インデックス i のテキスト |
| `embedding.embeddings.{i}.embedding.vector` |文字列 (JSON 配列) |インデックス i のベクトル |

**例：**```json
{
  "embedding.model_name": "text-embedding-ada-002",
  "embedding.embeddings.0.embedding.text": "最初のドキュメント",
  "embedding.embeddings.0.embedding.vector": "[0.1, 0.2, 0.3, ..., 0.5]",
  "embedding.embeddings.1.embedding.text": "2 番目のドキュメント",
  "embedding.embeddings.1.embedding.vector": "[0.6, 0.7, 0.8, ..., 0.9]"
}
「」### ベクトル形式

JSON 配列文字列として保存されたベクトル:
- 寸法: 通常、384、768、1536、または 3072
- 形式: `"[0.123, -0.456, 0.789, ...]"`
- 精度: 通常、小数点以下 3 ～ 6 桁

**ストレージに関する考慮事項:**
- ベクトルが大きいと、トレース サイズが大幅に増加する可能性があります
- 運用環境ではベクターを省略することを検討してください (デバッグ用に `embedding.text` を保持します)
- 実際の類似性検索には別のベクトルデータベースを使用します

## 例

### 単一の埋め込み```json
{
  "openinference.span.kind": "埋め込み",
  "embedding.model_name": "text-embedding-ada-002",
  "embedding.text": "機械学習とは何ですか?",
  "embedding.vector": "[0.023, -0.012, 0.045, ..., 0.001]",
  "input.value": "機械学習とは何ですか?",
  "output.value": "[0.023, -0.012, 0.045, ..., 0.001]"
}
「」### バッチ埋め込み```json
{
  "openinference.span.kind": "埋め込み",
  "embedding.model_name": "text-embedding-ada-002",
  "embedding.embeddings.0.embedding.text": "最初のドキュメント",
  "embedding.embeddings.0.embedding.vector": "[0.1, 0.2, 0.3]",
  "embedding.embeddings.1.embedding.text": "2 番目のドキュメント",
  "embedding.embeddings.1.embedding.vector": "[0.4, 0.5, 0.6]",
  "embedding.embeddings.2.embedding.text": "3 番目のドキュメント",
  "embedding.embeddings.2.embedding.vector": "[0.7, 0.8, 0.9]"
}
「」
