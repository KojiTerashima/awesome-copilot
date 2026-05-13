# 技術スタック

## コアセクション（必須）

### 1) ランタイム概要

| 領域 | 値 | 根拠 |
|------|-------|----------|
| 主要言語 | [VALUE] | [FILE_PATH] |
| ランタイム + バージョン | [VALUE] | [FILE_PATH] |
| パッケージマネージャー | [VALUE] | [FILE_PATH] |
| モジュール/ビルドシステム | [VALUE] | [FILE_PATH] |

### 2) 本番フレームワークと依存関係

高インパクトな本番依存関係（フレームワーク、データ、トランスポート、認証）のみを記載すること。

| 依存関係 | バージョン | システム内での役割 | 根拠 |
|------------|---------|----------------|----------|
| [NAME] | [VERSION] | [ROLE] | [FILE_PATH] |

### 3) 開発ツールチェーン

| ツール | 用途 | 根拠 |
|------|---------|----------|
| [TOOL] | [LINT/FORMAT/TEST/BUILD] | [FILE_PATH] |

### 4) 主要コマンド

```bash
[install command]
[build command]
[test command]
[lint command]
```

### 5) 環境と設定

- 設定ソース: [LIST FILES]
- 必須環境変数: [VAR_1], [VAR_2], [TODO]
- デプロイ/ランタイム制約: [SHORT NOTE]

### 6) 根拠

- [path/to/manifest]
- [path/to/runtime-config]
- [path/to/build-or-ci-config]

## 拡張セクション（任意）

複雑なリポジトリで必要な場合のみ追加:

- カテゴリ別の完全な依存関係分類
- コンパイラ/ランタイムフラグの詳細
- 環境マトリクス（dev/stage/prod）
- プロセスマネージャーとコンテナランタイムの詳細

