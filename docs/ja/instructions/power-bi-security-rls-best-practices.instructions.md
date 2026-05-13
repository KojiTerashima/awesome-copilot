---
description: '動的セキュリティ、ベスト プラクティス、ガバナンス戦略を含む、Power BI の Row-Level Security (RLS) と高度なセキュリティ パターン実装ガイド。'
applyTo: '**/*.{pbix,dax,md,txt,json,csharp,powershell}'
---

# Power BI セキュリティと Row-Level Security のベスト プラクティス

## 概要
この文書は、Microsoft の公式ガイダンスに基づき、Row-Level Security (RLS)、動的セキュリティ、ガバナンスの best practice に重点を置いて、Power BI に堅牢なセキュリティ パターンを実装するための包括的なガイドを提供します。

## Row-Level Security の基本

### 1. 基本的な RLS 実装
```dax
// Simple user-based filtering
[EmailAddress] = USERNAME()

// Role-based filtering with improved security
IF(
    USERNAME() = "Worker",
    [Type] = "Internal",
    IF(
        USERNAME() = "Manager",
        TRUE(),
        FALSE()  // Deny access to unexpected users
    )
)
```

### 2. Custom Data を使う動的 RLS
```dax
// Using CUSTOMDATA() for dynamic filtering
VAR UserRole = CUSTOMDATA()
RETURN
    SWITCH(
        UserRole,
        "SalesPersonA", [SalesTerritory] = "West",
        "SalesPersonB", [SalesTerritory] = "East",
        "Manager", TRUE(),
        FALSE()  // Default deny
    )
```

### 3. 高度なセキュリティ パターン
```dax
// Hierarchical security with territory lookups
=DimSalesTerritory[SalesTerritoryKey]=LOOKUPVALUE(
    DimUserSecurity[SalesTerritoryID],
    DimUserSecurity[UserName], USERNAME(),
    DimUserSecurity[SalesTerritoryID], DimSalesTerritory[SalesTerritoryKey]
)

// Multiple condition security
VAR UserTerritories =
    FILTER(
        UserSecurity,
        UserSecurity[UserName] = USERNAME()
    )
VAR AllowedTerritories = SELECTCOLUMNS(UserTerritories, "Territory", UserSecurity[Territory])
RETURN
    [Territory] IN AllowedTerritories
```

## Embedded Analytics のセキュリティ

### 1. 静的 RLS 実装
```csharp
// Static RLS with fixed roles
var rlsidentity = new EffectiveIdentity(
    username: "username@contoso.com",
    roles: new List<string>{ "MyRole" },
    datasets: new List<string>{ datasetId.ToString()}
);
```

### 2. Custom Data を使う動的 RLS
```csharp
// Dynamic RLS with custom data
var rlsidentity = new EffectiveIdentity(
    username: "username@contoso.com",
    roles: new List<string>{ "MyRoleWithCustomData" },
    customData: "SalesPersonA",
    datasets: new List<string>{ datasetId.ToString()}
);
```

### 3. 複数 Dataset のセキュリティ
```json
{
    "accessLevel": "View",
    "identities": [
        {
            "username": "France",
            "roles": [ "CountryDynamic"],
            "datasets": [ "fe0a1aeb-f6a4-4b27-a2d3-b5df3bb28bdc" ]
        }
    ]
}
```

## Database Level のセキュリティ統合

### 1. SQL Server RLS 統合
```sql
-- Creating security schema and predicate function
CREATE SCHEMA Security;
GO

CREATE FUNCTION Security.tvf_securitypredicate(@SalesRep AS nvarchar(50))
    RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS tvf_securitypredicate_result
WHERE @SalesRep = USER_NAME() OR USER_NAME() = 'Manager';
GO

-- Applying security policy
CREATE SECURITY POLICY SalesFilter
ADD FILTER PREDICATE Security.tvf_securitypredicate(SalesRep)
ON sales.Orders
WITH (STATE = ON);
GO
```

