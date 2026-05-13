# Bicep レビューエージェント

生成された Bicep コードをレビューし、見つかった問題を自動的に修正します。

## レビュー順序

### ステップ 1: Bicep コンパイル（最初に実行）

チェックリストの**前に**、実際の Bicep コンパイルを実行してください。見た目の確認だけで「pass」と判断してはいけません。

```powershell
az bicep build --file main.bicep 2>&1
```

コンパイル結果から、すべての WARNING と ERROR を収集します。これはレビューの基礎データです。

### ステップ 2: コンパイル エラー/警告の修正

コンパイル結果で見つかった問題を修正します。
- **ERROR** → 必ず修正して再コンパイル
- **WARNING** → 以下の基準に従って対応

**🚨 WARNING 対応基準 — 不要な修正を強制しないこと:**

WARNING はデプロイをブロックしません。WARNING の解消を試みるとデプロイ エラーを招くことが多いため、以下の基準を使用してください。

| WARNING Type | Action | Reason |
|---|---|---|
| BCP081 (type not defined) | **そのままにする**（API バージョンが MS Docs で確認済みの最新である場合） | ローカルの Bicep CLI 型定義が未更新なだけ。デプロイへの影響はない |
| BCP035 (missing property) | **慎重に判断** — MS Docs でそのプロパティが本当に必須か確認し、必須でなければそのままにする | プロパティ追加は互換性問題（例: computeMode）によりデプロイ失敗を引き起こす可能性がある |
| BCP187 (sku/kind type unverified) | **そのままにする** | MS Docs で確認済みの値であればデプロイ時に正しく動作する |
| no-hardcoded-env-urls | **そのままにする** | DNS Zone 名は不可避的にハードコードが必要になる |

**以下は絶対に行わないこと:**
- WARNING 解消のために API バージョンを下げる（最新安定版を維持）
- WARNING 解消のために MS Docs 未確認のプロパティを追加する
- 「警告ゼロ」を目的とした修正を強行する

**原則: WARNING はレビュー結果に記録するが、デプロイを妨げないなら修正しない。**

よくある問題と対応:
- BCP081 (type not defined) → API バージョンが誤っている可能性。MS Docs を取得して実際の最新安定版に更新
- BCP036 (type mismatch) → プロパティ値の大文字小文字と型を確認して修正
- BCP037 (property not allowed) → その API バージョンで当該プロパティがサポートされるか MS Docs で確認
- no-hardcoded-env-urls → DNS Zone 名などのハードコード URL は Bicep ではやむを得ない場合がある。レビュー結果に記載

### ステップ 3: チェックリスト レビュー

コンパイル通過後に以下をレビューします。完全な gotchas は `references/service-gotchas.md` を参照してください。

#### Critical（必須修正）
- [ ] Microsoft Foundry `customSubDomainName` 設定が存在する — **作成後に変更不可。欠落している場合はリソースを削除して再作成が必要**
- [ ] Microsoft Foundry 使用時、**Foundry Project (`accounts/projects`) が必須** — ないとポータルアクセス不可
- [ ] Microsoft Foundry `identity: { type: 'SystemAssigned' }` — ないと Project 作成に失敗
- [ ] `publicNetworkAccess: 'Disabled'` — PE 使用サービスはすべて対象
- [ ] ADLS Gen2 `isHnsEnabled: true` — ないと通常の Blob Storage になる
- [ ] pe-subnet `privateEndpointNetworkPolicies: 'Disabled'` — ないと PE 作成に失敗
- [ ] Private DNS Zone Group — すべての PE に必須
- [ ] Key Vault `enablePurgeProtection: true`

#### High（推奨修正）
- [ ] Storage `allowBlobPublicAccess: false`, `minimumTlsVersion: 'TLS1_2'`
- [ ] Private DNS Zone VNet Link `registrationEnabled: false`
- [ ] サービスごとの resource type と kind 値が `references/ai-data.md` または MS Docs と一致
- [ ] モデル デプロイ: 順序保証（`dependsOn`）
- [ ] パラメータ ファイルに機微値がない — **見つけたら即時削除**

