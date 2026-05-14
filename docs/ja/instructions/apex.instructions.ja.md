---
description: 'Salesforce プラットフォームでの Apex 開発のガイドラインとベストプラクティス'
applyTo: '**/*.cls, **/*.trigger'
---

# Apex開発

## 一般的な指示

- 常に最新の Apex 機能と Salesforce プラットフォームのベストプラクティスを使用してください。
- 各クラスとメソッドについて、ビジネスロジックと複雑な操作を説明する明確かつ簡潔なコメントを記述します。
- エッジケースを処理し、意味のあるエラーメッセージを含む適切な例外処理を実装します。
- バルク化に焦点を当てます。単一のレコードではなく、レコードのコレクションを処理するコードを作成します。
- ガバナ制限に留意し、効率的に拡張できるソリューションを設計してください。
- サービス層、ドメインクラス、セレクタークラスを使用して、懸念事項の適切な分離を実装します。
- 外部依存関係、統合ポイント、およびそれらの目的をコメントで文書化します。

## 命名規則

- **クラス**: クラス名には `PascalCase` を使用します。クラスには、その目的を反映したわかりやすい名前を付けます。
  - コントローラ: `Controller` の接尾辞 (例: `AccountController`)
  - トリガーハンドラー: `TriggerHandler` のサフィックス (例: `AccountTriggerHandler`)
  - サービスクラス: `Service` のサフィックス (例: `AccountService`)
  - セレクタークラス: `Selector` のサフィックス (例: `AccountSelector`)
  - テストクラス: `Test` の接尾辞 (例: `AccountServiceTest`)
  - バッチクラス: `Batch` の接尾辞 (例: `AccountCleanupBatch`)
  - キュー可能クラス: `Queueable` のサフィックス (例: `EmailNotificationQueueable`)

- **メソッド**: メソッド名には `camelCase` を使用します。動作を示すには動詞を使用します。
  - 良い: `getActiveAccounts()`、`updateContactEmail()`、`deleteExpiredRecords()`
  - 略語は避けてください: `getAccs()` → `getAccounts()`

- **変数**: 変数名には `camelCase` を使用します。わかりやすい名前を使用してください。
  - 良い: `accountList`、`emailAddress`、`totalAmount`
  - ループカウンター以外の単一文字は避けてください: `a` → `account`

- **定数**: 定数には `UPPER_SNAKE_CASE` を使用します。
  - 良い: `MAX_BATCH_SIZE`、`DEFAULT_EMAIL_TEMPLATE`、`ERROR_MESSAGE_PREFIX`

- **トリガー**: トリガーに `ObjectName` + トリガーイベントという名前を付けます (例: `AccountTrigger`、`ContactTrigger`)

## ベストプラクティス

### バルク化

- **常に一括化されたコードを作成します** - 個々のレコードではなく、レコードのコレクションを処理するようにすべてのコードを設計します。
- ループ内での SOQL クエリと DML ステートメントは避けてください。
- コレクション (`List<>`、`Set<>`、`Map<>`) を使用して、複数のレコードを効率的に処理します。

```apex
// Good Example - Bulkified
public static void updateAccountRating(List<Account> accounts) {
    for (Account acc : accounts) {
        if (acc.AnnualRevenue > 1000000) {
            acc.Rating = 'Hot';
        }
    }
    update accounts;
}

// Bad Example - Not bulkified
public static void updateAccountRating(Account account) {
    if (account.AnnualRevenue > 1000000) {
        account.Rating = 'Hot';
        update account; // DML in a method designed for single records
    }
}
```

### O(1) ルックアップのマップ

- **効率的なルックアップにマップを使用する** - O(n) リスト反復ではなく O(1) 定数時間ルックアップ用にリストをマップに変換します。
- `Map<Id, SObject>` コンストラクターを使用して、クエリ結果をマップにすばやく変換します。
- 関連するレコードの照合、検索、入れ子のループの回避に最適です。

```apex
// Good Example - Using Map for O(1) lookup
Map<Id, Account> accountMap = new Map<Id, Account>([
    SELECT Id, Name, Industry FROM Account WHERE Id IN :accountIds
]);

for (Contact con : contacts) {
    Account acc = accountMap.get(con.AccountId);
    if (acc != null) {
        con.Industry__c = acc.Industry;
    }
}

// Bad Example - Nested loop with O(n²) complexity
List<Account> accounts = [SELECT Id, Name, Industry FROM Account WHERE Id IN :accountIds];

for (Contact con : contacts) {
    for (Account acc : accounts) {
        if (con.AccountId == acc.Id) {
            con.Industry__c = acc.Industry;
            break;
        }
    }
}

// Good Example - Map for grouping records
Map<Id, List<Contact>> contactsByAccountId = new Map<Id, List<Contact>>();
for (Contact con : contacts) {
    if (!contactsByAccountId.containsKey(con.AccountId)) {
        contactsByAccountId.put(con.AccountId, new List<Contact>());
    }
    contactsByAccountId.get(con.AccountId).add(con);
}
```

### ガバナ制限

- Salesforce のガバナ制限に注意してください: SOQL クエリ (100)、DML ステートメント (150)、ヒープサイズ (6MB)、CPU 時間 (10 秒)。
- **`System.Limits` クラスを使用してガバナ制限を積極的に監視し**、制限に達する前に消費量をチェックします。
- 選択フィルタと適切なインデックスを備えた効率的な SOQL クエリを使用します。
- 大規模なデータセットを処理するために **SOQL for ループ** を実装します。
- 大量のデータ (50,000 レコードを超える) の操作には **Batch Apex** を使用してください。
- **プラットフォームキャッシュ**を活用して、冗長な SOQL クエリを削減します。

```apex
// Good Example - SOQL for loop for large data sets
public static void processLargeDataSet() {
    for (List<Account> accounts : [SELECT Id, Name FROM Account]) {
        // Process batch of 200 records
        processAccounts(accounts);
    }
}

// Good Example - Using WHERE clause to reduce query results
List<Account> accounts = [SELECT Id, Name FROM Account WHERE IsActive__c = true LIMIT 200];
```

