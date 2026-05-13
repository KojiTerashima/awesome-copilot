# テストパターン

## コアセクション（必須）

### 1) テストスタックとコマンド

- 主要なテストフレームワーク: [NAME + VERSION]
- アサーション／モックツール: [TOOLS]
- コマンド:

```bash
[run all tests]
[run unit tests]
[run integration/e2e tests]
[run coverage]
```

### 2) テストレイアウト

- テストファイルの配置パターン: [co-located/tests folder/etc]
- 命名規則: [pattern]
- セットアップファイルと実行場所: [paths]

### 3) テストスコープマトリクス

| スコープ | 対象? | 典型的な対象 | 備考 |
|-------|----------|----------------|-------|
| Unit | [yes/no] | [modules/services] | [notes] |
| Integration | [yes/no] | [API/data boundaries] | [notes] |
| E2E | [yes/no] | [user flows] | [notes] |

### 4) モックと分離戦略

- 主なモック手法: [module/class/network]
- 分離の保証: [what is reset and when]
- テストでよくある失敗パターン: [short note]

### 5) カバレッジと品質シグナル

- カバレッジツール + 閾値: [value or TODO]
- 現在報告されているカバレッジ: [value or TODO]
- 既知のギャップ／不安定な領域: [list]

### 6) 根拠

- [path/to/test-config]
- [path/to/representative-test-file]
- [path/to/ci-or-coverage-config]

## 拡張セクション（任意）

必要な場合にのみ追加:

- フレームワーク固有のスイートパターン
- 依存関係タイプごとの詳細なモックレシピ
- 過去の不安定テストカタログ
- テスト性能のボトルネックと最適化アイデア

