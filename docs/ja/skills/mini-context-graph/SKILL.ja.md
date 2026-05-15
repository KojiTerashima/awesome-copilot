---
name: mini-context-graph
description: |
  A persistent, compounding knowledge base combining Karpathy's LLM Wiki pattern
  with a structured knowledge graph. Ingest documents once — the LLM writes wiki
  pages, extracts entities/relations into the graph, and stores raw content for
  evidence retrieval. Knowledge accumulates and cross-references; it is never
  re-derived from scratch.
---

# ミニコンテキストグラフスキル

## 核となるアイデア

標準 RAG は、クエリごとに知識をゼロから再発見します。このスキルは異なります:

1. **Wiki レイヤー** — LLM は、永続的なマークダウン ページ (概要、エンティティ ページ、トピック合成) を作成し、維持します。相互参照はすでに存在します。 Wiki は取り込むたびに内容が充実していきます。
2. **グラフ レイヤー** — エンティティと関係は一度抽出され、ナビゲート可能なナレッジ グラフとして保存されます。 BFS トラバーサルは、ソースを再度読み取ることなく構造クエリに答えます。
3. **生のソース レイヤー** — 元のドキュメントはチャンクとともに不変に保存されます。来歴リンクは、すべてのグラフ ノードとエッジを、それをサポートする正確なテキストに結び付けます。

> LLM はこう書いています。 Python ツールはすべての簿記を処理します。

---

## 3層

|レイヤー |どこ | LLM の機能 | Python の機能 |
|------|------|---------------------|------|
| **原材料** | `data/documents.json` |読み取り (決して変更しない) |チャンク + メタデータを保存 |
| **ウィキ** | `wiki/` (マークダウン) |ページの書き込み/更新 | Index.md + log.md を管理 |
| **グラフ** | `data/graph.json` |エンティティ + リレーションを抽出します |永続化、重複排除、トラバース |

---

## ⚡ エージェント向けのクイックスタート
```python
from scripts.contextgraph import ContextGraphSkill
from scripts.tools import wiki_store

skill = ContextGraphSkill()

# ===== INGEST WITH FULL RAG + WIKI =====
# 1. Read references/ingestion.md and references/ontology.md first
# 2. Extract entities and relations (LLM reasoning step)
entities = [
    {"name": "memory leak",   "type": "issue",  "supporting_text": "memory leaks cause crashes"},
    {"name": "system crash",  "type": "issue",  "supporting_text": "system crashes due to memory leaks"},
]
relations = [
    {"source": "memory leak", "target": "system crash", "type": "causes",
     "confidence": 1.0, "supporting_text": "System crashes due to memory leaks."},
]

result = skill.ingest_with_content(
    doc_id="doc_001",
    title="System Crash Analysis",
    source="/docs/incident_report.pdf",
    raw_content="System crashes due to memory leaks. Memory leaks occur when objects are not released.",
    entities=entities,
    relations=relations,
)
# result = {"doc_id": "doc_001", "chunk_count": 1, "nodes_added": 2, "edges_added": 1}

# 3. Write a wiki summary page for this document
wiki_store.write_page(
    category="summary",
    title="System Crash Analysis Summary",
    content="""---
title: System Crash Analysis
source_document: doc_001
tags: [summary, incident]
---

# System Crash Analysis

**Source:** incident_report.pdf

## Key Claims

- [[memory-leak]] causes [[system-crash]] (confidence: 1.0)

## Entities

- [[memory-leak]] (issue)
- [[system-crash]] (issue)
""",
    summary="Incident report: memory leaks cause system crashes.",
)

# ===== QUERY WITH EVIDENCE =====
result = skill.query_with_evidence("Why does the system crash?")
# Returns: {"query": ..., "subgraph": ..., "supporting_documents": [...], "evidence_chain": ...}

# ===== WIKI SEARCH (read wiki before answering) =====
pages = wiki_store.search_wiki("memory leak")
# Returns: [{slug, category, path, snippet}, ...]
```

---

## 運営

### 摂取する

ユーザーが新しいドキュメントを提供すると、次のようになります。

1. `references/ingestion.md` — エンティティ/リレーション抽出ルールを読み取ります。
2. `references/ontology.md` を読み取ります — タイプの正規化ルール。
3. LLM 推論を使用してエンティティと関係を抽出します。
4. `skill.ingest_with_content(...)` を呼び出す — 生のコンテンツ + チャンク + グラフ ノード + 来歴を保存します。
5. **`wiki_store.write_page(category="summary", ...)` を使用して Wiki 概要ページを作成します**。
6. **エンティティ ページの更新** — 新規/更新されたエンティティごとに、`wiki_store.write_page(category="entity", ...)` を書き込むか更新します。
7. ドキュメントが既存の合成トピックに触れている場合は、**トピック ページを更新**します。
8. 通常、1 つのドキュメントの取り込みで 3 ～ 10 の Wiki ページが操作されます。