### セキュリティとデータアクセス

- **SOQL クエリまたは DML 操作を実行する前に、必ず CRUD/FLS 権限を確認してください**。
- SOQL クエリで `WITH SECURITY_ENFORCED` を使用して、項目レベルのセキュリティを強制します。
- ユーザーがアクセスできないフィールドを削除するには、`Security.stripInaccessible()` を使用します。
- 共有ルールを適用するクラスには `WITH SHARING` キーワードを実装します。
- `WITHOUT SHARING` は必要な場合にのみ使用し、その理由を文書化してください。
- 呼び出しコンテキストを継承するには、ユーティリティクラスに `INHERITED SHARING` を使用します。

```apex
// Good Example - Checking CRUD and using stripInaccessible
public with sharing class AccountService {
    public static List<Account> getAccounts() {
        if (!Schema.sObjectType.Account.isAccessible()) {
            throw new SecurityException('User does not have access to Account object');
        }

        List<Account> accounts = [SELECT Id, Name, Industry FROM Account WITH SECURITY_ENFORCED];

        SObjectAccessDecision decision = Security.stripInaccessible(
            AccessType.READABLE, accounts
        );

        return decision.getRecords();
    }
}

// Good Example - WITH SHARING for sharing rules
public with sharing class AccountController {
    // This class enforces record-level sharing
}
```

### 例外処理

- DML 操作とコールアウトには、常に try-catch ブロックを使用します。
- 特定のエラーシナリオ用のカスタム例外クラスを作成します。
- デバッグと監視のために例外を適切に記録します。
- ユーザーに意味のあるエラーメッセージを提供します。

```apex
// Good Example - Proper exception handling
public class AccountService {
    public class AccountServiceException extends Exception {}

    public static void safeUpdate(List<Account> accounts) {
        try {
            if (!Schema.sObjectType.Account.isUpdateable()) {
                throw new AccountServiceException('User does not have permission to update accounts');
            }
            update accounts;
        } catch (DmlException e) {
            System.debug(LoggingLevel.ERROR, 'DML Error: ' + e.getMessage());
            throw new AccountServiceException('Failed to update accounts: ' + e.getMessage());
        }
    }
}
```

### SOQL のベストプラクティス

- インデックス付きフィールド (`Id`、`Name`、`OwnerId`、カスタムインデックス付きフィールド) で選択クエリを使用します。
- 必要に応じて `LIMIT` 句を使用してクエリ結果を制限します。
- レコードが 1 つだけ必要な場合は、`LIMIT 1` を使用します。
- `SELECT *` は避けてください。常に必須フィールドを指定してください。
- リレーションシップクエリを使用して、SOQL クエリの数を最小限に抑えます。
- 可能な場合は、インデックス付きフィールドごとにクエリを順序付けします。
- **SOQL インジェクション攻撃を防ぐために、SOQL クエリでユーザー入力を使用する場合は、常に `String.escapeSingleQuotes()`** を使用してください。
- **クエリの選択性を確認します** - 10% を超える選択性を目指します (フィルターにより、結果が合計レコードの 10% 未満に減ります)。
- **クエリプラン**を使用して、クエリの効率とインデックスの使用状況を確認します。
- 現実的なデータ量でクエリをテストして、パフォーマンスを確認します。

```apex
// Good Example - Selective query with indexed fields
List<Account> accounts = [
    SELECT Id, Name, (SELECT Id, LastName FROM Contacts)
    FROM Account
    WHERE OwnerId = :UserInfo.getUserId()
    AND CreatedDate = THIS_MONTH
    LIMIT 100
];

// Good Example - LIMIT 1 for single record
Account account = [SELECT Id, Name FROM Account WHERE Name = 'Acme' LIMIT 1];

// Good Example - escapeSingleQuotes() to prevent SOQL injection
String searchTerm = String.escapeSingleQuotes(userInput);
List<Account> accounts = Database.query('SELECT Id, Name FROM Account WHERE Name LIKE \'%' + searchTerm + '%\'');

// Bad Example - Direct user input without escaping (SECURITY RISK)
List<Account> accounts = Database.query('SELECT Id, Name FROM Account WHERE Name LIKE \'%' + userInput + '%\'');

// Good Example - Selective query with indexed fields (high selectivity)
List<Account> accounts = [
    SELECT Id, Name FROM Account
    WHERE OwnerId = :UserInfo.getUserId()
    AND CreatedDate = TODAY
    LIMIT 100
];

// Bad Example - Non-selective query (scans entire table)
List<Account> accounts = [
    SELECT Id, Name FROM Account
    WHERE Description LIKE '%test%'  // Non-indexed field
];

// Check query performance in Developer Console:
// 1. Enable 'Use Query Plan' in Developer Console
// 2. Run SOQL query and review 'Query Plan' tab
// 3. Look for 'Index' usage vs 'TableScan'
// 4. Ensure selectivity > 10% for optimal performance
```

### トリガーのベストプラクティス

- 明確さを維持し、競合を避けるために、**オブジェクトごとに 1 つのトリガー**を使用します。
- トリガーロジックをトリガーに直接実装するのではなく、ハンドラークラスに実装します。
- 一貫したトリガー管理のためにトリガーフレームワークを使用します。
- トリガーコンテキスト変数を活用します: `Trigger.new`、`Trigger.old`、`Trigger.newMap`、`Trigger.oldMap`。
- トリガーコンテキストを確認してください: `Trigger.isBefore`、`Trigger.isAfter`、`Trigger.isInsert` など。

```apex
// Good Example - Trigger with handler pattern
trigger AccountTrigger on Account (before insert, before update, after insert, after update) {
    new AccountTriggerHandler().run();
}

// Handler Class
public class AccountTriggerHandler extends TriggerHandler {
    private List<Account> newAccounts;
    private List<Account> oldAccounts;
    private Map<Id, Account> newAccountMap;
    private Map<Id, Account> oldAccountMap;

    public AccountTriggerHandler() {
        this.newAccounts = (List<Account>) Trigger.new;
        this.oldAccounts = (List<Account>) Trigger.old;
        this.newAccountMap = (Map<Id, Account>) Trigger.newMap;
        this.oldAccountMap = (Map<Id, Account>) Trigger.oldMap;
    }

    public override void beforeInsert() {
        AccountService.setDefaultValues(newAccounts);
    }

    public override void afterUpdate() {
        AccountService.handleRatingChange(newAccountMap, oldAccountMap);
    }
}
```

