---
name: salesforce-component-standards
description: 'Quality standards for Salesforce Lightning Web Components (LWC), Aura components, and Visualforce pages. Covers SLDS 2 compliance, accessibility (WCAG 2.1 AA), data access pattern selection, component communication rules, XSS prevention, CSRF enforcement, FLS/CRUD in AuraEnabled methods, view state management, and Jest test requirements. Use this skill when building or reviewing any Salesforce UI component to enforce platform-specific security and quality standards.'
---
# Salesforce コンポーネントの品質基準

これらのチェックを、作成またはレビューするすべての LWC、Aura コンポーネント、および Visualforce ページに適用します。

## セクション 1 — LWC 品質基準

### 1.1 データアクセスパターンの選択

JavaScript コントローラー コードを記述する前に、適切なデータ アクセス パターンを選択してください。

|使用例 |パターン |なぜ |
|---|---|---|
|単一レコードを反応的に読み取ります (ナビゲーションに従います) | `@wire(getRecord, { recordId, fields })` | Lightning データ サービス — キャッシュ、リアクティブ |
|単一オブジェクトの標準 CRUD フォーム | `<lightning-record-form>` または `<lightning-record-edit-form>` |組み込みの FLS、CRUD、およびアクセシビリティ |
|複雑なサーバー クエリまたはフィルタリングされたリスト | `cacheable=true` メソッドの `@wire(apexMethodName, { param })` |キャッシュを許可します。パラメータ変更時にワイヤが再起動する |
|ユーザーがトリガーしたアクション、DML、またはキャッシュ不可能なサーバー呼び出し |命令型 `apexMethodName(params).then(...).catch(...)` | DML には必須 — 有線メソッドは `cacheable=true` なしで `@AuraEnabled` にすることはできません。
|コンポーネント間通信 (共有親なし) |ライトニング メッセージ サービス (LMS) |分離され、DOM の境界を越えて機能します |
|複数オブジェクトのグラフの関係 |グラフQL `@wire(gql, { query, variables })` |複雑な関連データの単一ラウンドトリップ |

### 1.2 セキュリティルール

|ルール |執行 |
|---|---|
| `innerHTML` に生のユーザー データがありません |テンプレートで `{expression}` バインディングを使用します。フレームワークは自動エスケープします。 `this.template.querySelector('.el').innerHTML = userValue` は決して使用しないでください。
| Apex `@AuraEnabled` メソッドは CRUD/FLS を強制します | SOQL で `WITH USER_MODE` を使用するか、明示的な `Schema.sObjectType` チェックを使用します。
|コンポーネント JavaScript に組織固有の ID をハードコードしない |クエリを実行するか、プロパティとして渡します。レコード ID をソースに埋め込まないでください。
|親からの `@api` プロパティ: 使用前に検証 |親は何でも渡すことができます。クエリ パラメーターとして使用する前に型と範囲を検証します。

### 1.3 SLDS 2 とスタイル標準

- **決して**ハードコード色: `color: #FF3366` → `color: var(--slds-c-button-brand-color-background)` またはセマンティック SLDS トークンを使用します。
- **決して** SLDS クラスを `!important` でオーバーライドしないでください。カスタム CSS プロパティを使用して作成します。
- `<lightning-*>` ベース コンポーネントが存在する場合は常に使用します: `lightning-button`、`lightning-input`、`lightning-datatable`、`lightning-card` など。
- 基本コンポーネントには、組み込みの SLDS 2、ダーク モード、およびアクセシビリティが含まれます。これらの動作の再実装は避けてください。
- カスタム CSS を使用している場合は、完了を宣言する前に **ライト モード** と **ダーク モード** の両方でテストしてください。

### 1.4 アクセシビリティ要件 (WCAG 2.1 AA)すべての LWC コンポーネントは、完了したとみなされる前に、次のすべてに合格する必要があります。

