# GDPR リファレンス — セキュリティ、運用とアーキテクチャ

次の実装詳細が必要なときに、このファイルを読み込んでください:
暗号化、パスワードハッシュ化、シークレット管理、匿名化/仮名化、
クラウド/DevOps プラクティス、CI/CD コントロール、インシデント対応、アーキテクチャパターン。

---

## 暗号化

### 保存時暗号化（At-Rest Encryption）

| データ機微性 | 最低基準 |
|---|---|
| 標準的な個人データ（氏名、住所、メール） | AES-256 のディスク/ボリューム暗号化（クラウドプロバイダー既定） |
| 機微な個人データ（健康、生体、金融、国民ID） | AES-256 の**カラムレベル**暗号化 + KMS によるエンベロープ暗号化 |
| 暗号鍵 | HSM バックド KMS（Azure Key Vault Premium / AWS KMS CMK / GCP Cloud KMS） |

**エンベロープ暗号化パターン:**
1. **Data Encryption Key (DEK)** でデータを暗号化する（AES-256、レコード単位またはテーブル単位で生成）。
2. KMS に保存された **Key Encryption Key (KEK)** で DEK を暗号化する。
3. 暗号化済みデータと一緒に、暗号化済み DEK を保存する。
4. KEK を削除する = それで暗号化されたすべてのデータに対する実質的な crypto-shredding。

### 通信時暗号化（In-Transit Encryption）

- **MUST** TLS 1.2 以上を強制し、TLS 1.3 を推奨する。
- **MUST** `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` を設定する。
- **MUST NOT** TLS 1.0、TLS 1.1、null cipher suites、または export-grade ciphers を許可しない。
- **MUST NOT** 本番環境で自己署名証明書を使用しない。

### 鍵管理

- DEK は最低でも毎年ローテーションし、漏えいの疑いがある場合は即時ローテーションする。
- 環境ごと（dev / staging / prod）に鍵ネームスペースを分離する。
- KMS 鍵へのすべてのアクセスイベントを記録し、異常なアクセスパターンにアラートを出す。
- 暗号鍵をソースコードや設定ファイルにハードコードしてはならない（MUST NOT）。

---

## パスワードハッシュ化

| アルゴリズム | パラメータ | 備考 |
|---|---|---|
| **Argon2id**  推奨 | memory ≥ 64 MB, iterations ≥ 3, parallelism ≥ 4 | OWASP と NIST が推奨 |
| **bcrypt**  許容 | cost factor ≥ 12 | 広くサポートされる。Argon2id が使えない場合に使用 |
| **scrypt**  許容 | N=32768, r=8, p=1 | 良い代替 |
| MD5  | — | 絶対に不可 — 容易に破られる |
| SHA-1 / SHA-256  | — | パスワード用途には絶対に不可 — この目的向けに設計されていない |

**MUST**
- パスワードごとに一意の salt を使う（上記 3 アルゴリズムはすべて組み込み対応）。
- 保存するのはハッシュのみ。平文や可逆エンコードは決して保存しない。
- 保存済みハッシュが古いアルゴリズムなら、ログイン時に再ハッシュし、透過的にアップグレードする。

**SHOULD**
- **pepper**（ハッシュ前に加えるサーバー側シークレット）を追加し、DB ではなく KMS に保存する。
- 登録時に既知の漏えいリストと照合する（`haveibeenpwned` API、k-anonymity モード）。
- 最小パスワード長を 12 文字にする。

**MUST NOT**
- 登録時やログイン失敗時を含め、いかなる形式でもパスワードをログ出力しない。
- URL やクエリ文字列でパスワードを送信しない。
- パスワードリセットトークンを平文保存しない。保存前にハッシュ化する。

---

## シークレット管理

**MUST**
- すべてのシークレットを専用のシークレットマネージャーに保存する: Azure Key Vault, AWS Secrets Manager,
  GCP Secret Manager, または HashiCorp Vault。
- シークレットのコミットを防ぐ pre-commit フックを使用する: `gitleaks`, `detect-secrets`, GitHub native secret scanning。
- 次の場合はシークレットを即時ローテーションする: 開発者の離任、漏えい疑い、年次スケジュール。
- **secrets inventory document** を維持し、すべてのシークレットに用途とローテーション日を記載する。

**SHOULD**
- 長期 API キーの代わりに、OIDC federation（GitHub Actions → Azure/AWS/GCP）による**短命クレデンシャル**を使う。
- すべての KMS シークレットアクセスを監査し、営業時間外や想定外ソースからのアクセスにアラートを出す。
- 環境ごとにシークレットネームスペースを分離する。