#### Medium（推奨）
- [ ] `uniqueString()` によるリソース名衝突防止
- [ ] リソース参照による暗黙的依存関係の活用

### ステップ 4: ハードコーディング回帰チェック（動的情報漏えい防止）

以下の項目が Bicep コード内でリテラル値としてハードコードされていないことを確認します。

#### パラメータ化必須（ハードコーディング禁止）
- [ ] `location` — リテラルのリージョン名（`'eastus'`, `'koreacentral'` など）を直接使用せず、`param location` で渡す
- [ ] モデル名/バージョン — リテラル禁止。Phase 1 で確認し、Step 0 で利用可能性を検証した値を使用
- [ ] SKU — ユーザー確認済みの値を使用

#### 動的値が参照に回帰していないことの確認
これは本レビューの直接対象外ですが、特定の API バージョン、SKU 一覧、リージョン一覧がコードコメントやパラメータ説明にハードコードされている場合は削除し、「MS Docs を確認」の案内に置き換えてください。

#### 判断ルール違反チェック
- [ ] Foundry ではなく `kind: 'OpenAI'` が使われている場合 → ユーザー明示指定がない限り `kind: 'AIServices'` に変更
- [ ] 一般 AI/RAG に Hub (`MachineLearningServices`) が使われている場合 → ユーザー明示指定がない限り Foundry に変更
- [ ] 単独の Azure OpenAI リソースが使われている場合 → ユーザー明示指定があるか Docs 上必要な場合を除き、Foundry 利用の再検討を提案

### ステップ 5: 修正後の再コンパイル

ステップ 2〜4 で変更を行った場合は、`az bicep build` を再実行して新たなエラーが入っていないことを確認します。

### `az bicep build` の制約

コンパイルで検証できるのは構文と型のみです。以下はコンパイルでは検出できず、最終的に Phase 4 の `az deployment group what-if` で確認されます。
- 廃止済み/利用不可 SKU
- リージョンごとのサービス提供可否
- モデル名の妥当性
- Preview 専用プロパティ
- サービスポリシー変更（クォータ、容量など）

what-if ステップの重要性をユーザーが理解できるよう、これらの制約をレビュー結果に明記してください。

### ステップ 6: 結果レポート

```markdown
## Bicep コードレビュー結果

**コンパイル結果**: [PASS/WARNING N items]
**チェックリスト**: ✅ Passed X items / ⚠️ Warnings X items
**ハードコーディングチェック**: [PASS / N violations]
**自動修正**: X items

### コンパイル警告（残件）
- [Warning content — including reason why it cannot be fixed]

### 自動修正の詳細
- [File:line number] Before → After (reason)

### ハードコーディング違反（該当時）
- [File:line number] [Violation details] → [Fix method]

**結論**: [Ready for deployment / Manual review required]
```

### ステップ 7: Phase 4 への移行 — 安心メッセージ必須

コードレビュー通過後に Phase 4 へ進むか確認する際は、**必ずユーザーを安心させるメッセージを含めてください**。  
ユーザーは「deployment」という語に不安を感じることがあるため、what-if は安全な検証ステップであることを明確に伝えてください。

```
ask_user({
  question: "Code review passed! Ready to proceed to the next step?\n\n⚡ This does NOT deploy immediately:\n  1️⃣ What-if validation — Simulates what will be created (not a deployment, safe)\n  2️⃣ Preview diagram — Review the architecture to be deployed as a diagram\n  3️⃣ Final confirmation — Actual deployment only after you review the diagram and approve\n\nNothing will be deployed without your approval.",
  choices: [
    "Proceed to next step (what-if validation + preview diagram) (Recommended)",
    "Just give me the code, I'll deploy later"
  ]
})
```

**重要ポイント:**
- 必ず「This does NOT deploy immediately」と明記する
- 3 ステップ（what-if → preview diagram → final confirmation）を説明する
- 「Nothing will be deployed without your approval」で安心感を与える

