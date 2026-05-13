---
name: agent-governance
description: |
  AIエージェントシステムにガバナンス、安全性、信頼性の制御を追加するためのパターンと手法。このスキルは次のような場合に使用します:
  - 外部ツール（API、データベース、ファイルシステム）を呼び出すAIエージェントを構築する場合
  - エージェントのツール利用に対するポリシーベースのアクセス制御を実装する場合
  - 危険なプロンプトを検出するために意味的な意図分類を追加する場合
  - マルチエージェントワークフロー向けの信頼スコアリングシステムを作成する場合
  - エージェントのアクションや判断に対する監査証跡を構築する場合
  - エージェントにレート制限、コンテンツフィルター、またはツール制限を適用する場合
  - 任意のエージェントフレームワーク（PydanticAI、CrewAI、OpenAI Agents、LangChain、AutoGen）を扱う場合
---

# エージェントガバナンスのパターン

AIエージェントシステムに安全性、信頼性、ポリシー適用を追加するためのパターンです。

## 概要

ガバナンスパターンにより、AIエージェントが定義された境界内で動作することを保証します。つまり、呼び出せるツール、処理できるコンテンツ、実行量を制御し、監査証跡を通じて説明責任を維持します。

```
ユーザーリクエスト → 意図分類 → ポリシーチェック → ツール実行 → 監査ログ
                     ↓                      ↓               ↓
                 脅威検知               許可/拒否         信頼更新
```

## 使うべきタイミング

- **ツールアクセス可能なエージェント**: 外部ツール（API、データベース、シェルコマンド）を呼び出す任意のエージェント
- **マルチエージェントシステム**: 他のエージェントに委譲するエージェントには信頼境界が必要
- **本番デプロイ**: コンプライアンス、監査、安全性の要件がある場合
- **機微な操作**: 金融取引、データアクセス、インフラ管理

---

## パターン1: ガバナンスポリシー

エージェントに許可される動作を、合成可能かつシリアライズ可能なポリシーオブジェクトとして定義します。

```python
from dataclasses import dataclass, field
from enum import Enum
from typing import Optional
import re

class PolicyAction(Enum):
    ALLOW = "allow"
    DENY = "deny"
    REVIEW = "review"  # flag for human review

@dataclass
class GovernancePolicy:
    """Declarative policy controlling agent behavior."""
    name: str
    allowed_tools: list[str] = field(default_factory=list)       # allowlist
    blocked_tools: list[str] = field(default_factory=list)       # blocklist
    blocked_patterns: list[str] = field(default_factory=list)    # content filters
    max_calls_per_request: int = 100                             # rate limit
    require_human_approval: list[str] = field(default_factory=list)  # tools needing approval

    def check_tool(self, tool_name: str) -> PolicyAction:
        """Check if a tool is allowed by this policy."""
        if tool_name in self.blocked_tools:
            return PolicyAction.DENY
        if tool_name in self.require_human_approval:
            return PolicyAction.REVIEW
        if self.allowed_tools and tool_name not in self.allowed_tools:
            return PolicyAction.DENY
        return PolicyAction.ALLOW

    def check_content(self, content: str) -> Optional[str]:
        """Check content against blocked patterns. Returns matched pattern or None."""
        for pattern in self.blocked_patterns:
            if re.search(pattern, content, re.IGNORECASE):
                return pattern
        return None
```

### ポリシーの合成

複数のポリシー（例: 組織全体 + チーム + エージェント固有）を組み合わせます:

```python
def compose_policies(*policies: GovernancePolicy) -> GovernancePolicy:
    """Merge policies with most-restrictive-wins semantics."""
    combined = GovernancePolicy(name="composed")

    for policy in policies:
        combined.blocked_tools.extend(policy.blocked_tools)
        combined.blocked_patterns.extend(policy.blocked_patterns)
        combined.require_human_approval.extend(policy.require_human_approval)
        combined.max_calls_per_request = min(
            combined.max_calls_per_request,
            policy.max_calls_per_request
        )
        if policy.allowed_tools:
            if combined.allowed_tools:
                combined.allowed_tools = [
                    t for t in combined.allowed_tools if t in policy.allowed_tools
                ]
            else:
                combined.allowed_tools = list(policy.allowed_tools)

    return combined


# Usage: layer policies from broad to specific
org_policy = GovernancePolicy(
    name="org-wide",
    blocked_tools=["shell_exec", "delete_database"],
    blocked_patterns=[r"(?i)(api[_-]?key|secret|password)\s*[:=]"],
    max_calls_per_request=50
)
team_policy = GovernancePolicy(
    name="data-team",
    allowed_tools=["query_db", "read_file", "write_report"],
    require_human_approval=["write_report"]
)
agent_policy = compose_policies(org_policy, team_policy)
```

