# FlowStudio MCP — アクション パターン: データ変換

配列操作、HTTP 呼び出し、解析、およびデータ変換パターン。

> すべての例は、`"runAfter"` が適切に設定されていることを前提としています。
> `<connectionName>` は、GUID ではなく、`connectionReferences` (例: `shared_sharepointonline`) の **キー** です。
> GUID はマップ値の `connectionName` プロパティに入力されます。

---

## 配列操作

### 選択 (配列の再形成/投影)

配列内の各項目を変換し、必要な列のみを保持するか、列の名前を変更します。
フローの残りの部分で大きなオブジェクトを運ぶことを回避します。```json
"Select_Needed_Columns": {
  "type": "Select",
  "runAfter": {},
  "inputs": {
    "from": "@outputs('HTTP_Get_Subscriptions')?['body/data']",
    "select": {
      "id":           "@item()?['id']",
      "status":       "@item()?['status']",
      "trial_end":    "@item()?['trial_end']",
      "cancel_at":    "@item()?['cancel_at']",
      "interval":     "@item()?['plan']?['interval']"
    }
  }
}
```結果参照: `@body('Select_Needed_Columns')` — 再形成されたオブジェクトの直接配列を返します。

> ループまたはフィルタリングの前に選択を使用して、ペイロード サイズを削減し、簡素化します。
> 下流の式。 SP 結果、HTTP 応答、変数など、あらゆる配列で動作します。
>
> **ヒント:**
> - **単一から配列への強制:** API が単一のオブジェクトを返すが、必要な場合
> 選択 (配列が必要) してラップします: `@array(body('Get_Employee')?['data'])`。
> 出力は 1 要素の配列です。`?[0]?['field']` 経由で結果にアクセスします。
> - **オプションのフィールドを Null 正規化します:** `@if(empty(item()?['field']), null, item()?['field'])` を使用します
> すべてのオプションのフィールドで、空の文字列、欠落しているプロパティ、空の文字列を正規化します。
> 明示的な `null` へのオブジェクト。一貫したダウンストリーム `@equals(..., @null)` チェックを保証します。
> - **ネストされたオブジェクトをフラット化する:** ネストされたプロパティをフラット フィールドに投影する:
>```
>   "manager_name": "@if(empty(item()?['manager']?['name']), null, item()?['manager']?['name'])"
>   ```> これにより、別のソースからのフラット スキーマとのフィールド レベルの直接比較が可能になります。

---

### フィルター配列 (クエリ)

条件に一致する項目に配列をフィルターします。アクション フォームを使用します (`filter()` ではありません)
式) は、複雑な複数条件ロジックの場合に使用します。これにより、より明確になり、保守が容易になります。```json
"Filter_Active_Subscriptions": {
  "type": "Query",
  "runAfter": {},
  "inputs": {
    "from": "@body('Select_Needed_Columns')",
    "where": "@and(or(equals(item().status, 'trialing'), equals(item().status, 'active')), equals(item().cancel_at, null))"
  }
}
```結果参照: `@body('Filter_Active_Subscriptions')` — 直接フィルターされた配列。

> ヒント: 同じソース配列に対して複数のフィルター配列アクションを実行して作成します
> 名前付きバケット (アクティブ、キャンセル中、完全にキャンセルなど) の場合は、次を使用します
> `coalesce(first(body('Filter_A')), first(body('Filter_B')), ...)` を選択します
> ループを含まない最も優先度の高い一致。

---

### CSV テーブルの作成 (配列 → CSV 文字列)

