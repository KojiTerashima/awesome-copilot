---
applyTo: '**'
---

# Dataverse SDK for Python — 実運用ユースケースとテンプレート

Dataverse のデータ移行および統合パターンに関する Microsoft 公式ガイダンスに基づいています。

## 1. レガシー システムからのデータ移行

### 移行アーキテクチャ

```
Legacy System → Staging Database → Dataverse
    (Extract)    (Transform)        (Load)
```

### 完全な移行例

```python
import pandas as pd
import time
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.errors import DataverseError
from azure.identity import DefaultAzureCredential

class DataMigrationPipeline:
    """レガシー システムから Dataverse へデータを移行する。"""

    def __init__(self, org_url: str):
        self.client = DataverseClient(
            base_url=org_url,
            credential=DefaultAzureCredential()
        )
        self.success_records = []
        self.failed_records = []

    def extract_from_legacy(self, legacy_db_connection, query: str):
        """source system からデータを抽出する。"""
        return pd.read_sql(query, legacy_db_connection)

    def transform_accounts(self, df: pd.DataFrame) -> list:
        """source data を Dataverse schema に変換する。"""
        payloads = []

        for _, row in df.iterrows():
            # source field を Dataverse へ対応付け
            payload = {
                "name": row["company_name"][:100],  # 100 文字に制限
                "telephone1": row["phone"],
                "websiteurl": row["website"],
                "revenue": float(row["annual_revenue"]) if row["annual_revenue"] else None,
                "numberofemployees": int(row["employees"]) if row["employees"] else None,
                # 照合用に source ID を保持
                "new_sourcecompanyid": str(row["legacy_id"]),
                "new_importsequencenumber": row["legacy_id"]
            }
            payloads.append(payload)

        return payloads

    def load_to_dataverse(self, payloads: list, batch_size: int = 200):
        """エラー追跡付きで Dataverse にデータを読み込む。"""
        total = len(payloads)

        for i in range(0, total, batch_size):
            batch = payloads[i:i + batch_size]

            try:
                ids = self.client.create("account", batch)
                self.success_records.extend(ids)
                print(f"✓ Created {len(ids)} records ({len(self.success_records)}/{total})")

                # rate limiting を防ぐ
                time.sleep(0.5)

            except DataverseError as e:
                self.failed_records.extend(batch)
                print(f"✗ Batch failed: {e.message}")

    def reconcile_migration(self, df: pd.DataFrame):
        """移行結果を検証し、結果を追跡する。"""

        # 作成済みレコードを照会
        created_accounts = self.client.get(
            "account",
            filter="new_importsequencenumber ne null",
            select=["accountid", "new_sourcecompanyid", "new_importsequencenumber"],
            top=10000
        )

        created_df = pd.DataFrame(list(created_accounts))

        # source table に Dataverse ID を反映
        merged = df.merge(
            created_df,
            left_on="legacy_id",
            right_on="new_importsequencenumber"
        )

        print(f"Successfully migrated {len(merged)} accounts")
        print(f"Failed: {len(self.failed_records)} records")

        return {
            "total_source": len(df),
            "migrated": len(merged),
            "failed": len(self.failed_records),
            "success_rate": len(merged) / len(df) * 100
        }

# Usage
pipeline = DataMigrationPipeline("https://myorg.crm.dynamics.com")

# Extract
source_data = pipeline.extract_from_legacy(
    legacy_connection,
    "SELECT id, company_name, phone, website, annual_revenue, employees FROM companies"
)

# Transform
payloads = pipeline.transform_accounts(source_data)

# Load
pipeline.load_to_dataverse(payloads, batch_size=300)

# Reconcile
results = pipeline.reconcile_migration(source_data)
print(results)
```

---

## 2. データ品質と重複排除エージェント

### 重複の検出とマージ