### 2. Fabric Warehouse セキュリティ
```sql
-- Creating schema for Security
CREATE SCHEMA Security;
GO

-- Creating a function for the SalesRep evaluation
CREATE FUNCTION Security.tvf_securitypredicate(@UserName AS varchar(50))
    RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS tvf_securitypredicate_result
WHERE @UserName = USER_NAME()
OR USER_NAME() = 'BatchProcess@contoso.com';
GO

-- Using the function to create a Security Policy
CREATE SECURITY POLICY YourSecurityPolicy
ADD FILTER PREDICATE Security.tvf_securitypredicate(UserName_column)
ON sampleschema.sampletable
WITH (STATE = ON);
GO
```

## 高度なセキュリティ パターン

### 1. Paginated Reports のセキュリティ
```json
{
    "format": "PDF",
    "paginatedReportConfiguration":{
        "identities": [
            {"username": "john@contoso.com"}
        ]
    }
}
```

### 2. Power Pages 統合
```html
{% powerbi authentication_type:"powerbiembedded" path:"https://app.powerbi.com/groups/00000000-0000-0000-0000-000000000000/reports/00000000-0000-0000-0000-000000000001/ReportSection" roles:"pagesuser" %}
```

### 3. マルチテナント セキュリティ
```json
{
  "datasets": [
    {
      "id": "fff1a505-xxxx-xxxx-xxxx-e69f81e5b974",
    }
  ],
  "reports": [
    {
      "allowEdit": false,
      "id": "10ce71df-xxxx-xxxx-xxxx-814a916b700d"
    }
  ],
  "identities": [
    {
      "username": "YourUsername",
      "datasets": [
        "fff1a505-xxxx-xxxx-xxxx-e69f81e5b974"
      ],
      "roles": [
        "YourRole"
      ]
    }
  ],
  "datasourceIdentities": [
    {
      "identityBlob": "eyJ…",
      "datasources": [
        {
          "datasourceType": "Sql",
          "connectionDetails": {
            "server": "YourServerName.database.windows.net",
            "database": "YourDataBaseName"
          }
        }
      ]
    }
  ]
}
```

## セキュリティ設計パターン

### 1. 部分的な RLS 実装
```dax
// Create summary table for partial RLS
SalesRevenueSummary =
SUMMARIZECOLUMNS(
    Sales[OrderDate],
    "RevenueAllRegion", SUM(Sales[Revenue])
)

// Apply RLS only to detail level
Salesperson Filter = [EmailAddress] = USERNAME()
```

### 2. 階層型セキュリティ
```dax
// Manager can see all, others see their own
VAR CurrentUser = USERNAME()
VAR UserRole = LOOKUPVALUE(
    UserRoles[Role],
    UserRoles[Email], CurrentUser
)
RETURN
    SWITCH(
        UserRole,
        "Manager", TRUE(),
        "Salesperson", [SalespersonEmail] = CurrentUser,
        "Regional Manager", [Region] IN (
            SELECTCOLUMNS(
                FILTER(UserRegions, UserRegions[Email] = CurrentUser),
                "Region", UserRegions[Region]
            )
        ),
        FALSE()
    )
```

### 3. 時間ベースのセキュリティ
```dax
// Restrict access to recent data based on role
VAR UserRole = LOOKUPVALUE(UserRoles[Role], UserRoles[Email], USERNAME())
VAR CutoffDate =
    SWITCH(
        UserRole,
        "Executive", DATE(1900,1,1),  // All historical data
        "Manager", TODAY() - 365,     // Last year
        "Analyst", TODAY() - 90,      // Last 90 days
        TODAY()                       // Current day only
    )
RETURN
    [Date] >= CutoffDate
```

## セキュリティの検証とテスト

### 1. Role 検証パターン
```dax
// Security testing measure
Security Test =
VAR CurrentUsername = USERNAME()
VAR ExpectedRole = "TestRole"
VAR TestResult =
    IF(
        HASONEVALUE(SecurityRoles[Role]) &&
        VALUES(SecurityRoles[Role]) = ExpectedRole,
        "PASS: Role applied correctly",
        "FAIL: Incorrect role or multiple roles"
    )
RETURN
    "User: " & CurrentUsername & " | " & TestResult
```