オブジェクトの配列を CSV 形式の文字列に変換します。コネクタ呼び出しやコードは必要ありません。
`Select` または `Filter Array` の後に使用して、データをエクスポートするか、ファイル書き込みアクションに渡します。```json
"Create_CSV": {
  "type": "Table",
  "runAfter": {},
  "inputs": {
    "from": "@body('Select_Output_Columns')",
    "format": "CSV"
  }
}
```結果参照: `@body('Create_CSV')` — ヘッダー行とデータ行を含むプレーン文字列。```json
// Custom column order / renamed headers:
"Create_CSV_Custom": {
  "type": "Table",
  "inputs": {
    "from": "@body('Select_Output_Columns')",
    "format": "CSV",
    "columns": [
      { "header": "Date",        "value": "@item()?['transactionDate']" },
      { "header": "Amount",      "value": "@item()?['amount']" },
      { "header": "Description", "value": "@item()?['description']" }
    ]
  }
}
```> `columns` を指定しないと、ヘッダーはソース配列内のオブジェクト プロパティ名から取得されます。
> `columns` を使用すると、ヘッダー名と列の順序を明示的に制御できます。
>
> 出力は生の文字列です。 `CreateFile` または `UpdateFile` を使用してファイルに書き込みます
> (`body` を `@body('Create_CSV')` に設定する)、または `SetVariable` を使用して変数に格納します。
>
> ソース データが Power BI の `ExecuteDatasetQuery` から取得された場合、列名は次のようになります。
> 角括弧で囲みます (例: `[Amount]`)。書き込む前にそれらを取り除きます。
> `@replace(replace(body('Create_CSV'),'[',''),']','')`

---

### range() + 配列生成用の選択

`range(0, N)` は整数シーケンス `[0, 1, 2, …, N-1]` を生成します。パイプで通す
日付シリーズ、インデックス グリッド、または任意の計算配列を生成するアクションを選択します。
ループなし:```json
// Generate 14 consecutive dates starting from a base date
"Generate_Date_Series": {
  "type": "Select",
  "inputs": {
    "from": "@range(0, 14)",
    "select": "@addDays(outputs('Base_Date'), item(), 'yyyy-MM-dd')"
  }
}
```結果: `@body('Generate_Date_Series')` → `["2025-01-06", "2025-01-07", …, "2025-01-19"]````json
// Flatten a 2D array (rows × cols) into 1D using arithmetic indexing
"Flatten_Grid": {
  "type": "Select",
  "inputs": {
    "from": "@range(0, mul(length(outputs('Rows')), length(outputs('Cols'))))",
    "select": {
      "row": "@outputs('Rows')[div(item(), length(outputs('Cols')))]",
      "col": "@outputs('Cols')[mod(item(), length(outputs('Cols')))]"
    }
  }
}
```> `range()` はゼロベースです。上記のデカルト積パターンでは `div(i, cols)` が使用されています
> 行インデックスの場合は `mod(i, cols)` 列インデックスの場合 -
> ネストされた for ループが 1 つのパスに平坦化されました。タイムスロット生成に便利×
> 日付グリッド、シフト×場所の割り当てなど。

---

### json(concat(join())) による動的辞書

実行時に O(1) キー→値の検索が必要で、Power Automate にネイティブがない場合
辞書タイプの場合、Select + join + json を使用して配列から辞書を構築します。```json
"Build_Key_Value_Pairs": {
  "type": "Select",
  "inputs": {
    "from": "@body('Get_Lookup_Items')?['value']",
    "select": "@concat('\"', item()?['Key'], '\":\"', item()?['Value'], '\"')"
  }
},
"Assemble_Dictionary": {
  "type": "Compose",
  "inputs": "@json(concat('{', join(body('Build_Key_Value_Pairs'), ','), '}'))"
}
```検索: `@outputs('Assemble_Dictionary')?['myKey']````json
// Practical example: date → rate-code lookup for business rules
"Build_Holiday_Rates": {
  "type": "Select",
  "inputs": {
    "from": "@body('Get_Holidays')?['value']",
    "select": "@concat('\"', formatDateTime(item()?['Date'], 'yyyy-MM-dd'), '\":\"', item()?['RateCode'], '\"')"
  }
},
"Holiday_Dict": {
  "type": "Compose",
  "inputs": "@json(concat('{', join(body('Build_Holiday_Rates'), ','), '}'))"
}
```次にループ内: `@coalesce(outputs('Holiday_Dict')?[item()?['Date']], 'Standard')`