### コード品質のベストプラクティス

- **`isEmpty()`** を使用する - サイズ比較の代わりに組み込みメソッドを使用して、コレクションが空かどうかを確認します。
- **カスタムラベルを使用する** - 国際化と保守性を高めるために、ユーザーに表示されるテキストをカスタムラベルに保存します。
- **定数を使用** - ハードコードされた値、エラーメッセージ、構成値の定数を定義します。
- **`String.isBlank()` と `String.isNotBlank()`** を使用する - null または空の文字列を適切にチェックします。
- **`String.valueOf()`** を使用する - null ポインター例外を回避するために、値を文字列に安全に変換します。
- **安全なナビゲーション演算子 `?.`** を使用する - null ポインター例外を発生させずに、プロパティとメソッドに安全にアクセスします。
- **null 合体演算子を使用する `??`** - null 式のデフォルト値を提供します。
- **ループ内の文字列連結には `+` の使用を避けてください** - パフォーマンスを向上させるには `String.join()` を使用してください。
- **コレクションメソッドを使用する** - よりクリーンなコードのために `List.clone()`、`Set.addAll()`、`Map.keySet()` を活用します。
- **三項演算子を使用します** - 読みやすさを向上させるための単純な条件付き代入の場合。
- **switch 式を使用する** - 読みやすさとパフォーマンスを向上させるための、if-else チェーンの最新の代替手段。
- **SObject クローンメソッドを使用する** - 意図しない参照を避けるために、必要に応じて SObject を適切にクローンします。

```apex
// Good Example - Switch expression (modern Apex)
String rating = switch on account.AnnualRevenue {
    when 0 { 'Cold'; }
    when 1, 2, 3 { 'Warm'; }
    when else { 'Hot'; }
};

// Good Example - Switch on SObjectType
String objectLabel = switch on record {
    when Account a { 'Account: ' + a.Name; }
    when Contact c { 'Contact: ' + c.LastName; }
    when else { 'Unknown'; }
};

// Bad Example - if-else chain
String rating;
if (account.AnnualRevenue == 0) {
    rating = 'Cold';
} else if (account.AnnualRevenue >= 1 && account.AnnualRevenue <= 3) {
    rating = 'Warm';
} else {
    rating = 'Hot';
}

// Good Example - SObject clone methods
Account original = new Account(Name = 'Acme', Industry = 'Technology');

// Shallow clone with ID and relationships
Account clone1 = original.clone(true, true);

// Shallow clone without ID or relationships
Account clone2 = original.clone(false, false);

// Deep clone with all relationships
Account clone3 = original.deepClone(true, true, true);

// Good Example - isEmpty() instead of size comparison
if (accountList.isEmpty()) {
    System.debug('No accounts found');
}

// Bad Example - size comparison
if (accountList.size() == 0) {
    System.debug('No accounts found');
}

// Good Example - Custom Labels for user-facing text
final String ERROR_MESSAGE = System.Label.Account_Update_Error;
final String SUCCESS_MESSAGE = System.Label.Account_Update_Success;

// Bad Example - Hardcoded strings
final String ERROR_MESSAGE = 'An error occurred while updating the account';

// Good Example - Constants for configuration values
public class AccountService {
    private static final Integer MAX_RETRY_ATTEMPTS = 3;
    private static final String DEFAULT_INDUSTRY = 'Technology';
    private static final String ERROR_PREFIX = 'AccountService Error: ';

    public static void processAccounts() {
        // Use constants
        if (retryCount > MAX_RETRY_ATTEMPTS) {
            throw new AccountServiceException(ERROR_PREFIX + 'Max retries exceeded');
        }
    }
}

// Good Example - isBlank() for null and empty checks
if (String.isBlank(account.Name)) {
    account.Name = DEFAULT_NAME;
}

// Bad Example - multiple null checks
if (account.Name == null || account.Name == '') {
    account.Name = DEFAULT_NAME;
}

// Good Example - String.valueOf() for safe conversion
String accountId = String.valueOf(account.Id);
String revenue = String.valueOf(account.AnnualRevenue);

// Good Example - Safe navigation operator (?.)
String ownerName = account?.Owner?.Name;
Integer contactCount = account?.Contacts?.size();

// Bad Example - Nested null checks
String ownerName;
if (account != null && account.Owner != null) {
    ownerName = account.Owner.Name;
}

// Good Example - Null-coalescing operator (??)
String accountName = account?.Name ?? 'Unknown Account';
Integer revenue = account?.AnnualRevenue ?? 0;
String industry = account?.Industry ?? DEFAULT_INDUSTRY;

// Bad Example - Ternary with null check
String accountName = account != null && account.Name != null ? account.Name : 'Unknown Account';

// Good Example - Combining ?. and ??
String email = contact?.Email ?? contact?.Account?.Owner?.Email ?? 'no-reply@example.com';

// Good Example - String concatenation in loops
List<String> accountNames = new List<String>();
for (Account acc : accounts) {
    accountNames.add(acc.Name);
}
String result = String.join(accountNames, ', ');

// Bad Example - String concatenation in loops
String result = '';
for (Account acc : accounts) {
    result += acc.Name + ', '; // Poor performance
}

// Good Example - Ternary operator
String status = isActive ? 'Active' : 'Inactive';

// Good Example - Collection methods
List<Account> accountsCopy = accountList.clone();
Set<Id> accountIds = new Set<Id>(accountMap.keySet());
```

### 再発防止

- **静的変数を使用**して再帰呼び出しを追跡し、無限ループを防ぎます。
- **サーキットブレーカー** パターンを実装して、しきい値を超えた後に実行を停止します。
- 再帰の制限と潜在的なリスクを文書化します。

