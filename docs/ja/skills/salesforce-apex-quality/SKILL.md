---
name: salesforce-apex-quality
description: 'Apex code quality guardrails for Salesforce development. Enforces bulk-safety rules (no SOQL/DML in loops), sharing model requirements, CRUD/FLS security, SOQL injection prevention, PNB test coverage (Positive / Negative / Bulk), and modern Apex idioms. Use this skill when reviewing or generating Apex classes, trigger handlers, batch jobs, or test classes to catch governor limit risks, security gaps, and quality issues before deployment.'
---
# Salesforce Apex 品質のガードレール

これらのチェックを、作成またはレビューするすべての Apex クラス、トリガ、テストファイルに適用します。

## ステップ 1 — ガバナ制限の安全性チェック

Apex ファイルが受け入れ可能であると宣言する前に、次のパターンをスキャンしてください。

### SOQL と DML のループ - 自動失敗```apex
// ❌ NEVER — causes LimitException at scale
for (Account a : accounts) {
    List<Contact> contacts = [SELECT Id FROM Contact WHERE AccountId = :a.Id]; // SOQL in loop
    update a; // DML in loop
}

// ✅ ALWAYS — collect, then query/update once
Set<Id> accountIds = new Map<Id, Account>(accounts).keySet();
Map<Id, List<Contact>> contactsByAccount = new Map<Id, List<Contact>>();
for (Contact c : [SELECT Id, AccountId FROM Contact WHERE AccountId IN :accountIds]) {
    if (!contactsByAccount.containsKey(c.AccountId)) {
        contactsByAccount.put(c.AccountId, new List<Contact>());
    }
    contactsByAccount.get(c.AccountId).add(c);
}
update accounts; // DML once, outside the loop
```ルール: `for` ループ本体内に `[SELECT` または `Database.query`、`insert`、`update`、`delete`、`upsert`、`merge` が含まれている場合は、続行する前に停止してリファクタリングします。

## ステップ 2 — 共有モデルの検証

すべてのクラスは、その共有意図を明示的に宣言する必要があります。宣言されていない共有は呼び出し元から継承され、予測できない動作が発生します。

|宣言 |いつ使用するか |
|---|---|
| `public with sharing class Foo` |すべてのサービス、ハンドラー、セレクター、コントローラー クラスのデフォルト |
| `public without sharing class Foo` |クラスを昇格して実行する必要がある場合のみ (システムレベルのロギング、トリガーのバイパスなど)。理由を説明するコード コメントが必要です。 |
| `public inherited sharing class Foo` |呼び出し元の共有コンテキストを尊重する必要があるフレームワーク エントリ ポイント |

クラスにこれら 3 つの宣言のいずれかがない場合は、**他に何かを記述する前に宣言を追加してください**。

## ステップ 3 — CRUD / FLS の適用

ユーザに代わってレコードを読み書きする Apex コードは、オブジェクトと項目のアクセスを検証する必要があります。プラットフォームは、Apex で FLS または CRUD を自動的に強制しません**。```apex
// Check before querying a field
if (!Schema.sObjectType.Contact.fields.Email.isAccessible()) {
    throw new System.NoAccessException();
}

// Or use WITH USER_MODE in SOQL (API 56.0+)
List<Contact> contacts = [SELECT Id, Email FROM Contact WHERE AccountId = :accId WITH USER_MODE];

// Or use Database.query with AccessLevel
List<Contact> contacts = Database.query('SELECT Id, Email FROM Contact', AccessLevel.USER_MODE);
```ルール: UI コンポーネント、REST エンドポイント、または `@InvocableMethod` から呼び出し可能な Apex メソッドは CRUD/FLS を強制する必要があります。信頼されたコンテキストからのみ呼び出される内部サービス メソッドは、代わりに `with sharing` を使用する場合があります。

## ステップ 4 — SOQL インジェクションの防止```apex
// ❌ NEVER — concatenates user input into SOQL string
String soql = 'SELECT Id FROM Account WHERE Name = \'' + userInput + '\'';

// ✅ ALWAYS — bind variable
String soql = [SELECT Id FROM Account WHERE Name = :userInput];

// ✅ For dynamic SOQL with user-controlled field names — validate against a whitelist
Set<String> allowedFields = new Set<String>{'Name', 'Industry', 'AnnualRevenue'};
if (!allowedFields.contains(userInput)) {
    throw new IllegalArgumentException('Field not permitted: ' + userInput);
}
```## ステップ 5 — 現代の Apex イディオム

現在の言語機能を優先します (API 62.0 / Winter '25+):

|古いパターン |最新の代替品 |
|---|---|
| `if (obj != null) { x = obj.Field__c; }` | `x = obj?.Field__c;` |
| `x = (y != null) ? y : defaultVal;` | `x = y ?? defaultVal;` |
| `System.assertEquals(expected, actual)` | `Assert.areEqual(expected, actual)` |
| `System.assert(condition)` | `Assert.isTrue(condition)` |
| `[SELECT ... WHERE ...]` 共有コンテキストなし | `[SELECT ... WHERE ... WITH USER_MODE]` |

## ステップ 6 — PNB テスト カバレッジ チェックリスト

すべての機能は 3 つのパスすべてにわたってテストする必要があります。これらのいずれかが欠けていると、品質が低下します。

### ポジティブパス
- 期待される入力→期待される出力。
- 例外がスローされなかったことだけでなく、正確なフィールド値、レコード数、または戻り値をアサートします。

### ネガティブパス
- 無効な入力、NULL 値、空のコレクション、およびエラー状態。
- 例外が正しいタイプとメッセージでスローされることをアサートします。
- 操作が正常に失敗するはずのときに、レコードが変更されていないことをアサートします。

### バルクパス
- 単一のテスト トランザクションで **200 ～ 251 レコード** を挿入/更新/削除します。
- すべてのレコードが正しく処理されたことをアサートします。ガバナ制限による部分的なエラーはありません。
- `Test.startTest()` / `Test.stopTest()` を使用して、非同期作業用のガバナー制限カウンターを分離します。

### テストクラスのルール```apex
@isTest(SeeAllData=false)   // Required — no exceptions without a documented reason
private class AccountServiceTest {

    @TestSetup
    static void makeData() {
        // Create all test data here — use a factory if one exists in the project
    }

    @isTest
    static void givenValidInput_whenProcessAccounts_thenFieldsUpdated() {
        // Positive path
        List<Account> accounts = [SELECT Id FROM Account LIMIT 10];
        Test.startTest();
        AccountService.processAccounts(accounts);
        Test.stopTest();
        // Assert meaningful outcomes — not just no exception
        List<Account> updated = [SELECT Status__c FROM Account WHERE Id IN :accounts];
        Assert.areEqual('Processed', updated[0].Status__c, 'Status should be Processed');
    }
}
```## ステップ 7 — トリガー アーキテクチャのチェックリスト

- [ ] オブジェクトごとに 1 つのトリガー。 2 番目のトリガーが存在する場合は、ハンドラーに統合します。
- [ ] トリガー本体には、コンテキスト チェック、ハンドラー呼び出し、およびルーティング ロジックのみが含まれます。
- [ ] トリガー本体にビジネス ロジック、SOQL、または DML を直接組み込むことはできません。
- [ ] トリガー フレームワーク (トリガー アクション フレームワーク、ff-apex-common、カスタム基本クラス) がすでに使用されている場合は、それを拡張します。平行パターンを作成しないでください。
- [ ] ハンドラー クラスは、トリガーに昇格されたアクセスが必要でない限り `with sharing` です。

## クイック リファレンス — ハードコードされたアンチパターンの概要

|パターン |アクション |
|---|---|
| `for` ループ内の SOQL |リファクタリング: ループの前にクエリを実行し、コレクションを操作します。
| `for` ループ内の DML |リファクタリング: ミューテーションを収集し、ループ後に DML を 1 回実行します。
|クラスに共有宣言がありません | `with sharing` を追加 (または `without sharing` の理由を文書化) |
|ユーザー データ (VF) に関する `escape="false"` |削除 — 自動エスケープにより XSS 防止が強制されます。
|空の `catch` ブロック |ログ記録と適切な再スローまたはエラー処理を追加します。
|ユーザー入力を使用した文字列連結 SOQL |バインド変数またはホワイトリスト検証で置き換える |
|アサーションなしでテストする |意味のある `Assert.*` 呼び出しを追加します。
| `System.assert` / `System.assertEquals` スタイル | `Assert.isTrue` / `Assert.areEqual` にアップグレード |
|ハードコードされたレコード ID (`'001...'`) |クエリまたは挿入されたテスト レコード ID に置き換えます |