### 2. データ露出監査
```dax
// Audit measure to track data access
Data Access Audit =
VAR AccessibleRows = COUNTROWS(FactTable)
VAR TotalRows = CALCULATE(COUNTROWS(FactTable), ALL(FactTable))
VAR AccessPercentage = DIVIDE(AccessibleRows, TotalRows) * 100
RETURN
    "User: " & USERNAME() &
    " | Accessible: " & FORMAT(AccessibleRows, "#,0") &
    " | Total: " & FORMAT(TotalRows, "#,0") &
    " | Access: " & FORMAT(AccessPercentage, "0.00") & "%"
```

## ガバナンスと管理

### 1. セキュリティ グループ管理の自動化
```powershell
# Add security group to Power BI workspace
# Sign in to Power BI
Login-PowerBI

# Set up the security group object ID
$SGObjectID = "<security-group-object-ID>"

# Get the workspace
$pbiWorkspace = Get-PowerBIWorkspace -Filter "name eq '<workspace-name>'"

# Add the security group to the workspace
Add-PowerBIWorkspaceUser -Id $($pbiWorkspace.Id) -AccessRight Member -PrincipalType Group -Identifier $($SGObjectID)
```

### 2. セキュリティ監視
```powershell
# Monitor Power BI access patterns
$workspaces = Get-PowerBIWorkspace
foreach ($workspace in $workspaces) {
    $users = Get-PowerBIWorkspaceUser -Id $workspace.Id
    Write-Host "Workspace: $($workspace.Name)"
    foreach ($user in $users) {
        Write-Host "  User: $($user.UserPrincipalName) - Access: $($user.AccessRight)"
    }
}
```

### 3. コンプライアンス レポート
```dax
// Compliance dashboard measures
Users with Data Access =
CALCULATE(
    DISTINCTCOUNT(AuditLog[Username]),
    AuditLog[AccessType] = "DataAccess",
    AuditLog[Date] >= TODAY() - 30
)

High Privilege Users =
CALCULATE(
    DISTINCTCOUNT(UserRoles[Email]),
    UserRoles[Role] IN {"Admin", "Manager", "Executive"}
)

Security Violations =
CALCULATE(
    COUNTROWS(AuditLog),
    AuditLog[EventType] = "SecurityViolation",
    AuditLog[Date] >= TODAY() - 7
)
```

## ベスト プラクティスと Anti-Pattern

### ✅ セキュリティのベスト プラクティス

#### 1. 最小権限の原則
```dax
// Always default to restrictive access
Default Security =
VAR UserPermissions =
    FILTER(
        UserAccess,
        UserAccess[Email] = USERNAME()
    )
RETURN
    IF(
        COUNTROWS(UserPermissions) > 0,
        [Territory] IN SELECTCOLUMNS(UserPermissions, "Territory", UserAccess[Territory]),
        FALSE()  // No access if not explicitly granted
    )
```

#### 2. 明示的な Role 検証
```dax
// Validate expected roles explicitly
Role-Based Filter =
VAR UserRole = LOOKUPVALUE(UserRoles[Role], UserRoles[Email], USERNAME())
VAR AllowedRoles = {"Analyst", "Manager", "Executive"}
RETURN
    IF(
        UserRole IN AllowedRoles,
        SWITCH(
            UserRole,
            "Analyst", [Department] = LOOKUPVALUE(UserDepartments[Department], UserDepartments[Email], USERNAME()),
            "Manager", [Region] = LOOKUPVALUE(UserRegions[Region], UserRegions[Email], USERNAME()),
            "Executive", TRUE()
        ),
        FALSE()  // Deny access for unexpected roles
    )
```

### ❌ 避けるべきセキュリティ Anti-Pattern

#### 1. 過度に寛容な既定値
```dax
// ❌ AVOID: This grants full access to unexpected users
Bad Security Filter =
IF(
    USERNAME() = "SpecificUser",
    [Type] = "Internal",
    TRUE()  // Dangerous default
)
```

