---
name: agent-owasp-compliance
description: |
  あらゆるAIエージェントのコードベースを、OWASP Agentic Security Initiative (ASI) Top 10リスクに照らしてチェックします。
  このスキルを使うタイミング:
  - 本番デプロイ前にエージェントシステムのセキュリティ態勢を評価するとき
  - OWASP ASI 2026標準への準拠チェックを実施するとき
  - 既存のセキュリティコントロールを10のエージェントリスクにマッピングするとき
  - セキュリティレビューや監査向けの準拠レポートを生成するとき
  - エージェントフレームワークのセキュリティ機能を標準と比較するとき
  - 「自分のエージェントはOWASP準拠？」「ASI準拠をチェックして」「agenticセキュリティ監査」といった依頼があったとき
---

# Agent OWASP ASI Compliance Check

AIエージェントシステムを、エージェントのセキュリティ態勢における業界標準である OWASP Agentic Security Initiative (ASI) Top 10 に対して評価します。

## 概要

OWASP ASI Top 10 は、自律型AIエージェントに特有の重要なセキュリティリスクを定義しています。対象はLLM単体やチャットボットではなく、ツールを呼び出し、システムにアクセスし、ユーザーに代わって行動するエージェントです。このスキルは、あなたのエージェント実装が各リスクに対応できているかを確認します。

```
Codebase → 各 ASI コントロールをスキャン:
  ASI-01: Prompt Injection Protection
  ASI-02: Tool Use Governance
  ASI-03: Agency Boundaries
  ASI-04: Escalation Controls
  ASI-05: Trust Boundary Enforcement
  ASI-06: Logging & Audit
  ASI-07: Identity Management
  ASI-08: Policy Integrity
  ASI-09: Supply Chain Verification
  ASI-10: Behavioral Monitoring
→ 準拠レポートを生成 (X/10 covered)
```

## 10のリスク

| Risk | Name | 何を確認するか |
|------|------|-----------------|
| ASI-01 | Prompt Injection | ツール呼び出し前の入力検証（LLM出力フィルタだけでは不十分） |
| ASI-02 | Insecure Tool Use | ツールの許可リスト、引数検証、生のシェル実行がないこと |
| ASI-03 | Excessive Agency | 権限境界、スコープ制限、最小権限の原則 |
| ASI-04 | Unauthorized Escalation | 機微操作前の権限チェック、自己昇格がないこと |
| ASI-05 | Trust Boundary Violation | エージェント間の信頼検証、署名付き認証情報、盲目的信頼がないこと |
| ASI-06 | Insufficient Logging | すべてのツール呼び出しに対する構造化監査証跡、改ざん検知可能なログ |
| ASI-07 | Insecure Identity | 文字列名だけでなく暗号学的なエージェントID |
| ASI-08 | Policy Bypass | 決定論的なポリシー強制、LLMベースの権限チェックがないこと |
| ASI-09 | Supply Chain Integrity | 署名済みプラグイン/ツール、完全性検証、依存関係監査 |
| ASI-10 | Behavioral Anomaly | ドリフト検知、サーキットブレーカー、キルスイッチ機能 |

---

## ASI-01 を確認: Prompt Injection Protection

LLM生成**後**ではなく、ツール実行**前**に動作する入力検証を確認します。

```python
import re
from pathlib import Path

def check_asi_01(project_path: str) -> dict:
    """ASI-01: Is user input validated before reaching tool execution?"""
    positive_patterns = [
        "input_validation", "validate_input", "sanitize",
        "classify_intent", "prompt_injection", "threat_detect",
        "PolicyEvaluator", "PolicyEngine", "check_content",
    ]
    negative_patterns = [
        r"eval\(", r"exec\(", r"subprocess\.run\(.*shell=True",
        r"os\.system\(",
    ]

    # Scan Python files for signals
    root = Path(project_path)
    positive_matches = []
    negative_matches = []

    for py_file in root.rglob("*.py"):
        content = py_file.read_text(errors="ignore")
        for pattern in positive_patterns:
            if pattern in content:
                positive_matches.append(f"{py_file.name}: {pattern}")
        for pattern in negative_patterns:
            if re.search(pattern, content):
                negative_matches.append(f"{py_file.name}: {pattern}")

    positive_found = len(positive_matches) > 0
    negative_found = len(negative_matches) > 0

    return {
        "risk": "ASI-01",
        "name": "Prompt Injection",
        "status": "pass" if positive_found and not negative_found else "fail",
        "controls_found": positive_matches,
        "vulnerabilities": negative_matches,
        "recommendation": "Add input validation before tool execution, not just output filtering"
    }
```