```apex
// Good Example - Recursion prevention with static variable
public class AccountTriggerHandler extends TriggerHandler {
    private static Boolean hasRun = false;

    public override void afterUpdate() {
        if (!hasRun) {
            hasRun = true;
            AccountService.updateRelatedContacts(Trigger.newMap.keySet());
        }
    }
}

// Good Example - Circuit breaker with counter
public class OpportunityService {
    private static Integer recursionCount = 0;
    private static final Integer MAX_RECURSION_DEPTH = 5;

    public static void processOpportunity(Id oppId) {
        recursionCount++;

        if (recursionCount > MAX_RECURSION_DEPTH) {
            System.debug(LoggingLevel.ERROR, 'Max recursion depth exceeded');
            return;
        }

        try {
            // Process opportunity logic
        } finally {
            recursionCount--;
        }
    }
}
```

### メソッドの可視性とカプセル化

- **デフォルトでは `private` を使用します** - パブリックにする必要があるメソッドのみを公開します。
- サブクラスがアクセスする必要があるメソッドには `protected` を使用します。
- `public` は、他のクラスが呼び出す必要がある API にのみ使用してください。
- **必要に応じて `final` キーワードを使用してメソッドのオーバーライドを防止します。
- 拡張すべきでないクラスには `final` のマークを付けます。

```apex
// Good Example - Proper encapsulation
public class AccountService {
    // Public API
    public static void updateAccounts(List<Account> accounts) {
        validateAccounts(accounts);
        performUpdate(accounts);
    }

    // Private helper - not exposed
    private static void validateAccounts(List<Account> accounts) {
        for (Account acc : accounts) {
            if (String.isBlank(acc.Name)) {
                throw new IllegalArgumentException('Account name is required');
            }
        }
    }

    // Private implementation - not exposed
    private static void performUpdate(List<Account> accounts) {
        update accounts;
    }
}

// Good Example - Final keyword to prevent extension
public final class UtilityHelper {
    // Cannot be extended
    public static String formatCurrency(Decimal amount) {
        return '$' + amount.setScale(2);
    }
}

// Good Example - Final method to prevent override
public virtual class BaseService {
    // Can be overridden
    public virtual void process() {
        // Implementation
    }

    // Cannot be overridden
    public final void validateInput() {
        // Critical validation that must not be changed
    }
}
```

### デザインパターン

- **サービスレイヤー パターン**: ビジネスロジックをサービスクラスにカプセル化します。
- **サーキットブレーカー パターン**: しきい値の後に実行を停止することで、失敗の繰り返しを防ぎます。
- **セレクタパターン**: SOQL クエリ専用のクラスを作成します。
- **ドメインレイヤー パターン**: レコード固有のロジックのドメインクラスを実装します。
- **トリガーハンドラー パターン**: トリガー管理には一貫したフレームワークを使用します。
- **ビルダーパターン**: 複雑なオブジェクトの構築に使用します。
- **戦略パターン**: 条件に基づいてさまざまな動作を実装します。

```apex
// Good Example - Service Layer Pattern
public class AccountService {
    public static void updateAccountRatings(Set<Id> accountIds) {
        List<Account> accounts = AccountSelector.selectByIds(accountIds);

        for (Account acc : accounts) {
            acc.Rating = calculateRating(acc);
        }

        update accounts;
    }

    private static String calculateRating(Account acc) {
        if (acc.AnnualRevenue > 1000000) {
            return 'Hot';
        } else if (acc.AnnualRevenue > 500000) {
            return 'Warm';
        }
        return 'Cold';
    }
}

// Good Example - Circuit Breaker Pattern
public class ExternalServiceCircuitBreaker {
    private static Integer failureCount = 0;
    private static final Integer FAILURE_THRESHOLD = 3;
    private static DateTime circuitOpenedTime;
    private static final Integer RETRY_TIMEOUT_MINUTES = 5;

    public static Boolean isCircuitOpen() {
        if (circuitOpenedTime != null) {
            // Check if retry timeout has passed
            if (DateTime.now() > circuitOpenedTime.addMinutes(RETRY_TIMEOUT_MINUTES)) {
                // Reset circuit
                failureCount = 0;
                circuitOpenedTime = null;
                return false;
            }
            return true;
        }
        return failureCount >= FAILURE_THRESHOLD;
    }

    public static void recordFailure() {
        failureCount++;
        if (failureCount >= FAILURE_THRESHOLD) {
            circuitOpenedTime = DateTime.now();
            System.debug(LoggingLevel.ERROR, 'Circuit breaker opened due to failures');
        }
    }

    public static void recordSuccess() {
        failureCount = 0;
        circuitOpenedTime = null;
    }

    public static HttpResponse makeCallout(String endpoint) {
        if (isCircuitOpen()) {
            throw new CircuitBreakerException('Circuit is open. Service unavailable.');
        }

        try {
            HttpRequest req = new HttpRequest();
            req.setEndpoint(endpoint);
            req.setMethod('GET');
            HttpResponse res = new Http().send(req);

            if (res.getStatusCode() == 200) {
                recordSuccess();
            } else {
                recordFailure();
            }
            return res;
        } catch (Exception e) {
            recordFailure();
            throw e;
        }
    }

    public class CircuitBreakerException extends Exception {}
}

// Good Example - Selector Pattern
public class AccountSelector {
    public static List<Account> selectByIds(Set<Id> accountIds) {
        return [
            SELECT Id, Name, AnnualRevenue, Rating
            FROM Account
            WHERE Id IN :accountIds
            WITH SECURITY_ENFORCED
        ];
    }

    public static List<Account> selectActiveAccountsWithContacts() {
        return [
            SELECT Id, Name, (SELECT Id, LastName FROM Contacts)
            FROM Account
            WHERE IsActive__c = true
            WITH SECURITY_ENFORCED
        ];
    }
}
```

### 構成管理

#### カスタムメタデータ タイプとカスタム設定