- [ ] すべてのフォーム入力には `<label>` または `aria-label` が含まれます。プレースホルダーを唯一のラベルとして使用しないでください。
- [ ] すべてのアイコンのみのボタンには、アクションを説明する `alternative-text` または `aria-label` が含まれています
- [ ] すべてのインタラクティブ要素はキーボード (Tab、Enter、Space、Escape) でアクセスおよび操作できます。
- [ ] ステータスを伝える唯一の手段ではありません - テキスト、アイコン、または `aria-*` 属性と組み合わせる
- [ ] エラー メッセージは、`aria-describedby` を介して入力に関連付けられます。
- [ ] モーダルでのフォーカス管理は正しい — フォーカスは開くとモーダルに移動し、閉じるとモーダルに戻ります。

### 1.5 コンポーネントの通信ルール

|方向 |メカニズム |
|---|---|
|親 → 子 | `@api` プロパティまたは `@api` メソッドの呼び出し |
|子→親 | `CustomEvent` — `this.dispatchEvent(new CustomEvent('eventname', { detail: data }))` |
|兄弟/無関係のコンポーネント |ライトニング メッセージ サービス (LMS) |
|決して使用しないでください | `document.querySelector`、`window.*`、または Pub/Sub ライブラリ |

フロー画面コンポーネントの場合:
- フロー ランタイムに到達する必要があるイベントは、`bubbles: true` および `composed: true` を設定する必要があります。
- Flow 変数との双方向バインディングのために `@api value` を公開します。

### 1.6 JavaScript パフォーマンス ルール

- **`connectedCallback`** には副作用はありません。DOM がアタッチされるたびに実行されます。ここでは DML、大量の計算、レンダリング状態の変更を避けてください。
- **ガード `renderedCallback`**: 無限のレンダリング ループを防ぐために、常にブール ガードを使用します。
- **リアクティブ プロパティ トラップを避ける**: `renderedCallback` 内にリアクティブ プロパティを設定すると、再レンダリングが発生します。必要で保護されている場合にのみ使用してください。
- **大きなデータセットをコンポーネント状態に保存しないでください** - 代わりに、大きな結果をページ分割するかストリーミングします。

### 1.7 Jest テストの要件

ユーザインタラクションを処理したり、Apex データを取得したりするすべてのコンポーネントには、Jest テストが必要です。```javascript
// Minimum test coverage expectations
it('renders the component with correct title', async () => { ... });
it('calls apex method and displays results', async () => { ... });  // Wire mock
it('dispatches event when button is clicked', async () => { ... });
it('shows error state when apex call fails', async () => { ... }); // Error path
````@salesforce/sfdx-lwc-jest` モック ユーティリティを使用します。
- `wire` アダプターのモック: `setImmediate` + `emit({ data, error })`
- Apex メソッドのモック: `jest.mock('@salesforce/apex/MyClass.myMethod', ...)`

---

## セクション 2 — Aura コンポーネントの標準

### 2.1 Aura と LWC を使用する場合

- **新しいコンポーネント: ターゲットコンテキストが Aura のみでない限り、常に LWC** (例: `force:appPage` の拡張、従来の管理パッケージでの Aura 固有のイベントの使用)。
- **Aura を LWC に移行**: LWC を優先し、コンポーネントごとに移行します。 LWC は Aura コンポーネント内に埋め込むことができます。

### 2.2 Aura セキュリティルール

- `@AuraEnabled` コントローラーメソッドは `with sharing` を宣言し、CRUD/FLS を強制する必要があります。Aura はこれらを自動的に強制しません**。
- `<div>` アンバインド ヘルパー内のエスケープされていないユーザー データでは `{!v.something}` を使用しないでください。エスケープするには `<ui:outputText value="{!v.text}" />` または `<c:something>` を使用してください。
- SOQL / Apex ロジックで使用する前に、コンポーネント属性からのすべての入力を検証します。

### 2.3 Aura イベントの設計

- 親子通信の **コンポーネント イベント** - 最も低い範囲。
- **アプリケーション イベント**は、コンポーネント イベントがターゲットに到達できない場合にのみ発生します。イベントはアプリ全体にブロードキャストされ、パフォーマンスとメンテナンスの問題になる可能性があります。
- ハイブリッド LWC + Aura スタックの場合: Lightning メッセージ サービスを使用して通信を分離します。LWC コンポーネントに到達する Aura アプリケーション イベントに依存しません。