```python
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import DefaultAzureCredential
import difflib

class DataQualityAgent:
    """データ品質を監視し改善する。"""

    def __init__(self, org_url: str):
        self.client = DataverseClient(
            base_url=org_url,
            credential=DefaultAzureCredential()
        )

    def find_potential_duplicates(self, table_name: str, match_fields: list):
        """重複候補レコードを見つける。"""

        records = []
        for page in self.client.get(table_name, select=match_fields, top=10000):
            records.extend(page)

        duplicates = []
        seen = {}

        for record in records:
            # match field からキーを作る
            key = tuple(
                record.get(field, "").lower().strip()
                for field in match_fields
            )

            if key in seen and key != ("",) * len(match_fields):
                duplicates.append({
                    "original": seen[key],
                    "duplicate": record,
                    "fields_matched": match_fields
                })
            else:
                seen[key] = record

        return duplicates, len(records)

    def merge_records(self, table_name: str, primary_id: str, duplicate_id: str,
                     mapping: dict):
        """重複レコードを primary にマージする。"""

        # duplicate から primary へデータをコピー
        updates = {}
        duplicate = self.client.get(table_name, duplicate_id)

        for source_field, target_field in mapping.items():
            if duplicate.get(source_field) and not primary.get(target_field):
                updates[target_field] = duplicate[source_field]

        # primary を更新
        if updates:
            self.client.update(table_name, primary_id, updates)

        # duplicate を削除
        self.client.delete(table_name, duplicate_id)

        return f"Merged {duplicate_id} into {primary_id}"

    def generate_quality_report(self, table_name: str) -> dict:
        """データ品質メトリクスを生成する。"""

        records = list(self.client.get(table_name, top=10000))

        report = {
            "table": table_name,
            "total_records": len(records),
            "null_values": {},
            "duplicates": 0,
            "completeness_score": 0
        }

        # null 値を確認
        all_fields = set()
        for record in records:
            all_fields.update(record.keys())

        for field in all_fields:
            null_count = sum(1 for r in records if not r.get(field))
            completeness = (len(records) - null_count) / len(records) * 100

            if completeness < 100:
                report["null_values"][field] = {
                    "null_count": null_count,
                    "completeness": completeness
                }

        # 重複を確認
        duplicates, _ = self.find_potential_duplicates(
            table_name,
            ["name", "emailaddress1"]
        )
        report["duplicates"] = len(duplicates)

        # 全体 completeness
        avg_completeness = sum(
            100 - ((d["null_count"] / len(records)) * 100)
            for d in report["null_values"].values()
        ) / len(report["null_values"]) if report["null_values"] else 100
        report["completeness_score"] = avg_completeness

        return report

# Usage
agent = DataQualityAgent("https://myorg.crm.dynamics.com")

# 重複を検出
duplicates, total = agent.find_potential_duplicates(
    "account",
    match_fields=["name", "emailaddress1"]
)

print(f"Found {len(duplicates)} potential duplicates out of {total} accounts")

# 確信が持てる場合だけマージ
for dup in duplicates[:5]:  # 上位 5 件を処理
    result = agent.merge_records(
        "account",
        primary_id=dup["original"]["accountid"],
        duplicate_id=dup["duplicate"]["accountid"],
        mapping={"telephone1": "telephone1", "websiteurl": "websiteurl"}
    )
    print(result)

# 品質レポート
report = agent.generate_quality_report("account")
print(f"Data Quality: {report['completeness_score']:.1f}%")
```

---

## 3. Contact と Account のデータ補完

### 外部ソースによる CRM データ補完

```python
import requests
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import DefaultAzureCredential

class DataEnrichmentAgent:
    """外部データで CRM レコードを補完する。"""

    def __init__(self, org_url: str, external_api_key: str):
        self.client = DataverseClient(
            base_url=org_url,
            credential=DefaultAzureCredential()
        )
        self.api_key = external_api_key

    def enrich_accounts_with_industry_data(self):
        """業種分類データで account を補完する。"""

        accounts = self.client.get(
            "account",
            select=["accountid", "name", "websiteurl"],
            filter="new_industrydata eq null",
            top=500
        )

        enriched_count = 0
        for page in accounts:
            for account in page:
                try:
                    # 外部 API を呼ぶ
                    industry = self._lookup_industry(account["name"])

                    if industry:
                        self.client.update(
                            "account",
                            account["accountid"],
                            {"new_industrydata": industry}
                        )
                        enriched_count += 1

                except Exception as e:
                    print(f"Failed to enrich {account['name']}: {e}")

        return enriched_count

    def enrich_contacts_with_social_profiles(self):
        """SNS プロフィールを見つけて関連付ける。"""

        contacts = self.client.get(
            "contact",
            select=["contactid", "fullname", "emailaddress1"],
            filter="new_linkedinurl eq null",
            top=500
        )

        for page in contacts:
            for contact in page:
                try:
                    # social profile を探す
                    profiles = self._find_social_profiles(
                        contact["fullname"],
                        contact["emailaddress1"]
                    )

                    if profiles:
                        self.client.update(
                            "contact",
                            contact["contactid"],
                            {
                                "new_linkedinurl": profiles.get("linkedin"),
                                "new_twitterhandle": profiles.get("twitter")
                            }
                        )

                except Exception as e:
                    print(f"Failed to enrich {contact['fullname']}: {e}")

    def _lookup_industry(self, company_name: str) -> str:
        """外部業種 API を呼ぶ。"""
        response = requests.get(
            "https://api.example.com/industry",
            params={"company": company_name},
            headers={"Authorization": f"Bearer {self.api_key}"}
        )

        if response.status_code == 200:
            return response.json().get("industry")
        return None

    def _find_social_profiles(self, name: str, email: str) -> dict:
        """人物の SNS プロフィールを見つける。"""
        response = requests.get(
            "https://api.example.com/social",
            params={"name": name, "email": email},
            headers={"Authorization": f"Bearer {self.api_key}"}
        )

        if response.status_code == 200:
            return response.json()
        return {}

# Usage
enricher = DataEnrichmentAgent(
    "https://myorg.crm.dynamics.com",
    api_key="your-api-key"
)

enriched = enricher.enrich_accounts_with_industry_data()
print(f"Enriched {enriched} accounts")
```

