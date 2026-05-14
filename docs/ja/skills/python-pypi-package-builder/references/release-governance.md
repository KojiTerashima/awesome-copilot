# リリースガバナンス — 分岐、保護、OIDC、およびアクセス制御

## 目次
1. [支店戦略](#1-支店戦略)
2. [ブランチ保護ルール](#2-ブランチ保護ルール)
3. [タグベースリリースモデル](#3-タグベースリリースモデル)
4. [ロールベースのアクセス制御](#4-role-based-access-control)
5. [OIDC を使用した安全な発行 (信頼された発行)](#5-secure-publishing-with-oidc-trusted-publishing)
6. [CI でのタグ作成者の検証](#6-validate-tag-author-in-ci)
7. [無効なリリースタグの防止](#7-prevent-invalid-release-tags)
8. [ガバナンス ゲートを含む完全な `publish.yml`](#8-full-publishyml-with-governance-gates)

---

## 1. ブランチ戦略

明確なブランチ階層を使用して、開発作業をリリース可能なコードから分離します。「」
メイン ← 安定。 Development または hotfix/* からのみ PR を受け取ります
開発 ← 統合ブランチ;すべての機能 PR が最初にここにマージされます
feature/* ← 新しい機能 (例: feature/add-redis-backend)
fix/* ← バグ修正 (例: fix/memory-leak-on-close)
hotfix/* ← 緊急の実稼働修正。メインに直接 PR + 開発のためのチェリーピック
release/* ← (オプション) リリースの準備 (例: release/v2.0.0)
「」### ルール

|ルール |なぜ |
|---|---|
| `main` への直接プッシュは禁止 |安定したブランチの偶発的な破損を防止 |
|すべての変更は PR 経由 |マージ前にレビュー + CI を強制します |
|少なくとも 1 つの承認が必要です |すべての変化を監視する 2 番目の目 |
| CI を渡す必要があります |壊れたコードは決してマージしないでください。
|タグのみがリリースをトリガーします。ブランチ プッシュからのアドホック パブリッシュはありません |

---

## 2. ブランチ保護ルール

**GitHub → 設定 → ブランチ → ルールの追加** で `main` と `develop` に対してこれらを設定します。

### `main` の場合```ヤムル
# 同等の GitHub ブランチ保護構成 (ドキュメント用)
ブランチ: メイン
ルール:
  - require_pull_request_reviews:
      必須承認レビュー数: 1
      dismiss_stale_reviews: true
  - require_status_checks_to_pass:
      コンテキスト:
        - 「Lint、フォーマット、タイプチェック」
        - 「テスト (Python 3.11)」 # 少なくとも;すべてのマトリックスのバージョンを追加
      strict: true # マージ前にブランチは最新である必要があります
  - プッシュを制限する:
      allowed_actors: [] # 誰も — PR マージのみ
  - require_linear_history: true # メインでのマージコミットを防止します
「」### `develop` の場合```ヤムル
ブランチ: 開発
ルール:
  - require_pull_request_reviews:
      必須承認レビュー数: 1
  - require_status_checks_to_pass:
      コンテキスト: ["CI"]
      strict: false # 統合ブランチの厳密性は低くなります
「」### GitHub CLI 経由「」バッシュ
# メインを保護 (gh CLI と管理者権限が必要)
gh api リポジトリ/{オーナー}/{リポジトリ}/ブランチ/メイン/保護 \
  --メソッド PUT \
  --input - <<'EOF'
{
  "required_status_checks": {
    "厳密": true、
    "contexts": ["Lint、形式と型のチェック"、"テスト (Python 3.11)"]
  }、
  "enforce_admins": false、
  "required_pull_request_reviews": {
    「必須承認レビュー数」: 1、
    "dismiss_stale_reviews": true
  }、
  「制限」: null
}
終了後
「」---

## 3. タグベースのリリースモデル

**`main` の注釈付きタグのみがリリースをトリガーします。** ブランチ プッシュと PR マージは公開されません。

### タグの命名規則「」
vMAJOR.MINOR.PATCH # 安定版: v1.2.3
vMAJOR.MINOR.PATCHaN # アルファ: v2.0.0a1
vMAJOR.MINOR.PATCHbN # ベータ版: v2.0.0b1
vMAJOR.MINOR.PATCHrcN # リリース候補: v2.0.0rc1
「」### リリースワークフロー「」バッシュ
# 1. PR 経由で開発→メインをマージ (レビュー済み、CI グリーン)

# 2. メインの CHANGELOG.md を更新する
# [未リリース] エントリを [vX.Y.Z] - YYYY-MM-DD に移動します

# 3. 変更ログをコミットする
git チェックアウト メイン
git pull オリジンメイン
git add CHANGELOG.md
git commit -m "雑用: リリース v1.2.3"

# 4. アノテーション付きタグを作成してプッシュする
git tag -a v1.2.3 -m "v1.2.3 をリリース"
git Push Origin v1.2.3 # ← タグのみ。 --tags ではありません (すべてのタグのプッシュを回避します)

# 5. 確認: GitHub アクションのpublish.yml が自動的にトリガーされる
# 監視: [アクション] タブ → ワークフローの公開
# 確認: https://pypi.org/project/your-package/
「」### 注釈付きタグを使用する理由

注釈付きタグ (`git tag -a`) は、タグ付け者の ID、日付、およびメッセージを伝えます。軽量タグは、
そうではありません。 `setuptools_scm` はどちらでも機能しますが、リリース ガバナンスにとっては注釈付きタグの方が安全です。
*誰*がタグを作成したかを記録します。

---

## 4. ロールベースのアクセス制御

|役割 |彼らにできること |
|---|---|
| **メンテナ** |リリース タグの作成、PR の承認、ブランチ保護の管理 |
| **寄稿者** | PR を `develop` に送信します。 `main` にプッシュしたり、リリース タグを作成したりできません |
| **CI (GitHub アクション)** | OIDC 経由で PyPI に公開します。コードをプッシュしたりタグを作成したりできません。

### GitHub チーム経由で実装する「」バッシュ
# メンテナー チームを作成し、タグの作成をそのチームに制限します
gh api リポジトリ/{owner}/{repo}/tags/protection \
  --メソッド POST \
  --フィールドパターン="v*"
# 次に、許可されたアクターを Maintainers チームのみに設定します
「」---

## 5. OIDC を使用した安全な公開 (信頼できる公開)

**PyPI API トークンを GitHub シークレットとして保存しないでください。** 代わりに Trusted Publishing (OIDC) を使用してください。
PyPI プロジェクトは、特定の GitHub リポジトリ + ワークフロー + 環境を認可します - 長期存続するものではありません
秘密が交わされる。

### ワンタイム PyPI セットアップ

1. https://pypi.org/manage/project/your-package/settings/publishing/ に移動します。
2. [**新しい発行者の追加**] をクリックします。
3. 以下を入力します。
   - **所有者:** あなたの Github ユーザー名
   - **リポジトリ:** あなたのリポジトリ名
   - **ワークフロー名:** `publish.yml`
   - **環境名:** `release` (ワークフローの `environment:` キーと一致する必要があります)
4. 保存します。トークンは必要ありません。

### GitHub 環境のセットアップ

1. **GitHub → 設定 → 環境 → 新しい環境** に移動し、`release` という名前を付けます。
2. 保護ルールを追加します: **必須レビュー担当者** (オプションですが、安全性を高めるために推奨)
3. デプロイメント ブランチ ルールを追加します: **`v*`** に一致するタグのみ

### OIDC を使用した最小限の `publish.yml````ヤムル
# .github/workflows/publish.yml
名前: PyPI に公開

に:
  プッシュ：
    タグ:
      - "v[0-9]+.[0-9]+.[0-9]+*" # v1.0.0、v2.0.0a1、v1.2.3rc1 に一致します

仕事:
  公開:
    名前: ビルドして公開する
    実行: ubuntu-最新
    環境: リリース # PyPI Trusted Publisher 環境名と一致する必要があります
    権限:
      id-token: write # OIDC に必要 — 有効期間の短いトークンを PyPI に付与します
      内容：読む

    手順:
      - 使用:actions/checkout@v4
        と:
          fetch- Depth: 0 # setuptools_scm には必須

      - 使用:actions/setup-python@v5
        と:
          Python バージョン: "3.11"

      - 名前: ビルドのインストール
        実行: pip インストール ビルド

      - 名前: ビルドディストリビューション
        実行: python -m build

      - 名前: ディストリビューションの検証
        実行: pip install Twine ;麻ひものチェック距離/*

      - 名前: PyPI に公開
        使用: pypa/gh-action-pypi-publish@release/v1
        # `password:` または `user:` は必要ありません — OIDC が認証を処理します
「」---

## 6. CI でのタグ作成者の検証

`GITHUB_ACTOR` を許可リストと照合して、リリースをトリガーできる人を制限します。
これを公開ジョブの**最初のステップ**として追加すると、迅速に失敗します。```ヤムル
- 名前: タグ作成者の検証
  実行: |
    ALLOWED_USERS=("あなたの Github ユーザー名" "共同メンテナのユーザー名")
    もし[[！ " ${ALLOWED_USERS[*]} " =~ " ${GITHUB_ACTOR} " ]];それから
      echo "::error::リリースがブロックされました: ${GITHUB_ACTOR} は認可されたリリーサーではありません。"
      出口1
    フィ
    echo "${GITHUB_ACTOR} のリリースが承認されました。"
「」### 注記

- `GITHUB_ACTOR` は、タグをプッシュした人の GitHub ユーザー名です。
- 保守性を高めるために、ホワイトリストを別のファイル (例: `.github/MAINTAINERS`) に保存します。
- チームの場合: ユーザー名のチェックを GitHub API 呼び出しに置き換えて、チームのメンバーシップを確認します。

---

## 7. 無効なリリースタグの防止

バージョン管理規則に従っていないタグによってトリガーされたワークフローの実行を拒否します。
これにより、`test`、`backup-old`、`v1` などのタグから誤って公開されることがなくなります。```ヤムル
- 名前: リリースタグ形式を検証します
  実行: |
    # 受け入れる: v1.0.0 v1.0.0a1 v1.0.0b2 v1.0.0rc1 v1.0.0.post1
    もし[[！ "${GITHUB_REF}" =~ ^refs/tags/v[0-9]+\.[0-9]+\.[0-9]+(a|b|rc|\.post)[0-9]*$ ]] && \
       [[ ! "${GITHUB_REF}" =~ ^refs/tags/v[0-9]+\.[0-9]+\.[0-9]+$ ]];それから
      echo "::error::タグ '${GITHUB_REF}' が必要な形式 v<MAJOR>.<MINOR>.<PATCH>[pre] と一致しません。"
      出口1
    フィ
    echo "有効なタグ形式: ${GITHUB_REF}"
「」### 正規表現の説明

|パターン |マッチ |
|---|---|
| `v[0-9]+\.[0-9]+\.[0-9]+` | `v1.0.0`、`v12.3.4` |
| `(a\|b\|rc)[0-9]*` | `v1.0.0a1`、`v2.0.0rc2` |
| `\.post[0-9]*` | `v1.0.0.post1` |

---

## 8. ガバナンス ゲートを備えた完全な `publish.yml`

タグの検証、作成者チェック、TestPyPI ゲート、実稼働パブリッシュを組み合わせた完全なワークフロー。```ヤムル
# .github/workflows/publish.yml
名前: PyPI に公開

に:
  プッシュ：
    タグ:
      - "v[0-9]+.[0-9]+.[0-9]+*"

仕事:
  公開:
    名前: ビルド、検証、公開
    実行: ubuntu-最新
    環境: リリース
    権限:
      IDトークン:書き込み
      内容：読む

    手順:
      - 名前: リリースタグ形式を検証します
        実行: |
          もし [[ ！ "${GITHUB_REF}" =~ ^refs/tags/v[0-9]+\.[0-9]+\.[0-9]+(a[0-9]*|b[0-9]*|rc[0-9]*|\.post[0-9]*)?$ ]];それから
            echo "::error::無効なタグ形式: ${GITHUB_REF}"
            出口1
          フィ

      - 名前: タグ作成者の検証
        実行: |
          ALLOWED_USERS=("あなたの Github ユーザー名")
          もし[[！ " ${ALLOWED_USERS[*]} " =~ " ${GITHUB_ACTOR} " ]];それから
            echo "::error::${GITHUB_ACTOR} はリリースする権限がありません。"
            出口1
          フィ

      - 使用:actions/checkout@v4
        と:
          フェッチ深度: 0

      - 使用:actions/setup-python@v5
        と:
          Python バージョン: "3.11"

      - 名前: ビルド ツールのインストール
        実行: pip install build Twine

      - 名前: ビルド
        実行: python -m build

      - 名前: ディストリビューションの検証
        実行: 撚り線チェック dist/*

      - 名前: TestPyPI に公開
        使用: pypa/gh-action-pypi-publish@release/v1
        と:
          リポジトリ URL: https://test.pypi.org/legacy/
        continue-on-error: true # 致命的ではありません。常にこれを通過させたい場合は削除してください

      - 名前: PyPI に公開
        使用: pypa/gh-action-pypi-publish@release/v1
「」### セキュリティチェックリスト

- [ ] PyPI Trusted Publishing が設定済み (GitHub に API トークンが保存されていない)
- [ ] GitHub `release` 環境にはブランチ保護があります: `v*` のみに一致するタグ
- [ ] タグ形式検証ステップはジョブの最初のステップです
- [ ] 許可されたユーザーのリストは維持され、定期的に見直されます
- [ ] ログにシークレットは出力されません (`echo` および `run` のすべてのステップを確認してください)
- [ ] `permissions:` は `id-token: write` のみにスコープされます — `write-all` はありません