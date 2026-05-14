---
name: mcp-security-audit
description: |
  MCP（Model Context Protocol）サーバーの設定をセキュリティ問題について監査します。以下の場合にこのスキルを使用してください：
  - .mcp.jsonファイルのセキュリティリスクをレビューする際
  - MCPサーバーの引数にハードコードされた秘密情報やシェルインジェクションのパターンがないか確認する際
  - MCPサーバーがピン留めされたバージョン（@latestではない）を使用しているか検証する際
  - MCPサーバー設定にアンピン留めの依存関係がないか検出する際
  - プロジェクトが登録しているMCPサーバーが承認リストにあるか監査する際
  - MCP設定で環境変数の使用とハードコードされた認証情報をチェックする際
  - 「私のMCP設定は安全ですか？」「MCPサーバーを監査してください」「.mcp.jsonをチェックしてください」などのリクエスト時
  keywords: [mcp, security, audit, secrets, shell-injection, supply-chain, governance]
---

# MCPセキュリティ監査

MCPサーバー設定のセキュリティ問題（秘密情報の漏洩、シェルインジェクション、アンピン留め依存関係、未承認サーバー）を監査します。

## 概要

MCPサーバーはエージェントに外部システムへの直接ツールアクセスを提供します。誤設定された`.mcp.json`は認証情報の漏洩、シェルインジェクション、信頼できないサーバーへの接続を招く可能性があります。このスキルはそれらの問題を本番環境に到達する前に検出します。

```
.mcp.json → サーバー解析 → 各サーバーをチェック：
  1. 引数や環境変数に秘密情報はあるか？
  2. シェルインジェクションのパターンはあるか？
  3. ピン留めされていないバージョン（@latest）はあるか？
  4. 危険なコマンド（eval、bash -c）はあるか？
  5. サーバーは承認リストにあるか？
→ レポート生成
```

## 使用タイミング

- プロジェクト内の任意の`.mcp.json`ファイルをレビューする時
- 新しいMCPサーバーをプロジェクトに導入する時
- モノレポやプラグインマーケットプレイス内の全MCPサーバーを監査する時
- MCP設定変更のコミット前チェック時
- エージェントツール設定のセキュリティレビュー時

---

## 監査チェック1：ハードコードされた秘密情報

MCPサーバーの引数や環境変数にハードコードされた認証情報がないかスキャンします。

```python
import json
import re
from pathlib import Path

SECRET_PATTERNS = [
    (r'(?i)(api[_-]?key|token|secret|password|credential)\s*[:=]\s*["\'][^"\']{8,}', "ハードコードされた秘密情報"),
    (r'(?i)Bearer\s+[A-Za-z0-9\-._~+/]+=*', "ハードコードされたベアラートークン"),
    (r'(?i)(ghp_|gho_|ghu_|ghs_|ghr_)[A-Za-z0-9]{30,}', "GitHubトークン"),
    (r'sk-[A-Za-z0-9]{20,}', "OpenAI APIキー"),
    (r'AKIA[0-9A-Z]{16}', "AWSアクセスキー"),
    (r'-----BEGIN\s+(RSA\s+)?PRIVATE\s+KEY-----', "プライベートキー"),
]

def check_secrets(mcp_config: dict) -> list[dict]:
    """MCPサーバー設定内のハードコードされた秘密情報をチェックします。"""
    findings = []
    raw = json.dumps(mcp_config)
    for pattern, description in SECRET_PATTERNS:
        matches = re.findall(pattern, raw)
        if matches:
            findings.append({
                "severity": "CRITICAL",
                "check": "hardcoded-secret",
                "message": f"{description}がMCP設定内で検出されました",
                "evidence": f"パターン一致: {pattern}",
                "fix": "環境変数参照を使用してください: ${ENV_VAR_NAME}"
            })
    return findings
```

**良い例 — 環境変数参照を使用：**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["server.js"],
      "env": {
        "API_KEY": "${MY_API_KEY}",
        "DB_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

**悪い例 — ハードコードされた認証情報：**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["server.js", "--api-key", "sk-abc123realkey456"],
      "env": {
        "DB_URL": "postgresql://admin:password123@prod-db:5432/main"
      }
    }
  }
}
```

---

## 監査チェック2：シェルインジェクションパターン

MCPサーバーの引数に危険なコマンドパターンがないか検出します。

```python
import json
import re

DANGEROUS_PATTERNS = [
    (r'\$\(', "コマンド置換 $(...)"),
    (r'`[^`]+`', "バックティックコマンド置換"),
    (r';\s*\w', "セミコロンによるコマンド連結"),
    (r'\|\s*\w', "パイプによる別コマンドへの連結"),
    (r'&&\s*\w', "&&によるコマンド連結"),
    (r'\|\|\s*\w', "||によるコマンド連結"),
    (r'(?i)eval\s', "evalの使用"),
    (r'(?i)bash\s+-c\s', "bash -c実行"),
    (r'(?i)sh\s+-c\s', "sh -c実行"),
    (r'>\s*/dev/tcp/', "TCPリダイレクト（リバースシェルパターン）"),
    (r'curl\s+.*\|\s*(ba)?sh', "curlからシェルへのパイプ"),
]

