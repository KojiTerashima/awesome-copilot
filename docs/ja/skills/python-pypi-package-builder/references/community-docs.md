# コミュニティ ドキュメント、PR チェックリスト、アンチパターン、リリース チェックリスト

## 目次
1. [README.md 必須セクション](#1-readmemd-required-sections)
2. [ドキュメント文字列 — Google スタイル](#2-docstrings--google-style)
3. [CONTRIBUTING.md テンプレート](#3-contributingmd)
4. [SECURITY.mdテンプレート](#4-securitymd)
5. [GitHub 問題テンプレート](#5-github-issue-templates)
6. [PR チェックリスト](#6-pr-チェックリスト)
7. [避けるべきアンチパターン](#7-避けるべきアンチパターン)
8. [マスターリリースチェックリスト](#8-マスターリリースチェックリスト)

---

## 1. `README.md` 必須セクション

優れた README は、採用にとって最も重要なファイルです。ユーザーは 30 秒以内に次のことを決定します。
README に基づいてライブラリを使用します。```マークダウン
# あなたのパッケージ

> 1 行の説明 — それが何をするのか、そしてなぜそれが役立つのか。

[![PyPI バージョン](https://badge.fury.io/py/your-package.svg)](https://pypi.org/project/your-package/)
[![Python バージョン](https://img.shields.io/pypi/pyversions/your-package)](https://pypi.org/project/your-package/)
[![CI](https://github.com/you/your-package/actions/workflows/ci.yml/badge.svg)](https://github.com/you/your-package/actions/workflows/ci.yml)
[![カバレッジ](https://codecov.io/gh/you/your-package/branch/master/graph/badge.svg)](https://codecov.io/gh/you/your-package)
[![ライセンス: MIT](https://img.shields.io/badge/License-MIT- yellow.svg)](ライセンス)

## インストール

pip パッケージをインストールします

# Redis バックエンドの場合:
pip install "your-package[redis]"

## クイックスタート

(コピー＆ペーストの動作例 - 実行するためのセットアップは必要ありません)

your_package から YourClient をインポート

client = YourClient(api_key="sk-...")
result = client.process({"入力": "値"})
印刷(結果)

## 特徴

- 特徴1
- 特徴2

## 構成

|パラメータ |タイプ |デフォルト |説明 |
|---|---|---|---|
| APIキー | str |必須 |認証資格情報 |
|タイムアウト |整数 | 30 |リクエストのタイムアウト (秒) |
|再試行 |整数 | 3 |再試行回数 |

## バックエンド

簡単な比較 (インメモリと Redis)、およびそれぞれをいつ使用するか。

## 貢献する

[CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください。

## 変更履歴

[CHANGELOG.md](./CHANGELOG.md) を参照してください。

## ライセンス

MIT — [ライセンス](./LICENSE) を参照
「」---

## 2. ドキュメント文字列 — Google スタイル

すべてのパブリック クラス、メソッド、関数に Google スタイルの docstring を使用します。 IDE はこれらを表示します
mkdocs/sphinx はツールチップとしてドキュメントを自動生成でき、意図を伝えます。
明らかに貢献者に。「」パイソン
クラス YourClient:
    「」
    <目的> のメインクライアント。

    引数:
        api_key: 認証資格情報。
        timeout: リクエストのタイムアウト (秒単位)。デフォルトは 30 です。
        retries: 再試行の回数。デフォルトは 3 です。

    発生するもの:
        ValueError: api_key が空であるか、タイムアウトが正でない場合。

    例:
        >>> your_package から YourClient をインポート
        >>> client = YourClient(api_key="sk-...")
        >>> 結果 = client.process({"入力": "値"})
    「」
「」---

## 3. `CONTRIBUTING.md````マークダウン
# パッケージに貢献する

## 開発セットアップ

git clone https://github.com/you/your-package
あなたのパッケージをCD化する
pip install -e ".[dev]"
プリコミットインストール

## テストの実行

pytest

## リンティングの実行

ラフチェック。
黒 。 --チェック
mypy your_package/

## PR を送信する

1. リポジトリをフォークする
2. 機能ブランチを作成します: `git checkout -b feat/your-feature`
3. テストで変更を加える
4. CI が合格することを確認します: `pre-commit run --all-files && pytest`
5. `[Unreleased]` の下の `CHANGELOG.md` を更新します
6. PR を開きます — PR テンプレートを使用します

## コミットメッセージの形式 (従来のコミット)

- @@コード4@@
- `fix: correct retry behavior on timeout`
- `docs: update README quick start`
- @@コード7@@
- `test: add edge cases for memory backend`

## バグの報告

GitHub の問題テンプレートを使用します。 Python のバージョン、パッケージのバージョン、
そして最小限の再現可能な例。
「」---

## 4. `SECURITY.md````マークダウン
# セキュリティポリシー

## サポートされているバージョン

|バージョン |サポートされている |
|---|---|
| 1.x.x |はい |
| < 1.0 |いいえ |

## 脆弱性の報告

セキュリティの脆弱性について GitHub でパブリックな問題を開かないでください。

レポート経由: GitHub プライベート セキュリティ レポート (推奨)
または電子メール: security@yourdomain.com

含める:
- 脆弱性の説明
- 再現手順
- 潜在的な影響
- 提案された修正 (ある場合)

48 時間以内に確認し、14 日以内に解決することを目指しています。
「」---

## 5. GitHub の問題テンプレート

### `.github/ISSUE_TEMPLATE/bug_report.md````マークダウン
---
名前: バグレポート
about: 再現可能なバグを報告する
ラベル: バグ
---

**Python バージョン:**
**パッケージバージョン:**

**バグの説明:**

**再現可能な最小限の例:**
「」パイソン
# ここにコードを貼り付けます「」

**期待される動作:**

**実際の動作:**
「」### `.github/ISSUE_TEMPLATE/feature_request.md````マークダウン
---
名前: 機能リクエスト
about: 新しい機能または拡張機能を提案する
ラベル:拡張機能
---

**これで解決できる問題:**

**提案された解決策:**

**考慮される代替案:**
「」---

## 6. PR チェックリスト

レビューをリクエストする前に、すべての項目を確認する必要があります。 CI は完全に緑色でなければなりません。

### コード品質ゲート「」
【】ラフチェック。 — エラーゼロ
[ ] 黒。 --check — フォーマットの問題はゼロ
[ ] 等。 --check-only — インポートは正しくソートされます
[ ] mypy your_package/ — ゼロタイプエラー
[ ] pytest — すべてのテストに合格します
[ ] カバレッジ >= 80% (pyproject.toml のfail_under によって強制)
[ ] すべての GitHub Actions ワークフローが緑色
「」＃＃＃ 構造「」
[ ] pyproject.toml: 名前、動的/バージョン、説明、Python が必要、ライセンス、作成者、
    キーワード (10 個以上)、分類子、依存関係、すべての [project.url] が入力されました
setuptools_scm を使用する場合 [ ] Dynamic = ["version"]
[ ] [tool.setuptools_scm] with local_scheme = "no-local-version"
[ ] setup.py シムが存在します (setuptools_scm を使用している場合)
[ ] py.typed マーカー ファイルがパッケージ ディレクトリに存在します (空のファイル)
[ ] py.typed は [tool.setuptools.package-data] にリストされています
[ ] pyproject.toml の「Typing :: Typed」分類子
[ ] __init__.py にはすべてのパブリック シンボルをリストする __all__ があります
[ ] importlib.metadata 経由の __version__ (ハードコードされた文字列ではない)
「」### テスト「」
[ ] conftest.py にはクライアントとバックエンドの共有フィクスチャがあります
[ ] テスト済みのコア ハッピー パス
[ ] テスト済みのエラー条件とエッジケース
[ ] 各バックエンドは個別に個別にテストされました
[ ] Redis バックエンドは Redis サービスを使用した別の CI ジョブでテストされました (該当する場合)
[ ] asyncio_mode = pyproject.toml の "auto" (非同期テスト用)
[ ] fetch- Depth: すべての CI チェックアウト ステップで 0
「」### オプションのバックエンド (該当する場合)「」
[ ] BaseBackend 抽象クラスはインターフェイスを定義します
[ ] MemoryBackend は追加の DEPS なしで動作します
[ ] RedisBackend は、Redis がインストールされていない場合、明確な pip インストール ヒントとともに ImportError を発生させます
[ ] 両方のバックエンドが個別に単体テスト済み
[ ] [project.optional-dependency] で宣言された redis extra
[ ] README には両方のインストール パス (base と [redis]) が示されています
「」### 変更履歴とドキュメント「」
[ ] CHANGELOG.md が [未公開] で更新されました
[ ] README には、説明、インストール、クイック スタート、構成テーブル、バッジ、ライセンスが含まれています
[ ] すべての公開シンボルには Google スタイルの docstring が含まれます
[ ] CONTRIBUTING.md: 開発セットアップ、テスト/lint コマンド、PR 命令
[ ] SECURITY.md: サポートされているバージョン、報告プロセス
[ ] .github/ISSUE_TEMPLATE/bug_report.md
[ ] .github/ISSUE_TEMPLATE/feature_request.md
「」###CI/CD「」
[ ] ci.yml: lint + mypy + テスト行列 (サポートされているすべての Python バージョン)
[ ] ci.yml: Redis サービスを使用した Redis バックエンドの別のジョブ
[ ] public.yml: v*.*.* タグでトリガーされ、信頼された公開 (OIDC) を使用します。
[ ] フェッチ深度: すべてのワークフロー チェックアウト ステップで 0
[ ] GitHub リポジトリの [設定] → [環境] で作成された pypi 環境
[ ] リポジトリ シークレットに API トークンがありません
「」---

## 7. 避けるべきアンチパターン

|アンチパターン |なぜダメなのか |正しいアプローチ |
|---|---|---|
| `__version__ = "1.0.0"` setuptools_scm でハードコーディング |最初の git タグの後は古くなります | `importlib.metadata.version()` を使用します。
| CI チェックアウトに `fetch-depth: 0` がありません | setuptools_scm でタグが見つかりません → version = `0.0.0+dev` | `fetch-depth: 0` を **すべて** チェックアウト ステップに追加します。
| `local_scheme` が設定されていません | `+g<hash>` サフィックスにより PyPI アップロードが中断される (ローカル バージョンは拒否される) | `local_scheme = "no-local-version"` |
| `py.typed` ファイルがありません | IDE と mypy はパッケージを入力どおりに認識しません。パッケージ root | に空の `py.typed` を作成します。
| `py.typed` は `package-data` にありません |インストールされたホイールにファイルがありません - 役に立たない | `[tool.setuptools.package-data]` に追加 |
|モジュール先頭でオプションの dep をインポート | `ImportError` 上の `import your_package` すべてのユーザー向け |それを必要とする関数/クラス内の遅延インポート |
| `setup.py` でメタデータを複製しています | `pyproject.toml` と競合します。ドリフト | `setup.py` を 3 行のシムのみとして保持します。
|カバレッジ設定に `fail_under` がありません |カバレッジの後退は気づかれない | `fail_under = 80` を設定 |
| CI に mypy はありません |タイプエラーは静かに蓄積されます。 mypy ステップを `ci.yml` に追加 |
| GitHub Secrets for PyPI の API トークン |セキュリティリスク、ローテーションの負担 |信頼できる発行 (OIDC) を使用する |
| `main`/`master` に直接コミットする | CI チェックをバイパスします。 `no-commit-to-branch` コミット前フックを介して強制する |
| CHANGELOG に `[Unreleased]` セクションがありません |変更が積み重なり、リリース時には忘れ去られる | `[Unreleased]` を PR ごとに更新する |
|正確な dep バージョンをライブラリに固定する |ユーザーの依存関係の解決を中断します。 `>=` 下限のみを使用してください。 `==` は避けてください。
| `__init__.py` に `__all__` はありません |ユーザーが誤って内部ヘルパーをインポートする可能性があります。すべてのパブリック シンボルで `__all__` を宣言します。
| `from your_package import *` テスト中 |インポートが壊れていてもテストは合格します。常に明示的なインポートを使用してください。
|いいえ `SECURITY.md` |責任ある脆弱性開示の道はない |応答タイムラインを含むファイルを追加 |
| `Any` 型ヒント内のあらゆる場所 | mypy を完全に破る |真に任意の値には `object` を使用します。
| `Union` 戻り値の型 |すべての呼び出し元に `isinstance()` チェックの書き込みを強制します。具体的な型を返します。オーバーロードを使用する |
| `setup.cfg` + `pyproject.toml` 両方ともアクティブ |コントリビューターにとっての矛盾と混乱 |すべてを `pyproject.toml` に移行する |
|タグなしのコミットでのリリース |バージョン番号は意味がありません |リリース前に必ずタグ付けする |
|サポートされているすべての Python バージョンでテストしていない |破損はあなたではなくユーザーによって発見されました | CI でのマトリックス テスト |
| `license = {text = "MIT"}` (旧形式) |廃止されました。 PEP 639 は SPDX 文字列を使用します。 `license = "MIT"` |
|問題のテンプレートはありません |バグレポートに一貫性がない | `bug_report.md` + `feature_request.md` を追加 |

---

## 8. マスターリリースチェックリスト

リリースタグを押す前に、すべての項目を確認してください。 CI は完全に緑色でなければなりません。

### コードの品質「」
【】ラフチェック。 — エラーゼロ
[ ] ラフ形式。 --check — フォーマットの問題はゼロ
[ ] mypy src/your_package/ — ゼロタイプのエラー
[ ] pytest — すべてのテストに合格します
[ ] カバレッジ >= 80% (pyproject.toml で強制的に fail_under)
[ ] すべての GitHub Actions CI ジョブは緑色 (lint + テスト マトリックス)
「」### プロジェクトの構造「」
[ ] pyproject.toml — 名前、説明、Requires-Python、ライセンス (SPDX 文字列)、作成者、
    キーワード (10 以上)、分類子 (Python バージョン + Typing :: Typed)、URL (5 つのフィールドすべて)
[ ] 動的 = ["バージョン"] セット (setuptools_scm または hatch-vcs を使用する場合)
[ ] [tool.setuptools_scm] with local_scheme = "no-local-version"
[ ] setup.py シムが存在します (setuptools_scm を使用している場合)
[ ] py.typed マーカー ファイルが存在します (パッケージ ルートに空のファイル)
[ ] py.typed は [tool.setuptools.package-data] にリストされています
[ ] pyproject.toml の「Typing :: Typed」分類子
[ ] __init__.py にはすべてのパブリック シンボルをリストする __all__ があります
[ ] __version__ は importlib.metadata から読み取ります (ハードコードされていません)
「」### テスト「」
[ ] conftest.py にはクライアントとバックエンドの共有フィクスチャがあります
[ ] テスト済みのコア ハッピー パス
[ ] テスト済みのエラー条件とエッジケース
[ ] 各バックエンドは個別に個別にテストされました
[ ] asyncio_mode = pyproject.toml の "auto" (非同期テスト用)
[ ] fetch- Depth: すべての CI チェックアウト ステップで 0
「」### 変更履歴とドキュメント「」
[ ] CHANGELOG.md: [未リリース] エントリが [x.y.z] に移動されました - YYYY-MM-DD
[ ] README には、説明、インストール コマンド、クイック スタート、構成テーブル、バッジが含まれています
[ ] すべての公開シンボルには Google スタイルの docstring が含まれます
[ ] CONTRIBUTING.md: 開発セットアップ、テスト/lint コマンド、PR 命令
[ ] SECURITY.md: サポートされているバージョン、タイムラインを含むレポート プロセス
「」### バージョン管理「」
[ ] すべての CI チェックは、タグ付けする予定のコミットに渡されます
[ ] CHANGELOG.md が更新され、コミットされました
[ ] Git タグは形式 v1.2.3 に従います (semver、v プレフィックス)
[ ] ビルドされたホイール名に古い local_scheme サフィックスは表示されません
「」###CI/CD「」
[ ] ci.yml: lint + mypy + テスト行列 (サポートされているすべての Python バージョン)
[ ] public.yml: v*.*.* タグでトリガーされ、信頼された公開 (OIDC) を使用します。
[ ] GitHub リポジトリの [設定] → [環境] で作成された pypi 環境
[ ] リポジトリ シークレットに API トークンが保存されていません
「」### リリースコマンドシーケンス「」バッシュ
# 1. 完全なローカル検証を実行する
ラフチェック。 ;ラフフォーマット。 - チェック ; mypy src/your_package/ ; pytest

# 2. CHANGELOG.md を更新します — [未リリース] を [x.y.z] に移動します
# 3. 変更ログをコミットする
git add CHANGELOG.md
git commit -m "雑務: リリース vX.Y.Z の準備"

# 4. タグ付けしてプッシュ - これにより、publish.yml が自動的にトリガーされます
git タグ vX.Y.Z
git Push Origin main --tags

# 5. モニター: https://github.com/<you>/<pkg>/actions
# 6. 確認: https://pypi.org/project/your-package/
「」