- **展開可能な構成データにはカスタムメタデータ タイプ (CMT)** を優先します。
- 環境によって異なるユーザー固有または組織固有のデータには、**カスタム設定** を使用します。
- CMT はパッケージ化、展開可能であり、検証ルールや式で使用できます。
- カスタム設定は階層 (組織、プロファイル、ユーザー) をサポートしますが、展開できません。

```apex
// Good Example - Using Custom Metadata Type
List<API_Configuration__mdt> configs = [
    SELECT Endpoint__c, Timeout__c, Max_Retries__c
    FROM API_Configuration__mdt
    WHERE DeveloperName = 'Production_API'
    LIMIT 1
];

if (!configs.isEmpty()) {
    String endpoint = configs[0].Endpoint__c;
    Integer timeout = Integer.valueOf(configs[0].Timeout__c);
}

// Good Example - Using Custom Settings (user-specific)
User_Preferences__c prefs = User_Preferences__c.getInstance(UserInfo.getUserId());
Boolean darkMode = prefs.Dark_Mode_Enabled__c;

// Good Example - Using Custom Settings (org-level)
Org_Settings__c orgSettings = Org_Settings__c.getOrgDefaults();
Integer maxRecords = Integer.valueOf(orgSettings.Max_Records_Per_Query__c);
```

#### 名前付き資格情報とHTTPコールアウト

- **外部 API エンドポイントと認証には常に名前付き資格情報を使用してください**。
- コード内で URL、トークン、または資格情報をハードコーディングしないでください。
- 安全でデプロイ可能な統合を実現するには、`callout:NamedCredential` 構文を使用します。
- **常に HTTP ステータスコードを確認し**、エラーを適切に処理してください。
- コールアウトの長時間実行を防ぐために、適切なタイムアウトを設定します。
- Queueable クラスと Batchable クラスには `Database.AllowsCallouts` インターフェイスを使用します。

```apex
// Good Example - Using Named Credentials
public class ExternalAPIService {
    private static final String NAMED_CREDENTIAL = 'callout:External_API';
    private static final Integer TIMEOUT_MS = 120000; // 120 seconds

    public static Map<String, Object> getExternalData(String recordId) {
        HttpRequest req = new HttpRequest();
        req.setEndpoint(NAMED_CREDENTIAL + '/api/records/' + recordId);
        req.setMethod('GET');
        req.setTimeout(TIMEOUT_MS);
        req.setHeader('Content-Type', 'application/json');

        try {
            Http http = new Http();
            HttpResponse res = http.send(req);

            if (res.getStatusCode() == 200) {
                return (Map<String, Object>) JSON.deserializeUntyped(res.getBody());
            } else if (res.getStatusCode() == 404) {
                throw new NotFoundException('Record not found: ' + recordId);
            } else if (res.getStatusCode() >= 500) {
                throw new ServiceUnavailableException('External service error: ' + res.getStatus());
            } else {
                throw new CalloutException('Unexpected response: ' + res.getStatusCode());
            }
        } catch (System.CalloutException e) {
            System.debug(LoggingLevel.ERROR, 'Callout failed: ' + e.getMessage());
            throw new ExternalAPIException('Failed to retrieve data', e);
        }
    }

    public class ExternalAPIException extends Exception {}
    public class NotFoundException extends Exception {}
    public class ServiceUnavailableException extends Exception {}
}

// Good Example - POST request with JSON body
public static String createExternalRecord(Map<String, Object> data) {
    HttpRequest req = new HttpRequest();
    req.setEndpoint(NAMED_CREDENTIAL + '/api/records');
    req.setMethod('POST');
    req.setTimeout(TIMEOUT_MS);
    req.setHeader('Content-Type', 'application/json');
    req.setBody(JSON.serialize(data));

    HttpResponse res = new Http().send(req);

    if (res.getStatusCode() == 201) {
        Map<String, Object> result = (Map<String, Object>) JSON.deserializeUntyped(res.getBody());
        return (String) result.get('id');
    } else {
        throw new CalloutException('Failed to create record: ' + res.getStatus());
    }
}
```

### 共通の注釈

- `@AuraEnabled` - Lightning Web コンポーネントおよび Aura コンポーネントにメソッドを公開します。
- `@AuraEnabled(cacheable=true)` - 読み取り専用メソッドのクライアント側キャッシュを有効にします。
- `@InvocableMethod` - フローおよびプロセスビルダーからメソッドを呼び出し可能にします。
- `@InvocableVariable` - 呼び出し可能なメソッドの入出力パラメータを定義します。
- `@TestVisible` - プライベートメンバーをテストクラスのみに公開します。
- `@SuppressWarnings('PMD.RuleName')` - 特定の PMD 警告を抑制します。
- `@RemoteAction` - Visualforce JavaScript リモート処理のメソッドを公開します (レガシー)。
- `@Future` - メソッドを非同期に実行します。
- `@Future(callout=true)` - 今後のメソッドで HTTP コールアウトを許可します。

```apex
// Good Example - AuraEnabled for LWC
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccounts() {
        return [SELECT Id, Name FROM Account WITH SECURITY_ENFORCED LIMIT 10];
    }

    @AuraEnabled
    public static void updateAccount(Id accountId, String newName) {
        Account acc = new Account(Id = accountId, Name = newName);
        update acc;
    }
}

// Good Example - InvocableMethod for Flow
public class FlowActions {
    @InvocableMethod(label='Send Email Notification' description='Sends email to account owner')
    public static List<Result> sendNotification(List<Request> requests) {
        List<Result> results = new List<Result>();

        for (Request req : requests) {
            Result result = new Result();
            try {
                // Send email logic
                result.success = true;
                result.message = 'Email sent successfully';
            } catch (Exception e) {
                result.success = false;
                result.message = e.getMessage();
            }
            results.add(result);
        }
        return results;
    }

    public class Request {
        @InvocableVariable(required=true label='Account ID')
        public Id accountId;

        @InvocableVariable(label='Email Template')
        public String templateName;
    }

    public class Result {
        @InvocableVariable
        public Boolean success;

        @InvocableVariable
        public String message;
    }
}

// Good Example - TestVisible for testing private methods
public class AccountService {
    @TestVisible
    private static Boolean validateAccountName(String name) {
        return String.isNotBlank(name) && name.length() > 3;
    }
}
```