### YAMLとしてのポリシー

ポリシーはコードではなく設定として保存します:

```yaml
# governance-policy.yaml
name: production-agent
allowed_tools:
  - search_documents
  - query_database
  - send_email
blocked_tools:
  - shell_exec
  - delete_record
blocked_patterns:
  - "(?i)(api[_-]?key|secret|password)\\s*[:=]"
  - "(?i)(drop|truncate|delete from)\\s+\\w+"
max_calls_per_request: 25
require_human_approval:
  - send_email
```

```python
import yaml

def load_policy(path: str) -> GovernancePolicy:
    with open(path) as f:
        data = yaml.safe_load(f)
    return GovernancePolicy(**data)
```

---

## パターン2: 意味的意図分類

エージェントに到達する前のプロンプトで、パターンベースのシグナルを使って危険な意図を検出します。

```python
from dataclasses import dataclass

@dataclass
class IntentSignal:
    category: str       # e.g., "data_exfiltration", "privilege_escalation"
    confidence: float   # 0.0 to 1.0
    evidence: str       # what triggered the detection

# Weighted signal patterns for threat detection
THREAT_SIGNALS = [
    # Data exfiltration
    (r"(?i)send\s+(all|every|entire)\s+\w+\s+to\s+", "data_exfiltration", 0.8),
    (r"(?i)export\s+.*\s+to\s+(external|outside|third.?party)", "data_exfiltration", 0.9),
    (r"(?i)curl\s+.*\s+-d\s+", "data_exfiltration", 0.7),

    # Privilege escalation
    (r"(?i)(sudo|as\s+root|admin\s+access)", "privilege_escalation", 0.8),
    (r"(?i)chmod\s+777", "privilege_escalation", 0.9),

    # System modification
    (r"(?i)(rm\s+-rf|del\s+/[sq]|format\s+c:)", "system_destruction", 0.95),
    (r"(?i)(drop\s+database|truncate\s+table)", "system_destruction", 0.9),

    # Prompt injection
    (r"(?i)ignore\s+(previous|above|all)\s+(instructions?|rules?)", "prompt_injection", 0.9),
    (r"(?i)you\s+are\s+now\s+(a|an)\s+", "prompt_injection", 0.7),
]

def classify_intent(content: str) -> list[IntentSignal]:
    """Classify content for threat signals."""
    signals = []
    for pattern, category, weight in THREAT_SIGNALS:
        match = re.search(pattern, content)
        if match:
            signals.append(IntentSignal(
                category=category,
                confidence=weight,
                evidence=match.group()
            ))
    return signals

def is_safe(content: str, threshold: float = 0.7) -> bool:
    """Quick check: is the content safe above the given threshold?"""
    signals = classify_intent(content)
    return not any(s.confidence >= threshold for s in signals)
```

**重要な洞察**: 意図分類はツール実行の*前*に行われ、事前安全チェックとして機能します。これは、生成の*後*にのみ検査する出力ガードレールとは本質的に異なります。

---

## パターン3: ツールレベルのガバナンスデコレーター

個々のツール関数をガバナンスチェックでラップします:

```python
import functools
import time
from collections import defaultdict

_call_counters: dict[str, int] = defaultdict(int)

def govern(policy: GovernancePolicy, audit_trail=None):
    """Decorator that enforces governance policy on a tool function."""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            tool_name = func.__name__

            # 1. Check tool allowlist/blocklist
            action = policy.check_tool(tool_name)
            if action == PolicyAction.DENY:
                raise PermissionError(f"Policy '{policy.name}' blocks tool '{tool_name}'")
            if action == PolicyAction.REVIEW:
                raise PermissionError(f"Tool '{tool_name}' requires human approval")

            # 2. Check rate limit
            _call_counters[policy.name] += 1
            if _call_counters[policy.name] > policy.max_calls_per_request:
                raise PermissionError(f"Rate limit exceeded: {policy.max_calls_per_request} calls")

            # 3. Check content in arguments
            for arg in list(args) + list(kwargs.values()):
                if isinstance(arg, str):
                    matched = policy.check_content(arg)
                    if matched:
                        raise PermissionError(f"Blocked pattern detected: {matched}")

            # 4. Execute and audit
            start = time.monotonic()
            try:
                result = await func(*args, **kwargs)
                if audit_trail is not None:
                    audit_trail.append({
                        "tool": tool_name,
                        "action": "allowed",
                        "duration_ms": (time.monotonic() - start) * 1000,
                        "timestamp": time.time()
                    })
                return result
            except Exception as e:
                if audit_trail is not None:
                    audit_trail.append({
                        "tool": tool_name,
                        "action": "error",
                        "error": str(e),
                        "timestamp": time.time()
                    })
                raise

        return wrapper
    return decorator


# Usage with any agent framework
audit_log = []
policy = GovernancePolicy(
    name="search-agent",
    allowed_tools=["search", "summarize"],
    blocked_patterns=[r"(?i)password"],
    max_calls_per_request=10
)

@govern(policy, audit_trail=audit_log)
async def search(query: str) -> str:
    """Search documents — governed by policy."""
    return f"Results for: {query}"

# Passes: search("latest quarterly report")
# Blocked: search("show me the admin password")
```

