---
description: '安全、高速、明確なフックと再利用可能なフックの例をオーサリングするためのポータブルなガイダンス'
applyTo: '.github/hooks/**, hooks/**'
---

# フックオーサリングガイドライン

フックは、特定のライフサイクル イベントで実行される **小さな決定論的なコマンドまたはスクリプト**です。
素晴らしいフックは 1 つの明確な仕事を実行し、迅速に実行され、その副作用を明示します。

## フォルダー構造

GitHub Copilot フックはリポジトリ内の `.github/hooks/` にあります。
```text
.github/
└── hooks/
    ├── block-dangerous-commands.json   ← hook config (which event, which script, options)
    └── scripts/
        ├── block-dangerous-commands.sh  ← Bash implementation
        └── block-dangerous-commands.ps1 ← PowerShell implementation (optional if Bash-only)
```

複数の `.json` ファイルを作成できます。各ファイルは 1 つ以上のイベントのフックを登録します。ホストはそれらをすべてロードします。

## 設定ファイル

各 `.json` ファイルは、イベントをフック エントリの配列にマップします。

- **コマンド フック** (`type: "command"`): ローカル スクリプトを実行します。ホストは標準入力でイベント JSON を渡し、スクリプトは終了コードと標準出力を通じて応答します。

### 設定例
```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "matcher": "bash",
        "type": "command",
        "bash": "./.github/hooks/scripts/block-dangerous-commands.sh",
        "powershell": "./.github/hooks/scripts/block-dangerous-commands.ps1",
        "cwd": ".",
        "timeoutSec": 5,
        "env": {
          "BLOCK_MODE": "deny"
        }
      }
    ]
  }
}
```

### 設定フィールド

|フィールド |必須 |何をするのか |
| ---- | ---- | ---- |
| `type` |はい |スクリプトの場合は `"command"` |
| `matcher` |いいえ |ホストレベルのフィルター — ツール名がこの値と一致する場合にのみフックが起動します (例: `"bash"`、`"powershell"`、`"edit"`、`"create"`)。 Copilot CLI v1.0.36 で動作することをローカルで検証済み。リポジトリフックサンプルではまだ使用されていません。 |
| `bash` | 1 つまたは両方 | Unix / Bash 対応ホスト上で呼び出されるコマンド ライン |
| `powershell` | 1 つまたは両方 | Windows / PowerShell 対応ホストで呼び出されるコマンド ライン |
| `cwd` |いいえ |作業ディレクトリ (リポジトリ ルートからの相対) |
| `timeoutSec` |いいえ |ホストがプロセスを強制終了するまでの最大秒数 (デフォルトは 30) |
| `env` |いいえ |スクリプトに渡される追加のプロセス環境変数 |

### マッチャーが重要な理由

マッチャーがない場合、すべての `preToolUse` フックは **すべて** ツール呼び出しで起動されます。スクリプトは次のようなボイラープレートで始まります。
```bash
tool_name="$(printf '%s' "$payload" | jq -r '.toolName')"
[[ "$tool_name" != "bash" ]] && exit 0
```

マッチャーを使用すると、ホストがこのフィルタリングを実行します。ボイラープレートや無関係なツールのプロセス生成は必要ありません。機能が安定すると、これが標準パターンになる可能性があります。

フックが CLI とクラウド エージェント (または古い CLI バージョン) の両方で動作する必要がある場合は、マッチャーを使用する場合でもスクリプト内フィルタリングをフォールバックとして保持してください。

### `env` — スクリプトの静的設定

`env` は **標準のホスト フィールド**です。その中のキーは **作成者定義の変数** です。名前と値はユーザーが選択します。

これらは、stdin JSON ペイロード内ではなく、**プロセス環境変数**として到着します。ハードコーディングすべきでない静的構成にはこれらを使用します。

|パターン |例 |
| ---- | ---- |
|モードフラグ | `"BLOCK_MODE": "deny"` — 同じスクリプトが 1 つのリポジトリにログされ、別のリポジトリではブロックされます。
|しきい値 | `"MAX_CHANGED_FILES": "20"` |
|パス | `"AUDIT_LOG_PATH": ".github/logs/hooks.log"` |
|機能切り替え | `"ENABLE_NOTIFICATIONS": "false"` |

### `bash` および `powershell` — どちらかまたは両方を指定する場合

