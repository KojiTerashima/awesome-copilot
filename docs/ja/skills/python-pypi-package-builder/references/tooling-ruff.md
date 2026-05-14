# ツール — Ruff のみのセットアップとコード品質

## 目次
1. [Ruff のみを使用 (black、isort、flake8 を置き換える)](#1-use-only-ruff-replaces-black-isort-flake8)
2. [pyproject.toml の Ruff 設定](#2-ruff-configuration-in-pyprojecttoml)
3. [mypy 構成](#3-mypy-configuration)
4. [コミット前の構成](#4-コミット前の構成)
5. [pytest とカバレッジ構成](#5-pytest-and-coverage-configuration)
6. [pyproject.toml の開発依存関係](#6-dev-dependency-in-pyprojecttoml)
7. [CI lint ジョブ — Ruff のみ](#7-ci-lint-job--ruff-only)
8. [移行ガイド — black と isort の削除](#8-migration-guide--removing-black-and-isort)

---

## 1. Ruff のみを使用します (black、isort、flake8 を置き換えます)

**決定:** `ruff` を単一の lint および書式設定ツールとして使用します。 `black` と `isort` を削除します。

|古い (避ける) |新品(使用) |何をするのか |
|---|---|---|
| `black` | `ruff format` |コードのフォーマット |
| `isort` | `ruff check --select I` |インポートの並べ替え |
| `flake8` | `ruff check` |スタイルとエラーのリンティング |
| `pyupgrade` | `ruff check --select UP` |構文を最新の Python にアップグレードする |
| `bandit` | `ruff check --select S` |セキュリティリンティング |
|上記のすべて | `ruff` | 1 つのツール、1 つの構成セクション |

**なぜラフなのですか?**
- 代替ツール (Rust で書かれた) よりも 10 ～ 100 倍高速です。
- `pyproject.toml` の単一の設定セクション — `.flake8`、`.isort.cfg`、`pyproject.toml[tool.black]` のスプロールなし。
- Astral によって積極的に保守されています。置き換えられるツールと同じルールに従います。
- `ruff format` は黒と互換性があります。つまり、既存の黒でフォーマットされたコードは変更せずに渡されます。

---

## 2. pyproject.toml の Ruff 設定```トムル
[ツール.ラフ]
target-version = "py310" # サポートされる最小の Python バージョン
line-length = 88 # 黒互換のデフォルト
src = ["src", "テスト"]

[ツール.ruff.lint]
選択 = [
    "E"、# pycodestyle エラー
    "W"、# pycodestyle 警告
    "F"、# パイフレーク
    「私」、# isort
    "B"、# flake8-bugbear (意見はありますが、非常に便利です)
    "C4"、# flake8-comprehensions
    "UP"、# pyupgrade (最新の構文)
    「SIM」、#flake8-simplify
    "TCH"、# flake8-type-checking (インポートを TYPE_CHECKING ブロックに移動)
    "ANN"、# flake8-annotations (型ヒントを強制します - 厳密すぎる場合は削除します)
    "S"、# flake8-bandit (セキュリティ)
    "N"、# pep8-naming
】
無視 = [
    "ANN101", # `self` の型アノテーションがありません
    "ANN102", # `cls` の型アノテーションがありません
    "S101"、# `assert` の使用 — テストで必要
    "S603"、 # シェル = True のないサブプロセス — 多くの場合、意図的です
    "B008", # デフォルト引数で関数呼び出しを実行しないでください (FastAPI/Typer での誤検知)
】

[tool.ruff.lint.isort]
既知のファーストパーティ = ["your_package"]

[tool.ruff.lint.ファイルごとの無視]
"tests/**" = ["S101", "ANN", "D"] # テストでアノテーション/docstring のアサートとスキップを許可します

[tool.ruff.format]
quote-style = "double" # 黒互換
インデントスタイル = "スペース"
スキップマジック末尾のカンマ = false
行末 = "自動"
「」### 便利な ruff コマンド「」バッシュ
# lint の問題をチェックする (変更なし)
ラフチェック。

# 修正可能な問題を自動修正する
ruff check --fix 。

# フォーマットコード (黒を置き換えます)
ラフフォーマット。

# ファイルを変更せずにフォーマットをチェックする (CI モード)
ruff 形式 --check 。

# 1 つのコマンドで lint チェックとフォーマット チェックの両方を実行します (CI の場合)
ラフチェック。 && ラフ形式 --check 。
「」---

## 3. mypy の設定```トムル
[ツール.mypy]
python_version = "3.10"
厳密 = 真
warn_return_any = true
warn_unused_ignores = true
warn_redundant_casts = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
show_error_codes = true

# タイプを同梱しないサードパーティパッケージの欠落したスタブを無視します
[[tool.mypy.overrides]]
module = ["redis.*", "pydantic_settings.*"]
ignore_missing_imports = true
「」### mypy の実行 — src レイアウトとフラット レイアウトの両方を処理する「」バッシュ
# ソースレイアウト:
mypy src/your_package/

# フラットレイアウト:
mypy your_package/
「」CI でレイアウトを動的に検出します。```ヤムル
- 名前: mypy を実行します。
  実行: |
    if [ -d "src" ];それから
        mypy src/
    それ以外の場合
        mypy your_package/
    フィ
「」---

## 4. コミット前の構成```ヤムル
# .pre-commit-config.yaml
リポジトリ:
  - リポジトリ: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4 # 特定のリリースに固定します。 `pre-commit autoupdate` を使用して定期的に更新します
    フック:
      - ID: ラフ
        args: [--fix] # 修正できるものは自動修正します
      - id: ruff-format # フォーマット (黒いフックを置き換えます)

  - リポジトリ: https://github.com/pre-commit/mirrors-mypy
    リビジョン: v1.10.0
    フック:
      - ID: mypy
        追加の依存関係:
          - タイプリクエスト
          - タイプ-redis
          # パッケージで使用される型付き依存関係のスタブを追加します

  - リポジトリ: https://github.com/pre-commit/pre-commit-hooks
    リビジョン: v4.6.0
    フック:
      - ID: 末尾の空白
      - ID: ファイルの終わりの修正者
      - ID: チェックトム
      - ID: check-yaml
      - ID: チェックマージ競合
      - ID: 追加された大きなファイルをチェック
        引数: ["--maxkb=500"]
「」### ❌ これらのフックを取り外します (ラフに置き換えます)```ヤムル
# 削除するか、追加しないでください:
- リポジトリ: https://github.com/psf/black # ruff-format に置き換えられました
- リポジトリ: https://github.com/PyCQA/isort # ruff lint I ルールに置き換えられます
- リポジトリ: https://github.com/PyCQA/flake8 # ruff check に置き換えられました
- リポジトリ: https://github.com/PyCQA/autoflake # ruff check F401 に置き換えられました
「」＃＃＃ 設定「」バッシュ
pip install プリコミット
pre-commit install # git フックをインストールします — コミットごとに実行します
pre-commit run --all-files # すべてのファイルに対して手動で実行します
pre-commit autoupdate # すべてのフックを最新の固定バージョンに更新します
「」---

## 5. pytest とカバレッジ設定```トムル
[tool.pytest.ini_options]
テストパス = ["テスト"]
addopts = "-ra -q --strict-markers --cov=your_package --cov-report=term-missing"
asyncio_mode = "auto" # @pytest.mark.asyncio デコレータなしで非同期テストを有効にします

[ツール.カバレッジ.実行]
ソース = ["あなたのパッケージ"]
ブランチ = true
omit = ["**/__main__.py", "**/cli.py"] # カバレッジからエントリ ポイントを省略します

[ツール.カバレッジ.レポート]
show_missing = true
スキップ_カバー = false
failed_under = 85 # カバレッジが 85% を下回る場合、CI は失敗します
exclude_lines = [
    "プラグマ: カバーなし",
    "TYPE_CHECKING の場合:",
    "NotImplementedError を発生させる",
    "@abstractメソッド",
】
「」### asyncio_mode = "auto" — @pytest.mark.asyncio を削除します

`asyncio_mode = "auto"` が `pyproject.toml` に設定されている場合は、**`@pytest.mark.asyncio` を追加しないでください**
機能をテストするため。デコレータは冗長であるため、最新の pytest-asyncio では警告が表示されます。「」パイソン
# 誤り — asyncio_mode = "auto" の場合、デコレーターは非推奨になります。
@pytest.mark.asyncio
async def test_async_operation():
    結果 = my_async_func() を待ちます
    アサート結果 == 期待される

# 正しい — async def を使用するだけです。
async def test_async_operation():
    結果 = my_async_func() を待ちます
    アサート結果 == 期待される
「」---

## 6. pyproject.toml の開発依存関係

すべての開発/テスト ツールを `dev` という名前の `[extras]` グループで宣言します。```トムル
[プロジェクト.オプションの依存関係]
開発 = [
    "pytest>=8",
    "pytest-asyncio>=0.23",
    "pytest-cov>=5",
    "ラフ>=0.4"、
    "mypy>=1.10",
    "事前コミット>=3.7",
    "httpx>=0.27", # HTTP トランスポートをテストする場合
    "respx>=0.21", # テストで httpx をモックする場合
】
レディス = [
    "redis>=5",
】
ドキュメント = [
    "mkdocs-material>=9",
    "mkdocstrings[python]>=0.25",
】
「」開発依存関係をインストールします。「」バッシュ
pip install -e ".[dev]"
pip install -e ".[dev,redis]" # オプションの追加機能を含める
「」---

## 7. CI リント ジョブ — Ruff のみ

個別の `black`、`isort`、`flake8` ステップを 1 つの `ruff` ステップに置き換えます。```ヤムル
# .github/workflows/ci.yml — lint ジョブ
糸くず:
  名前: リントとタイプチェック
  実行: ubuntu-最新
  手順:
    - 使用:actions/checkout@v4

    - 使用:actions/setup-python@v5
      と:
        Python バージョン: "3.11"

    - 名前: 開発依存関係のインストール
      実行: pip install -e ".[dev]"

    # シングルステップ: ruff が black + isort + flake8 を置き換えます
    - 名前：ラフ・リント
      実行：ラフチェック。

    - 名前: ruff フォーマットチェック
      実行: ruff 形式 --check 。

    - 名前：マイピー
      実行: |
        if [ -d "src" ];それから
            mypy src/
        それ以外の場合
            mypy $(basename $(ls -d */))/ 2>/dev/null ||マイピー 。
        フィ
「」---

## 8. 移行ガイド — black と isort の削除

`black` と `isort` を使用した既存のプロジェクトを変換する場合:「」バッシュ
# 1. black と isort を開発依存関係から削除する
pipアンインストールブラックアイソート

# 2. pyproject.toml から black および isort config セクションを削除します。
# [tool.black] ← このセクションを削除
# [tool.isort] ← このセクションを削除

# 3. ruff を開発依存関係に追加します (構成についてはセクション 2 を参照)

# 4. ruff format を実行して、既存のコードに互換性があることを確認する
ruff 形式 --check 。
# ruff 形式は黒と互換性があります。出力は同一である必要があります

# 5. .pre-commit-config.yaml を更新します (セクション 4 を参照)
# 黒いフックとイソソートフックを削除します。ラフおよびラフ形式のフックを追加する

# 6. CI を更新する (セクション 7 を参照)
# black、isort、flake8 ステップを削除します。 ruff チェック + ruff フォーマットを追加 --check

# 7. プリコミットフックを再インストールする
コミット前のアンインストール
プリコミットインストール
pre-commit run --all-files # クリーンであることを確認する
「」
