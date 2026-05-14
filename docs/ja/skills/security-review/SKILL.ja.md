---
name: security-review
description: 'AI-powered codebase security scanner that reasons about code like a security researcher — tracing data flows, understanding component interactions, and catching vulnerabilities that pattern-matching tools miss. Use this skill when asked to scan code for security vulnerabilities, find bugs, check for SQL injection, XSS, command injection, exposed API keys, hardcoded secrets, insecure dependencies, access control issues, or any request like "is my code secure?", "review for security issues", "audit this codebase", or "check for vulnerabilities". Covers injection flaws, authentication and access control bugs, secrets exposure, weak cryptography, insecure dependencies, and business logic issues across JavaScript, TypeScript, Python, Java, PHP, Go, Ruby, and Rust.'
---
# セキュリティレビュー

人間のセキュリティと同じようにコードベースを推論する、AI を活用したセキュリティ スキャナー
研究者なら、データ フローを追跡し、コンポーネントの相互作用を理解し、状況を把握するでしょう。
パターンマッチングツールが見逃す脆弱性。

## このスキルを使用する場合

このスキルは、リクエストに以下が含まれる場合に使用します。

- コードベースまたはファイルのセキュリティ脆弱性のスキャン
- セキュリティレビューまたは脆弱性チェックの実行
- SQL インジェクション、XSS、コマンド インジェクション、またはその他のインジェクションの欠陥のチェック
- コード内の公開された API キー、ハードコードされたシークレット、または認証情報の検索
- 既知の CVE の依存関係を監査する
- 認証、認可、またはアクセス制御ロジックのレビュー
- 安全でない暗号または弱いランダム性の検出
- データ フロー分析を実行して、危険なシンクへのユーザー入力を追跡します。
- 「私のコードは安全ですか?」、「このファイルをスキャンしてください」、「リポジトリに脆弱性がないか確認してください」などのリクエストの表現
- `/security-review` または `/security-review <path>` の実行

## このスキルの仕組み

パターンを照合する従来の静的分析ツールとは異なり、このスキルは次のことを行います。
1. **セキュリティ研究者のようにコードを読む** — コンテキスト、意図、データ フローを理解する
2. **ファイル全体のトレース** — ユーザー入力がアプリケーション内をどのように移動するかを追跡します
3. **結果を自己検証** — 各結果を再検査して誤検知をフィルタリングします
4. **重大度評価の割り当て** — クリティカル / 高 / 中 / 低 / 情報
5. **対象を絞ったパッチを提案します** — すべての発見には具体的な修正が含まれます
6. **人間の承認が必要です** — 何も自動適用されません。あなたはいつも最初にレビューします

## 実行ワークフロー

毎回、**順番に**次の手順を実行してください。

### ステップ 1 — スコープの解決
何をスキャンするかを決定します。
- パスが指定されている場合 (`/security-review src/auth/`)、そのスコープのみをスキャンします
- パスが指定されていない場合は、ルートから開始して **プロジェクト全体**をスキャンします
- 使用している言語とフレームワークを特定します (package.json、requirements.txt、
  go.mod、Cargo.toml、pom.xml、Gemfile、composer.json など)
- `references/language-patterns.md` を読んで、言語固有の脆弱性パターンをロードします### ステップ 2 — 依存関係の監査
ソース コードをスキャンする前に、まず依存関係を監査します (早いもの勝ち)。
- **Node.js**: 既知の脆弱なパッケージについては `package.json` + `package-lock.json` を確認してください
- **Python**: `requirements.txt` / `pyproject.toml` / `Pipfile` を確認してください
- **Java**: `pom.xml` / `build.gradle` を確認してください
- **Ruby**: `Gemfile.lock` を確認してください
- **Rust**: `Cargo.toml` を確認してください
- **Go**: `go.sum` を確認してください
- 既知の CVE、非推奨の暗号ライブラリ、または疑わしい古い固定バージョンを含むパッケージにフラグを立てます
- 厳選されたウォッチリストについては `references/vulnerable-packages.md` をご覧ください

### ステップ 3 — 秘密と暴露スキャン
以下のすべてのファイル (config、env、CI/CD、Dockerfile、IaC を含む) をスキャンします。
- ハードコードされた API キー、トークン、パスワード、秘密キー
- `.env` ファイルが誤ってコミットされました
- コメントまたはデバッグ ログ内の秘密
- クラウド認証情報 (AWS、GCP、Azure、Stripe、Twilio など)
- 資格情報が埋め込まれたデータベース接続文字列
- 適用する正規表現パターンとエントロピー ヒューリスティックについては、`references/secret-patterns.md` を参照してください。

### ステップ 4 — 脆弱性の詳細スキャン
これがコアスキャンです。コードに関する理由 — 単にパターン一致するだけではありません。
各カテゴリの詳細については、`references/vuln-categories.md` を参照してください。

**射出欠陥**
- SQL インジェクション: 文字列補間を使用した生のクエリ、ORM の誤用、二次 SQLi
- XSS: エスケープされていない出力、dangerlySetInnerHTML、innerHTML、テンプレート インジェクション
- コマンドインジェクション: ユーザー入力による exec/spawn/system
- LDAP、XPath、ヘッダー、ログインジェクション

**認証とアクセス制御**
- 機密性の高いエンドポイントで認証が欠落している
- オブジェクトレベルの認証の破損 (BOLA/IDOR)
- JWT の弱点 (alg:none、弱いシークレット、有効期限検証なし)
- セッション固定、CSRF 保護の欠如
- 権限昇格パス
- 質量割り当て/パラメータ汚染