> `json(concat('{', join(...), '}'))` パターンは文字列値に対して機能します。数値の場合
> またはブール値の場合は、値部分を囲む内側のエスケープ引用符を省略します。
> キーは一意である必要があります。重複したキーは以前のキーを警告なく上書きします。
> これは、深くネストされた `if(equals(key,'A'),'X', if(equals(key,'B'),'Y', ...))` チェーンを置き換えます。

---

### 変更フィールド検出用の Union()

複数のフィールドの *いずれか* が変更されたレコードを検索する必要がある場合は、次のいずれかを実行します。
`Filter Array` フィールドごとと `union()` 結果。そうすることでコンプレックスを回避できる
複数条件フィルターを使用して、クリーンな重複排除されたセットを生成します。```json
"Filter_Name_Changed": {
  "type": "Query",
  "inputs": { "from": "@body('Existing_Records')",
              "where": "@not(equals(item()?['name'], item()?['dest_name']))" }
},
"Filter_Status_Changed": {
  "type": "Query",
  "inputs": { "from": "@body('Existing_Records')",
              "where": "@not(equals(item()?['status'], item()?['dest_status']))" }
},
"All_Changed": {
  "type": "Compose",
  "inputs": "@union(body('Filter_Name_Changed'), body('Filter_Status_Changed'))"
}
```参照: `@outputs('All_Changed')` — 変更があった行の重複排除された配列。

> `union()` はオブジェクト ID によって重複を排除するため、両方のフィールドで変更された行
> が 1 回表示されます。必要に応じて、`Filter_*_Changed` 入力を `union()` に追加します。
> @@コード4@@

---

### ファイルコンテンツ変更ゲート

ファイルまたは BLOB に対して負荷の高い処理を実行する前に、その現在の内容を比較してください
保存されたベースラインに。何も変更されていない場合は完全にスキップ — 同期フローを作成します
冪等で安全に再実行したり、積極的にスケジュールしたりできます。```json
"Get_File_From_Source": { ... },
"Get_Stored_Baseline": { ... },
"Condition_File_Changed": {
  "type": "If",
  "expression": {
    "not": {
      "equals": [
        "@base64(body('Get_File_From_Source'))",
        "@body('Get_Stored_Baseline')"
      ]
    }
  },
  "actions": {
    "Update_Baseline": { "...": "overwrite stored copy with new content" },
    "Process_File":    { "...": "all expensive work goes here" }
  },
  "else": { "actions": {} }
}
```> ベースラインをファイルとして SharePoint または BLOB ストレージに保存します — `base64()` をエンコードします
> 比較する前にライブコンテンツを実行するため、バイナリファイルとテキストファイルが均一に処理されます。
> 部分的な失敗後に再実行できるように、新しいベースラインを処理の**前**に作成します。
> 同じファイルを再度再処理しません。

---

### 同期のためのセット結合 (ネストされたループを使用しない更新検出)

ソース コレクションを宛先に同期するとき (例: API 応答 → SharePoint リスト、
CSV → データベース)、変更されたレコードを検索するためにネストされた `Apply to each` ループを回避します。
代わりに、**フラット キー配列を投影**し、`contains()` を使用して集合演算を実行します。
ネストされたループはゼロで、最後のループは変更された項目のみに触れます。

