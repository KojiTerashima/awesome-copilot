# pyproject.toml、バックエンド、バージョン管理、および型付きパッケージ

## 目次
1. [完全な pyproject.toml — setuptools + setuptools_scm](#1-complete-pyprojecttoml)
2. [孵化したばかりの子 (モダン、ゼロ構成)](#2-hatchling-modern-zero-config)
3. [flit (最小、`__version__` のバージョン)](#3-flit-minimal-version-from-__version__)
4. [poetry (統合された dep マネージャー)](#4-poetry-integrated-dep-manager)
5. [バージョン管理戦略 — PEP 440、semver、dep 指定子](#5-versioning-strategy)
6. [setuptools_scm — git タグからの動的バージョン](#6-dynamic-versioning-with-setuptools_scm)
7. [従来の編集可能なインストール用の setup.py shim](#7-setuppy-shim)
8. [PEP 561 型付きパッケージ (py.typed)](#8-typed-package-pep-561)

---

## 1. pyproject.toml を完成させます

### setuptools + setuptools_scm (git タグのバージョン管理に推奨)```トムル
[ビルドシステム]
Required = ["setuptools>=68", "wheel", "setuptools_scm"]
build-backend = "setuptools.build_meta"

[プロジェクト]
名前 = "あなたのパッケージ"
Dynamic = ["version"] # バージョンは setuptools_scm 経由で git タグから取得されます
description = "<説明> — <主な機能 1>、<主な機能 2>"
readme = "README.md"
必要な-Python = ">=3.10"
ライセンス = "MIT" # PEP 639 SPDX 式 (文字列、{text = "MIT"} ではありません)
ライセンスファイル = ["ライセンス"]
著者 = [
    {名前 = "あなたの名前"、メール = "you@example.com"}、
】
メンテナー = [
    {名前 = "あなたの名前"、メール = "you@example.com"}、
】
キーワード = [
    「パイソン」、
    # ライブラリを説明する 10 ～ 15 個の特定のキーワードを追加します - これらは PyPI の発見可能性に影響します
】
分類子 = [
    "開発ステータス :: 3 - アルファ"、# 安定版リリースでは 5 に変更されます
    "対象読者:: 開発者",
    "ライセンス :: OSI 承認済み :: MIT ライセンス",
    "オペレーティング システム :: OS に依存しない",
    "プログラミング言語 :: Python :: 3",
    "プログラミング言語 :: Python :: 3.10",
    "プログラミング言語 :: Python :: 3.11",
    "プログラミング言語 :: Python :: 3.12",
    "プログラミング言語 :: Python :: 3.13",
    "トピック :: ソフトウェア開発 :: ライブラリ :: Python モジュール",
    "Typing :: Typed", # py.typed を出荷するときにこれを追加します
】
依存関係 = [
    # ここにランタイムの依存関係をリストします。それらは最小限にしてください。
    # 例: "httpx>=0.24"、"pydantic>=2.0"
    # ライブラリに必要なランタイム DEPS がない場合は、空のままにしておきます。
】

[プロジェクト.オプションの依存関係]
レディス = [
    "redis>=4.2", # オプションの重いバックエンド
】
開発 = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
    "httpx>=0.24",
    "pytest-cov>=4.0",
    "ラフ>=0.4"、
    "ブラック>=24.0"、
    "isort>=5.13",
    "mypy>=1.0",
    "事前コミット>=3.0",
    「構築する」、
    「麻ひも」、
】

[プロジェクト.url]
ホームページ = "https://github.com/あなたのユーザー名/あなたのパッケージ"
ドキュメント = "https://github.com/yourusername/your-package#readme"
リポジトリ = "https://github.com/yourusername/your-package"
「バグトラッカー」=「https://github.com/yourusername/your-package/issues」
変更ログ = "https://github.com/yourusername/your-package/blob/master/CHANGELOG.md"

# --- Setuptools 構成 ---
[tool.setuptools.packages.find]
include = ["your_package*"] # フラットレイアウト
# src/layout の場合は、次を使用します。
# ここで = ["ソース"]

[tool.setuptools.パッケージデータ]
your_package = ["py.typed"] # py.typed マーカーをホイールに含めて送信します

# --- setuptools_scm: git タグからのバージョン ---
[tool.setuptools_scm]
version_scheme = "リリース後"
local_scheme = "no-local-version" # +local サフィックスによる PyPI アップロードの破壊を防止します

# --- ラフ（糸くず） ---
[ツール.ラフ]
ターゲットバージョン = "py310"
行の長さ = 100

[ツール.ruff.lint]
select = ["E"、"F"、"W"、"I"、"N"、"UP"、"B"、"SIM"、"C4"、"PTH"、"RUF"]
ignore = ["E501"] # フォーマッタによって強制される行の長さ[tool.ruff.lint.ファイルごとの無視]
"tests/*" = ["S101", "ANN"] # テストでのアサーションと欠落したアノテーションを許可します
"scripts/*" = ["T201"] # スクリプトでの印刷を許可します

[tool.ruff.format]
引用スタイル = "ダブル"

# --- 黒 (書式設定) ---
[ツール.ブラック]
行の長さ = 100
ターゲットバージョン = ["py310", "py311", "py312", "py313"]

# --- isort (インポートソート) ---
[ツール.isort]
プロフィール = "黒"
行の長さ = 100

# --- mypy (静的型チェック) ---
[ツール.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
warn_unused_ignores = true
disallow_untyped_defs = true
disallow_any_generics = true
ignore_missing_imports = true
strict = false # 最大限の厳密性を得るには true を設定します

[[tool.mypy.overrides]]
module = "テスト.*"
disallow_untyped_defs = false # テストで緩和

# --- pytest ---
[tool.pytest.ini_options]
asyncio_mode = "自動"
テストパス = ["テスト"]
pythonpath = ["."] # フラット レイアウトの場合。 src/ の削除
python_files = "test_*.py"
python_classes = "テスト*"
python_functions = "テスト_*"
addopts = "-v --tb=short --cov=your_package --cov-report=term-missing"

# --- 対象範囲 ---
[ツール.カバレッジ.実行]
ソース = ["あなたのパッケージ"]
省略 = ["tests/*"]

[ツール.カバレッジ.レポート]
失敗アンダー = 80
show_missing = true
exclude_lines = [
    "プラグマ: カバーなし",
    "def __repr__",
    "NotImplementedError を発生させる",
    "TYPE_CHECKING の場合:",
    "@abstractメソッド",
】
「」---

## 2. 孵化したばかりの子 (最新、ゼロ構成)

C 拡張機能を必要としない新しい純粋な Python プロジェクトに最適です。 `setup.py` は必要ありません。使用する
git タグのバージョン管理の場合は `hatch-vcs`、手動でバージョンをバンプする場合は省略します。```トムル
[ビルドシステム]
Required = ["hatchling", "hatch-vcs"] # git タグのバージョン管理用の hatch-vcs
build-backend = "hatchling.build"

[プロジェクト]
名前 = "あなたのパッケージ"
Dynamic = ["version"] # 手動バージョン管理用に version = "1.0.0" を削除して追加します
description = "1 行の説明"
readme = "README.md"
必要な-Python = ">=3.10"
ライセンス = "MIT"
ライセンスファイル = ["ライセンス"]
著者 = [{名前 = "あなたの名前", 電子メール = "you@example.com"}]
キーワード = ["Python"]
分類子 = [
    "開発状況 :: 3 - アルファ",
    "対象読者:: 開発者",
    "ライセンス :: OSI 承認済み :: MIT ライセンス",
    "オペレーティング システム :: OS に依存しない",
    "プログラミング言語 :: Python :: 3",
    "入力 :: 入力済み",
】
依存関係 = []

[プロジェクト.オプションの依存関係]
dev = ["pytest>=8.0", "pytest-cov>=5.0", "ruff>=0.6", "mypy>=1.10"]

[プロジェクト.url]
ホームページ = "https://github.com/あなたのユーザー名/あなたのパッケージ"
変更ログ = "https://github.com/yourusername/your-package/blob/master/CHANGELOG.md"

# --- ヒナのビルド構成 ---
[tool.hatch.build.targets.wheel]
パッケージ = ["src/your_package"] # src/ レイアウト
# パッケージ = ["your_package"] # ← フラット レイアウト

[ツール.ハッチング.バージョン]
source = "vcs" # hatch-vcs による git タグのバージョン管理

[tool.hatch.version.raw-options]
local_scheme = "ローカルバージョンなし"

# ruff、mypy、pytest、カバレッジ セクション — 上記の setuptools テンプレートと同じ
「」---

## 3. flit (最小、`__version__` のバージョン)

非常に単純な単一モジュールのパッケージに最適です。設定ゼロ。バージョンは直接読み込まれます
@@コード1@@。 `__version__` には常に **静的文字列** が必要です。```トムル
[ビルドシステム]
必要 = ["flit_core>=3.9"]
ビルドバックエンド = "flit_core.buildapi"

[プロジェクト]
名前 = "あなたのパッケージ"
Dynamic = ["version", "description"] # __init__.py __version__ と docstring から読み取る
readme = "README.md"
必要な-Python = ">=3.10"
ライセンス = "MIT"
著者 = [{名前 = "あなたの名前", 電子メール = "you@example.com"}]
分類子 = [
    "ライセンス :: OSI 承認済み :: MIT ライセンス",
    "プログラミング言語 :: Python :: 3",
    "入力 :: 入力済み",
】
依存関係 = []

[プロジェクト.url]
ホームページ = "https://github.com/あなたのユーザー名/あなたのパッケージ"

# flit は your_package/__init__.py から __version__ を自動的に読み取ります。
# __init__.py に次の内容が含まれていることを確認します: __version__ = "1.0.0" (静的文字列 - flit はサポートしていません)
# 動的バージョン検出用の importlib.metadata)
「」---

## 4. 詩 (統合された依存関係 + ビルド マネージャー)

デプスの管理、構築、公開を 1 つのツールで行いたいチームに最適です。詩 v2+
標準の `[project]` テーブルをサポートします。```トムル
[ビルドシステム]
必要 = ["詩コア>=2.0"]
build-backend = "poetry.core.masonry.api"

[プロジェクト]
名前 = "あなたのパッケージ"
バージョン = "1.0.0"
description = "1 行の説明"
readme = "README.md"
必要な-Python = ">=3.10"
ライセンス = "MIT"
著者 = [{名前 = "あなたの名前", 電子メール = "you@example.com"}]
分類子 = [
    "プログラミング言語 :: Python :: 3",
    "入力 :: 入力済み",
】
dependency = [] # 詩 v2+ は標準の [プロジェクト] テーブルを使用します

[プロジェクト.オプションの依存関係]
dev = ["pytest>=8.0", "ruff>=0.6", "mypy>=1.10"]

# オプション: [tool.poetry] は詩固有の機能にのみ使用します
[tool.poetry.group.dev.dependency]
# 詩固有のグループ構文 ([project.optional-dependency] の代替)
pytest = ">=8.0"
「」---

## 5. バージョン管理戦略

### PEP 440 — スタンダード「」
正規形式: N[.N]+[{a|b|rc}N][.postN][.devN]

例:
  1.0.0 安定版リリース
  1.0.0a1 アルファ (プレリリース)
  1.0.0b2 ベータ版
  1.0.0rc1 リリース候補
  1.0.0.post1 リリース後 (例: パッケージ修正のみ - コード変更なし)
  1.0.0.dev1 開発スナップショット (PyPI 用ではありません)
「」### セマンティック バージョニング (SemVer) — すべてのライブラリにこれを使用します「」
メジャーパッチ、マイナーパッチ

メジャー: API の重大な変更 (パブリック関数/クラス/引数の削除/名前変更)
マイナー: 新機能、完全な下位互換性
パッチ: バグ修正、API 変更なし
「」|変更 |何がぶつかる |例 |
|---|---|---|
|パブリック関数を削除/名前変更する |メジャー | `1.2.3 → 2.0.0` |
|新しいパブリック関数を追加 |マイナー | `1.2.3 → 1.3.0` |
|バグ修正、API 変更なし |パッチ | `1.2.3 → 1.2.4` |
|新しいプレリリース |接尾辞 | `2.0.0a1`、`2.0.0rc1` |

### コード内のバージョン — パッケージのメタデータから読み取る「」パイソン
# your_package/__init__.py
importlib.metadata インポート バージョンから、PackageNotFoundError

試してみてください:
    __version__ = version("あなたのパッケージ")
PackageNotFoundError を除く:
    __version__ = "0.0.0-dev" # アンインストールされた開発チェックアウトのフォールバック
「」setuptools_scm を使用する場合は、`__version__ = "1.0.0"` をハードコードしないでください。setuptools_scm を使用すると、無効になります。
最初の git タグ。常に `importlib.metadata` を使用してください。

### 依存関係に関するバージョン指定子のベスト プラクティス```トムル
# [プロジェクト] の依存関係内 — ライブラリの場合:
"httpx>=0.24" # 最小バージョン - ライブラリに推奨
"httpx>=0.24,<1.0" # 既知の重大な変更が存在する場合のみ上限

# アプリケーションのみ (ライブラリには決して使用しないでください):
"httpx==0.27.0" # 正確にピン留めします - ライブラリの dep 解像度を壊します

# ライブラリではこれを決して行わないでください:
# "httpx~=0.24.0" # 互換性のあるリリース演算子 — 厳しすぎます
# "httpx==0.27.*" # ワイルドカード ピン — 壊れやすい
「」---

## 6. `setuptools_scm` を使用した動的バージョニング

`setuptools_scm` は git タグを読み取り、パッケージのバージョンを自動的に設定します。手動で行う必要はありません
各リリースの前にバージョン文字列を編集します。

### 仕組み「」
git タグ v1.0.0 → パッケージ バージョン = 1.0.0
git タグ v1.1.0 → パッケージ バージョン = 1.1.0
(タグの後にコミット) → version = 1.1.0.post1+g<hash> (PyPI 用に削除)
「」`local_scheme = "no-local-version"` は `+g<hash>` サフィックスを取り除き、PyPI アップロードが失敗しないようにします。
「ローカル バージョン ラベルは許可されていません」エラー。

### 実行時のバージョンへのアクセス「」パイソン
# your_package/__init__.py
importlib.metadata インポート バージョンから、PackageNotFoundError

試してみてください:
    __version__ = version("あなたのパッケージ")
PackageNotFoundError を除く:
    __version__ = "0.0.0-dev" # アンインストールされた開発チェックアウトのフォールバック
「」setuptools_scm を使用する場合は、`__version__ = "1.0.0"` をハードコードしないでください。setuptools_scm を使用すると、無効になります。
最初のタグ。

### 完全なリリース フロー (これだけです。他には何も必要ありません)「」バッシュ
git タグ v1.2.0
git Push Origin master --tags
# GitHub アクションのpublish.yml が自動的にトリガーされる
「」---

## 7. `setup.py` シム

一部の古いツールや IDE では、依然として `setup.py` が必要です。 3 行のシムとして保持します - すべて本物です
設定は `pyproject.toml` に残ります。「」パイソン
# setup.py — 薄いシムのみ。すべての設定は pyproject.toml にあります。
setuptoolsからセットアップをインポート

セットアップ()
「」`name`、`version`、`dependencies`、またはその他のメタデータを `pyproject.toml` から複製しないでください。
`setup.py` に変換します。そこに何かをコピーすると、最終的にはずれてしまい、混乱を招く競合が発生します。

---

## 8. 型付きパッケージ (PEP 561)

適切に宣言された型付きパッケージとは、mypy、pyright、および IDE が自動的に型を取得することを意味します。
ユーザーが追加の構成を行うことなく、ヒントを得ることができます。

### ステップ 1: マーカー ファイルを作成する「」バッシュ
# ファイルは存在する必要があります。その内容は重要ではありません。その存在が信号です。
your_package/py.typed にタッチします
「」### ステップ 2: ホイールに含める

上記のテンプレートにはすでに含まれています:```トムル
[tool.setuptools.パッケージデータ]
your_package = ["py.typed"]
「」### ステップ 3: PyPI 分類子を追加する```トムル
分類子 = [
    ...
    "入力 :: 入力済み",
】
「」### ステップ 4: すべてのパブリック関数に型アノテーションを付ける「」パイソン
# Good — fully typed
def process(
    自分自身、
    data: dict[str, object],
    *、
    タイムアウト: int = 30、
) -> dict[str, object]:
    ...

# 悪い — mypy はこれにフラグを立てますが、IDE はユーザーに補完を提供しません
def プロセス (自己、データ、タイムアウト = 30):
    ...
「」### ステップ 5: py.typed がホイールに組み込まれていることを確認する「」バッシュ
Python -m ビルド
unzip -l dist/your_package-*.whl | unzip -l dist/your_package-*.whl | unzip -l dist/your_package-*.whl grep py.typed
# 表示する必要があります: your_package/py.typed
「」見つからない場合は、`[tool.setuptools.package-data]` 構成を確認してください。