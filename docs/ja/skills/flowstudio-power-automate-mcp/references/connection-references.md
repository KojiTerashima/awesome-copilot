# FlowStudio MCP — 接続リファレンス

接続参照は、フローのコネクタ アクションを実際に認証されたものに結び付けます。
Power Platform での接続。電話をかけるときは必ず必要です
`update_live_flow` とコネクタ アクションを使用する定義。

---

## フロー定義内の構造```json
{
  "properties": {
    "definition": { ... },
    "connectionReferences": {
      "shared_sharepointonline": {
        "connectionName": "shared-sharepointonl-62599557c-1f33-4aec-b4c0-a6e4afcae3be",
        "id": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
        "displayName": "SharePoint"
      },
      "shared_office365": {
        "connectionName": "shared-office365-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "id": "/providers/Microsoft.PowerApps/apis/shared_office365",
        "displayName": "Office 365 Outlook"
      }
    }
  }
}
```キーは **論理参照名** (例: `shared_sharepointonline`) です。
これらは、各アクションの `host` ブロック内の `connectionName` フィールドと一致します。

---

## 接続 GUID の検索

同じ接続を使用する**既存のフロー**で `get_live_flow` を呼び出します
`connectionReferences` ブロックをコピーします。コネクタ接頭辞の後の GUID は次のとおりです。
認証ユーザーが所有する接続インスタンス。```python
flow = mcp("get_live_flow", environmentName=ENV, flowName=EXISTING_FLOW_ID)
conn_refs = flow["properties"]["connectionReferences"]
# conn_refs["shared_sharepointonline"]["connectionName"]
# → "shared-sharepointonl-62599557c-1f33-4aec-b4c0-a6e4afcae3be"
```> ⚠️ 接続参照は **ユーザースコープ** です。接続が所有されている場合
> 別のアカウントの場合、`update_live_flow` は 403 を返します
> @@コード1@@。に属する接続を使用する必要があります
> `x-api-key` ヘッダーにトークンが含まれているアカウント。

---

## `connectionReferences` を `update_live_flow` に渡す```python
result = mcp("update_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,
    definition=modified_definition,
    connectionReferences={
        "shared_sharepointonline": {
            "connectionName": "shared-sharepointonl-62599557c-1f33-4aec-b4c0-a6e4afcae3be",
            "id": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
        }
    }
)
```定義で実際に使用される接続のみを含めます。

---

## 共通コネクタ API ID

|サービス | API ID |
|---|---|
| SharePointオンライン | `/providers/Microsoft.PowerApps/apis/shared_sharepointonline` |
| Office 365 の見通し | `/providers/Microsoft.PowerApps/apis/shared_office365` |
|マイクロソフトチーム | `/providers/Microsoft.PowerApps/apis/shared_teams` |
|ビジネス向け OneDrive | `/providers/Microsoft.PowerApps/apis/shared_onedriveforbusiness` |
| Azure AD | `/providers/Microsoft.PowerApps/apis/shared_azuread` |
| Azure AD を使用した HTTP | `/providers/Microsoft.PowerApps/apis/shared_webcontents` |
| SQLサーバー | `/providers/Microsoft.PowerApps/apis/shared_sql` |
|データバース | `/providers/Microsoft.PowerApps/apis/shared_commondataserviceforapps` |
| Azure Blob ストレージ | `/providers/Microsoft.PowerApps/apis/shared_azureblob` |
|承認 | `/providers/Microsoft.PowerApps/apis/shared_approvals` |
| Office 365 ユーザー | `/providers/Microsoft.PowerApps/apis/shared_office365users` |
|フロー管理 | `/providers/Microsoft.PowerApps/apis/shared_flowmanagement` |

---

## Teams アダプティブ カードのデュアル接続要件

アダプティブ カードを送信するフロー ** およびポスト フォローアップ メッセージ** には 2 つが必要です
個別の Teams 接続:```json
"connectionReferences": {
  "shared_teams": {
    "connectionName": "shared-teams-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "id": "/providers/Microsoft.PowerApps/apis/shared_teams"
  },
  "shared_teams_1": {
    "connectionName": "shared-teams-yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
    "id": "/providers/Microsoft.PowerApps/apis/shared_teams"
  }
}
```どちらも **同じ基礎となる Teams アカウント**を指すことができますが、登録する必要があります
2 つの異なる接続参照として。 Webhook (`OpenApiConnectionWebhook`)
`shared_teams` を使用し、後続のメッセージ アクションは `shared_teams_1` を使用します。