---
name: react-audit-grep-patterns
description: 'Provides the complete, verified grep scan command library for auditing React codebases before a React 18.3.1 or React 19 upgrade. Use this skill whenever running a migration audit - for both the react18-auditor and react19-auditor agents. Contains every grep pattern needed to find deprecated APIs, removed APIs, unsafe lifecycle methods, batching vulnerabilities, test file issues, dependency conflicts, and React 19 specific removals. Always use this skill when writing audit scan commands - do not rely on memory for grep syntax, especially for the multi-line async setState patterns which require context flags.'
---
# React 監査 Grep パターン

React 18.3.1 および React 19 移行監査用の完全なスキャン コマンド ライブラリ。

## 使用法

ターゲットに関連するセクションを読んでください。
- **`references/react18-scans.md`** - React 16/17 → 18.3.1 監査のすべてのスキャン
- **`references/react19-scans.md`** - React 18 → 19 監査のすべてのスキャン
- **`references/test-scans.md`** - テスト ファイル固有のスキャン (両方の監査人が使用)
- **`references/dep-scans.md`** - 依存関係とピアの競合スキャン

## すべてのスキャンで使用される基本パターン「」バッシュ
# 全体で使用される標準フラグ:
# -r = 再帰的
# -n = 行番号を表示
# -l = ファイル名のみを表示します (影響を受けるファイルをカウントするため)
# --include="*.js" --include="*.jsx" = JS/JSX ファイルのみ
# | grep -v "\.test\.\|\.spec\.\|__tests__" = テスト ファイルを除外します
# | grep -v "node_modules" = 安全性 (通常、node_modules をスキャンしないことで処理されます)
# 2>/dev/null = 「ファイルが見つかりません」エラーを抑制します

# ソース ファイルのみ (テストは除く):
SRC_FLAGS='--include="*.js" --include="*.jsx"'
EXCLUDE_TESTS='grep -v "\.test\.\|\.spec\.\|__tests__"'

# テストファイルのみ:
TEST_FLAGS='--include="*.test.js" --include="*.test.jsx" --include="*.spec.js" --include="*.spec.jsx"'
「」