**完全な挿入/更新/削除同期パターン:**```json
// Step 1 — Project a flat key array from the DESTINATION (e.g. SharePoint)
"Select_Dest_Keys": {
  "type": "Select",
  "inputs": {
    "from": "@outputs('Get_Dest_Items')?['body/value']",
    "select": "@item()?['Title']"
  }
}
// → ["KEY1", "KEY2", "KEY3", ...]

// Step 2 — INSERT: source rows whose key is NOT in destination
"Filter_To_Insert": {
  "type": "Query",
  "inputs": {
    "from": "@body('Source_Array')",
    "where": "@not(contains(body('Select_Dest_Keys'), item()?['key']))"
  }
}
// → Apply to each Filter_To_Insert → CreateItem

// Step 3 — INNER JOIN: source rows that exist in destination
"Filter_Already_Exists": {
  "type": "Query",
  "inputs": {
    "from": "@body('Source_Array')",
    "where": "@contains(body('Select_Dest_Keys'), item()?['key'])"
  }
}

// Step 4 — UPDATE: one Filter per tracked field, then union them
"Filter_Field1_Changed": {
  "type": "Query",
  "inputs": {
    "from": "@body('Filter_Already_Exists')",
    "where": "@not(equals(item()?['field1'], item()?['dest_field1']))"
  }
}
"Filter_Field2_Changed": {
  "type": "Query",
  "inputs": {
    "from": "@body('Filter_Already_Exists')",
    "where": "@not(equals(item()?['field2'], item()?['dest_field2']))"
  }
}
"Union_Changed": {
  "type": "Compose",
  "inputs": "@union(body('Filter_Field1_Changed'), body('Filter_Field2_Changed'))"
}
// → rows where ANY tracked field differs

// Step 5 — Resolve destination IDs for changed rows (no nested loop)
"Select_Changed_Keys": {
  "type": "Select",
  "inputs": { "from": "@outputs('Union_Changed')", "select": "@item()?['key']" }
}
"Filter_Dest_Items_To_Update": {
  "type": "Query",
  "inputs": {
    "from": "@outputs('Get_Dest_Items')?['body/value']",
    "where": "@contains(body('Select_Changed_Keys'), item()?['Title'])"
  }
}
// Step 6 — Single loop over changed items only
"Apply_to_each_Update": {
  "type": "Foreach",
  "foreach": "@body('Filter_Dest_Items_To_Update')",
  "actions": {
    "Get_Source_Row": {
      "type": "Query",
      "inputs": {
        "from": "@outputs('Union_Changed')",
        "where": "@equals(item()?['key'], items('Apply_to_each_Update')?['Title'])"
      }
    },
    "Update_Item": {
      "...": "...",
      "id": "@items('Apply_to_each_Update')?['ID']",
      "item/field1": "@first(body('Get_Source_Row'))?['field1']"
    }
  }
}

// Step 7 — DELETE: destination keys NOT in source
"Select_Source_Keys": {
  "type": "Select",
  "inputs": { "from": "@body('Source_Array')", "select": "@item()?['key']" }
}
"Filter_To_Delete": {
  "type": "Query",
  "inputs": {
    "from": "@outputs('Get_Dest_Items')?['body/value']",
    "where": "@not(contains(body('Select_Source_Keys'), item()?['Title']))"
  }
}
// → Apply to each Filter_To_Delete → DeleteItem
```> **これがネストされたループに勝る理由**: 単純なアプローチ (dest 項目ごとに、ソースをスキャン)
> は O(n × m) であり、大きなリストでは Power Automate の 100k アクションの実行制限にすぐに達します。
> このパターンは O(n + m) です。キー配列の構築に 1 つのパス、フィルターごとに 1 つのパス。
> ステップ 6 の更新ループは、*変更された* レコードのみを反復します (多くの場合、ごく一部です)
> フルコレクションの。さらに高速化するには、**並列スコープ**でステップ 2/4/7 を実行します。

---

### 最初または Null の単一行の検索

結果配列で `first()` を使用して、ループなしで 1 つのレコードを抽出します。
次に、出力を null チェックして、ダウンストリームのアクションを保護します。```json
"Get_First_Match": {
  "type": "Compose",
  "runAfter": { "Get_SP_Items": ["Succeeded"] },
  "inputs": "@first(outputs('Get_SP_Items')?['body/value'])"
}
```条件で、**`@null` リテラル** (`empty()` ではない) との不一致をテストします。```json
"Condition": {
  "type": "If",
  "expression": {
    "not": {
      "equals": [
        "@outputs('Get_First_Match')",
        "@null"
      ]
    }
  }
}
```一致した行のフィールドにアクセスします: `@outputs('Get_First_Match')?['FieldName']`

