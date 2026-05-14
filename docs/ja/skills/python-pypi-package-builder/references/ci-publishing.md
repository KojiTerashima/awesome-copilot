# CI/CD、公開、および変更ログ

## 目次
1. [変更ログ形式](#1-変更ログ形式)
2. [ci.yml — lint、型チェック、テストマトリックス](#2-ciyml)
3. [publish.yml — バージョンタグでトリガー](#3-publishyml)
4. [PyPI Trusted Publishing (API トークンなし)](#4-pypi-trusted-publishing)
5. [手動公開フォールバック](#5-manual-publish-fallback)
6. [リリースチェックリスト](#6-リリースチェックリスト)
7. [py.typed の船が操舵室にあることを確認](#7-verify-pytyped-ships-in-the-wheel)
8. [サーバー変更タイプガイド](#8-semver-change-type-guide)

---

## 1. 変更ログの形式

[変更ログを保持する](https://keepachangelog.com/) の規則に従って `CHANGELOG.md` を保持します。
すべての PR は `[Unreleased]` セクションを更新する必要があります。リリースする前に、これらのエントリを
新しいバージョンのセクションに日付が記載されています。```マークダウン
# 変更履歴

このプロジェクトに対するすべての重要な変更は、このファイルに文書化されます。

形式は[変更ログを保持する](https://keepachangelog.com/ja/1.1.0/)に基づいています。
そしてこのプロジェクトは [セマンティック バージョニング](https://semver.org/spec/v2.0.0.html) に準拠しています。

---

## [未公開]

### 追加
- (進行中の機能はここにあります)

---

## [1.0.0] - 2026-04-02

### 追加
- 初期の安定版リリース
- `YourMiddleware` 段階的モード、厳密モード、複合モード
- インメモリ バックエンド (追加の DEP なし)
- オプションの Redis バックエンド (`pip install pkg[redis]`)
- `Depends(RouteThrottle(...))` によるルートごとのオーバーライド
- `py.typed` マーカー — PEP 561 型付きパッケージ
- GitHub アクション CI: lint、mypy、テスト マトリックス、信頼できる公開

### 変更されました
### 修正済み
### 削除されました

---

## [0.1.0] - 2026-03-01

### 追加
- 初期プロジェクトの足場

[未リリース]: https://github.com/you/your-package/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/you/your-package/compare/v0.1.0...v1.0.0
[0.1.0]: https://github.com/you/your-package/releases/tag/v0.1.0
「」### Semver — 何が何を衝突させるのか

|タイプの変更 |バンプ |例 |
|---|---|---|
| API の重大な変更 |メジャー | `1.0.0 → 2.0.0` |
|新機能、下位互換性 |マイナー | `1.0.0 → 1.1.0` |
|バグ修正 |パッチ | `1.0.0 → 1.0.1` |

---

## 2. `ci.yml`

すべてのプッシュおよびプル リクエストで実行されます。サポートされているすべての Python バージョンにわたってテストします。```ヤムル
# .github/workflows/ci.yml
名前：CI

に:
  プッシュ：
    ブランチ: [メイン、マスター]
  プルリクエスト:
    ブランチ: [メイン、マスター]

仕事:
  糸くず:
    名前: lint、フォーマットと型のチェック
    実行: ubuntu-最新
    手順:
      - 使用:actions/checkout@v4
      - 使用:actions/setup-python@v5
        と:
          Python バージョン: "3.11"
      - 名前: 開発依存関係のインストール
        実行: pip install -e ".[dev]"
      - 名前：ラフ・リント
        実行：ラフチェック。
      - 名前: ruff フォーマットチェック
        実行: ruff 形式 --check 。
      - 名前：マイピー
        実行: |
          if [ -d "src" ];それから
              mypy src/
          それ以外の場合
              mypy {mod}/
          フィ

  テスト:
    名前: テスト (Python ${{ math.python-version }})
    実行: ubuntu-最新
    戦略:
      マトリックス:
        Python バージョン: ["3.10"、"3.11"、"3.12"、"3.13"]

    手順:
      - 使用:actions/checkout@v4
        と:
          fetch- Depth: 0 # setuptools_scm が git タグを読み取るために必要です

      - 使用:actions/setup-python@v5
        と:
          Python バージョン: ${{ マトリックス.python バージョン }}

      - 名前: 依存関係をインストールします。
        実行: pip install -e ".[dev]"

      - 名前: カバレッジを指定してテストを実行します。
        実行: pytest --cov --cov-report=xml

      - 名前: カバレッジのアップロード
        使用: codecov/codecov-action@v4
        と:
          トークン: ${{ Secrets.CODECOV_TOKEN }}
          失敗_ci_if_error: false

  テスト-redis:
    名前: Redis バックエンドのテスト
    実行: ubuntu-最新
    サービス:
      レディス:
        画像: redis:7-alpine
        ポート: ["6379:6379"]
    手順:
      - 使用:actions/checkout@v4
        と:
          フェッチ深度: 0

      - 使用:actions/setup-python@v5
        と:
          Python バージョン: "3.11"

      - 名前: Redis エクストラでインストール
        実行: pip install -e ".[dev,redis]"

      - 名前: Redis テストの実行
        実行: pytest テスト/test_redis_backend.py -v
「」> **`setuptools_scm` を使用する場合は、すべてのチェックアウト ステップに常に `fetch-depth: 0`** を追加してください。
> 完全な git 履歴がないと、`setuptools_scm` はタグを見つけることができず、あるバージョンでビルドが失敗します
> 検出エラーです。

---

## 3. `publish.yml`

`v*.*.*` に一致するタグをプッシュすると自動的にトリガーされます。信頼された発行 (OIDC) を使用します —
リポジトリ シークレットに API トークンがありません。```ヤムル
# .github/workflows/publish.yml
名前: PyPI に公開

に:
  プッシュ：
    タグ:
      - 「v*.*.*」

仕事:
  ビルド:
    名前: ビルドディストリビューション
    実行: ubuntu-最新
    手順:
      - 使用:actions/checkout@v4
        と:
          fetch- Depth: 0 # setuptools_scm にとって重要

      - 使用:actions/setup-python@v5
        と:
          Python バージョン: "3.11"

      - 名前: ビルド ツールのインストール
        実行: pip install build Twine

      - 名前: ビルドパッケージ
        実行: python -m build

      - 名前: チェック配布
        実行: 撚り線チェック dist/*

      - 使用:actions/upload-artifact@v4
        と:
          名前: ディスト
          パス: dist/

  公開:
    名前: PyPI に公開
    ニーズ: 構築
    実行: ubuntu-最新
    環境: pypi
    権限:
      id-token: write # Trusted Publishing (OIDC) に必要

    手順:
      - 使用:actions/download-artifact@v4
        と:
          名前: ディスト
          パス: dist/

      - 名前: PyPI に公開
        使用: pypa/gh-action-pypi-publish@release/v1
「」---

## 4. PyPI 信頼できる公開

信頼できる公開は OpenID Connect (OIDC) を使用するため、PyPI は公開がユーザーからのものであることを確認できます。
特定の GitHub Actions ワークフロー - 有効期間の長い API トークンは必要なく、ローテーションの負担もありません。

### ワンタイムセットアップ

1. https://pypi.org でアカウントを作成します
2. **「アカウント」→「公開」→「新しい保留中の発行元を追加」** に移動します。
3. 以下を入力します。
   - GitHub 所有者 (ユーザー名または組織)
   - リポジトリ名
   - ワークフローファイル名: `publish.yml`
   - 環境名：`pypi`
4. GitHub に `pypi` 環境を作成します。
   **リポジトリ → 設定 → 環境 → 新しい環境 → `pypi`** という名前を付けます

それだけです。次回 `v*.*.*` タグをプッシュすると、ワークフローは自動的に認証します。

---

## 5. 手動パブリッシュフォールバック

CI がまだ設定されていない場合、またはマシンから公開する必要がある場合:「」バッシュ
pip インストール ビルド ツイン

# ビルドホイール + SDIST
Python -m ビルド

# アップロードする前に検証する
麻ひものチェック距離/*

# PyPIにアップロードする
麻紐アップロード dist/*

# OR を最初に TestPyPI でテストします (最初のリリースに推奨)
ひもアップロード --repository testpypi dist/*
pip install --index-url https://test.pypi.org/simple/ your-package
python -c "your_packageをインポート; print(your_package.__version__)"
「」---

## 6. リリースチェックリスト「」
[ ] すべてのテストはメイン/マスターで合格します
[ ] CHANGELOG.md が更新されました — [未リリース] 項目を日付付きの新しいバージョンのセクションに移動します
[ ] CHANGELOG の下部にある差分比較リンクを更新します
[ ] git タグ vX.Y.Z
[ ] git Push Origin master --tags
[ ] GitHubアクションのpublish.yml実行の監視
[ ] PyPI で確認します: pip install your-package==X.Y.Z
[ ] インストールされているバージョンをテストします。
    python -c "your_packageをインポート; print(your_package.__version__)"
「」---

## 7. py.typed Ships in the Wheelを確認する

ビルドするたびに、入力されたマーカーが含まれていることを確認します。「」バッシュ
Python -m ビルド
unzip -l dist/your_package-*.whl | unzip -l dist/your_package-*.whl | unzip -l dist/your_package-*.whl grep py.typed
# 印刷する必要があります: your_package/py.typed
# 見つからない場合は、pyproject.toml の [tool.setuptools.package-data] を確認してください
「」ホイールから欠落している場合、コードが正しくても、ユーザーは型情報を取得できません。
完全に入力されています。これはサイレントエラーです。リリースする前に必ず確認してください。

---

## 8. Semver 変更タイプのガイド

|変更 |バージョンバンプ |例 |
|---|---|---|
| API の重大な変更 (パブリック シンボルの削除/名前変更) |メジャー | `1.2.3 → 2.0.0` |
|新機能、完全な下位互換性 |マイナー | `1.2.3 → 1.3.0` |
|バグ修正、API 変更なし |パッチ | `1.2.3 → 1.2.4` |
|プレリリース |接尾辞 | `2.0.0a1 → 2.0.0rc1 → 2.0.0` |
|パッケージングのみの修正 (コード変更なし) |リリース後 | `1.2.3 → 1.2.3.post1` |