**`.gitignore` MUST include:**
```
.env
.env.*
*.pem
*.key
*.pfx
*.p12
secrets/
appsettings.*.json   # 接続文字列を含む可能性がある場合
```

**MUST NOT**
- シークレットをソースコードリポジトリにコミットしない。
- シークレットを平文の CLI 引数として渡さない（プロセス一覧やシェル履歴に残る）。
- シークレットを、暗号化されていない環境変数デフォルトとしてコードに保存しない。

---

## 匿名化と仮名化

### 定義

| 用語 | 可逆性 | GDPR の適用範囲? | ユースケース |
|---|---|---|---|
| **Anonymization** | いいえ | GDPR の適用範囲外 | 削除後も保持する記録、分析データセット |
| **Pseudonymization** | はい（鍵あり） | 依然として個人データ | 分析パイプライン、監査ログ、リスク低減処理 |

### 匿名化手法

| 手法 | 方法 | いつ使うか |
|---|---|---|
| Suppression | フィールドを完全に削除 | 分析価値がないフィールド |
| Masking | 固定プレースホルダーに置換（`"ANONYMIZED_USER"`） | 削除後の監査ログ識別子 |
| Generalization | 正確値を範囲に置換（34歳 → "30–40"） | 分析 |
| Noise addition | 数値に統計ノイズを追加 | 集計分析 |
| Aggregation | 個別値ではなくグループ統計を報告 | レポーティング |
| K-anonymity | 各レコードを他の k-1 件と識別不能にする | 分析データセット |

### 仮名化手法

| 手法 | 方法 |
|---|---|
| HMAC-SHA256 with secret key | 一貫性がある、不可逆、鍵付き。分析でのユーザー ID に使用。鍵は KMS に保存。 |
| Tokenization | 値を不透明なトークンに置換し、マッピングは別の安全な vault に保存。 |
| Encryption with separate key | 明示的な KMS 認可がある場合のみ復号可能。 |

**MUST**
- ユーザー削除時、保持が必要な記録（財務、監査ログ）は**匿名化**する。識別フィールドを `"ANONYMIZED"` またはハッシュ化プレースホルダーに置換する。
- 仮名化鍵は KMS に保存し、仮名化データと同じデータベースには保存しない。
- 匿名化処理はアサーションでテストし、出力から元の値を復元できないこと（MUST NOT）を確認する。

**Crypto-shredding pattern (event sourcing):**
イベント内の個人データをユーザー単位の DEK で暗号化し、DEK は KMS に保存する。
削除時: KMS から DEK を削除する → そのユーザーの全イベントは実質的に匿名化される。

**MUST NOT**
- 他データセットとの突合で再識別可能な場合、そのデータを「匿名化済み」と呼ばない。
- 仮名化を適用し、マッピング鍵を仮名化データと同じテーブルに保存しない。

---

## クラウド & DevOps プラクティス

**MUST**
- すべてのクラウドストレージで保存時暗号化を有効にする: blobs, managed databases, queues, caches。
- **private endpoints** を使用し、データベースを公開アクセス可能にしてはならない（MUST NOT）。
- network security groups / firewall rules を適用し、DB アクセスをアプリケーション層のみに制限する。
- クラウドネイティブ監査ログを有効化する: Azure Monitor / AWS CloudTrail / GCP Cloud Audit Logs。
- 個人データは**承認済み地理リージョン**（EEA、または十分性認定 / SCCs）にのみ保存する。
- 個人データを処理するすべてのクラウドリソースに `DataClassification` タグを付与する。

**SHOULD**
- Microsoft Defender for Cloud / AWS Security Hub / GCP SCC を有効化し、推奨事項を毎週レビューする。
- 長期アクセスキーではなく **managed identities**（Azure）または **IAM roles**（AWS/GCP）を使う。
- オブジェクトストレージで soft delete と versioning を有効化する。
- 保護されていないバケットへ書き込まれた PII を検出するため、クラウドストレージに DLP ポリシーを適用する。
- 機微テーブルへの SELECT に対する DB レベル監査ログを有効化する。

**MUST NOT**
- アクセス制御なしの公開ストレージバケットに個人データを保存しない。
- 本番環境で public IP 付きのデータベースをデプロイしない。
- データが混在する可能性がある場合、本番と非本番で同じクラウドアカウント/サブスクリプションを使わない。

---

## CI/CD コントロール

**MUST**
- すべてのコミットで **secret scanning** を実行する: `gitleaks`, `detect-secrets`, GitHub secret scanning。
- すべてのビルドで **dependency vulnerability scanning** を実行する: `npm audit`, `dotnet list package --vulnerable`, `trivy`, `snyk`。
- CI テストジョブで実在の個人データを使用してはならない（MUST NOT）。
- CI パイプラインで環境変数をログ出力してはならない（MUST NOT）。すべてのシークレットをマスクする。