> 一致するレコードが 1 つだけ必要な場合は、`Apply to each` の代わりにこれを使用します。
> `first()` が空の配列の場合は `null` を返します。 `empty()` は配列/文字列用です。
> スカラーではありません — `first()` の結果に対して使用すると、実行時エラーが発生します。

---

## HTTP と解析

### HTTP アクション (外部 API)```json
"Call_External_API": {
  "type": "Http",
  "runAfter": {},
  "inputs": {
    "method": "POST",
    "uri": "https://api.example.com/endpoint",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer @{variables('apiToken')}"
    },
    "body": {
      "data": "@outputs('Compose_Payload')"
    },
    "retryPolicy": {
      "type": "Fixed",
      "count": 3,
      "interval": "PT10S"
    }
  }
}
```応答参照: `@outputs('Call_External_API')?['body']`

#### バリアント: ActiveDirectoryOAuth (サービス間)

Azure AD クライアント資格情報を必要とする API (Microsoft Graph など) を呼び出す場合、
Bearer トークン変数の代わりにインライン OAuth を使用します。```json
"Call_Graph_API": {
  "type": "Http",
  "runAfter": {},
  "inputs": {
    "method": "GET",
    "uri": "https://graph.microsoft.com/v1.0/users?$search=\"employeeId:@{variables('Code')}\"&$select=id,displayName",
    "headers": {
      "Content-Type": "application/json",
      "ConsistencyLevel": "eventual"
    },
    "authentication": {
      "type": "ActiveDirectoryOAuth",
      "authority": "https://login.microsoftonline.com",
      "tenant": "<tenant-id>",
      "audience": "https://graph.microsoft.com",
      "clientId": "<app-registration-id>",
      "secret": "@parameters('graphClientSecret')"
    }
  }
}
```> **使用する場合:** Microsoft Graph、Azure Resource Manager、またはその他の呼び出し
> プレミアム コネクタを使用しないフローからの Azure AD で保護された API。
>
> `authentication` ブロックは、OAuth クライアント資格情報フロー全体を処理します
> 透過的 — 手動によるトークン取得手順は必要ありません。
>
> `ConsistencyLevel: eventual` は、Graph `$search` クエリに必要です。
> これがないと、`$search` は 400 を返します。
>
> PATCH/PUT 書き込みの場合、同じ `authentication` ブロックが機能します - 変更するだけです
> `method` に `body` を追加します。
>
> ⚠️ **`secret` をインラインでハードコーディングしないでください。** `@parameters('graphClientSecret')` を使用してください
> フローの `parameters` ブロックで宣言します (`securestring` と入力します)。これ
> シークレットが実行履歴に表示されたり、シークレットが読み取り可能になったりすることを防ぎます。
> `get_live_flow`。次のようにパラメータを宣言します。
>```json
> "parameters": {
>   "graphClientSecret": { "type": "securestring", "defaultValue": "" }
> }
> ```> 次に、フローの接続または環境変数を介して実際の値を渡します
> — 決してソース管理にコミットしないでください。

---

### HTTP レスポンス (呼び出し元に戻る)

HTTP によってトリガーされるフローで使用され、構造化された応答を呼び出し元に送り返します。
フローがタイムアウトする前に実行する必要があります (同期 HTTP のデフォルトは 2 分)。```json
"Response": {
  "type": "Response",
  "runAfter": {},
  "inputs": {
    "statusCode": 200,
    "headers": {
      "Content-Type": "application/json"
    },
    "body": {
      "status": "success",
      "message": "@{outputs('Compose_Result')}"
    }
  }
}
```> **PowerApps / ローコード呼び出し元パターン**: 常に `statusCode: 200` を返します。
> 本文の `status` フィールド (`"success"` / `"error"`)。 PowerApps HTTP アクション
> 非 2xx 応答を適切に処理しない - 呼び出し元は検査する必要がある
> HTTP ステータス コードではなく `body.status`。
>
> 複数の応答アクション (ブランチごとに 1 つ) を使用して、各パスが返されるようにします。
> 適切なメッセージ。 1 回の実行につき 1 つだけが実行されます。

---

### 子フロー呼び出し (HTTP POST 経由の親→子)

Power Automate は、子フローの呼び出しによる親→子のオーケストレーションをサポートします。
HTTP トリガー URL を直接指定します。親は HTTP POST を送信し、
子は `Response` アクションを返します。子フローは `manual` (リクエスト) トリガーを使用します。```json
// PARENT — call child flow and wait for its response
"Call_Child_Flow": {
  "type": "Http",
  "inputs": {
    "method": "POST",
    "uri": "https://prod-XX.australiasoutheast.logic.azure.com:443/workflows/<workflowId>/triggers/manual/paths/invoke?api-version=2016-06-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<SAS>",
    "headers": { "Content-Type": "application/json" },
    "body": {
      "ID": "@triggerBody()?['ID']",
      "WeekEnd": "@triggerBody()?['WeekEnd']",
      "Payload": "@variables('dataArray')"
    },
    "retryPolicy": { "type": "none" }
  },
  "operationOptions": "DisableAsyncPattern",
  "runtimeConfiguration": {
    "contentTransfer": { "transferMode": "Chunked" }
  },
  "limit": { "timeout": "PT2H" }
}
```

```json
// CHILD — manual trigger receives the JSON body
// (trigger definition)
"manual": {
  "type": "Request",
  "kind": "Http",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "ID": { "type": "string" },
        "WeekEnd": { "type": "string" },
        "Payload": { "type": "array" }
      }
    }
  }
}