### クエリ

ユーザーが質問すると:

1. **最初に Wiki を確認してください** — `wiki_store.search_wiki(query)` で関連ページを見つけてください。読んでみてください。
2. Wiki に適切な答えがある場合は、Wiki ページから合成します (高速パス)。
3. より深いグラフ走査が必要な場合は、`skill.query_with_evidence(query)` を呼び出します。
4. `supporting_documents` からの証拠を引用して回答を返します。
5. 答えが価値がある場合は、新しい Wiki トピック ページとしてファイルに戻してください。

### 糸くず

定期的に Wiki の健全性をチェックします。
```python
from scripts.tools import wiki_store
issues = wiki_store.lint_wiki()
# Returns: {orphan_pages, missing_pages, broken_wikilinks, isolated_pages}
```

LLM に、リンク切れ、孤立したページ、古い主張、相互参照の欠落などを確認して修正するよう依頼してください。完全な lint ワークフローについては、`references/lint.md` を参照してください。

---

## 摂取の制約

- ❌ 本文に存在しない実体を幻覚に見せないでください。
- ❌ 明示的なテキスト証拠がない限り関係を追加しないでください
- ❌ 0.6 未満の信頼度でエッジを追加しないでください。
- ✅ すべてのエンティティとリレーションに `supporting_text` を指定します。これにより出所を確認できます。
- ✅ 取り込んだドキュメントごとに Wiki の概要ページを作成します
- ✅ 新しい情報が到着したら既存のエンティティページを更新します
- ✅ 新しいデータが古い主張と矛盾する場合、Wiki ページ内の矛盾にフラグを立てます

---

## 取得の制約

- 🔒 トラバース深度は 2 を超えてはなりません (構成: MAX_GRAPH_DEPTH)
- 🔒 信頼度 ≥ 0.6 のエッジのみ (構成: MIN_CONFIDENCE)
- 🔒 最大 50 ノードが返されます (構成: MAX_NODES)
- ❌ グラフにないノードやエッジを作成しないでください。

---

## 完全な Python API リファレンス

|方法 |目的 |いつ使用するか |
|----------|----------|---------------|
| `skill.ingest_with_content(doc_id, title, source, raw_content, entities, relations)` | RAG の完全な取り込み: 生のドキュメント + グラフ + 来歴 |すべての新しいドキュメント |
| `skill.add_node(name, node_type)` |単一エンティティを追加 (出所なし) |ソースドキュメントなしで簡単に追加 |
| `skill.add_edge(source_name, target_name, relation, confidence)` |単一のリレーションを追加 |ソースドキュメントなしで簡単に追加 |
| `skill.query(query)` |グラフのみの検索 → 部分グラフ |構造クエリ |
| `skill.query_with_evidence(query)` |グラフ + 来歴 → サブグラフ + ソースチャンク |引用が必要なクエリ |
| `wiki_store.write_page(category, title, content, summary)` | Wiki ページを作成/更新する |毎回の摂取後。質問に答えた後 |
| `wiki_store.read_page(category, title)` | Wiki ページを読む |答える前に;相互参照用 |
| `wiki_store.search_wiki(query)` | Wiki 全体のキーワード検索 |グラフ走査前の高速パス |
| `wiki_store.list_pages(category)` |すべての Wiki ページをリストする |概要を知る |
| `wiki_store.get_log(last_n)` |最近の操作を読む | Wiki の歴史を理解する |
| `wiki_store.lint_wiki()` |健康診断 |定期メンテナンス |
| `documents_store.list_documents()` |取り込まれたすべての生のソースをリストする |監査/出所チェック |
| `documents_store.search_chunks(query)` |チャンクレベルの検索 |具体的な証拠を見つける |

---

## 設計理念

> 「Wiki は永続的で複合的な成果物です。相互参照はすでにそこにあります。統合には、あなたが読んだすべてがすでに反映されています。」 — カルパシー

|レイヤー |何が起こるのか |誰が所有するのか |
|------|-----------|---------------|
| **LLM 推論** |ウィキページの抽出、合成、作成 |エージェント (.md ガイダンス ファイル) |
| **Wiki の永続性** |インデックス、ログ、ファイル I/O | `wiki_store.py` |
| **グラフの永続性** |重複排除、インデックス、BFS トラバース | `graph_store.py`、`retrieval_engine.py` |
| **ローソースストレージ** |不変のドキュメント + チャンク + 来歴 | `documents_store.py` |

人間は情報源を厳選し、質問します。 LLM は wiki を作成し、グラフを抽出し、引用とともに回答します。 Python はすべての簿記を処理します。
