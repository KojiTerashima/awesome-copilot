# コーディング規約

## コアセクション（必須）

### 1) 命名ルール

| 項目 | ルール | 例 | 根拠 |
|------|------|---------|----------|
| ファイル | [RULE] | [EXAMPLE] | [FILE] |
| 関数/メソッド | [RULE] | [EXAMPLE] | [FILE] |
| 型/インターフェース | [RULE] | [EXAMPLE] | [FILE] |
| 定数/環境変数 | [RULE] | [EXAMPLE] | [FILE] |

### 2) フォーマットとLint

- Formatter: [TOOL + CONFIG FILE]
- Linter: [TOOL + CONFIG FILE]
- 特に重要な適用ルール: [RULE_1], [RULE_2], [RULE_3]
- 実行コマンド: [COMMANDS]

### 3) インポートとモジュールの規約

- インポートのグルーピング/順序: [RULE]
- エイリアス import と相対 import の方針: [RULE]
- 公開エクスポート/バレルの方針: [RULE]

### 4) エラーとログの規約

- レイヤーごとのエラー戦略: [SHORT SUMMARY]
- ログのスタイルと必須コンテキスト項目: [SUMMARY]
- 機密データのマスキング規則: [SUMMARY]

### 5) テスト規約

- テストファイルの命名/配置ルール: [RULE]
- モック戦略の基本方針: [RULE]
- カバレッジ期待値: [RULE or TODO]

### 6) 根拠

- [path/to/lint-config]
- [path/to/format-config]
- [path/to/representative-source-file]

## 拡張セクション（任意）

大規模または規約が不統一なコードベースでのみ追加:

- レイヤー別エラー処理マトリクス
- 言語別の厳格性オプション
- リポジトリ固有のコミット/ブランチ運用規約
- 解消すべき既知の規約違反