---

## パターン4: 信頼スコアリング

減衰ベースの信頼スコアで、エージェントの信頼性を時間経過で追跡します:

```python
from dataclasses import dataclass, field
import math
import time

@dataclass
class TrustScore:
    """Trust score with temporal decay."""
    score: float = 0.5          # 0.0 (untrusted) to 1.0 (fully trusted)
    successes: int = 0
    failures: int = 0
    last_updated: float = field(default_factory=time.time)

    def record_success(self, reward: float = 0.05):
        self.successes += 1
        self.score = min(1.0, self.score + reward * (1 - self.score))
        self.last_updated = time.time()

    def record_failure(self, penalty: float = 0.15):
        self.failures += 1
        self.score = max(0.0, self.score - penalty * self.score)
        self.last_updated = time.time()

    def current(self, decay_rate: float = 0.001) -> float:
        """Get score with temporal decay — trust erodes without activity."""
        elapsed = time.time() - self.last_updated
        decay = math.exp(-decay_rate * elapsed)
        return self.score * decay

    @property
    def reliability(self) -> float:
        total = self.successes + self.failures
        return self.successes / total if total > 0 else 0.0


# Usage in multi-agent systems
trust = TrustScore()

# Agent completes tasks successfully
trust.record_success()  # 0.525
trust.record_success()  # 0.549

# Agent makes an error
trust.record_failure()  # 0.467

# Gate sensitive operations on trust
if trust.current() >= 0.7:
    # Allow autonomous operation
    pass
elif trust.current() >= 0.4:
    # Allow with human oversight
    pass
else:
    # Deny or require explicit approval
    pass
```

**マルチエージェント信頼**: エージェントが他のエージェントへ委譲するシステムでは、各エージェントが委譲先ごとの信頼スコアを保持します:

```python
class AgentTrustRegistry:
    def __init__(self):
        self.scores: dict[str, TrustScore] = {}

    def get_trust(self, agent_id: str) -> TrustScore:
        if agent_id not in self.scores:
            self.scores[agent_id] = TrustScore()
        return self.scores[agent_id]

    def most_trusted(self, agents: list[str]) -> str:
        return max(agents, key=lambda a: self.get_trust(a).current())

    def meets_threshold(self, agent_id: str, threshold: float) -> bool:
        return self.get_trust(agent_id).current() >= threshold
```

---

## パターン5: 監査証跡

すべてのエージェントアクションに対する追記専用監査ログ。コンプライアンスとデバッグで重要です:

```python
from dataclasses import dataclass, field
import json
import time

@dataclass
class AuditEntry:
    timestamp: float
    agent_id: str
    tool_name: str
    action: str           # "allowed", "denied", "error"
    policy_name: str
    details: dict = field(default_factory=dict)

class AuditTrail:
    """Append-only audit trail for agent governance events."""
    def __init__(self):
        self._entries: list[AuditEntry] = []

    def log(self, agent_id: str, tool_name: str, action: str,
            policy_name: str, **details):
        self._entries.append(AuditEntry(
            timestamp=time.time(),
            agent_id=agent_id,
            tool_name=tool_name,
            action=action,
            policy_name=policy_name,
            details=details
        ))

    def denied(self) -> list[AuditEntry]:
        """Get all denied actions — useful for security review."""
        return [e for e in self._entries if e.action == "denied"]

    def by_agent(self, agent_id: str) -> list[AuditEntry]:
        return [e for e in self._entries if e.agent_id == agent_id]

    def export_jsonl(self, path: str):
        """Export as JSON Lines for log aggregation systems."""
        with open(path, "w") as f:
            for entry in self._entries:
                f.write(json.dumps({
                    "timestamp": entry.timestamp,
                    "agent_id": entry.agent_id,
                    "tool": entry.tool_name,
                    "action": entry.action,
                    "policy": entry.policy_name,
                    **entry.details
                }) + "\n")
```