// CHILD — return result to parent
"Response_Success": {
  "type": "Response",
  "inputs": {
    "statusCode": 200,
    "headers": { "Content-Type": "application/json" },
    "body": { "Result": "Success", "Count": "@length(variables('processed'))" }
  }
}
```> **`retryPolicy: none`** — 親の HTTP 呼び出しで重要です。それがなければ子供は
> フロー タイムアウトにより再試行がトリガーされ、重複した子の実行が生成されます。
>
> **`DisableAsyncPattern`** — 親が 202 Accepted を次のように扱うことを防ぎます。
> 完成です。親は、子が `Response` を送信するまでブロックします。
>
> **`transferMode: Chunked`** — 大きな配列 (>100 KB) を子に渡すときに有効にします。
> リクエストサイズの制限を回避します。
>
> **`limit.timeout: PT2H`** — 長時間実行する場合、デフォルトの 2 分の HTTP タイムアウトを引き上げます
>子供たち。最大はPT24Hです。
>
> 子フローのトリガー URL には、認証を行う SAS トークン (`sig=...`) が含まれています。
> 電話です。子フローのトリガー プロパティ パネルからコピーします。 URLが変わります
> トリガーが削除され、再作成された場合。

---

### JSON を解析する```json
"Parse_Response": {
  "type": "ParseJson",
  "runAfter": {},
  "inputs": {
    "content": "@outputs('Call_External_API')?['body']",
    "schema": {
      "type": "object",
      "properties": {
        "id": { "type": "integer" },
        "name": { "type": "string" },
        "items": {
          "type": "array",
          "items": { "type": "object" }
        }
      }
    }
  }
}
```解析された値にアクセスします: `@body('Parse_Response')?['name']`

---

### 手動 CSV → JSON (プレミアム アクションなし)

組み込みの式のみを使用して、生の CSV 文字列をオブジェクトの配列に解析します。
プレミアムの「CSV 解析」コネクタ アクションを回避します。```json
"Delimiter": {
  "type": "Compose",
  "inputs": ","
},
"Strip_Quotes": {
  "type": "Compose",
  "inputs": "@replace(body('Get_File_Content'), '\"', '')"
},
"Detect_Line_Ending": {
  "type": "Compose",
  "inputs": "@if(equals(indexOf(outputs('Strip_Quotes'), decodeUriComponent('%0D%0A')), -1), if(equals(indexOf(outputs('Strip_Quotes'), decodeUriComponent('%0A')), -1), decodeUriComponent('%0D'), decodeUriComponent('%0A')), decodeUriComponent('%0D%0A'))"
},
"Headers": {
  "type": "Compose",
  "inputs": "@split(first(split(outputs('Strip_Quotes'), outputs('Detect_Line_Ending'))), outputs('Delimiter'))"
},
"Data_Rows": {
  "type": "Compose",
  "inputs": "@skip(split(outputs('Strip_Quotes'), outputs('Detect_Line_Ending')), 1)"
},
"Select_CSV_Body": {
  "type": "Select",
  "inputs": {
    "from": "@outputs('Data_Rows')",
    "select": {
      "@{outputs('Headers')[0]}": "@split(item(), outputs('Delimiter'))[0]",
      "@{outputs('Headers')[1]}": "@split(item(), outputs('Delimiter'))[1]",
      "@{outputs('Headers')[2]}": "@split(item(), outputs('Delimiter'))[2]"
    }
  }
},
"Filter_Empty_Rows": {
  "type": "Query",
  "inputs": {
    "from": "@body('Select_CSV_Body')",
    "where": "@not(equals(item()?[outputs('Headers')[0]], null))"
  }
}
```結果: `@body('Filter_Empty_Rows')` — ヘッダー名をキーとして持つオブジェクトの配列。

> **`Detect_Line_Ending`** は CRLF (Windows)、LF (Unix)、および CR (古い Mac) を自動的に処理します
> `indexOf()` を `decodeUriComponent('%0D%0A' / '%0A' / '%0D')` とともに使用します。
>
> **`Select`** の動的キー名: `@{outputs('Headers')[0]}` 内の JSON キーとして
> `Select` シェイプは、実行時にヘッダー行から出力プロパティ名を設定します —
> これは、式が `@{...}` 補間構文である限り機能します。
>
> **カンマが埋め込まれた列**: フィールド値に区切り文字を含めることができる場合、
> スイッチで `length(split(row, ','))` を使用して列数を検出し、手動で
> 分割されたフラグメントを再構成します: `@concat(split(item(),',')[1],',',split(item(),',')[2])`

---

### ConvertTimeZone (組み込み、コネクタなし)

API 呼び出しやコネクタ ライセンスのコストをかけずに、タイムゾーン間のタイムスタンプを変換します。
フォーマット文字列 `"g"` は、短いロケール日付+時刻 (`M/d/yyyy h:mm tt`) を生成します。```json
"Convert_to_Local_Time": {
  "type": "Expression",
  "kind": "ConvertTimeZone",
  "runAfter": {},
  "inputs": {
    "baseTime": "@{outputs('UTC_Timestamp')}",
    "sourceTimeZone": "UTC",
    "destinationTimeZone": "Taipei Standard Time",
    "formatString": "g"
  }
}
```結果の参照: `@body('Convert_to_Local_Time')` — ほとんどのアクションとは異なり、**`outputs()` ではありません。

一般的な `formatString` 値: `"g"` (短縮)、`"f"` (完全)、`"yyyy-MM-dd"`、`"HH:mm"`

一般的なタイムゾーン文字列: `"UTC"`、`"AUS Eastern Standard Time"`、`"Taipei Standard Time"`、
`"Singapore Standard Time"`、`"GMT Standard Time"`

> これは `type: Expression, kind: ConvertTimeZone` — 組み込みの Logic Apps アクションです。
>コネクタではありません。接続参照は必要ありません。出力を参照するには
> `body()` (`outputs()` ではない)、それ以外の場合、式は null を返します。