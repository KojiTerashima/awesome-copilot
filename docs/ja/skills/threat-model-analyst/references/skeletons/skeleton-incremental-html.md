# スケルトン：incremental-comparison.html

> **⛔ 自己完結型 HTML — すべての CSS インライン。 CDN リンクはありません。この 8 セクションの構造に従ってください。**

---

HTML レポートには、この順序でちょうど 8 つのセクションがあります。各セクションが存在する必要があります。

## セクション 1: ヘッダー + 比較カード```html
<div class="header">
  <div class="report-badge">INCREMENTAL THREAT MODEL COMPARISON</div>
  <h1>[FILL: repo name]</h1>
</div>
<div class="comparison-cards">
  <div class="compare-card baseline">
    <div class="card-label">BASELINE</div>
    <div class="card-hash">[FILL: baseline SHA]</div>
    <div class="card-date">[FILL: baseline commit date from git log]</div>
    <div class="risk-badge [FILL: old-class]">[FILL: old rating]</div>
  </div>
  <div class="compare-arrow">→</div>
  <div class="compare-card target">
    <div class="card-label">TARGET</div>
    <div class="card-hash">[FILL: target SHA]</div>
    <div class="card-date">[FILL: target commit date from git log]</div>
    <div class="risk-badge [FILL: new-class]">[FILL: new rating]</div>
  </div>
  <div class="compare-card trend">
    <div class="card-label">TREND</div>
    <div class="trend-direction [FILL: color]">[FILL: Improving / Worsening / Stable]</div>
    <div class="trend-duration">[FILL: N months]</div>
  </div>
</div>
```<!-- 基本手順: セクション 2 (リスクシフト) は上記のセクション 1 に統合されます。古い個別のリスクシフト div は削除されました。比較カード div は、古いサブタイトル + リスクシフト + 間隔ボックスの両方を置き換えます。 -->

## セクション 2: メトリック バー (5 ボックス)```html
<div class="metrics-bar">
  [FILL: Components: old → new (±N)]
  [FILL: Trust Boundaries: old → new (±N)]
  [FILL: Threats: old → new (±N)]
  [FILL: Findings: old → new (±N)]
  [FILL: Code Changes: N commits, M PRs — use git rev-list --count and git log --oneline --merges --grep="Merged PR"]
</div>
```**5 つの指標の 1 つとして信頼境界を含める必要があります。 5 番目のボックスはコード変更です (時間間隔ではありません)。**

## セクション 3: ステータス概要カード (色付き)```html
<div class="status-cards">
  <!-- Green card --> Fixed: [FILL: count] [FILL: 1-sentence summary, NO IDs]
  <!-- Red card --> New: [FILL: count] [FILL: 1-sentence summary, NO IDs]
  <!-- Amber card --> Previously Unidentified: [FILL: count] [FILL: 1-sentence summary, NO IDs]
  <!-- Gray card --> Still Present: [FILL: count] [FILL: 1-sentence summary, NO IDs]
</div>
```<!-- スケルトンの説明: ステータス カードには COUNT + 人間が読める短い文のみが表示されます。
  脅威 ID (T06.S、T02.E)、検出結果 ID (FIND-14)、またはコンポーネント名は含めないでください。
  良好: 「認証情報処理の脆弱性が 1 件修正されました」
  良い: 「21 の新しい脅威を含む 4 つの新しいコンポーネントが特定されました」
  良い: 「新たな脅威や調査結果は導入されていません」
  悪い: 「T06.S: DefaultAzureCredential → ManagedIdentityCredential」
  悪い: 「ConfigurationOrchestrator — 5 つの脅威 (T16.*)、LLMService — 6 つの脅威 (T17.*)」
  ID を含む項目ごとの詳細な内訳は、セクション 5 (脅威/調査状況の内訳) に属します。 -->
**ステータス情報はここにのみ表示され、メトリクス バーには表示されません。**

## セクション 4: コンポーネント ステータス グリッド```html
<table class="component-grid">
  <tr><th>Component</th><th>Type</th><th>Status</th><th>Source Files</th></tr>
  [REPEAT: one row per component with color-coded status badge]
  <tr><td>[FILL]</td><td>[FILL]</td><td><span class="badge-[FILL: status]">[FILL]</span></td><td>[FILL]</td></tr>
  [END-REPEAT]
</table>
```## セクション 5: 脅威/調査状況の内訳```html
<div class="status-breakdown">
  [FILL: Grouped by status — Fixed items, New items, etc.]
  [REPEAT: Each item: ID | Title | Component | Status]
  [END-REPEAT]
</div>
```## セクション 6: デルタを含む STRIDE ヒートマップ```html
<table class="stride-heatmap">
  <thead>
    <tr>
      <th>Component</th>
      <th>S</th><th>T</th><th>R</th><th>I</th><th>D</th><th>E</th><th>A</th>
      <th>Total</th>
      <th class="divider"></th>
      <th>T1</th><th>T2</th><th>T3</th>
    </tr>
  </thead>
  <tbody>
    [REPEAT: one row per component]
    <tr>
      <td>[FILL: component]</td>
      <td>[FILL: S value] [FILL: delta indicator ▲/▼]</td>
      ... [same for T, R, I, D, E, A, Total] ...
      <td class="divider"></td>
      <td>[FILL: T1]</td><td>[FILL: T2]</td><td>[FILL: T3]</td>
    </tr>
    [END-REPEAT]
  </tbody>
</table>
```**13 列が必要です: コンポーネント + S + T + R + I + D + E + A + 合計 + 除算器 + T1 + T2 + T3**

## セクション 7: 検証が必要```html
<div class="needs-verification">
  [REPEAT: items where analysis disagrees with old report]
  [FILL: item description]
  [END-REPEAT]
</div>
```## セクション 8: フッター```html
<div class="footer">
  Model: [FILL] | Duration: [FILL]
  Baseline: [FILL: folder] at [FILL: SHA]
  Generated: [FILL: timestamp]
</div>
```---

**修正された CSS 変数 (`<style>` ブロックで使用):**```css
--red: #dc3545;    /* new vulnerability */
--green: #28a745;  /* fixed/improved */
--amber: #fd7e14;  /* previously unidentified */
--gray: #6c757d;   /* still present */
--accent: #2171b5; /* modified/info */
```**修正されたルール:**
- インライン `<style>` ブロック内のすべての CSS — 外部スタイルシートなし
- `@media print` スタイルを含める
- ヒートマップにはディバイダーの後に T1/T2/T3 列が必要です
- メトリクスバーには信頼境界を含める必要があります
- カード内のステータス データのみ - メトリクス バーには複製されません
- HTML の脅威/検出結果の合計は、マークダウン STRIDE 概要の合計と一致する必要があります。