def check_shell_injection(server_config: dict) -> list[dict]:
    """MCPサーバーの引数にシェルインジェクションリスクがないかチェックします。"""
    findings = []
    args_text = json.dumps(server_config.get("args", []))
    for pattern, description in DANGEROUS_PATTERNS:
        if re.search(pattern, args_text):
            findings.append({
                "severity": "HIGH",
                "check": "shell-injection",
                "message": f"MCPサーバー引数に危険なパターン検出: {description}",
                "fix": "シェル補間ではなく直接コマンド実行を使用してください"
            })
    return findings
```

---

## 監査チェック3：アンピン留め依存関係

MCPサーバーのパッケージ参照に`@latest`が使われていないかを検出します。

```python
def check_pinned_versions(server_config: dict) -> list[dict]:
    """MCPサーバーの依存関係が@latestではなくピン留めされたバージョンを使っているかチェックします。"""
    findings = []
    args = server_config.get("args", [])
    for arg in args:
        if isinstance(arg, str):
            if "@latest" in arg:
                findings.append({
                    "severity": "MEDIUM",
                    "check": "unpinned-dependency",
                    "message": f"アンピン留め依存関係: {arg}",
                    "fix": f"特定バージョンにピン留めしてください: {arg.replace('@latest', '@1.2.3')}"
                })
            # npxでバージョン指定なしパッケージ
            if arg.startswith("-y") or (not "@" in arg and not arg.startswith("-")):
                pass  # npxフラグまたは単純な引数なので問題なし
    # -yなしのnpx使用（CIで対話プロンプトになる可能性）
    command = server_config.get("command", "")
    if command == "npx" and "-y" not in args:
        findings.append({
            "severity": "LOW",
            "check": "npx-interactive",
            "message": "npxに-yフラグがないためCIで対話プロンプトが発生する可能性があります",
            "fix": "npx -y パッケージ名のように-yフラグを追加してください"
        })
    return findings
```

**良い例 — ピン留めされたバージョン：**
```json
{ "args": ["-y", "my-mcp-server@2.1.0"] }
```

**悪い例 — アンピン留め：**
```json
{ "args": ["-y", "my-mcp-server@latest"] }
```

---

## 監査チェック4：フル監査ランナー

すべてのチェックを組み合わせて一括監査を実行します。

```python
def audit_mcp_config(mcp_path: str) -> dict:
    """.mcp.jsonファイルに対してフルセキュリティ監査を実行します。"""
    path = Path(mcp_path)
    if not path.exists():
        return {"error": f"{mcp_path} が見つかりません"}

    config = json.loads(path.read_text(encoding="utf-8"))
    servers = config.get("mcpServers", {})
    results = {"file": str(path), "servers": {}, "summary": {}}
    total_findings = []

    # 秘密情報チェックは設定全体に対して一度だけ実行（サーバーごとではない）
    config_level_findings = check_secrets(config)
    total_findings.extend(config_level_findings)

    for name, server_config in servers.items():
        if not isinstance(server_config, dict):
            continue
        findings = []
        findings.extend(check_shell_injection(server_config))
        findings.extend(check_pinned_versions(server_config))
        results["servers"][name] = {
            "command": server_config.get("command", ""),
            "findings": findings,
        }
        total_findings.extend(findings)

    # サマリー集計
    by_severity = {}
    for f in total_findings:
        sev = f["severity"]
        by_severity[sev] = by_severity.get(sev, 0) + 1

    results["summary"] = {
        "total_servers": len(servers),
        "total_findings": len(total_findings),
        "by_severity": by_severity,
        "passed": len(total_findings) == 0,
    }
    return results
```

**使用例：**
```python
results = audit_mcp_config(".mcp.json")
if not results["summary"]["passed"]:
    for server, data in results["servers"].items():
        for finding in data["findings"]:
            print(f"[{finding['severity']}] {server}: {finding['message']}")
            print(f"  修正案: {finding['fix']}")
```

---

## 出力フォーマット

```
MCPセキュリティ監査 — .mcp.json
═══════════════════════════════
スキャンしたサーバー数: 5
検出件数: 3 (CRITICAL 1件, HIGH 1件, MEDIUM 1件)

[CRITICAL] my-api-server: MCP設定内にハードコードされた秘密情報が検出されました
  修正案: 環境変数参照を使用してください: ${ENV_VAR_NAME}

[HIGH] data-processor: MCPサーバー引数に危険なパターン検出: bash -c実行
  修正案: シェル補間ではなく直接コマンド実行を使用してください

[MEDIUM] analytics: アンピン留め依存関係: analytics-mcp@latest
  修正案: 特定バージョンにピン留めしてください: analytics-mcp@2.1.0
```

---

## 関連リソース

- [MCP仕様](https://modelcontextprotocol.io/)
- [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) — MCPトラストプロキシを含む完全なガバナンスフレームワーク
- [OWASP ASI-02: Insecure Tool Use](https://owasp.org/www-project-agentic-ai-threats/)