### 非同期アペックス

- 単純な非同期操作とコールアウトには **@future** メソッドを使用します。
- チェーンを必要とする複雑な非同期操作には、**Queueable Apex** を使用します。
- 大量のデータ (50,000 レコードを超える) を処理するには、**Batch Apex** を使用します。
  - `Database.Stateful` を使用して、バッチ実行全体 (カウンター、集計など) の状態を維持します。
  - `Database.Stateful` がない場合、バッチクラスはステートレスとなり、インスタンス変数はバッチ間でリセットされます。
  - ステートフルバッチを使用する場合は、ガバナ制限に注意してください。
- 定期的な操作には **スケジュールされた Apex** を使用します。
  - バッチジョブをスケジュールするには、別の **スケジュール可能クラス** を作成します。
  - `Database.Batchable` と `Schedulable` の両方を同じクラスに実装しないでください。
- イベント駆動型のアーキテクチャと分離された統合には、**プラットフォームイベント** を使用します。
  - `EventBus.publish()` を使用して、非同期のファイアアンドフォーゲット通信を使用してイベントを発行します。
  - プラットフォームイベント オブジェクトのトリガーを使用してイベントをサブスクライブします。
  - 統合、マイクロサービス、組織間の通信に最適です。
- **処理の複雑さとガバナ制限に基づいてバッチサイズを最適化**します。
  - デフォルトのバッチサイズは 200 ですが、1 ～ 2000 の範囲で調整できます。
  - 複雑な処理またはコールアウトの場合は、より小さいバッチ (50 ～ 100)。
  - 単純な DML 操作の場合は、より大きなバッチ (200)。
  - 現実的なデータ量でテストして、最適なサイズを見つけます。

```apex
// Good Example - Platform Events for decoupled communication
public class OrderEventPublisher {
    public static void publishOrderCreated(List<Order> orders) {
        List<Order_Created__e> events = new List<Order_Created__e>();

        for (Order ord : orders) {
            Order_Created__e event = new Order_Created__e(
                Order_Id__c = ord.Id,
                Order_Amount__c = ord.TotalAmount,
                Customer_Id__c = ord.AccountId
            );
            events.add(event);
        }

        // Publish events
        List<Database.SaveResult> results = EventBus.publish(events);

        // Check for errors
        for (Database.SaveResult result : results) {
            if (!result.isSuccess()) {
                for (Database.Error error : result.getErrors()) {
                    System.debug('Error publishing event: ' + error.getMessage());
                }
            }
        }
    }
}

// Good Example - Platform Event Trigger (Subscriber)
trigger OrderCreatedTrigger on Order_Created__e (after insert) {
    List<Task> tasksToCreate = new List<Task>();

    for (Order_Created__e event : Trigger.new) {
        Task t = new Task(
            Subject = 'Follow up on order',
            WhatId = event.Order_Id__c,
            Priority = 'High'
        );
        tasksToCreate.add(t);
    }

    if (!tasksToCreate.isEmpty()) {
        insert tasksToCreate;
    }
}

// Good Example - Batch size optimization based on complexity
public class ComplexProcessingBatch implements Database.Batchable<SObject>, Database.AllowsCallouts {
    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator([
            SELECT Id, Name FROM Account WHERE IsActive__c = true
        ]);
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        // Complex processing with callouts - use smaller batch size
        for (Account acc : scope) {
            // Make HTTP callout
            HttpResponse res = ExternalAPIService.getAccountData(acc.Id);
            // Process response
        }
    }

    public void finish(Database.BatchableContext bc) {
        System.debug('Batch completed');
    }
}

// Execute with smaller batch size for callout-heavy processing
Database.executeBatch(new ComplexProcessingBatch(), 50);

// Good Example - Simple DML batch with default size
public class SimpleDMLBatch implements Database.Batchable<SObject> {
    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator([
            SELECT Id, Status__c FROM Order WHERE Status__c = 'Draft'
        ]);
    }

    public void execute(Database.BatchableContext bc, List<Order> scope) {
        for (Order ord : scope) {
            ord.Status__c = 'Pending';
        }
        update scope;
    }

    public void finish(Database.BatchableContext bc) {
        System.debug('Batch completed');
    }
}

// Execute with larger batch size for simple DML
Database.executeBatch(new SimpleDMLBatch(), 200);

// Good Example - Queueable Apex
public class EmailNotificationQueueable implements Queueable, Database.AllowsCallouts {
    private List<Id> accountIds;

    public EmailNotificationQueueable(List<Id> accountIds) {
        this.accountIds = accountIds;
    }

    public void execute(QueueableContext context) {
        List<Account> accounts = [SELECT Id, Name, Email__c FROM Account WHERE Id IN :accountIds];

        for (Account acc : accounts) {
            sendEmail(acc);
        }

        // Chain another job if needed
        if (hasMoreWork()) {
            System.enqueueJob(new AnotherQueueable());
        }
    }

    private void sendEmail(Account acc) {
        // Email sending logic
    }

    private Boolean hasMoreWork() {
        return false;
    }
}

// Good Example - Stateless Batch Apex (default)
public class AccountCleanupBatch implements Database.Batchable<SObject> {
    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator([
            SELECT Id, Name FROM Account WHERE LastActivityDate < LAST_N_DAYS:365
        ]);
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        delete scope;
    }

    public void finish(Database.BatchableContext bc) {
        System.debug('Batch completed');
    }
}

// Good Example - Stateful Batch Apex (maintains state across batches)
public class AccountStatsBatch implements Database.Batchable<SObject>, Database.Stateful {
    private Integer recordsProcessed = 0;
    private Integer totalRevenue = 0;

    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator([
            SELECT Id, Name, AnnualRevenue FROM Account WHERE IsActive__c = true
        ]);
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        for (Account acc : scope) {
            recordsProcessed++;
            totalRevenue += (Integer) acc.AnnualRevenue;
        }
    }

    public void finish(Database.BatchableContext bc) {
        // State is maintained: recordsProcessed and totalRevenue retain their values
        System.debug('Total records processed: ' + recordsProcessed);
        System.debug('Total revenue: ' + totalRevenue);

        // Send summary email or create summary record
    }
}

// Good Example - Schedulable class to schedule a batch
public class AccountCleanupScheduler implements Schedulable {
    public void execute(SchedulableContext sc) {
        // Execute the batch with batch size of 200
        Database.executeBatch(new AccountCleanupBatch(), 200);
    }
}

// Schedule the batch to run daily at 2 AM
// Execute this in Anonymous Apex or in setup code:
// String cronExp = '0 0 2 * * ?';
// System.schedule('Daily Account Cleanup', cronExp, new AccountCleanupScheduler());
```