ホストは、現在の環境に一致するエントリを選択します。両方を実行したり、一方からもう一方にフォールバックしたりすることはありません。

|状況 |提供 |
| ---- | ---- |
|プライベートフック、既知のプラットフォームの 1 つ |そのプラットフォームのエントリのみ |
|クロスプラットフォームのサポートを主張するフックを公開 |両方のエントリ |
|単一のクロスプラットフォーム ランタイム (Python、Node、pwsh) |両方のエントリを通じて同じスクリプトを公開します。
| Bash のみの依存関係 | `bash` のみ |
| Windows のみの依存関係 | `powershell` のみ |

両方のエントリで Python を使用したクロスプラットフォームの例:
```json
{
  "type": "command",
  "bash": "python3 ./.github/hooks/scripts/check.py",
  "powershell": "python .\\.github\\hooks\\scripts\\check.py"
}
```

## 脚本契約

すべてのフック スクリプトは同じ基本規約に従います。標準入力から JSON を読み取り、作業を実行し、終了コード、標準出力、および標準エラー出力を通じて応答します。

**重要**: `toolArgs` は **JSON 文字列**であり、ネストされたオブジェクトではありません。そのフィールドにアクセスするには、もう一度解析する必要があります。

### 標準入力の読み取りと応答 — Bash と PowerShell

**バッシュ**:
```bash
#!/usr/bin/env bash
set -euo pipefail
payload="$(cat)"
tool_name="$(printf '%s' "$payload" | jq -r '.toolName')"
tool_args="$(printf '%s' "$payload" | jq -r '.toolArgs')"
command="$(printf '%s' "$tool_args" | jq -r '.command // ""')"
```

**PowerShell**:
```powershell
Set-StrictMode -Version Latest
$payload = [Console]::In.ReadToEnd() | ConvertFrom-Json
$toolArgs = $payload.toolArgs | ConvertFrom-Json
$command = $toolArgs.command
```

`preToolUse` (PowerShell) で拒否するには:
```powershell
@{ permissionDecision = 'deny'; permissionDecisionReason = 'Blocked by policy' } |
    ConvertTo-Json -Compress
exit 0
```

### スクリプトが受け取るもの

|入力 |何を運ぶのか |
| ---- | ---- |
| `stdin` |現在のイベントを説明する 1 つの JSON ペイロード |
|プロセス環境 |通常の環境変数に加えて、設定の `env` で定義したもの。
|作業ディレクトリ | `cwd` 構成から、またはホストのデフォルト |

### スクリプトの応答方法

|チャンネル |目的 |
| ---- | ---- |
| `0` を終了 |スクリプトは成功しました - 標準出力が構造化拒否を伝送しない限り、ホストは続行します。
|ゼロ以外の終了 | **トリガーアクションをブロック**し、フックの失敗を通知します |
| `stdout` |構造化された機械可読出力 - 標準出力スキーマ (`preToolUse` など) を文書化するイベントのみ。
| `stderr` |人間が判読できるログの診断 |

### 終了コードと拒否: 全体像

拒否メカニズムは **イベントによって異なります**:

|イベントの種類 |許可する方法 |拒否/ブロックする方法 |
| ---- | ---- | ---- |
| `preToolUse` | exit `0`、空、または標準出力の `{"permissionDecision":"allow"}` | **推奨**: 標準出力で `0` + `{"permissionDecision":"deny","permissionDecisionReason":"..."}` を終了 — ホストに表示する理由を与えます。 **これも機能します**: ゼロ以外の exit はツール呼び出しをブロックしますが、構造化された理由はありません。 |
| `userPromptSubmitted` | `0` を終了 |ゼロ以外の exit はプロンプトをブロックします (このイベントでは stdout は無視されます)。
| `agentStop` | `0` を終了 |ゼロ以外の exit はアクションをブロックします。
|その他のイベント (`sessionStart`、`sessionEnd`、`postToolUse`、`errorOccurred`) | `0` を終了 |ゼロ以外の終了信号は失敗を示します。ホストはそのイベントの後続のフックをスキップする可能性があります。

