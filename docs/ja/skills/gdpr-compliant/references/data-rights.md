# GDPR リファレンス — データ主体の権利、説明責任とガバナンス

以下の実装詳細が必要な場合にこのファイルを読み込んでください：
ユーザー権利エンドポイント、データ主体請求（DSR）ワークフロー、
処理活動記録（RoPA）、同意管理。

---

## ユーザー権利の実装（第15〜22条）

本番稼働前に、すべての権利に対してテスト済みの API エンドポイント、または文書化されたバックオフィス手順が **必須** です。本人確認済みの請求には **30 暦日以内** に対応してください。

| 権利 | 条文 | エンジニアリング実装 |
|---|---|---|
| アクセス権 | 15 | `GET /api/v1/me/data-export` — すべての個人データ（JSON または CSV） |
| 訂正権 | 16 | `PUT /api/v1/me/profile` — すべての下流ストアに反映 |
| 消去権 | 17 | `DELETE /api/v1/me` — 消去チェックリストに従って全ストアを消去 |
| 処理制限権 | 18 | ユーザーレコードに `ProcessingRestricted` フラグを設定し、非必須処理を制御 |
| データポータビリティの権利 | 20 | アクセス権エンドポイントと同じ。構造化され、機械可読（JSON） |
| 異議申立権 | 21 | 正当利益に基づく処理のオプトアウト用エンドポイントを提供し、即時反映 |
| 自動化された意思決定 | 22 | 人によるレビュー経路とロジックの説明を提供 |

### 消去チェックリスト — すべてのストアを対象にすること（必須）

`DELETE /api/v1/me` が呼び出されたとき、消去パイプラインは次を消去する必要があります（必須）：

- メインのリレーショナルデータベース（行の匿名化または削除）
- リードレプリカ
- 検索インデックス（Elasticsearch、Azure Cognitive Search など）
- インメモリキャッシュ（Redis、IMemoryCache）
- オブジェクトストレージ（S3、Azure Blob — プロフィール画像、文書）
- メールサービスログ（Brevo、SendGrid — 配信ログ）
- 分析プラットフォーム（Mixpanel、Amplitude、GA4 — ユーザー削除 API）
- 監査ログ（識別可能な項目は匿名化 — イベント自体は削除しない）
- バックアップ（バックアップ TTL を文書化し、自然失効を許容）
- CDN エッジキャッシュ（個人データがキャッシュされ得る場合はパージ）
- サードパーティの再委託先（削除 API を実行、または手動手順を文書化）

### データエクスポート形式（`GET /api/v1/me/data-export`）

```json
{
  "exportedAt": "2025-03-30T10:00:00Z",
  "subject": {
    "id": "uuid",
    "email": "user@example.com",
    "createdAt": "2024-01-15T08:30:00Z"
  },
  "profile": { ... },
  "orders": [ ... ],
  "consents": [ ... ],
  "auditEvents": [ ... ]
}
```

- 機械可読であること（JSON 推奨、CSV も可）が必須。
- PDF のスクリーンショットや HTML ページであってはならない（MUST NOT）。
- このユーザーについて RoPA に記載されたすべてのストアを含めることが必須。

### DSR トラッカー（バックオフィス）

以下を備えた **データ主体請求トラッカー** を実装してください：
- 請求受付日
- 請求種別（access / rectification / erasure / portability / restriction / objection）
- 検証ステータス（本人確認済み y/n）
- 期限（受付日 + 30 日）
- 担当者
- 完了日と結果
- 備考

メインストアの消去は自動化し、サードパーティストアの手動手順を文書化してください。

---

## 処理活動記録（RoPA）

リポジトリ内でバージョン管理された生きた文書（Markdown、YAML、または JSON）として維持してください。
処理活動を導入する **すべて** の新機能ごとに更新してください。

### 処理活動ごとの最小項目