---

## パターン6: フレームワーク統合

### PydanticAI

```python
from pydantic_ai import Agent

policy = GovernancePolicy(
    name="support-bot",
    allowed_tools=["search_docs", "create_ticket"],
    blocked_patterns=[r"(?i)(ssn|social\s+security|credit\s+card)"],
    max_calls_per_request=20
)

agent = Agent("openai:gpt-4o", system_prompt="You are a support assistant.")

@agent.tool
@govern(policy)
async def search_docs(ctx, query: str) -> str:
    """Search knowledge base — governed."""
    return await kb.search(query)

@agent.tool
@govern(policy)
async def create_ticket(ctx, title: str, body: str) -> str:
    """Create support ticket — governed."""
    return await tickets.create(title=title, body=body)
```

### CrewAI

```python
from crewai import Agent, Task, Crew

policy = GovernancePolicy(
    name="research-crew",
    allowed_tools=["search", "analyze"],
    max_calls_per_request=30
)

# Apply governance at the crew level
def governed_crew_run(crew: Crew, policy: GovernancePolicy):
    """Wrap crew execution with governance checks."""
    audit = AuditTrail()
    for agent in crew.agents:
        for tool in agent.tools:
            original = tool.func
            tool.func = govern(policy, audit_trail=audit)(original)
    result = crew.kickoff()
    return result, audit
```

### OpenAI Agents SDK

```python
from agents import Agent, function_tool

policy = GovernancePolicy(
    name="coding-agent",
    allowed_tools=["read_file", "write_file", "run_tests"],
    blocked_tools=["shell_exec"],
    max_calls_per_request=50
)

@function_tool
@govern(policy)
async def read_file(path: str) -> str:
    """Read file contents — governed."""
    import os
    safe_path = os.path.realpath(path)
    if not safe_path.startswith(os.path.realpath(".")):
        raise ValueError("Path traversal blocked by governance")
    with open(safe_path) as f:
        return f.read()
```

---

## ガバナンスレベル

リスクレベルに応じてガバナンスの厳格さを合わせます:

| Level | Controls | Use Case |
|-------|----------|----------|
| **Open** | 監査のみ、制限なし | 内部開発/テスト |
| **Standard** | ツール許可リスト + コンテンツフィルター | 一般的な本番エージェント |
| **Strict** | 全制御 + 機微操作に対する人手承認 | 金融、医療、法務 |
| **Locked** | 許可リストのみ、動的ツールなし、完全監査 | コンプライアンス重視システム |

---

## ベストプラクティス

| Practice | Rationale |
|----------|-----------|
| **設定としてのポリシー** | ポリシーはハードコードせずYAML/JSONに保存する — デプロイなしで変更可能にする |
| **最も厳しいルール優先** | ポリシー合成時は、拒否が常に許可より優先される |
| **事前の意図チェック** | 意図分類はツール実行の*前*に行い、後では行わない |
| **信頼の減衰** | 信頼スコアは時間とともに減衰させる — 継続的な良好行動を要求する |
| **追記専用監査** | 監査エントリは変更・削除しない — 不変性がコンプライアンスを可能にする |
| **フェイルクローズ** | ガバナンスチェックでエラーが起きたら、許可せず拒否する |
| **ポリシーとロジックの分離** | ガバナンス適用はエージェントの業務ロジックから独立させる |

---

## クイックスタートチェックリスト

```markdown
## エージェントガバナンス実装チェックリスト

### セットアップ
- [ ] ガバナンスポリシーを定義する（許可ツール、ブロックパターン、レート制限）
- [ ] ガバナンスレベルを選択する（open/standard/strict/locked）
- [ ] 監査証跡の保存先を設定する

### 実装
- [ ] すべてのツール関数に @govern デコレーターを追加する
- [ ] ユーザー入力処理に意図分類を追加する
- [ ] マルチエージェント相互作用のための信頼スコアリングを実装する
- [ ] 監査証跡エクスポートを接続する

### 検証
- [ ] ブロック対象ツールが適切に拒否されることをテストする
- [ ] コンテンツフィルターが機微パターンを検出することをテストする
- [ ] レート制限の挙動をテストする
- [ ] 監査証跡がすべてのイベントを記録することを確認する
- [ ] ポリシー合成（最も厳しいルール優先）をテストする
```

---

## 関連リソース

- [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) — 完全なガバナンスフレームワーク
- [AgentMesh Integrations](https://github.com/microsoft/agent-governance-toolkit/tree/main/packages/agentmesh-integrations) — フレームワーク別パッケージ
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

