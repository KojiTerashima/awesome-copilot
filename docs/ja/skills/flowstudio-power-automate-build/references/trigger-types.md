# FlowStudio MCP — トリガーの種類

Power Automate フロー定義のトリガー定義をコピーして貼り付けます。

---

## 再発

スケジュールに従って実行します。```json
"Recurrence": {
  "type": "Recurrence",
  "recurrence": {
    "frequency": "Day",
    "interval": 1,
    "startTime": "2026-01-01T08:00:00Z",
    "timeZone": "AUS Eastern Standard Time"
  }
}
```毎週特定の日に:```json
"Recurrence": {
  "type": "Recurrence",
  "recurrence": {
    "frequency": "Week",
    "interval": 1,
    "schedule": {
      "weekDays": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
    },
    "startTime": "2026-01-05T09:00:00Z",
    "timeZone": "AUS Eastern Standard Time"
  }
}
```一般的な `timeZone` 値:
- `"AUS Eastern Standard Time"` — シドニー/メルボルン (UTC+10/+11)
- `"UTC"` — 世界時
- `"E. Australia Standard Time"` — ブリスベン (UTC+10 夏時間なし)
- `"New Zealand Standard Time"` — オークランド (UTC+12/+13)
- `"Pacific Standard Time"` — ロサンゼルス (UTC-8/-7)
- `"GMT Standard Time"` — ロンドン (UTC+0/+1)

---

## マニュアル (HTTP リクエスト / Power Apps)

JSON 本文を含む HTTP POST を受信します。```json
"manual": {
  "type": "Request",
  "kind": "Http",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "value": { "type": "integer" }
      },
      "required": ["name"]
    }
  }
}
```アクセス値: `@triggerBody()?['name']`  
保存後に使用できるトリガー URL: `@listCallbackUrl()`

#### スキーマなしバリアント (任意の JSON を受け入れる)

受信ペイロード構造が不明または異なる場合は、スキーマを省略します。
検証なしで有効な JSON 本文を受け入れるには:```json
"manual": {
  "type": "Request",
  "kind": "Http",
  "inputs": {
    "schema": {}
  }
}
```任意のフィールドに動的にアクセス: `@triggerBody()?['anyField']`

> これを外部 Webhook (Stripe、GitHub、Employment Hero など) に使用します。
> ペイロードの形状は変更されるか、完全に文書化されていない可能性があります。フローは何でも受け入れます
> 予期しないプロパティに対して 400 を返さない JSON。

---

## 自動化 (SharePoint アイテムが作成されました)```json
"When_an_item_is_created": {
  "type": "OpenApiConnectionNotification",
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "OnNewItem"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList"
    },
    "subscribe": {
      "body": { "notificationUrl": "@listCallbackUrl()" },
      "queries": {
        "dataset": "https://mytenant.sharepoint.com/sites/mysite",
        "table": "MyList"
      }
    }
  }
}
```アクセストリガーデータ：`@triggerBody()?['ID']`、`@triggerBody()?['Title']`など

---

## 自動化 (SharePoint アイテムの変更)```json
"When_an_existing_item_is_modified": {
  "type": "OpenApiConnectionNotification",
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "OnUpdatedItem"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList"
    },
    "subscribe": {
      "body": { "notificationUrl": "@listCallbackUrl()" },
      "queries": {
        "dataset": "https://mytenant.sharepoint.com/sites/mysite",
        "table": "MyList"
      }
    }
  }
}
```---

## 自動化 (Outlook: 新しいメールの到着時)```json
"When_a_new_email_arrives": {
  "type": "OpenApiConnectionNotification",
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connectionName": "<connectionName>",
      "operationId": "OnNewEmail"
    },
    "parameters": {
      "folderId": "Inbox",
      "to": "monitored@contoso.com",
      "isHTML": true
    },
    "subscribe": {
      "body": { "notificationUrl": "@listCallbackUrl()" }
    }
  }
}
```---

## 子フロー (別のフローによって呼び出される)```json
"manual": {
  "type": "Request",
  "kind": "Button",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "items": {
          "type": "array",
          "items": { "type": "object" }
        }
      }
    }
  }
}
```親が提供するデータにアクセスする: `@triggerBody()?['items']`

データを親に返すには、`Response` アクションを追加します。```json
"Respond_to_Parent": {
  "type": "Response",
  "runAfter": { "Compose_Result": ["Succeeded"] },
  "inputs": {
    "statusCode": 200,
    "body": "@outputs('Compose_Result')"
  }
}
```