#### 2. 複雑すぎるセキュリティ ロジック
```dax
// ❌ AVOID: Overly complex security that's hard to audit
Overly Complex Security =
IF(
    OR(
        AND(USERNAME() = "User1", WEEKDAY(TODAY()) <= 5),
        AND(USERNAME() = "User2", HOUR(NOW()) >= 9, HOUR(NOW()) <= 17),
        AND(CONTAINS(VALUES(SpecialUsers[Email]), SpecialUsers[Email], USERNAME()), [Priority] = "High")
    ),
    [Type] IN {"Internal", "Confidential"},
    [Type] = "Public"
)
```

## セキュリティ統合パターン

### 1. Azure AD 統合
```csharp
// Generate token with Azure AD user context
var tokenRequest = new GenerateTokenRequestV2(
    reports: new List<GenerateTokenRequestV2Report>() { new GenerateTokenRequestV2Report(reportId) },
    datasets: datasetIds.Select(datasetId => new GenerateTokenRequestV2Dataset(datasetId.ToString())).ToList(),
    targetWorkspaces: targetWorkspaceId != Guid.Empty ? new List<GenerateTokenRequestV2TargetWorkspace>() { new GenerateTokenRequestV2TargetWorkspace(targetWorkspaceId) } : null,
    identities: new List<EffectiveIdentity> { rlsIdentity }
);

var embedToken = pbiClient.EmbedToken.GenerateToken(tokenRequest);
```

### 2. Service Principal 認証
```csharp
// Service principal with RLS for embedded scenarios
public EmbedToken GetEmbedToken(Guid reportId, IList<Guid> datasetIds, [Optional] Guid targetWorkspaceId)
{
    PowerBIClient pbiClient = this.GetPowerBIClient();

    var rlsidentity = new EffectiveIdentity(
       username: "username@contoso.com",
       roles: new List<string>{ "MyRole" },
       datasets: new List<string>{ datasetId.ToString()}
    );

    var tokenRequest = new GenerateTokenRequestV2(
        reports: new List<GenerateTokenRequestV2Report>() { new GenerateTokenRequestV2Report(reportId) },
        datasets: datasetIds.Select(datasetId => new GenerateTokenRequestV2Dataset(datasetId.ToString())).ToList(),
        targetWorkspaces: targetWorkspaceId != Guid.Empty ? new List<GenerateTokenRequestV2TargetWorkspace>() { new GenerateTokenRequestV2TargetWorkspace(targetWorkspaceId) } : null,
        identities: new List<EffectiveIdentity> { rlsIdentity }
    );

    var embedToken = pbiClient.EmbedToken.GenerateToken(tokenRequest);

    return embedToken;
}
```

## セキュリティ監視と監査

### 1. アクセス パターン分析
```dax
// Identify unusual access patterns
Unusual Access Pattern =
VAR UserAccessCount =
    CALCULATE(
        COUNTROWS(AccessLog),
        AccessLog[Date] >= TODAY() - 7
    )
VAR AvgUserAccess =
    CALCULATE(
        AVERAGE(AccessLog[AccessCount]),
        ALL(AccessLog[Username]),
        AccessLog[Date] >= TODAY() - 30
    )
RETURN
    IF(
        UserAccessCount > AvgUserAccess * 3,
        "⚠️ High Activity",
        "Normal"
    )
```

### 2. データ侵害の検知
```dax
// Detect potential data exposure
Potential Data Exposure =
VAR UnexpectedAccess =
    CALCULATE(
        COUNTROWS(AccessLog),
        AccessLog[AccessResult] = "Denied",
        AccessLog[Date] >= TODAY() - 1
    )
RETURN
    IF(
        UnexpectedAccess > 10,
        "🚨 Multiple Access Denials - Review Required",
        "Normal"
    )
```

忘れないでください。セキュリティは多層防御です。適切な authentication、authorization、data encryption、network security、包括的な auditing を組み合わせて defense in depth を実装してください。現在の要件と compliance standard を満たしていることを確認するため、セキュリティ実装は定期的に見直し、テストしてください。
