---
name: python-pypi-package-builder
description: 'End-to-end skill for building, testing, linting, versioning, and publishing a production-grade Python library to PyPI. Covers all four build backends (setuptools+setuptools_scm, hatchling, flit, poetry), PEP 440 versioning, semantic versioning, dynamic git-tag versioning, OOP/SOLID design, type hints (PEP 484/526/544/561), Trusted Publishing (OIDC), and the full PyPA packaging flow. Use for: creating Python packages, pip-installable SDKs, CLI tools, framework plugins, pyproject.toml setup, py.typed, setuptools_scm, semver, mypy, pre-commit, GitHub Actions CI/CD, or PyPI publishing.'
---
# Python PyPI パッケージ ビルダー スキル

ビルド、テスト、lint、バージョン管理、タイピング、および
本番グレードの Python ライブラリを PyPI に公開する — 最初のコミットからコミュニティで使用できるようになるまで
解放する。

> **AI エージェントの指示:** コードを 1 行書く前に、このファイル全体を読んでください。
> 任意のファイルを作成します。あらゆる決定 - レイアウト、バックエンド、バージョン管理戦略、パターン、CI -
> ここに決定ルールがあります。デシジョン ツリーを順番にたどってください。このスキルはどんなものにも当てはまります
> Python パッケージ タイプ (ユーティリティ、SDK、CLI、プラグイン、データ ライブラリ)。セクションをスキップしないでください。

---

## クイックナビゲーション

