# FlowStudio MCP — フロー定義スキーマ

`update_live_flow` によって予期される完全な JSON 構造 (そして `get_live_flow` によって返される)。

---

## 最上位の形状```json
{
  "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "$connections": {
      "defaultValue": {},
      "type": "Object"
    }
  },
  "triggers": {
    "<TriggerName>": { ... }
  },
  "actions": {
    "<ActionName>": { ... }
  },
  "outputs": {}
}
```---

## `triggers`

フロー定義ごとにトリガーは 1 つだけです。キー名は任意ですが、
従来の名前が使用されます (例: `Recurrence`、`manual`、`When_a_new_email_arrives`)。

すべてのトリガー テンプレートについては、[trigger-types.md](trigger-types.md) を参照してください。

---

## @@コード4@@

一意のアクション名をキーとするアクション定義のディクショナリ。
キー名にはスペースを含めることはできません。アンダースコアを使用してください。

各アクションには以下を含める必要があります。
- `type` — アクション タイプの識別子
- `runAfter` — 上流アクション名のマップ → ステータス条件配列
- `inputs` — アクション固有の入力構成

[action-patterns-core.md](action-patterns-core.md)、[action-patterns-data.md](action-patterns-data.md) を参照してください。
テンプレートの場合は [action-patterns-connectors.md](action-patterns-connectors.md) です。

### オプションのアクションのプロパティ

必須の `type`、`runAfter`、`inputs` 以外にも、次のアクションを含めることができます。

|プロパティ |目的 |
|---|---|
| `runtimeConfiguration` |ページネーション、同時実行性、安全なデータ、チャンク転送 |
| `operationOptions` | Foreach の場合は `"Sequential"`、HTTP の場合は `"DisableAsyncPattern"` |
| `limit` |タイムアウトオーバーライド (例: `{"timeout": "PT2H"}`) |

#### `runtimeConfiguration` バリアント

**ページネーション** (大きなリストを含む SharePoint Get Items):```json
"runtimeConfiguration": {
  "paginationPolicy": {
    "minimumItemCount": 5000
  }
}
```> これを行わないと、Get Items の結果は 256 件に制限されます。 `minimumItemCount`を設定します
> 予想される最大行数まで。 256 項目を超える SharePoint リストに必要です。

**同時実行性** (並列 Foreach):```json
"runtimeConfiguration": {
  "concurrency": {
    "repetitions": 20
  }
}
```**安全な入力/出力** (実行履歴内のマスク値):```json
"runtimeConfiguration": {
  "secureData": {
    "properties": ["inputs", "outputs"]
  }
}
```> 認証情報、トークン、または PII を処理するアクションで使用します。マスクされた値が表示されます
> フロー実行履歴 UI および API 応答内の `"<redacted>"` として。

**チャンク転送** (大きな HTTP ペイロード):```json
"runtimeConfiguration": {
  "contentTransfer": {
    "transferMode": "Chunked"
  }
}
```> 100 KB を超える本文を送信または受信する HTTP アクションで有効にします (例: 親→子)
> 大きな配列を使用したフロー呼び出し)。

---

## `runAfter` ルール

ブランチ内の最初のアクションには `"runAfter": {}` が含まれます (空 - トリガー後に実行されます)。

後続のアクションは依存関係を宣言します。```json
"My_Action": {
  "runAfter": {
    "Previous_Action": ["Succeeded"]
  }
}
```複数の上流依存関係:```json
"runAfter": {
  "Action_A": ["Succeeded"],
  "Action_B": ["Succeeded", "Skipped"]
}
```エラー処理アクション (アップストリームが失敗したときに実行):```json
"Log_Error": {
  "runAfter": {
    "Risky_Action": ["Failed"]
  }
}
```---

## `parameters` (フローレベル入力パラメータ)

オプション。フローレベルで再利用可能な値を定義します。```json
"parameters": {
  "listName": {
    "type": "string",
    "defaultValue": "MyList"
  },
  "maxItems": {
    "type": "integer",
    "defaultValue": 100
  }
}
```参照: 式文字列内の `@parameters('listName')`。

---

## `outputs`

クラウド フローではほとんど使用されません。フローが呼び出されない限り、`{}` のままにしておきます
子フローとして、値を返す必要があります。

データを返す子フローの場合:```json
"outputs": {
  "resultData": {
    "type": "object",
    "value": "@outputs('Compose_Result')"
  }
}
```---

## スコープ付きアクション (スコープ ブロック内)

エラー処理または明確にするためにグループ化する必要があるアクション:```json
"Scope_Main_Process": {
  "type": "Scope",
  "runAfter": {},
  "actions": {
    "Step_One": { ... },
    "Step_Two": { "runAfter": { "Step_One": ["Succeeded"] }, ... }
  }
}
```---

## 完全な最小限の例```json
{
  "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
  "contentVersion": "1.0.0.0",
  "triggers": {
    "Recurrence": {
      "type": "Recurrence",
      "recurrence": {
        "frequency": "Week",
        "interval": 1,
        "schedule": { "weekDays": ["Monday"] },
        "startTime": "2026-01-05T09:00:00Z",
        "timeZone": "AUS Eastern Standard Time"
      }
    }
  },
  "actions": {
    "Compose_Greeting": {
      "type": "Compose",
      "runAfter": {},
      "inputs": "Good Monday!"
    }
  },
  "outputs": {}
}
```