**データの処理**
- ログ、エラー メッセージ、または API 応答内の機密データ
- 保存中または転送中の暗号化が欠落している
- 安全でない逆シリアル化
- パストラバーサル / ディレクトリトラバーサル
- XXE (XML 外部エンティティ) 処理
- SSRF (サーバーサイドリクエストフォージェリ)

**暗号化**
- セキュリティ目的での MD5、SHA1、DES の使用
- ハードコードされた IV またはソルト
- 弱い乱数生成 (トークンの Math.random())
- TLS 証明書の検証が欠落しています**ビジネス ロジック**
- 競合状態 (TOCTOU)
- 財務計算における整数のオーバーフロー
- 機密性の高いエンドポイントにレート制限がない
- 予測可能なリソース識別子

### ステップ 5 — ファイル間のデータ フロー分析
ファイルごとのスキャン後、**全体的なレビュー**を実行します。
- エントリ ポイントからのユーザー制御入力のトレース (HTTP パラメータ、ヘッダー、本文、ファイル アップロード)
  シンクに至るまで (DB クエリ、実行呼び出し、HTML 出力、ファイル書き込み)
- 複数のファイルを一緒に見た場合にのみ現れる脆弱性を特定します
- サービスまたはモジュール間の安全でない信頼境界を確認します。

### ステップ 6 — 自己検証パス
それぞれの結果について:
1. 関連するコードを新鮮な目で読み直します
2. 「これは実際に悪用可能ですか? それとも、私が見逃したサニタイズはありますか?」と尋ねます。
3. フレームワークまたはミドルウェアがすでにこのアップストリームを処理しているかどうかを確認します
4. 本物の脆弱性ではない検出結果をダウングレードまたは破棄する
5. 最終重大度を割り当てます: CRITICAL / HIGH / MEDIUM / LOW / INFO

### ステップ 7 — セキュリティ レポートの生成
`references/report-format.md` で定義された形式で完全なレポートを出力します。

### ステップ 8 — パッチを提案する
すべての CRITICAL および HIGH の検出結果に対して、具体的なパッチを生成します。
- 脆弱なコードを表示します (前)
- 修正されたコードを表示(後)
- 何が変わったのか、なぜ変わったのか説明する
- 元のコード スタイル、変数名、構造を保持します。
- 修正を説明するコメントをインラインで追加します

**「適用する前に各パッチを確認してください。まだ何も変更されていません。」** と明示的に述べます。

## 重大度ガイド

|重大度 |意味 |例 |
|----------|-----------|----------|
| 🔴 クリティカル |差し迫った悪用リスク、データ侵害の可能性 | SQLi、RCE、認証バイパス |
| 🟠高い |深刻な脆弱性、悪用パスが存在 | XSS、IDOR、ハードコードされたシークレット |
| 🟡 ミディアム |条件または連鎖で悪用可能 | CSRF、オープンリダイレクト、弱い暗号 |
| 🔵 低い |ベストプラクティス違反、直接的なリスクは低い |詳細なエラー、ヘッダーの欠落 |
| ⚪ 情報 |脆弱性ではなく、注目に値する観察結果 |古い依存関係 (CVE なし) |

## 出力ルール- **常に** 最初に調査結果の概要表を作成します (重大度別にカウント)
- **決してパッチを自動適用しないでください** - 人間によるレビューのためにのみパッチを提示してください
- **常に** 結果ごとに信頼度評価 (高 / 中 / 低) を含めます
- **結果をファイル別ではなくカテゴリ別にグループ化**
- **具体的に** — ファイル パス、行番号、脆弱なコード スニペットを正確に含めます
- **リスクについて説明** 平易な英語で — 攻撃者はこれを使って何ができるでしょうか?
- コードベースがクリーンな場合は、スキャンされた内容で「脆弱性は見つかりませんでした」とはっきりと伝えます。

## 参照ファイル

詳細な検出ガイダンスについては、必要に応じて次の参照ファイルをロードしてください。

- `references/vuln-categories.md` — 検出シグナル、安全なパターン、エスカレーション チェッカーを備えたあらゆる脆弱性カテゴリの詳細なリファレンス
  - 検索パターン: `SQL injection`、`XSS`、`command injection`、`SSRF`、`BOLA`、`IDOR`、`JWT`、`CSRF`、`secrets`、`cryptography`、`race condition`、`path traversal`
- `references/secret-patterns.md` — 正規表現パターン、エントロピーベースの検出、CI/CD シークレットのリスク
  - 検索パターン: `API key`、`token`、`private key`、`connection string`、`entropy`、`.env`、`GitHub Actions`、`Docker`、`Terraform`
- `references/language-patterns.md` — JavaScript、Python、Java、PHP、Go、Ruby、Rust のフレームワーク固有の脆弱性パターン
  - 検索パターン：`Express`、`React`、`Next.js`、`Django`、`Flask`、`FastAPI`、`Spring Boot`、`PHP`、`Go`、`Rails`、`Rust`
- `references/vulnerable-packages.md` — npm、pip、Maven、Rubygems、Cargo、Go モジュール用に厳選された CVE ウォッチリスト
  - 検索パターン: `lodash`、`axios`、`jsonwebtoken`、`Pillow`、`log4j`、`nokogiri`、`CVE`
- `references/report-format.md` — 検索カード、依存関係の監査、シークレットのスキャン、およびパッチ提案の書式設定を含むセキュリティ レポートの構造化された出力テンプレート
  - 検索パターン: `report`、`format`、`template`、`finding`、`patch`、`summary`、`confidence`