|このファイル内のセクション |内容 |
|---|---|
| [1.スキルトリガー](#1-スキルトリガー) |このスキルをロードするタイミング |
| [2.パッケージ タイプの決定](#2-パッケージ タイプの決定) |何を構築しているのかを特定する |
| [3.フォルダー構造の決定](#3-フォルダー構造の決定) | src/ vs flat vs monorepo |
| [4.ビルド バックエンドの決定](#4-build-backend-decion) | setuptools / 孵化したばかりの子 / フリット / 詩 |
| [5. PyPA パッケージング フロー](#5-pypa-packaging-flow) |正規のパブリッシュ パイプライン |
| [6.プロジェクト構造テンプレート](#6-プロジェクト構造-テンプレート) |すべてのオプションの完全なレイアウト |
| [7.バージョン管理戦略](#7-バージョン管理戦略) | PEP 440、セムバー、動的と静的 |

|参照ファイル |内容 |
|---|---|
| `references/pyproject-toml.md` | 4 つのバックエンド テンプレートすべて、`setuptools_scm`、`py.typed`、ツール構成 |
| `references/library-patterns.md` | OOP/SOLID、タイプヒント、コアクラス設計、ファクトリ、プロトコル、CLI |
| `references/testing-quality.md` | `conftest.py`、ユニット/バックエンド/非同期テスト、ruff/mypy/pre-commit |
| `references/ci-publishing.md` | `ci.yml`、`publish.yml`、信頼できる公開、TestPyPI、CHANGELOG、リリース チェックリスト |
| `references/community-docs.md` | README、docstrings、貢献、セキュリティ、アンチパターン、マスター チェックリスト |
| `references/architecture-patterns.md` |バックエンド システム (プラグイン/戦略)、構成層、トランスポート層、CLI、バックエンド インジェクション |
| `references/versioning-strategy.md` | PEP 440、SemVer、プレリリース、setuptools_scm の詳細、フリット静的、デシジョン エンジン |
| `references/release-governance.md` |ブランチ戦略、ブランチ保護、OIDC、タグ作成者の検証、無効なタグの防止 |
| `references/tooling-ruff.md` | Ruff のみのセットアップ (black/isort を置き換える)、mypy config、pre-commit、asyncio_mode=auto |

**足場スクリプト:** `python skills/python-pypi-package-builder/scripts/scaffold.py --name your-package-name` を実行します
1 つのコマンドでディレクトリ レイアウト全体、スタブ ファイル、および `pyproject.toml` を生成します。

---

## 1. スキルトリガー

ユーザーが希望するときはいつでも、このスキルをロードします。- Python パッケージまたはライブラリを作成、スキャフォールディング、または PyPI に公開する
- pip インストール可能な SDK、ユーティリティ、CLI ツール、またはフレームワーク拡張機能を構築する
- Python プロジェクトの `pyproject.toml`、lint、mypy、pre-commit、または GitHub アクションをセットアップする
- バージョン管理を理解する (`setuptools_scm`、PEP 440、semver、静的バージョン管理)
- PyPA 仕様を理解する: `py.typed`、`MANIFEST.in`、`RECORD`、分類子
- 信頼された公開 (OIDC) または API トークンを使用して PyPI に公開する
- 最新の Python パッケージ標準に準拠するように既存のパッケージをリファクタリングします
- タイプヒント、プロトコル、ABC、またはデータクラスを Python ライブラリに追加します
- OOP/SOLID デザイン パターンを Python パッケージに適用する
- ビルド バックエンド (setuptools、hatchling、flit、poetry) から選択します。

**次のようなフレーズでもトリガーされます:** 「Python SDK をビルドする」、「ライブラリを公開する」、「PyPI CI をセットアップする」、
"pip パッケージを作成する"、"PyPI に公開するにはどうすればよいですか"、"pyproject.toml ヘルプ"、"PEP 561 の入力"、
「setuptools_scm バージョン」、「semver Python」、「PEP 440」、「git タグ リリース」、「信頼された公開」。

---

## 2. パッケージタイプの決定

コードを記述する前に、ユーザーが何を構築しているかを特定します。それぞれのタイプには異なるパターンがあります。

### デシジョンテーブル

|タイプ |コアパターン |エントリーポイント |キーデプス |パッケージ例 |
|---|---|---|---|---|
| **ユーティリティ ライブラリ** |純粋関数 + ヘルパーのモジュール |インポート API のみ |最小限 | `arrow`、`humanize`、`boltons`、`more-itertools` |
| **API クライアント / SDK** |メソッド、認証、再試行ロジックを備えたクラス |インポート API のみ | `httpx` または `requests` | `boto3`、`stripe-python`、`openai` |
| **CLI ツール** |コマンド関数 + 引数パーサー | `[project.scripts]` または `[project.entry-points]` | `click` または `typer` | `black`、`ruff`、`httpie`、`rich` |
| **フレームワーク プラグイン** |プラグインクラス、フック登録 | `[project.entry-points."framework.plugin"]` |フレームワーク開発 | `pytest-*`、`django-*`、`flask-*` |
| **データ処理ライブラリ** |クラス + 機能パイプライン |インポート API のみ |オプション: `numpy`、`pandas` | `pydantic`、`marshmallow`、`cerberus` |
| **混合/汎用** |上記の組み合わせ |さまざま |さまざま |多くの現実世界のパッケージ |

**決定ルール:** 不明な場合はユーザーに質問してください。パッケージはタイプを組み合わせることができます (例: SDK と CLI)
エントリ ポイント) — 構造上の決定に主タイプを使用し、その上に副タイプ パターンを追加します。

各タイプの実装パターンについては、`references/library-patterns.md` を参照してください。

### パッケージの命名規則

- PyPI 名: すべて小文字、ハイフン — `my-python-library`
- Python インポート名: アンダースコア — `my_python_library`
- 開始前に利用可能かどうかを確認してください: https://pypi.org/search/
- 人気のあるパッケージのシャドウイングを避ける (最初に `pip install <name>` が失敗することを確認する)

---

## 3. フォルダー構造の決定

### デシジョン ツリー「」
パッケージには 5 つ以上の内部モジュール、または複数のコントリビュータ、または複雑なサブパッケージが含まれていますか?
§── YES → src/layout を使用
│ 理由: 開発中にアンインストールされたコードが誤ってインポートされるのを防ぎます。
│ ソースをプロジェクトのルート ファイルから分離します。 PyPA - 大規模プロジェクトに推奨。
│
§── いいえ → それは単一モジュールに焦点を当てたパッケージ (例: 1 つのファイル + ヘルパー) ですか?
│ §── YES → フラットレイアウトを使用する
│ └── NO (中程度の複雑さ) → フラット レイアウトを使用し、大きくなった場合は src/ に移行します
│
━── 1 つの名前空間 (myorg.http、myorg.db など) に複数の関連パッケージがありますか?
          └── YES → ネームスペース/モノリポジトリレイアウトを使用
「」### 簡単なルールの概要

|状況 |使用 |
|---|---|
|新しいプロジェクト、将来の規模は不明 | `src/` レイアウト (最も安全なデフォルト) |
|単一目的、1 ～ 4 モジュール |フラットレイアウト |
|大規模なライブラリ、多くの寄稿者 | `src/` レイアウト |
| 1 つのリポジトリ内の複数のパッケージ |名前空間 / モノリポジトリ |
|古いフラット プロジェクトの移行 |平らに保ちます。次のメジャー バージョンで `src/` に移行 |

---

## 4. バックエンドの構築に関する決定

### デシジョン ツリー「」
ユーザーは git タグから自動的に派生したバージョンを必要としますか?
§── YES → setuptools + setuptools_scm を使用する
│ (git tag v1.0.0 → それがリリースワークフローです)
│
└── いいえ → ユーザーはオールインワン ツール (deps + build + public) を望んでいますか?
          §── はい → 詩を使用します (v2+ は標準の [プロジェクト] テーブルをサポートします)
          │
          └── いいえ → パッケージは C 拡張機能のない純粋な Python ですか?
                    §── はい、最小限の構成が望ましい → flit を使用します
                    │ (設定不要、__version__ からバージョンを自動検出)
                    │
                    └── はい、モダンで速いものが望ましい → 孵化したばかりの子を使用する
                        (ゼロ構成、プラグイン システム、setup.py は不要)

パッケージには C/Cython/Fortran 拡張機能が含まれていますか?
└── はい → setuptools を使用しなければなりません (完全なネイティブ拡張機能をサポートするバックエンドのみ)
「」### バックエンドの比較

|バックエンド |バージョンソース |構成 | C 拡張機能 |こんな方に最適 |
|---|---|---|---|---|
| `setuptools` + `setuptools_scm` | git タグ (自動) | `pyproject.toml` + オプションの `setup.py` シム |はい | gittag リリースのあるプロジェクト。あらゆる複雑さ |
| `hatchling` |マニュアルまたはプラグイン | `pyproject.toml` のみ |いいえ |新しい純粋な Python プロジェクト。速く、モダン |
| `flit` | `__version__` `__init__.py` | `pyproject.toml` のみ |いいえ |非常にシンプルな単一モジュールのパッケージ |
| `poetry` | `pyproject.toml` フィールド | `pyproject.toml` のみ |いいえ |統合された Dep 管理を必要とするチーム |

4 つの完全な `pyproject.toml` テンプレートすべてについては、`references/pyproject-toml.md` を参照してください。

---

## 5. PyPA のパッケージ化フロー

これは、ソース コードからユーザーのインストールまでの正規のエンドツーエンド フローです。
**公開する前に、すべての手順を理解する必要があります。**「」
1. ソースツリー
   バージョン管理 (git) 内のコード
   └── pyproject.toml にはメタデータ + ビルドシステムが記述されています

2. ビルド
   Python -m ビルド
   └── dist/ に 2 つのアーティファクトを生成します。
       §── *.tar.gz → ソース配布(sdist)
       └── *.whl → ビルドされたディストリビューション (ホイール) — pip が推奨

3. 検証する
   麻ひものチェック距離/*
   └── メタデータ、README レンダリング、および PyPI 互換性をチェックします

4. テストパブリッシュ (初回リリースのみ)
   ひもアップロード --repository testpypi dist/*
   └── 確認: pip install --index-url https://test.pypi.org/simple/ your-package

5. 公開する
   Twine アップロード dist/* ← 手動フォールバック
   または GitHub アクションの public.yml ← 推奨 (信頼された公開 / OIDC)

6. ユーザーによるインストール
   pip パッケージをインストールします
   pip install "your-package[extra]"
「」### PyPA の重要な概念

|コンセプト |意味 |
|---|---|
| **sdist** |ソースの配布 - ソース + メタデータ。使用可能なホイールがない場合に使用されます。
| **ホイール (.whl)** |事前に構築されたバイナリ — pip はサイトパッケージに直接抽出します。ビルドステップがありません |
| **PEP 517/518** | `pyproject.toml [build-system]` テーブル経由の標準ビルド システム インターフェイス |
| **PEP 621** | `pyproject.toml` の標準 `[project]` テーブル。最新のバックエンドはすべてサポートしています。
| **PEP 639** | `{text = "MIT"}` ではなく、SPDX 文字列としての `license` キー (例: `"MIT"`、`"Apache-2.0"`)
| **PEP 561** | `py.typed` 空のマーカー ファイル — このパッケージには型情報が含まれることを mypy/IDE に伝えます。

完全な CI ワークフローと公開設定については、`references/ci-publishing.md` を参照してください。

---

## 6. プロジェクト構造テンプレート

### A. src/ Layout (新しいプロジェクトの推奨デフォルト)「」
あなたのパッケージ/
§── src/
│ └── your_package/
│ §── __init__.py # パブリック API: __all__、__version__
│ §── py.typed # PEP 561 マーカー — 空のファイル
│ §── core.py # 一次実装
│ §── client.py # (API クライアントの種類) または削除
│ §── cli.py # (CLI タイプ) click/typer コマンド、または削除
│ §── config.py # 設定/構成データクラス
│ §──Exceptions.py # カスタム例外階層
│ §── models.py # データクラス、Pydantic モデル、TypedDicts
│ §── utils.py # 内部ヘルパー (プライベートの場合は接頭辞 _utils)
│ §── types.py # 共有型エイリアスと TypeVars
│ └── backends/ # (プラグイン パターン) — 不要な場合は削除します
│ §── __init__.py # プロトコル/ABCインターフェース定義
│ §──memory.py # デフォルトのゼロデプ実装
│ └── redis.py # オプションの重い実装
§── テスト/
│ §── __init__.py
│ §── conftest.py # 共有フィクスチャ
│ §── 単位/
│ │ §── __init__.py
│ │ §── test_core.py
│ │ §── test_config.py
│ │ └─ test_models.py
│ §── 統合/
│ │ §── __init__.py
│ │ └─ test_backends.py
│ └─ e2e/ # オプション: エンドツーエンドのテスト
│ └── __init__.py
§── docs/ # オプション: mkdocs または sphinx
§── スクリプト/
│ └── scaffold.py
§── .github/
│ §── ワークフロー/
│ │ §── ci.yml
│ │ └──publish.yml
│ └── ISSUE_TEMPLATE/
│ §── bug_report.md
│ └── feature_request.md
§── .pre-commit-config.yaml
§── pyproject.toml
§── CHANGELOG.md
§── 投稿.md
§── SECURITY.md
§── ライセンス
§── README.md
└── .gitignore
「」### B. フラット レイアウト (小規模/集中パッケージ)「」
あなたのパッケージ/
§── your_package/ # ← src/ 内ではなくルートにあります
│ §── __init__.py
│ §── py.typed
│ └── ... (内部構造は同じ)
§── テスト/
└── ... (同じ最上位ファイル)
「」### C. 名前空間 / Monorepo レイアウト (複数の関連パッケージ)「」
あなたの組織/
§── パッケージ/
│ §── your-org-core/
│ │ §── src/your_org/core/
│ │ └─ pyproject.toml
│ §── your-org-http/
│ │ §── src/your_org/http/
│ │ └─ pyproject.toml
│ └── your-org-cli/
│ §── src/your_org/cli/
│ └── pyproject.toml
§── .github/workflows/
━── README.md
「」各サブパッケージには独自の `pyproject.toml` があります。これらは PEP 420 を介して `your_org` 名前空間を共有します
暗黙的な名前空間パッケージ (名前空間ルートに `__init__.py` がありません)。

### 内部モジュールのガイドライン

|ファイル |目的 |いつ含めるか |
|---|---|---|
| `__init__.py` |パブリック API サーフェス。再輸出。 `__version__` |常に |
| `py.typed` | PEP 561 型付きパッケージ マーカー (空) |常に |
| `core.py` |プライマリクラス / メインロジック |常に |
| `config.py` |設定データクラスまたは Pydantic モデル |構成可能な場合 |
| `exceptions.py` |例外階層（`YourBaseError`→詳細） |常に |
| `models.py` |データ モデル / DTO / TypedDicts |データ量が多い場合 |
| `utils.py` |内部ヘルパー (パブリック API の一部ではありません) |必要に応じて |
| `types.py` | `TypeVar`、`TypeAlias`、`Protocol` の定義を共有 |複雑な入力の場合 |
| `cli.py` | CLI エントリ ポイント (クリック/タイプ) | CLI タイプのみ |
| `backends/` |プラグイン/戦略パターン |スワップ可能な実装の場合 |
| `_compat.py` | Python バージョン互換性シム | 3.9 ～ 3.13 の互換性が必要な場合 |

---

## 7. バージョン管理戦略

### PEP 440 — スタンダード「」
正規形式: N[.N]+[{a|b|rc}N][.postN][.devN]

例:
  1.0.0 安定版リリース
  1.0.0a1 アルファ (プレリリース)
  1.0.0b2 ベータ版
  1.0.0rc1 リリース候補
  1.0.0.post1 リリース後 (例: パッケージ修正のみ)
  1.0.0.dev1 開発スナップショット (PyPI 用ではありません)
「」### セマンティック バージョニング (推奨)「」
メジャーパッチ、マイナーパッチ

メジャー: API の重大な変更 (パブリック関数/クラス/引数の削除/名前変更)
マイナー: 新機能、完全な下位互換性
パッチ: バグ修正、API 変更なし
「」### setuptools_scm を使用した動的バージョン管理 (git-tag ワークフローに推奨)「」バッシュ
# 仕組み:
git tag v1.0.0 → インストールされているバージョン = 1.0.0
git tag v1.1.0 → インストールされているバージョン = 1.1.0
(タグの後にコミット) → バージョン = 1.1.0.post1 (PyPI 用にサフィックスを削除)

# コード内 — setuptools_scm を使用する場合は決してハードコードしないでください。
importlib.metadata インポート バージョンから、PackageNotFoundError
試してみてください:
    __version__ = version("あなたのパッケージ")
PackageNotFoundError を除く:
    __version__ = "0.0.0-dev" # アンインストールされた開発チェックアウトのフォールバック
「」必須の `pyproject.toml` 構成:```トムル
[tool.setuptools_scm]
version_scheme = "リリース後"
local_scheme = "no-local-version" # +g<hash> による PyPI アップロードの中断を防止します
「」**重要:** すべての CI チェックアウト ステップで常に `fetch-depth: 0` を設定します。完全な Git 履歴がないと、
`setuptools_scm` はタグを見つけることができず、ビルド バージョンはサイレントに `0.0.0+dev` にフォールバックします。

### 静的バージョン管理 (フリット、孵化マニュアル、詩)「」パイソン
# your_package/__init__.py
__version__ = "1.0.0" # リリースごとにこれを更新します
「」### 依存関係に関するバージョン指定子のベスト プラクティス```トムル
# [プロジェクト] 依存関係内:
"httpx>=0.24" # 最小バージョン - ライブラリに推奨
"httpx>=0.24,<1.0" # 既知の重大な変更が存在する場合のみ上限
"httpx==0.27.0" # ライブラリではなくアプリケーション内でのみピン留めします

# これはライブラリでは絶対に行わないでください。ユーザーの依存関係の解決が中断されます。
# "httpx~=0.24.0" # きつすぎます
# "httpx==0.27.*" # 壊れやすい
「」### バージョンアップ→リリースの流れ「」バッシュ
# 1. CHANGELOG.md を更新します — [未リリース] エントリを [x.y.z] - YYYY-MM-DD に移動します
# 2. 変更ログをコミットする
git add CHANGELOG.md
git commit -m "雑務: リリース vX.Y.Z の準備"
# 3. タグ付けしてプッシュ - これにより、publish.yml が自動的にトリガーされます
git タグ vX.Y.Z
git Push Origin main --tags
# 4. GitHub アクションを監視する → https://pypi.org/project/your-package/ で確認する
「」4 つすべてのバックエンドの完全な pyproject.toml テンプレートについては、`references/pyproject-toml.md` を参照してください。

---

## 次にどこへ行くか

決定と構造を理解した後:

1. **`pyproject.toml`** → `references/pyproject-toml.md` を設定します
   4 つのバックエンド テンプレート (setuptools+scm、hatchling、flit、poetry)、完全なツール構成、
   `py.typed` セットアップ、バージョン管理設定。

2. **ライブラリ コードを記述します** → `references/library-patterns.md`
   OOP/SOLID 原則、型ヒント (PEP 484/526/544/561)、コア クラス設計、ファクトリ関数、
   `__init__.py`、プラグイン/バックエンド パターン、CLI エントリ ポイント。

3. **テストとコード品質を追加** → `references/testing-quality.md`
   `conftest.py`、ユニット/バックエンド/非同期テスト、パラメータ化、ruff/mypy/pre-commit セットアップ。

4. **CI/CD を設定して公開** → `references/ci-publishing.md`
   `ci.yml`、`publish.yml`、信頼された発行 (OIDC、API トークンなし)、CHANGELOG 形式、
   リリースチェックリスト。

5. **コミュニティ/OSS 用のポーランド語** → `references/community-docs.md`
   README セクション、docstring 形式、CONTRIBUTING、SECURITY、問題テンプレート、アンチパターン
   表とマスターリリースチェックリスト。

6. **バックエンド、構成、トランスポート、CLI の設計** → `references/architecture-patterns.md`
   バックエンド システム (プラグイン/戦略パターン)、設定データクラス、HTTP トランスポート層、
   クリック/タイパー、バックエンド インジェクション ルールを備えた CLI。

7. **バージョン管理戦略の選択と実装** → `references/versioning-strategy.md`
   PEP 440 正規形式、SemVer ルール、プレリリース識別子、setuptools_scm の詳細、
   flit 静的バージョン管理、意思決定エンジン (DEFAULT/BEGINNER/MINIMAL)。

8. **リリースを管理し、公開パイプラインを保護** → `references/release-governance.md`
   ブランチ戦略、ブランチ保護ルール、OIDC Trusted Publishing セットアップ、タグ作成者
   CI での検証、タグ形式の強制、完全に管理された `publish.yml`。

9. **Ruff を使用してツールを簡素化** → `references/tooling-ruff.md`
   black/isort/flake8、mypy config、pre-commit フックを置き換える Ruff のみのセットアップ、
   asyncio_mode=auto (@pytest.mark.asyncio を削除)、移行ガイド。