**SHOULD**
- すべての PR で **SAST** を実行する: SonarQube, Semgrep, または CodeQL。
- **container image scanning** を実行する: `trivy`, Snyk Container, または AWS ECR scanning。
- パイプラインに **GDPR compliance gate** を追加する:
  - 保持期間が文書化されていない新規 migration がある → fail。
  - 既知の PII フィールド名を含むログ文がある → warn。

**Pipeline secret rules:**
```yaml
# MUST: 使用前にシークレットをマスクする
- name: Mask secret
  run: echo "::add-mask::${{ secrets.MY_SECRET }}"

# MUST NOT: シークレットをコンソールに出力する
- run: echo "Key=$API_KEY"   # Never

# SHOULD: OIDC federation を使用する（長期鍵なし）
- uses: azure/login@v1
  with:
    client-id: ${{ vars.AZURE_CLIENT_ID }}
    tenant-id: ${{ vars.AZURE_TENANT_ID }}
    subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

---

## インシデントと侵害対応

### 規制上のタイムライン

| 期間 | 義務 |
|---|---|
| 認知から **72時間** 以内 | 監督機関（CNIL, APD, ICO…）へ通知 — 侵害が個人にリスクを与える可能性が低い場合を除く |
| **不当な遅延なく** | 侵害により本人の権利に**高リスク**が生じる可能性が高い場合、影響を受けるデータ主体へ通知 |

DPA への通知が不要なものも含め、すべての個人データ侵害を内部で記録する。

### 侵害対応 Runbook（テンプレート）

1. **Detection** — 判定基準を定義: 何をインシデント起点とするか（認証情報漏えい、DB ダンプ露出、ランサムウェア、誤った公開バケット）。
2. **Severity classification** — データ機微性と件数に基づき Low / Medium / High / Critical を判定。
3. **Containment** — 侵害された認証情報を失効し、影響システムを隔離し、証拠を保全する（ログを削除しないこと）。
4. **Assessment** — どのデータが露出したか？対象者数は？リスクレベルは？
5. **DPA notification** — 監督機関のオンラインポータルを利用し、次を含める: 侵害の性質、データ主体のカテゴリと概数、記録のカテゴリと概数、連絡先、想定される影響、実施済み対策。
6. **Data subject notification** — 高リスク時: 明確な文言で、侵害の性質、想定される影響、実施済み対策、DPO 連絡先を通知。
7. **Post-incident review** — 根本原因分析、是正措置、runbook 更新。

### 自動侵害検知アラート

次に対してアラートを設定する:
- 異常なデータエクスポート量（1時間あたりしきい値）
- 営業時間外の機微テーブルアクセス
- 一括削除イベント
- 認証失敗の急増
- 公開侵害データベースに新しい認証情報が出現（HaveIBeenPwned monitoring）

侵害記録は内部で少なくとも **5年間** 保持する。

---

## アーキテクチャパターン

### データストア分離
運用データ（トランザクション DB）と分析データ（データウェアハウス）を分離する。
それぞれに異なる保持期間とアクセス制御を適用する。
分析ストアは本番の運用テーブルを直接参照してはならない（MUST NOT）。

### 専用 Consent Store
同意は、ユーザーテーブルの boolean カラムではなく、別ストアの不変イベントログとして追跡する。
これにより、監査可能な同意履歴、バージョン追跡、データ損失なしの容易な撤回が可能になる。

### 監査ログ分離
監査ログは別の追記専用ストアに保存する。
アプリケーションのサービスアカウントは監査ログエントリを削除できてはならない（MUST NOT）。
監査テーブルには INSERT 専用権限の別 DB ユーザーを使う。

### DSR キューパターン
Data Subject Requests は非同期ワークフローとして実装する:
`POST /api/v1/me/erasure-request` → ジョブをキュー投入 → ワーカーが全ストアをスクラブ → 完了時にユーザー通知。
これにより、複数ストアにまたがるスクラブの複雑さを確実に処理でき、リトライ機構も提供できる。

### 仮名化ゲートウェイ
分析パイプラインでは、運用システムと分析システムの境界に仮名化サービスを実装する。
マッピング鍵（HMAC secret または tokenization vault）は運用ゾーン外へ出さない。
分析ゾーンが受け取るのは仮名化済み識別子のみとする。

### Crypto-Shredding（Event Sourcing）
イベント内の個人データを KMS に保存したユーザー単位 DEK で暗号化する。
ユーザー削除時: DEK を削除する → そのユーザーのすべての過去イベントは、イベントログを変更せずに
実質的に匿名化される。