**合格状態の例:**
```python
# GOOD: Validate before tool execution
result = policy_engine.evaluate(user_input)
if result.action == "deny":
    return "Request blocked by policy"
tool_result = await execute_tool(validated_input)
```

**不合格状態の例:**
```python
# BAD: User input goes directly to tool
tool_result = await execute_tool(user_input)  # No validation
```

---

## ASI-02 を確認: Insecure Tool Use

ツールに許可リスト、引数検証、無制限実行の禁止があることを確認します。

**検索すべきポイント:**
- 明示的な許可リストによるツール登録（無制限でない）
- ツール実行前の引数検証
- ユーザー制御入力を伴う `subprocess.run(shell=True)` がない
- サンドボックスなしでエージェント生成コードに `eval()` や `exec()` を使っていない

**合格例:**
```python
ALLOWED_TOOLS = {"search", "read_file", "create_ticket"}

def execute_tool(name: str, args: dict):
    if name not in ALLOWED_TOOLS:
        raise PermissionError(f"Tool '{name}' not in allowlist")
    # validate args...
    return tools[name](**validated_args)
```

---

## ASI-03 を確認: Excessive Agency

エージェントの能力が無制限ではなく、境界づけられていることを確認します。

**検索すべきポイント:**
- 明示的な能力リストまたは実行リング
- エージェントがアクセスできる範囲のスコープ制限
- ツールアクセスに最小権限の原則が適用されている

**不合格:** エージェントがデフォルトで全ツールにアクセス可能。  
**合格:** エージェント能力が固定の許可リストで定義され、未知のツールは拒否される。

---

## ASI-04 を確認: Unauthorized Escalation

エージェントが自分自身の権限を昇格できないことを確認します。

**検索すべきポイント:**
- 機微操作前の権限レベルチェック
- 自己昇格パターンがない（エージェントが自身の trust score や role を変更）
- 昇格には外部証明が必要（人間またはSRE witness）

**不合格:** エージェントが自身の設定や権限を変更できる。  
**合格:** 権限変更には帯域外承認が必要（例: Ring 0 には SRE attestation が必要）。

---

## ASI-05 を確認: Trust Boundary Violation

マルチエージェントシステムでは、指示を受け入れる前にエージェント同士が互いのIDを検証することを確認します。

**検索すべきポイント:**
- エージェントID検証（DID、署名付きトークン、APIキー）
- 委任タスク受け入れ前の trust score チェック
- エージェント間メッセージの盲目的信頼がない
- 委任範囲の縮小（child scope <= parent scope）

**合格例:**
```python
def accept_task(sender_id: str, task: dict):
    trust = trust_registry.get_trust(sender_id)
    if not trust.meets_threshold(0.7):
        raise PermissionError(f"Agent {sender_id} trust too low: {trust.current()}")
    if not verify_signature(task, sender_id):
        raise SecurityError("Task signature verification failed")
    return process_task(task)
```

---

## ASI-06 を確認: Insufficient Logging

すべてのエージェント操作で、構造化され改ざん検知可能な監査エントリが生成されることを確認します。

**検索すべきポイント:**
- すべてのツール呼び出しに対する構造化ログ（print文だけでない）
- 監査エントリに含まれる項目: timestamp, agent ID, tool name, args, result, policy decision
- 追記専用またはハッシュチェーン型ログ形式
- ログがエージェント書き込み可能ディレクトリと分離して保存されている

**不合格:** エージェント操作が `print()` のみ、または未記録。  
**合格:** チェーンハッシュ付き構造化JSONL監査証跡を安全なストレージへエクスポートしている。

---

## ASI-07 を確認: Insecure Identity

エージェントが文字列名ではなく暗号学的IDを持つことを確認します。

**不合格の兆候:**
- `agent_name = "my-agent"` のような識別（文字列のみ）
- エージェント間認証がない
- エージェント間で認証情報を共有している

**合格の兆候:**
- DIDベースID（`did:web:`, `did:key:`）
- Ed25519 などの暗号署名
- ローテーション可能なエージェント単位の認証情報
- IDが特定の能力に結び付いている

---

## ASI-08 を確認: Policy Bypass

ポリシー強制がLLMベースではなく決定論的であることを確認します。