---

## 4. レポート データの自動エクスポート

### CRM データを Excel に出力する

```python
import pandas as pd
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import DefaultAzureCredential
from datetime import datetime

class ReportExporter:
    """Dataverse データをレポートへ出力する。"""

    def __init__(self, org_url: str):
        self.client = DataverseClient(
            base_url=org_url,
            credential=DefaultAzureCredential()
        )

    def export_sales_summary(self, output_file: str):
        """レポート用に sales データを export する。"""

        accounts = []
        for page in self.client.get(
            "account",
            select=["accountid", "name", "revenue", "numberofemployees",
                   "createdon", "modifiedon"],
            filter="statecode eq 0",  # Active のみ
            orderby=["revenue desc"],
            top=10000
        ):
            accounts.extend(page)

        # Opportunities
        opportunities = []
        for page in self.client.get(
            "opportunity",
            select=["opportunityid", "name", "estimatedvalue",
                   "statuscode", "parentaccountid", "createdon"],
            top=10000
        ):
            opportunities.extend(page)

        # DataFrame 作成
        df_accounts = pd.DataFrame(accounts)
        df_opportunities = pd.DataFrame(opportunities)

        # レポート生成
        with pd.ExcelWriter(output_file) as writer:
            df_accounts.to_excel(writer, sheet_name="Accounts", index=False)
            df_opportunities.to_excel(writer, sheet_name="Opportunities", index=False)

            # Summary sheet
            summary = pd.DataFrame({
                "Metric": [
                    "Total Accounts",
                    "Total Opportunities",
                    "Total Revenue",
                    "Export Date"
                ],
                "Value": [
                    len(df_accounts),
                    len(df_opportunities),
                    df_accounts["revenue"].sum() if "revenue" in df_accounts else 0,
                    datetime.now().isoformat()
                ]
            })
            summary.to_excel(writer, sheet_name="Summary", index=False)

        return output_file

    def export_activity_log(self, days_back: int = 30) -> str:
        """監査用に最近の activity を export する。"""

        from_date = pd.Timestamp.now(tz='UTC') - pd.Timedelta(days=days_back)

        activities = []
        for page in self.client.get(
            "activitypointer",
            select=["activityid", "subject", "activitytypecode",
                   "createdon", "ownerid"],
            filter=f"createdon gt {from_date.isoformat()}",
            orderby=["createdon desc"],
            top=10000
        ):
            activities.extend(page)

        df = pd.DataFrame(activities)
        output = f"activity_log_{datetime.now():%Y%m%d}.csv"
        df.to_csv(output, index=False)

        return output

# Usage
exporter = ReportExporter("https://myorg.crm.dynamics.com")
report_file = exporter.export_sales_summary("sales_report.xlsx")
print(f"Report saved to {report_file}")
```

---

## 5. Workflow 統合 - 一括操作

### 条件に基づいてレコードを処理する

```python
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import DefaultAzureCredential
from enum import IntEnum

class AccountStatus(IntEnum):
    PROSPECT = 1
    ACTIVE = 2
    CLOSED = 3

class BulkWorkflow:
    """一括操作を自動化する。"""

    def __init__(self, org_url: str):
        self.client = DataverseClient(
            base_url=org_url,
            credential=DefaultAzureCredential()
        )

    def mark_accounts_as_inactive_if_no_activity(self, days_no_activity: int = 90):
        """最近 activity のない account を無効化する。"""

        from_date = f"2025-{datetime.now().month:02d}-01T00:00:00Z"

        inactive_accounts = self.client.get(
            "account",
            select=["accountid", "name"],
            filter=f"modifiedon lt {from_date} and statecode eq 0",
            top=5000
        )

        accounts_to_deactivate = []
        for page in inactive_accounts:
            accounts_to_deactivate.extend([a["accountid"] for a in page])

        # Bulk update
        if accounts_to_deactivate:
            self.client.update(
                "account",
                accounts_to_deactivate,
                {"statecode": AccountStatus.CLOSED}
            )
            print(f"Deactivated {len(accounts_to_deactivate)} inactive accounts")

    def update_opportunity_status_based_on_amount(self):
        """見込金額に応じて opportunity stage を更新する。"""

        opportunities = self.client.get(
            "opportunity",
            select=["opportunityid", "estimatedvalue"],
            filter="statuscode ne 7",  # 未クローズ
            top=5000
        )

        updates = []
        ids = []

        for page in opportunities:
            for opp in page:
                value = opp.get("estimatedvalue", 0)

                # stage を決定
                if value < 10000:
                    stage = 1  # Qualification
                elif value < 50000:
                    stage = 2  # Proposal
                else:
                    stage = 3  # Proposal Review

                updates.append({"stageid": stage})
                ids.append(opp["opportunityid"])

        # Bulk update
        if ids:
            self.client.update("opportunity", ids, updates)
            print(f"Updated {len(ids)} opportunities")

# Usage
workflow = BulkWorkflow("https://myorg.crm.dynamics.com")
workflow.mark_accounts_as_inactive_if_no_activity(days_no_activity=90)
workflow.update_opportunity_status_based_on_amount()
```

