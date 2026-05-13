# コードベース構造

## コアセクション（必須）

### 1) トップレベルマップ

意味のあるトップレベルのディレクトリとファイルのみを列挙してください。

| Path | Purpose | Evidence |
|------|---------|----------|
| [path/] | [purpose] | [source] |

### 2) エントリーポイント

- メインのランタイムエントリ: [FILE]
- セカンダリエントリポイント（worker/cli/jobs）: [FILES or NONE]
- エントリの選択方法（script/config）: [NOTE]

### 3) モジュール境界

| Boundary | What belongs here | What must not be here |
|----------|-------------------|------------------------|
| [module/layer] | [responsibility] | [forbidden logic] |

### 4) 命名および構成ルール

- ファイル命名パターン: [kebab/camel/Pascal + examples]
- ディレクトリ構成パターン: [feature/layer/domain]
- import エイリアスまたはパス規約: [RULE]

### 5) 根拠

- [path/to/root-tree-source]
- [path/to/entry-config]
- [path/to/key-module]

## 拡張セクション（任意）

リポジトリの複雑さに応じて必要な場合にのみ追加してください。

- 機能/レイヤー別のサブディレクトリ詳細マップ
- ミドルウェア/起動順の詳細
- 生成物とソースのレイアウト境界
- モノレポにおけるワークスペースレベルの構造マップ