## テスト

- **実稼働コードでは常に 100% のコードカバレッジ** を達成します (最低 75% が必要)。
- コードカバレッジだけでなくビジネスロジックを検証する**意味のあるテスト**を作成します。
- `@TestSetup` メソッドを使用して、テストメソッド間で共有されるテストデータを作成します。
- `Test.startTest()` と `Test.stopTest()` を使用してガバナ制限をリセットします。
- **ポジティブシナリオ**、**ネガティブシナリオ**、**一括シナリオ** (200 以上のレコード) をテストします。
- `System.runAs()` を使用して、さまざまなユーザーコンテキストと権限をテストします。
- `Test.setMock()` を使用して外部コールアウトを模擬します。
- `@SeeAllData=true` は決して使用しないでください。テストでは必ずテストデータを作成してください。
- **アサーションには、非推奨の `System.assert*()` メソッドの代わりに `Assert` クラスメソッドを使用してください**。
- 明確にするために、アサーションには必ず説明的な失敗メッセージを追加してください。

```apex
// Good Example - Comprehensive test class
@IsTest
private class AccountServiceTest {
    @TestSetup
    static void setupTestData() {
        List<Account> accounts = new List<Account>();
        for (Integer i = 0; i < 200; i++) {
            accounts.add(new Account(
                Name = 'Test Account ' + i,
                AnnualRevenue = i * 10000
            ));
        }
        insert accounts;
    }

    @IsTest
    static void testUpdateAccountRatings_Positive() {
        // Arrange
        List<Account> accounts = [SELECT Id FROM Account];
        Set<Id> accountIds = new Map<Id, Account>(accounts).keySet();

        // Act
        Test.startTest();
        AccountService.updateAccountRatings(accountIds);
        Test.stopTest();

        // Assert
        List<Account> updatedAccounts = [
            SELECT Id, Rating FROM Account WHERE AnnualRevenue > 1000000
        ];
        for (Account acc : updatedAccounts) {
            Assert.areEqual('Hot', acc.Rating, 'Rating should be Hot for high revenue accounts');
        }
    }

    @IsTest
    static void testUpdateAccountRatings_NoAccess() {
        // Create user with limited access
        User testUser = createTestUser();

        List<Account> accounts = [SELECT Id FROM Account LIMIT 1];
        Set<Id> accountIds = new Map<Id, Account>(accounts).keySet();

        Test.startTest();
        System.runAs(testUser) {
            try {
                AccountService.updateAccountRatings(accountIds);
                Assert.fail('Expected SecurityException');
            } catch (SecurityException e) {
                Assert.isTrue(true, 'SecurityException thrown as expected');
            }
        }
        Test.stopTest();
    }

    @IsTest
    static void testBulkOperation() {
        List<Account> accounts = [SELECT Id FROM Account];
        Set<Id> accountIds = new Map<Id, Account>(accounts).keySet();

        Test.startTest();
        AccountService.updateAccountRatings(accountIds);
        Test.stopTest();

        List<Account> updatedAccounts = [SELECT Id, Rating FROM Account];
        Assert.areEqual(200, updatedAccounts.size(), 'All accounts should be processed');
    }

    private static User createTestUser() {
        Profile p = [SELECT Id FROM Profile WHERE Name = 'Standard User' LIMIT 1];
        return new User(
            Alias = 'testuser',
            Email = 'testuser@test.com',
            EmailEncodingKey = 'UTF-8',
            LastName = 'Testing',
            LanguageLocaleKey = 'en_US',
            LocaleSidKey = 'en_US',
            ProfileId = p.Id,
            TimeZoneSidKey = 'America/Los_Angeles',
            UserName = 'testuser' + DateTime.now().getTime() + '@test.com'
        );
    }
}
```

## 一般的なコードの匂いとアンチパターン

- **ループ内の DML/SOQL** - ガバナ制限例外を回避するために、コードは常にバルク化してください。
- **ハードコードされた ID** - 代わりにカスタム設定、カスタムメタデータ、または動的クエリを使用します。
- **深くネストされた条件文** - わかりやすくするためにロジックを別のメソッドに抽出します。
- **大規模なメソッド** - メソッドを 1 つの責任に集中させます (最大 30 ～ 50 行)。
- **マジックナンバー** - 明確さと保守性を高めるために名前付き定数を使用します。
- **重複コード** - 共通ロジックを再利用可能なメソッドまたはクラスに抽出します。
- **null チェックの欠落** - 入力パラメータとクエリ結果を常に検証します。

```apex
// Bad Example - DML in loop
for (Account acc : accounts) {
    acc.Rating = 'Hot';
    update acc; // AVOID: DML in loop
}

// Good Example - Bulkified DML
for (Account acc : accounts) {
    acc.Rating = 'Hot';
}
update accounts;

// Bad Example - Hardcoded ID
Account acc = [SELECT Id FROM Account WHERE Id = '001000000000001'];

// Good Example - Dynamic query
Account acc = [SELECT Id FROM Account WHERE Name = :accountName LIMIT 1];

// Bad Example - Magic number
if (accounts.size() > 200) {
    // Process
}

// Good Example - Named constant
private static final Integer MAX_BATCH_SIZE = 200;
if (accounts.size() > MAX_BATCH_SIZE) {
    // Process
}
```