**検索すべきポイント:**
- ポリシー評価が決定論的ロジックを使用（YAML rules、コード述語）
- 強制経路にLLM呼び出しがない
- ポリシーチェックがエージェントによりスキップ/上書きできない
- フェイルクローズ動作（ポリシーチェックでエラー時は操作拒否）

**不合格:** エージェントがプロンプトで自身の権限を判断（"Am I allowed to...?"）。  
**合格:** PolicyEvaluator.evaluate() が <0.1ms で allow/deny を返し、LLMは関与しない。

---

## ASI-09 を確認: Supply Chain Integrity

エージェントのプラグインとツールで完全性検証があることを確認します。

**検索すべきポイント:**
- SHA-256ハッシュ付きの `INTEGRITY.json` またはマニフェストファイル
- プラグインインストール時の署名検証
- 依存関係の固定（`@latest` や上限なし `>=` を使わない）
- SBOM生成

---

## ASI-10 を確認: Behavioral Anomaly

システムがエージェントの行動ドリフトを検知・対応できることを確認します。

**検索すべきポイント:**
- 反復失敗時に作動するサーキットブレーカー
- 時間経過による trust score 減衰（temporal decay）
- キルスイッチまたは緊急停止機能
- ツール呼び出しパターン（頻度、対象、タイミング）の異常検知

**不合格:** 問題行動を起こすエージェントを自動停止する仕組みがない。  
**合格:** N回失敗後にサーキットブレーカーが作動し、非活動時にtrustが減衰し、キルスイッチが利用可能。

---

## 準拠レポート形式

```markdown
# OWASP ASI Compliance Report
Generated: 2026-04-01
Project: my-agent-system

## Summary: 7/10 Controls Covered

| Risk | Status | Finding |
|------|--------|---------|
| ASI-01 Prompt Injection | PASS | PolicyEngine validates input before tool calls |
| ASI-02 Insecure Tool Use | PASS | Tool allowlist enforced in governance.py |
| ASI-03 Excessive Agency | PASS | Execution rings limit capabilities |
| ASI-04 Unauthorized Escalation | PASS | Ring promotion requires attestation |
| ASI-05 Trust Boundary | FAIL | No identity verification between agents |
| ASI-06 Insufficient Logging | PASS | AuditChain with SHA-256 chain hashes |
| ASI-07 Insecure Identity | FAIL | Agents use string names, no crypto identity |
| ASI-08 Policy Bypass | PASS | Deterministic PolicyEvaluator, no LLM in path |
| ASI-09 Supply Chain | FAIL | No integrity manifests or plugin signing |
| ASI-10 Behavioral Anomaly | PASS | Circuit breakers and trust decay active |

## Critical Gaps
- ASI-05: Add agent identity verification using DIDs or signed tokens
- ASI-07: Replace string agent names with cryptographic identity
- ASI-09: Generate INTEGRITY.json manifests for all plugins

## Recommendation
Install agent-governance-toolkit for reference implementations of all 10 controls:
pip install agent-governance-toolkit
```

---

## クイック評価質問

エージェントシステムを迅速に評価するために以下を使ってください:

1. **ユーザー入力は、いずれかのツールに到達する前に検証を通過しますか？** (ASI-01)
2. **エージェントが呼び出せるツールの明示的な一覧がありますか？** (ASI-02)
3. **エージェントは何でもできますか、それとも能力に境界がありますか？** (ASI-03)
4. **エージェントは自分自身の権限を昇格できますか？** (ASI-04)
5. **エージェント同士はタスク受け入れ前に互いのIDを検証しますか？** (ASI-05)
6. **すべてのツール呼び出しは、再現可能な十分な詳細で記録されていますか？** (ASI-06)
7. **各エージェントは一意の暗号学的IDを持っていますか？** (ASI-07)
8. **ポリシー強制は決定論的ですか（LLMベースではない）？** (ASI-08)
9. **プラグイン/ツールは使用前に完全性検証されていますか？** (ASI-09)
10. **サーキットブレーカーまたはキルスイッチはありますか？** (ASI-10)

これらのいずれかに「no」と答えるなら、対処すべきギャップです。

---

## 関連リソース

- [OWASP Agentic AI Threats](https://owasp.org/www-project-agentic-ai-threats/)
- [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) — 10/10 の ASI コントロールをカバーするリファレンス実装
- [agent-governance skill](https://github.com/github/awesome-copilot/tree/main/skills/agent-governance) — エージェントシステム向けガバナンスパターン