---

## 6. 定期ジョブ テンプレート

### 定期実行用 Azure Function

```python
# scheduled_migration_job.py
import azure.functions as func
from datetime import datetime
from DataMigrationPipeline import DataMigrationPipeline
import logging

def main(timer: func.TimerRequest) -> None:
    """スケジュール (例: 毎日) で移行ジョブを実行する。"""

    if timer.past_due:
        logging.info('The timer is past due!')

    try:
        logging.info(f'Migration job started at {datetime.utcnow()}')

        # 移行を実行
        pipeline = DataMigrationPipeline("https://myorg.crm.dynamics.com")

        # extract, transform, load
        source_data = pipeline.extract_from_legacy(...)
        payloads = pipeline.transform_accounts(source_data)
        pipeline.load_to_dataverse(payloads)

        # 結果取得
        results = pipeline.reconcile_migration(source_data)

        logging.info(f'Migration completed: {results}')

    except Exception as e:
        logging.error(f'Migration failed: {e}')
        raise

# function_app.py - Azure Functions setup
app = func.FunctionApp()

@app.schedule_trigger(schedule="0 0 * * *")  # 毎日 0 時
def migration_job(timer: func.TimerRequest) -> None:
    main(timer)
```

---

## 7. 完全なスターター テンプレート

```python
#!/usr/bin/env python3
"""
Dataverse SDK for Python - 完全スターター テンプレート
"""

from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.config import DataverseConfig
from PowerPlatform.Dataverse.core.errors import DataverseError
import logging

# logging を設定
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DataverseApp:
    """Dataverse application の基底 class。"""

    def __init__(self, org_url: str):
        self.org_url = org_url
        self.client = self._create_client()

    def _create_client(self) -> DataverseClient:
        """認証済み client を作成する。"""
        cfg = DataverseConfig()
        cfg.logging_enable = False

        return DataverseClient(
            base_url=self.org_url,
            credential=DefaultAzureCredential(),
            config=cfg
        )

    def create_account(self, name: str, phone: str = None) -> str:
        """account レコードを作成する。"""
        try:
            payload = {"name": name}
            if phone:
                payload["telephone1"] = phone

            id = self.client.create("account", payload)[0]
            logger.info(f"Created account: {id}")
            return id

        except DataverseError as e:
            logger.error(f"Failed to create account: {e.message}")
            raise

    def get_accounts(self, filter_expr: str = None, top: int = 100) -> list:
        """account レコードを取得する。"""
        try:
            accounts = self.client.get(
                "account",
                filter=filter_expr,
                select=["accountid", "name", "telephone1", "createdon"],
                orderby=["createdon desc"],
                top=top
            )

            all_accounts = []
            for page in accounts:
                all_accounts.extend(page)

            logger.info(f"Retrieved {len(all_accounts)} accounts")
            return all_accounts

        except DataverseError as e:
            logger.error(f"Failed to get accounts: {e.message}")
            raise

    def update_account(self, account_id: str, **kwargs) -> None:
        """account レコードを更新する。"""
        try:
            self.client.update("account", account_id, kwargs)
            logger.info(f"Updated account: {account_id}")

        except DataverseError as e:
            logger.error(f"Failed to update account: {e.message}")
            raise

if __name__ == "__main__":
    # Usage
    app = DataverseApp("https://myorg.crm.dynamics.com")

    # Create
    account_id = app.create_account("Acme Inc", "555-0100")

    # Get
    accounts = app.get_accounts(filter_expr="statecode eq 0", top=50)
    print(f"Found {len(accounts)} active accounts")

    # Update
    app.update_account(account_id, telephone1="555-0199")
```

---

## 8. 関連資料

- [Dataverse Data Migration](https://learn.microsoft.com/en-us/power-platform/architecture/key-concepts/data-migration/workflow-complex-data-migration)
- [Working with Data (SDK)](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/sdk-python/work-data)
- [SDK Examples on GitHub](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/tree/main/examples)
