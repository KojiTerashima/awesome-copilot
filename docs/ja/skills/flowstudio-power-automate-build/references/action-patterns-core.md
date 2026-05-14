# FlowStudio MCP — アクションパターン: コア

Power Automate フロー定義の変数、制御フロー、および式パターン。

> すべての例は、`"runAfter"` が適切に設定されていることを前提としています。
> `<connectionName>` を、`connectionReferences` マップで使用した **キー** に置き換えます
> (例: `shared_teams`、`shared_office365`) — 接続 GUID ではありません。

---

## データと変数

### 作成 (値の保存)```json
"Compose_My_Value": {
  "type": "Compose",
  "runAfter": {},
  "inputs": "@variables('myVar')"
}
```参照: `@outputs('Compose_My_Value')`

---

### 変数の初期化```json
"Init_Counter": {
  "type": "InitializeVariable",
  "runAfter": {},
  "inputs": {
    "variables": [{
      "name": "counter",
      "type": "Integer",
      "value": 0
    }]
  }
}
```タイプ: `"Integer"`、`"Float"`、`"Boolean"`、`"String"`、`"Array"`、`"Object"`

---

### 変数を設定する```json
"Set_Counter": {
  "type": "SetVariable",
  "runAfter": {},
  "inputs": {
    "name": "counter",
    "value": "@add(variables('counter'), 1)"
  }
}
```---

### 配列変数に追加```json
"Collect_Item": {
  "type": "AppendToArrayVariable",
  "runAfter": {},
  "inputs": {
    "name": "resultArray",
    "value": "@item()"
  }
}
```---

### 変数をインクリメントします```json
"Increment_Counter": {
  "type": "IncrementVariable",
  "runAfter": {},
  "inputs": {
    "name": "counter",
    "value": 1
  }
}
```> ループ内のカウンターには `IncrementVariable` (`SetVariable` と `add()` ではない) を使用します —
> これはアトミックであり、変数が他の場所で使用される場合の式エラーを回避します。
> 同じ繰り返しです。 `value` には、任意の整数または式を指定できます。 @@コード4@@
> Unix タイムスタンプ カーソルを N 分進めます。

---

## 制御フロー

### 条件 (If/Else)```json
"Check_Status": {
  "type": "If",
  "runAfter": {},
  "expression": {
    "and": [{ "equals": ["@item()?['Status']", "Active"] }]
  },
  "actions": {
    "Handle_Active": {
      "type": "Compose",
      "runAfter": {},
      "inputs": "Active user: @{item()?['Name']}"
    }
  },
  "else": {
    "actions": {
      "Handle_Inactive": {
        "type": "Compose",
        "runAfter": {},
        "inputs": "Inactive user"
      }
    }
  }
}
```比較演算子: `equals`、`not`、`greater`、`greaterOrEquals`、`less`、`lessOrEquals`、`contains`  
論理: `and: [...]`、`or: [...]`

---

### スイッチ```json
"Route_By_Type": {
  "type": "Switch",
  "runAfter": {},
  "expression": "@triggerBody()?['type']",
  "cases": {
    "Case_Email": {
      "case": "email",
      "actions": { "Process_Email": { "type": "Compose", "runAfter": {}, "inputs": "email" } }
    },
    "Case_Teams": {
      "case": "teams",
      "actions": { "Process_Teams": { "type": "Compose", "runAfter": {}, "inputs": "teams" } }
    }
  },
  "default": {
    "actions": { "Unknown_Type": { "type": "Compose", "runAfter": {}, "inputs": "unknown" } }
  }
}
```---

### スコープ (グループ化/トライキャッチ)

関連するアクションをスコープでラップして共有名を付け、
デザイナー、そして最も重要なことは、エラーを 1 つの単位として処理することです。```json
"Scope_Get_Customer": {
  "type": "Scope",
  "runAfter": {},
  "actions": {
    "HTTP_Get_Customer": {
      "type": "Http",
      "runAfter": {},
      "inputs": {
        "method": "GET",
        "uri": "https://api.example.com/customers/@{variables('customerId')}"
      }
    },
    "Compose_Email": {
      "type": "Compose",
      "runAfter": { "HTTP_Get_Customer": ["Succeeded"] },
      "inputs": "@outputs('HTTP_Get_Customer')?['body/email']"
    }
  }
},
"Handle_Scope_Error": {
  "type": "Compose",
  "runAfter": { "Scope_Get_Customer": ["Failed", "TimedOut"] },
  "inputs": "Scope failed: @{result('Scope_Get_Customer')?[0]?['error']?['message']}"
}
```> 参照スコープの結果: `@result('Scope_Get_Customer')` はアクションの配列を返します
> 結果。フォローアップ アクションでは `runAfter: {"MyScope": ["Failed", "TimedOut"]}` を使用してください
> Terminate を使用せずに try/catch セマンティクスを作成します。

---

### Foreach (シーケンシャル)```json
"Process_Each_Item": {
  "type": "Foreach",
  "runAfter": {},
  "foreach": "@outputs('Get_Items')?['body/value']",
  "operationOptions": "Sequential",
  "actions": {
    "Handle_Item": {
      "type": "Compose",
      "runAfter": {},
      "inputs": "@item()?['Title']"
    }
  }
}
```> 並列化が意図されている場合を除き、常に `"operationOptions": "Sequential"` を含めてください。

---

### Foreach (同時実行制限と並行)```json
"Process_Each_Item_Parallel": {
  "type": "Foreach",
  "runAfter": {},
  "foreach": "@body('Get_SP_Items')?['value']",
  "runtimeConfiguration": {
    "concurrency": {
      "repetitions": 20
    }
  },
  "actions": {
    "HTTP_Upsert": {
      "type": "Http",
      "runAfter": {},
      "inputs": {
        "method": "POST",
        "uri": "https://api.example.com/contacts/@{item()?['Email']}"
      }
    }
  }
}
```> `repetitions` を設定して、同時に処理される項目の数を制御します。
> 実際の値: 外部 API 呼び出しの場合は `5–10` (レート制限を考慮)、
> `20–50` 内部/高速操作用。
> プラットフォームのデフォルトでは `runtimeConfiguration.concurrency` を完全に省略します
>（現在50歳）。 `"operationOptions": "Sequential"` と同​​時実行性を一緒に使用しないでください。

---

### 待機 (遅延)```json
"Delay_10_Minutes": {
  "type": "Wait",
  "runAfter": {},
  "inputs": {
    "interval": {
      "count": 10,
      "unit": "Minute"
    }
  }
}
```有効な `unit` 値: `"Second"`、`"Minute"`、`"Hour"`、`"Day"`

> 重複排除ガードとして遅延と再フェッチを使用します。競合するプロセスを待ちます。
> 完了するには、行動する前にレコードをもう一度読んでください。これにより二重処理が回避されます
> 複数のトリガーまたは手動編集が同じ項目で競合する可能性がある場合。

---

### 終了 (成功または失敗)```json
"Terminate_Success": {
  "type": "Terminate",
  "runAfter": {},
  "inputs": {
    "runStatus": "Succeeded"
  }
},
"Terminate_Failure": {
  "type": "Terminate",
  "runAfter": { "Risky_Action": ["Failed"] },
  "inputs": {
    "runStatus": "Failed",
    "runError": {
      "code": "StepFailed",
      "message": "@{outputs('Get_Error_Message')}"
    }
  }
}
```---

### Do until (条件までループ)

終了条件が true になるまで、アクションのブロックを繰り返します。
反復回数が事前に不明な場合に使用します (例: API のページ分割、
時間範囲を歩き回り、ステータスが変化するまでポーリングします)。```json
"Do_Until_Done": {
  "type": "Until",
  "runAfter": {},
  "expression": "@greaterOrEquals(variables('cursor'), variables('endValue'))",
  "limit": {
    "count": 5000,
    "timeout": "PT5H"
  },
  "actions": {
    "Do_Work": {
      "type": "Compose",
      "runAfter": {},
      "inputs": "@variables('cursor')"
    },
    "Advance_Cursor": {
      "type": "IncrementVariable",
      "runAfter": { "Do_Work": ["Succeeded"] },
      "inputs": {
        "name": "cursor",
        "value": 1
      }
    }
  }
}
```> `limit.count` と `limit.timeout` を常に明示的に設定します。プラットフォームのデフォルトは次のとおりです。
> 低 (60 回の反復、1 時間)。時間範囲ウォーカーの場合は `limit.count: 5000` を使用し、
> `limit.timeout: "PT5H"` (ISO 8601 期間)。
>
> 終了条件は各反復の前**に評価されます。カーソルを初期化する
> ループの前に変数を追加して、最初のパスで条件が正しく評価できるようにします。

---

### RequestId 相関を使用した非同期ポーリング

API が長時間実行ジョブを非同期で開始するとき (例: Power BI データセットの更新、
レポート生成、バッチ エクスポートなど）、トリガー呼び出しはリクエスト ID を返します。捕まえてください
**応答ヘッダー**から取得し、その正確な ID でフィルタリングしてステータス エンドポイントをポーリングします。```json
"Start_Job": {
  "type": "Http",
  "inputs": { "method": "POST", "uri": "https://api.example.com/jobs" }
},
"Capture_Request_ID": {
  "type": "Compose",
  "runAfter": { "Start_Job": ["Succeeded"] },
  "inputs": "@outputs('Start_Job')?['headers/X-Request-Id']"
},
"Initialize_Status": {
  "type": "InitializeVariable",
  "inputs": { "variables": [{ "name": "jobStatus", "type": "String", "value": "Running" }] }
},
"Poll_Until_Done": {
  "type": "Until",
  "expression": "@not(equals(variables('jobStatus'), 'Running'))",
  "limit": { "count": 60, "timeout": "PT30M" },
  "actions": {
    "Delay": { "type": "Wait", "inputs": { "interval": { "count": 20, "unit": "Second" } } },
    "Get_History": {
      "type": "Http",
      "runAfter": { "Delay": ["Succeeded"] },
      "inputs": { "method": "GET", "uri": "https://api.example.com/jobs/history" }
    },
    "Filter_This_Job": {
      "type": "Query",
      "runAfter": { "Get_History": ["Succeeded"] },
      "inputs": {
        "from": "@outputs('Get_History')?['body/items']",
        "where": "@equals(item()?['requestId'], outputs('Capture_Request_ID'))"
      }
    },
    "Set_Status": {
      "type": "SetVariable",
      "runAfter": { "Filter_This_Job": ["Succeeded"] },
      "inputs": {
        "name": "jobStatus",
        "value": "@first(body('Filter_This_Job'))?['status']"
      }
    }
  }
},
"Handle_Failure": {
  "type": "If",
  "runAfter": { "Poll_Until_Done": ["Succeeded"] },
  "expression": { "equals": ["@variables('jobStatus')", "Failed"] },
  "actions": { "Terminate_Failed": { "type": "Terminate", "inputs": { "runStatus": "Failed" } } },
  "else": { "actions": {} }
}
```アクセス応答ヘッダー: `@outputs('Start_Job')?['headers/X-Request-Id']`

> **ステータス変数の初期化**: 前にセンチネル値 (`"Running"`、`"Unknown"`) を設定します。
> ループです。終了条件は、センチネル以外の値をテストします。
> この方法では、空のポーリング結果 (まだ履歴にないジョブ) では変数が変更されないままになります。
> そしてループは継続します。誤って null で終了することはありません。
>
> **抽出する前にフィルタリング**: 常に `Filter Array` 履歴を特定のファイルに保存します
> `first()` を呼び出す前に ID をリクエストしてください。履歴エンドポイントはすべてのジョブを返します。なしで
> フィルタリングすると、別の同時ジョブからのステータスによってポーリングが破損する可能性があります。

---

### runAfter フォールバック (失敗 → 代替アクション)

プライマリ アクションが失敗したときに、Condition ブロックを使用せずにフォールバック アクションにルーティングします。
フォールバックに `runAfter` を設定して、プライマリから `["Failed"]` を受け入れるだけです。```json
"HTTP_Get_Hi_Res": {
  "type": "Http",
  "runAfter": {},
  "inputs": { "method": "GET", "uri": "https://api.example.com/data?resolution=hi-res" }
},
"HTTP_Get_Low_Res": {
  "type": "Http",
  "runAfter": { "HTTP_Get_Hi_Res": ["Failed"] },
  "inputs": { "method": "GET", "uri": "https://api.example.com/data?resolution=low-res" }
}
```> 続くアクションでは、`["Succeeded", "Skipped"]` の両方を受け入れる `runAfter` を使用できます。
> いずれかのパスを処理します。以下の **ファンイン参加ゲート** を参照してください。

---

### ファンイン結合ゲート (相互に排他的な 2 つのブランチをマージ)

2 つのブランチが相互に排他的である場合 (実行ごとに 1 つのブランチのみが成功できる)、単一のブランチを使用します。
**両方**のブランチから `["Succeeded", "Skipped"]` を受け入れるダウンストリーム アクション。
どのブランチが実行されたかに関係なく、ゲートは 1 回だけ起動します。```json
"Increment_Count": {
  "type": "IncrementVariable",
  "runAfter": {
    "Update_Hi_Res_Metadata":  ["Succeeded", "Skipped"],
    "Update_Low_Res_Metadata": ["Succeeded", "Skipped"]
  },
  "inputs": { "name": "LoopCount", "value": 1 }
}
```> これにより、各ブランチでの下流アクションの重複が回避されます。重要な洞察:
> どのブランチがスキップされたとしても `Skipped` が報告されます — ゲートはその状態を受け入れ、
> 1回発火します。 2 つのブランチが真に相互排他的である場合にのみ正常に動作します。
> (例: 1 つは `runAfter: [...Failed]` で、もう 1 つは `runAfter: [...Failed]` です)。

---

## 式

### 一般的な表現パターン```
Null-safe field access:    @item()?['FieldName']
Null guard:                @coalesce(item()?['Name'], 'Unknown')
String format:             @{variables('firstName')} @{variables('lastName')}
Date today:                @utcNow()
Formatted date:            @formatDateTime(utcNow(), 'dd/MM/yyyy')
Add days:                  @addDays(utcNow(), 7)
Array length:              @length(variables('myArray'))
Filter array:              Use the "Filter array" action (no inline filter expression exists in PA)
Union (new wins):          @union(body('New_Data'), outputs('Old_Data'))
Sort:                      @sort(variables('myArray'), 'Date')
Unix timestamp → date:     @formatDateTime(addseconds('1970-1-1', triggerBody()?['created']), 'yyyy-MM-dd')
Date → Unix milliseconds:  @div(sub(ticks(startOfDay(item()?['Created'])), ticks(formatDateTime('1970-01-01Z','o'))), 10000)
Date → Unix seconds:       @div(sub(ticks(item()?['Start']), ticks('1970-01-01T00:00:00Z')), 10000000)
Unix seconds → datetime:   @addSeconds('1970-01-01T00:00:00Z', int(variables('Unix')))
Coalesce as no-else:       @coalesce(outputs('Optional_Step'), outputs('Default_Step'))
Flow elapsed minutes:      @div(float(sub(ticks(utcNow()), ticks(outputs('Flow_Start')))), 600000000)
HH:mm time string:         @formatDateTime(outputs('Local_Datetime'), 'HH:mm')
Response header:           @outputs('HTTP_Action')?['headers/X-Request-Id']
Array max (by field):      @reverse(sort(body('Select_Items'), 'Date'))[0]
Integer day span:          @int(split(dateDifference(outputs('Start'), outputs('End')), '.')[0])
ISO week number:           @div(add(dayofyear(addDays(subtractFromTime(date, sub(dayofweek(date),1), 'Day'), 3)), 6), 7)
Join errors to string:     @if(equals(length(variables('Errors')),0), null, concat(join(variables('Errors'),', '),' not found.'))
Normalize before compare:  @replace(coalesce(outputs('Value'),''),'_',' ')
Robust non-empty check:    @greater(length(trim(coalesce(string(outputs('Val')), ''))), 0)
```### 式内の改行

> **`\n` は Power Automate 式内で改行を生成しません。**
> リテラルのバックスラッシュ + `n` として扱われ、そのまま表示されるか、
> 検証エラーです。

改行文字が必要な場合は常に `decodeUriComponent('%0a')` を使用します。```
Newline (LF):   decodeUriComponent('%0a')
CRLF:           decodeUriComponent('%0d%0a')
```例 - 複数行のチームまたは `concat()` 経由の電子メール本文:```json
"Compose_Message": {
  "type": "Compose",
  "inputs": "@concat('Hi ', outputs('Get_User')?['body/displayName'], ',', decodeUriComponent('%0a%0a'), 'Your report is ready.', decodeUriComponent('%0a'), '- The Team')"
}
```例 — `join()` と改行区切り文字:```json
"Compose_List": {
  "type": "Compose",
  "inputs": "@join(body('Select_Names'), decodeUriComponent('%0a'))"
}
```> これは、動的に構築された文字列に改行を埋め込む唯一の信頼できる方法です
> Power Automate フロー定義内 (Logic Apps ランタイムに対して確認済み)。

---

### 配列の合計 (XPath トリック)

Power Automate にはネイティブの `sum()` 関数がありません。代わりに XML で XPath を使用します。```json
"Prepare_For_Sum": {
  "type": "Compose",
  "runAfter": {},
  "inputs": { "root": { "numbers": "@body('Select_Amounts')" } }
},
"Sum": {
  "type": "Compose",
  "runAfter": { "Prepare_For_Sum": ["Succeeded"] },
  "inputs": "@xpath(xml(outputs('Prepare_For_Sum')), 'sum(/root/numbers)')"
}
````Select_Amounts` は、数値のフラットな配列を出力する必要があります (**選択** アクションを使用して、最初に単一の数値フィールドを抽出します)。結果は、条件または計算で直接使用できる数値になります。

> これは、Power Automate でループを使用せずに配列を集計 (合計/最小/最大) する唯一の方法です。