**経験則**: イベントに構造化された stdout スキーマ (`preToolUse` など) がある場合は、それを使用します。これは明確な理由を示し、公式に文書化された拒否パスです。構造化された標準出力のないイベントの場合、ゼロ以外の終了が実際的なブロック メカニズムです。これはリポジトリ サンプルとラーニング ハブのドキュメントで確認されていますが、公式の GitHub リファレンスでは、契約の保証として "非ゼロ = ブロック" が明示的に文書化されていません。

### 例 1: コミット ゲート — lint、型、およびテストが合格するまでコミットをブロックする

**このパターンが重要な理由**: 拒否理由には実際のエラーが含まれるため、エージェントは何が壊れているかを確認し、再試行する前に修正します。これにより、自己修正フィードバック ループが作成されます。これは、フックが実行できる最も強力な機能です。

Error 500 (Server Error)!!1500.That’s an error.There was an error. Please try again later.That’s all we know.

**構成** — `.github/hooks/commit-gate.json`:
```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "./.github/hooks/scripts/commit-gate.sh",
        "cwd": ".",
        "timeoutSec": 120
      }
    ]
  }
}
```

**スクリプト** — `.github/hooks/scripts/commit-gate.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

payload="$(cat)"
tool_name="$(printf '%s' "$payload" | jq -r '.toolName')"

# Only gate bash commands that are git commits
if [[ "$tool_name" != "bash" ]]; then exit 0; fi
command="$(printf '%s' "$payload" | jq -r '.toolArgs' | jq -r '.command // ""')"
if ! printf '%s' "$command" | grep -q "git commit"; then exit 0; fi

CWD="$(printf '%s' "$payload" | jq -r '.cwd')"
ERRORS=""

# 1. TypeScript type check
if [[ -f "$CWD/tsconfig.json" ]]; then
  TSC_OUT=$(cd "$CWD" && npx tsc --noEmit 2>&1) || ERRORS="${ERRORS}
=== TypeScript Errors ===
$(echo "$TSC_OUT" | head -30)"
fi

# 2. Lint
if [[ -f "$CWD/package.json" ]]; then
  HAS_LINT=$(jq -r '.scripts.lint // empty' "$CWD/package.json" 2>/dev/null)
  if [[ -n "$HAS_LINT" ]]; then
    LINT_OUT=$(cd "$CWD" && npm run lint --silent 2>&1) || ERRORS="${ERRORS}
=== Lint Errors ===
$(echo "$LINT_OUT" | tail -30)"
  fi

  # 3. Tests
  HAS_TEST=$(jq -r '.scripts.test // empty' "$CWD/package.json" 2>/dev/null)
  if [[ -n "$HAS_TEST" ]]; then
    TEST_OUT=$(cd "$CWD" && CI=true npm test -- --watchAll=false 2>&1) || ERRORS="${ERRORS}
=== Test Failures ===
$(echo "$TEST_OUT" | tail -30)"
  fi
fi

if [[ -n "$ERRORS" ]]; then
  jq -nc --arg reason "Cannot commit — fix these issues first:
$ERRORS" \
    '{permissionDecision:"deny",permissionDecisionReason:$reason}'
fi
exit 0
```

**実行時に何が起こるか:**

|シナリオ |標準出力 |終了 |ホストのアクション |
| ---- | ---- | ---- | ---- |
|すべてのチェックに合格します |空 | `0` |コミット収益 |
| lint が失敗する | `{"permissionDecision":"deny","permissionDecisionReason":"Cannot commit — fix these issues first:\n=== Lint Errors ===\n..."}` | `0` |コミットをブロックします。エージェントはエラーを確認して修正します。
| jqが見つかりません |空 |ゼロ以外の |フックの失敗 |

### 例 2: ファイル編集後の自動フォーマット

**このパターンが重要な理由**: エージェントがコードを作成すると、その直後にフォーマッタが実行されます。手動の手順は必要ありません。エージェントによるそのファイルの次回の読み取りでは、フォーマットされたバージョンが表示されます。

**イベント**: `postToolUse` — `edit` または `create` ツール呼び出し後に発生します

**構成** — `.github/hooks/format-on-save.json`:
```json
{
  "version": 1,
  "hooks": {
    "postToolUse": [
      {
        "type": "command",
        "bash": "./.github/hooks/scripts/format-on-save.sh",
        "cwd": ".",
        "timeoutSec": 15
      }
    ]
  }
}
```