```yaml
- name: "User account management"
  purpose: "Create and manage user accounts for service access"
  legalBasis: "Contract (Art. 6(1)(b))"
  dataSubjects: ["Registered users"]
  personalDataCategories: ["Name", "Email", "Password hash", "IP address"]
  recipients: ["Internal engineering team", "Brevo (email delivery)"]
  retentionPeriod: "Account lifetime + 12 months"
  transfers:
    outside_eea: true
    safeguard: "Brevo — Standard Contractual Clauses (SCCs)"
  securityMeasures: ["TLS 1.3", "AES-256 at rest", "bcrypt password hashing"]
  dpia_required: false
```

### 法的根拠の選択肢（第6条）

| 根拠 | 使用する場面 |
|---|---|
| `Contract (6(1)(b))` | サービス契約の履行に必要な処理 |
| `Legitimate interest (6(1)(f))` | 不正防止、セキュリティ、分析（利益衡量テストが必要） |
| `Consent (6(1)(a))` | マーケティング、非必須 Cookie、任意のプロファイリング |
| `Legal obligation (6(1)(c))` | 税務記録、マネーロンダリング防止 |
| `Vital interest (6(1)(d))` | 緊急時のみ |
| `Public task (6(1)(e))` | 公的機関 |

---

## 同意管理

### MUST

- 同意は可変のブールフラグではなく、**不変のイベントログ** として保存する。
- 記録項目：何に同意したか、いつ同意したか、プライバシーポリシーのどのバージョンか、同意取得の手段。
- 分析／マーケティング SDK は **条件付き** で読み込むこと。つまり、同意付与後のみ。
- 同意撤回手段は、同意付与と同じくらい容易に使えるようにする。

### 同意ストアのスキーマ（最小）

```sql
CREATE TABLE ConsentRecords (
    Id          UUID PRIMARY KEY,
    UserId      UUID NOT NULL,
    Purpose     VARCHAR(100) NOT NULL,   -- e.g. "marketing_emails", "analytics"
    Granted     BOOLEAN NOT NULL,
    PolicyVersion VARCHAR(20) NOT NULL,
    ConsentedAt TIMESTAMPTZ NOT NULL,
    IpAddressHash VARCHAR(64),           -- HMAC-SHA256 of anonymized IP
    UserAgent   VARCHAR(500)
);
```

### MUST NOT

- 同意チェックボックスを事前チェックしてはならない。
- マーケティング同意をサービス提供同意と抱き合わせにしてはならない。
- マーケティング同意をサービス利用の条件にしてはならない。
- ダークパターンを使用してはならない（例：「すべて同意」を目立たせ、「拒否」を見つけにくくする）。

---

## 再委託先（Sub-processor）管理

個人データに触れる新しい SaaS ツールまたはクラウドサービスを導入するたびに、
更新される **再委託先リスト** を維持してください。

再委託先ごとの最小項目：

| 項目 | 例 |
|---|---|
| Name | Brevo |
| Service | Transactional email |
| Data categories transferred | Email address, name, email content |
| Processing location | EU (Paris) |
| DPA signed |  2024-01-10 |
| DPA URL / reference | [link] |
| SCCs applicable | N/A (EU-based) |

再委託先リストは毎年および変更時に **必ず** 見直すこと。
DPA 締結前に新しい再委託先へデータを流すことは **禁止**。

---

## DPIA のトリガー（第35条）

高リスクを生じる可能性が高い処理の前には、DPIA が **必須** です。トリガー例：

- 個人に重大な影響を与える、体系的かつ大規模なプロファイリング
- 要配慮個人データ（健康、生体情報、人種的出自、性的指向、宗教）の大規模処理
- 公共にアクセス可能な場所の体系的監視（CCTV、位置追跡）
- 子どものデータの大規模処理
- プライバシー影響が未知の革新的技術
- 複数ソースのデータセット照合または結合

迷う場合：それでも DPIA を実施してください。結果を文書化してください。