## ドキュメントとコメント

- クラスとメソッドには JavaDoc スタイルのコメントを使用します。
- 追跡用に `@author` タグと `@date` タグを含めます。
- `@description`、`@param`、`@return`、および `@throws` タグを含めます。
- 該当する場合は、**のみ** `@param`、`@return`、および `@throws` タグを含めます。
- 何も返さないメソッドには `@return void` を使用しないでください。
- 複雑なビジネスロジックと設計上の決定を文書化します。
- コードの変更に関するコメントを最新の状態に保ちます。

```apex
/**
 * @author Your Name
 * @date 2025-01-01
 * @description Service class for managing Account records
 */
public with sharing class AccountService {

    /**
     * @author Your Name
     * @date 2025-01-01
     * @description Updates the rating for accounts based on annual revenue
     * @param accountIds Set of Account IDs to update
     * @throws AccountServiceException if user lacks update permissions
     */
    public static void updateAccountRatings(Set<Id> accountIds) {
        // Implementation
    }
}
```

## 導入とDevOps

- ソース駆動開発には **Salesforce CLI** を使用します。
- **スクラッチ組織**を開発とテストに活用します。
- Salesforce CLI、GitHub Actions、Jenkins などのツールを使用して **CI/CD パイプライン**を実装します。
- モジュラー展開には **ロック解除されたパッケージ** を使用してください。
- 導入検証の一環として **Apex テスト**を実行します。
- **Salesforce Code Analyzer** を使用してコードをスキャンし、品質とセキュリティの問題がないか確認します。

```bash
# Salesforce CLI commands (sf)
sf project deploy start                    # Deploy source to org
sf project deploy start --dry-run          # Validate deployment without deploying
sf apex run test --test-level RunLocalTests # Run local Apex tests
sf apex get test --test-run-id <id>        # Get test results
sf project retrieve start                  # Retrieve source from org

# Salesforce Code Analyzer commands
sf code-analyzer rules                     # List all available rules
sf code-analyzer rules --rule-selector eslint:Recommended  # List recommended ESLint rules
sf code-analyzer rules --workspace ./force-app             # List rules for specific workspace
sf code-analyzer run                       # Run analysis with recommended rules
sf code-analyzer run --rule-selector pmd:Recommended       # Run PMD recommended rules
sf code-analyzer run --rule-selector "Security"           # Run rules with Security tag
sf code-analyzer run --workspace ./force-app --target "**/*.cls"  # Analyze Apex classes
sf code-analyzer run --severity-threshold 3               # Run analysis with severity threshold
sf code-analyzer run --output-file results.html           # Output results to HTML file
sf code-analyzer run --output-file results.csv            # Output results to CSV file
sf code-analyzer run --view detail                        # Show detailed violation information
```

## パフォーマンスの最適化

- インデックス付きフィールドでは **選択的 SOQL クエリ** を使用します。
- 負荷の高い操作には **遅延読み込み** を実装します。
- 長時間実行される操作には **非同期処理** を使用します。
- **デバッグログ**と**イベントモニタリング**を使用して監視します。
- パフォーマンスに関する洞察を得るには、**ApexGuru** と **Scale Center** を使用してください。

### プラットフォームキャッシュ

- **プラットフォームキャッシュ**を使用して、頻繁にアクセスされるデータを保存し、SOQL クエリを減らします。
- `Cache.OrgPartition` - 組織内のすべてのユーザーとセッションで共有されます。
- `Cache.SessionPartition` - ユーザーのセッションに固有。
- 適切なキャッシュ無効化戦略を実装します。
- データベースクエリへのフォールバックにより、キャッシュミスを適切に処理します。

```apex
// Good Example - Using Org Cache
public class AccountCacheService {
    private static final String CACHE_PARTITION = 'local.AccountCache';
    private static final Integer TTL_SECONDS = 3600; // 1 hour

    public static Account getAccount(Id accountId) {
        Cache.OrgPartition orgPart = Cache.Org.getPartition(CACHE_PARTITION);
        String cacheKey = 'Account_' + accountId;

        // Try to get from cache
        Account acc = (Account) orgPart.get(cacheKey);

        if (acc == null) {
            // Cache miss - query database
            acc = [
                SELECT Id, Name, Industry, AnnualRevenue
                FROM Account
                WHERE Id = :accountId
                LIMIT 1
            ];

            // Store in cache with TTL
            orgPart.put(cacheKey, acc, TTL_SECONDS);
        }

        return acc;
    }

    public static void invalidateCache(Id accountId) {
        Cache.OrgPartition orgPart = Cache.Org.getPartition(CACHE_PARTITION);
        String cacheKey = 'Account_' + accountId;
        orgPart.remove(cacheKey);
    }
}

// Good Example - Using Session Cache
public class UserPreferenceCache {
    private static final String CACHE_PARTITION = 'local.UserPrefs';

    public static Map<String, Object> getUserPreferences() {
        Cache.SessionPartition sessionPart = Cache.Session.getPartition(CACHE_PARTITION);
        String cacheKey = 'UserPrefs_' + UserInfo.getUserId();

        Map<String, Object> prefs = (Map<String, Object>) sessionPart.get(cacheKey);

        if (prefs == null) {
            // Load preferences from database or custom settings
            prefs = new Map<String, Object>{
                'theme' => 'dark',
                'language' => 'en_US'
            };
            sessionPart.put(cacheKey, prefs);
        }

        return prefs;
    }
}
```

## 構築と検証

- コードを追加または変更した後、プロジェクトが引き続き正常にビルドされることを確認します。
- 関連するすべての Apex テストクラスを実行して、回帰がないことを確認します。
- Salesforce CLI を使用します: `sf apex run test --test-level RunLocalTests`
- コードカバレッジが 75% の最低要件を満たしていることを確認します (100% を目指します)。
- Salesforce Code Analyzer を使用してコード品質の問題をチェックします: `sf code-analyzer run --severity-threshold 2`
- 導入前に違反を確認し、対処してください。