**スクリプト** — `.github/hooks/scripts/format-on-save.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

payload="$(cat)"
tool_name="$(printf '%s' "$payload" | jq -r '.toolName')"
result_type="$(printf '%s' "$payload" | jq -r '.toolResult.resultType // ""')"

# Only format after successful file writes
case "$tool_name" in
  edit|create) ;;
  *) exit 0 ;;
esac
[[ "$result_type" != "success" ]] && exit 0

file_path="$(printf '%s' "$payload" | jq -r '.toolArgs' | jq -r '.path // ""')"
[[ -z "$file_path" || ! -f "$file_path" ]] && exit 0

# Run the project's formatter — adapt to your stack
if command -v npx >/dev/null 2>&1 && [[ -f "package.json" ]]; then
  npx prettier --write "$file_path" 2>/dev/null || true
elif command -v dotnet >/dev/null 2>&1 && [[ "$file_path" == *.cs ]]; then
  dotnet format --include "$file_path" 2>/dev/null || true
fi
exit 0
```

**実行時に何が起こるか:**

|シナリオ |フックの役割 |終了 |
| ---- | ---- | ---- |
|エージェントは `src/app.ts` を正常に編集しました | `prettier --write src/app.ts` を実行します | `0` |
|エージェントは `bash ls` を実行します | Skips (ファイル書き込みツールではありません) | `0` |
| Prettier がインストールされていません |表示せずに書式設定をスキップします。 `0` |

### 例 3: 構造化拒否を使用して危険なコマンドをブロックする

**このパターンが重要な理由**: 最も単純なガードレール — エージェントが読み取ることができる明確な理由を示して、破壊的なシェル コマンドを実行前に防止します。

**イベント**: `preToolUse` — ツール呼び出しの前に発生します

**構成** — `.github/hooks/block-dangerous.json`:
```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "./.github/hooks/scripts/block-dangerous.sh",
        "cwd": ".",
        "timeoutSec": 5,
        "env": {
          "BLOCK_MODE": "deny"
        }
      }
    ]
  }
}
```

**スクリプト** — `.github/hooks/scripts/block-dangerous.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

payload="$(cat)"
block_mode="${BLOCK_MODE:-log}"
tool_name="$(printf '%s' "$payload" | jq -r '.toolName')"

[[ "$tool_name" != "bash" ]] && exit 0

command="$(printf '%s' "$payload" | jq -r '.toolArgs' | jq -r '.command // ""')"

if printf '%s' "$command" | grep -qE 'rm -rf /|git reset --hard|git clean -fd|git push.*--force'; then
  # Truncate command to avoid leaking secrets in deny reason or logs
  short_cmd="$(printf '%.80s' "$command")"
  if [[ "$block_mode" == "deny" ]]; then
    jq -cn --arg reason "Destructive command blocked: ${short_cmd}..." \
      '{permissionDecision:"deny",permissionDecisionReason:$reason}'
  else
    echo "Would block: ${short_cmd}..." >&2
  fi
fi
exit 0
```

**実行時に何が起こるか:**

|シナリオ |ブロックモード |標準出力 |終了 |ホストのアクション |
| ---- | ---- | ---- | ---- | ---- |
|安全なコマンド |任意 |空 | `0` |収益 |
| `git push --force` | `deny` | `{"permissionDecision":"deny",...}` | `0` |理由のあるブロック |
| `git push --force` | `log` |空 | `0` |収益 (ログのみ) |

## イベントの種類

完全なフックのリファレンスは信頼できます。 **フックを作成する前に、最新のペイロード形状を常に確認してください**。