---

## セクション 3 — Visualforce セキュリティ標準

### 3.1 XSS の防止```xml
<!-- ❌ NEVER — renders raw user input as HTML -->
<apex:outputText value="{!userInput}" escape="false" />

<!-- ✅ ALWAYS — auto-escaping on -->
<apex:outputText value="{!userInput}" />
<!-- Default escape="true" — platform HTML-encodes the output -->
```ルール: `escape="false"` は、ユーザー制御のデータには決して受け入れられません。リッチ テキストをレンダリングする必要がある場合は、出力前にホワイトリストを使用してサーバー側をサニタイズします。

### 3.2 CSRF 保護

すべてのポストバック アクションには `<apex:form>` を使用します。プラットフォームは CSRF トークンをフォームに自動的に挿入します。 CSRF 保護をバイパスする生の `<form method="POST">` HTML 要素は**使用しないでください**。

### 3.3 コントローラでの SOQL インジェクションの防止```apex
// ❌ NEVER
String soql = 'SELECT Id FROM Account WHERE Name = \'' + ApexPages.currentPage().getParameters().get('name') + '\'';
List<Account> results = Database.query(soql);

// ✅ ALWAYS — bind variable
String nameParam = ApexPages.currentPage().getParameters().get('name');
List<Account> results = [SELECT Id FROM Account WHERE Name = :nameParam];
```### 3.4 状態管理チェックリストの表示

- [ ] ビューステートが 135 KB 未満である (ブラウザ開発者ツールまたは Salesforce の [ビューステート] タブで確認してください)
- [ ] サーバー側の計算のみに使用されるフィールドは `transient` として宣言されます
- [ ] 大規模なコレクションがポストバック間で不必要に保持されない
- [ ] `readonly="true"` は、ビューステートのシリアル化をスキップするために読み取り専用ページの `<apex:page>` に設定されます

### 3.5 Visualforce コントローラの FLS / CRUD```apex
// Before reading a field
if (!Schema.sObjectType.Account.fields.Revenue__c.isAccessible()) {
    ApexPages.addMessage(new ApexPages.Message(ApexPages.Severity.ERROR, 'You do not have access to this field.'));
    return null;
}

// Before performing DML
if (!Schema.sObjectType.Account.isDeletable()) {
    throw new System.NoAccessException();
}
```標準コントローラーは、バインドされたフィールドに対して FLS を自動的に適用します。 **カスタム コントローラーには適用されません** — FLS は手動で適用する必要があります。

---

## クイック リファレンス — コンポーネントのアンチパターンの概要

|アンチパターン |テクノロジー |リスク |修正 |
|---|---|---|---|
| `innerHTML` とユーザー データ | ＬＷＣ | XSS |テンプレート バインディングを使用する `{expression}` |
|ハードコードされた 16 進数の色 | LWC/オーラ |ダークモード / SLDS 2 ブレーク | SLDS CSS カスタム プロパティを使用する |
|アイコン ボタンに `aria-label` がありません。 LWC/オーラ/VF |アクセシビリティの失敗 | `alternative-text` または `aria-label` を追加 |
| `renderedCallback` にはガードがありません | ＬＷＣ |無限の再レンダリング ループ | `hasRendered` ブール値ガードを追加 |
|親子で応募イベント |オーラ |不要なブロードキャスト スコープ |代わりにコンポーネント イベントを使用してください。
| `escape="false"` ユーザーデータ |ビジュアルフォース | XSS |削除 — デフォルトのエスケープを使用 |
|生の `<form>` ポストバック |ビジュアルフォース | CSRF の脆弱性 | `<apex:form>` を使用してください。
|カスタム コントローラに `with sharing` がありません | VF/アペックス |データ漏洩 | `with sharing` 宣言を追加 |
|カスタム コントローラーで FLS がチェックされていません | VF/アペックス |権限昇格 | `Schema.sObjectType` チェックを追加 |
| URL パラメータと連結された SOQL | VF/アペックス | SOQL インジェクション |バインド変数を使用する |