- [フック設定リファレンス](https://docs.github.com/en/copilot/reference/hooks-configuration)
- [フックについて](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-hooks)

|イベント |標準出力 |一般的な使用方法 |
| ---- | ---- | ---- |
| `sessionStart` | **解析済み** — 標準出力の `additionalContext` がセッションに挿入されます。セットアップ、検証、コンテキスト挿入、ロギング |
| `sessionEnd` |無視 |クリーンアップ、要約 |
| `userPromptSubmitted` |無視 |監査、プロンプトのブロック |
| `preToolUse` | **解析済み** — `permissionDecision`、`modifiedArgs`/`updatedInput`、`additionalContext` |ガードレール、拒否/ブロック、引数の変更 |
| `postToolUse` |無視 |ロギング、フォーマット |
| `postToolUseFailure` | — |失敗したツールの実行後の回復 |
| `agentStop` | — |最終検証 |
| `subagentStart` | — |サブエージェントの監査 |
| `subagentStop` | — |サブエージェント出力の検証 |
| `errorOccurred` |無視 |診断、アラート |
| `preCompact` | — |事前締固め作業 |
| `permissionRequest` | — |承認ワークフロー |

### 一般的なイベントのペイロード スキーマ

これらはフックのリファレンスからのペイロードの形状です。最新のフィールドについては、[公式リファレンス](https://docs.github.com/en/copilot/reference/hooks-configuration) を常に参照してください。

**`sessionStart`**
```json
{
  "timestamp": 1704614400000,
  "cwd": "/path/to/project",
  "source": "new",
  "initialPrompt": "Create a new feature"
}
```

`source` は、`"new"`、`"resume"`、または `"startup"` です。 `initialPrompt` は、指定されている場合、ユーザーの最初のプロンプトです。

**`sessionStart` stdout 出力** — ホストは次の stdout を解析します。
```json
{
  "additionalContext": "Current branch: main. Deploy target: staging."
}
```

`additionalContext` はセッション会話に直接挿入され、フックが環境固有のコンテキストを動的に提供できるようにします。

**`sessionEnd`**
```json
{
  "timestamp": 1704618000000,
  "cwd": "/path/to/project",
  "reason": "complete"
}
```

`reason` は、`"complete"`、`"error"`、`"abort"`、`"timeout"`、または `"user_exit"` です。

**`userPromptSubmitted`**
```json
{
  "timestamp": 1704614500000,
  "cwd": "/path/to/project",
  "prompt": "Fix the authentication bug"
}
```

フィールドは `prompt` で、ユーザーが送信した正確なテキストです。

**`preToolUse`**
```json
{
  "timestamp": 1704614600000,
  "cwd": "/path/to/project",
  "toolName": "bash",
  "toolArgs": "{\"command\":\"rm -rf dist\",\"description\":\"Clean build directory\"}"
}
```

`toolArgs` は **JSON 文字列**です。フィールドにアクセスするには 2 回目に解析します。

**`preToolUse` stdout 出力** — ホストは次の stdout を解析します。

|フィールド |何をするのか |
| ---- | ---- |
| `permissionDecision` | `"deny"` はツール呼び出しをブロックします。 `"allow"` および `"ask"` も受け入れられます。 `"deny"` のみが現在処理されています。 |
| `permissionDecisionReason` |ユーザーに表示される人間が判読できる理由 |
| `modifiedArgs` または `updatedInput` |置換ツール引数 — 元の引数の代わりに使用されます。
| `additionalContext` |このターンでエージェントのコンテキストに挿入されたテキスト |

**`postToolUse`**
```json
{
  "timestamp": 1704614700000,
  "cwd": "/path/to/project",
  "toolName": "bash",
  "toolArgs": "{\"command\":\"npm test\"}",
  "toolResult": {
    "resultType": "success",
    "textResultForLlm": "All tests passed (15/15)"
  }
}
```

`resultType` は、`"success"`、`"failure"`、または `"denied"` です。

**`errorOccurred`**
```json
{
  "timestamp": 1704614800000,
  "cwd": "/path/to/project",
  "error": {
    "message": "Network timeout",
    "name": "TimeoutError",
    "stack": "TimeoutError: Network timeout\n    at ..."
  }
}
```

**`agentStop`**
```json
{
  "timestamp": 1704618000000,
  "cwd": "/path/to/project"
}
```

最小限のペイロード — `git diff --stat` の実行や最終検証などのセッション終了アクションをトリガーするために使用します。

## フックが間違ったツールである場合

| | のフックは避けてください。より良いフィット感 |
| ---- | ---- |
|自由形式の推論またはスタイルのガイダンス |指示、プロンプト、またはエージェント |
|メモリ、再試行、または分岐を伴う長い複数ステップのワークフロー |エージェント、スクリプト、またはワークフロー エンジン |
|バックグラウンド デーモン、ウォッチャー、デバウンス ループ、または非同期ジョブ |専用の自動化、サービス、または CI |
|リポジトリ全体にわたる大量の検証 | CI、スケジュールされたジョブ、または専用の自動化 |

## ユニバーサルデザインルール

|ルール |なぜそれが重要なのか |
| ---- | ---- |
|フックは 1 つ、責任は 1 つ |小さなフックは信頼しやすく、デバッグしやすいです。
|デフォルトは **最初に観察** |ブロックまたは突然変異は明示的に選択する必要があります。
|フックを同期、境界付き、非対話型に保つ |フックはクリティカル パスで実行されます。
|フックを決定的かつ冪等にする |再実行によりドリフトが発生しないようにする必要があります。
|デフォルトではブランチ、インデックス、またはワークツリーの状態を変更しません。 Git を破壊する行為は高リスクです |
|プロンプト、ツール引数、およびツール出力を信頼できない機密情報として扱います。入力は敵対的または個人的なものである可能性があります。
|ログからシークレット、認証情報、トークン、プライベート コンテンツを編集する |ログはフックの実行よりも長く存続することがよくあります。

## スクリプト作成ルール

- 実際に使用する JSON フィールドを検証する
- シェル変数を引用符で囲み、生の入力からコマンドを構築しないでください。
- ホストが構造化された出力を必要としない限り、stdout をクリーンな状態に保ちます
- 厳密モードを使用します: Bash `set -euo pipefail`、PowerShell `Set-StrictMode -Version Latest`
- 依存関係を早期に確認し、依存関係が欠落している場合は明らかに失敗します
- 実行中のプロンプト、非表示のインストール、または環境の変更を回避します。
- 代表的な JSON ペイロードを手動でパイプしてスクリプトをテストする

## 実行可能な最小の実装を選択する

1. **PowerShell 7**、**Node.js**、または **Python** (広範囲に移植可能なフック用)
2. **Bash** ここで、Bash は明示的な要件または安全な仮定です。
3. **既存のプロジェクト CLI** (リポジトリがすでに依存している場合)

通常のフックを実装するためだけに、新しいコンパイル済みランタイムを導入しないでください。

## 再利用可能なフックのパッケージ化

- 構成、スクリプト、ドキュメントをまとめてパッケージ化する
- トリガーイベント、目的、副作用、依存関係、および無効化パスを文書化します。
- フックが何を読み取り、何を書き込み、何をブロックするかを説明する

## アンチパターン

- 長時間実行されるフック、ウォッチャー、バックグラウンド デーモン、またはファイア アンド フォーゲット非同期作業
- より狭いトリガーで実行する場合、すべてのイベントで大量のスキャンが行われる
- クリティカル パスでの隠されたネットワーク呼び出しまたはアップロード
- デフォルトでの Git 状態のサイレントミューテーション (チェックアウト、リセット、クリーン、スタッシュ、ステージ、コミット、プッシュ、または履歴の書き換え)
- インタラクティブなプロンプトまたは暗黙的な承認ステップ
- ノイズの多い標準出力、アドホック出力形式、またはマシンと人間の混合出力
- 生のプロンプト、シークレット、認証情報、または大規模なツール出力のログ記録
- 無関係な責任が混在するモノリシックなフック

## 携帯性

### GitHub Copilot: CLI、VS Code、およびクラウド エージェント

同じ `.github/hooks/*.json` 構成、同じペイロード スキーマ、および同じスクリプト コントラクトは、CLI、VS Code、およびクラウド エージェント全体で機能します。イベント名は、キャメルケース (`preToolUse`) とパスカルケース (`PreToolUse`) の両方を受け入れます。ツール引数の文書化されたペイロード フィールドは `toolArgs` (JSON 文字列) です。

知っておくべき点が 1 つあります。クラウド エージェントはリポジトリの **デフォルト ブランチ**からのみフックをロードします。 hooks.json が機能ブランチ上にのみ存在する場合、クラウド エージェントはそれを認識しません。

### クロード・コード

Claude コードは別のフック システムを使用します。

- `~/.claude/settings.json` および `.claude/settings.json` の設定
- さまざまなイベント名とマッチャー構文 (正規表現、`if` 条件)
- 終了 2 = ブロック、終了 1 = 非ブロッキング エラー (GitHub Copilot とは異なります)
- 5 種類のフック (コマンド、http、mcp_tool、プロンプト、エージェント)
- `FileChanged`、`CwdChanged`、`ConfigChange` を含む 29 以上のイベント

共有されるベスト プラクティスも同じです。フックは小さく、決定論的に保ち、I/O については明示的にし、副作用については厳密にします。