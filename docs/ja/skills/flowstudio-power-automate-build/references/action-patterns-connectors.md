# FlowStudio MCP — アクション パターン: コネクタ

SharePoint、Outlook、Teams、および承認コネクタのアクション パターン。

> すべての例は、`"runAfter"` が適切に設定されていることを前提としています。
> `<connectionName>` を `connectionReferences` で使用した **キー** に置き換えます
> (例: `shared_sharepointonline`、`shared_teams`)。これは接続ではありません
> GUID — アクションをそのエントリにリンクする論理参照名です。
> `connectionReferences` マップ。

---

## シェアポイント

### SharePoint — アイテムの取得```json
"Get_SP_Items": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "GetItems"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList",
      "$filter": "Status eq 'Active'",
      "$top": 500
    }
  }
}
```結果の参照: `@outputs('Get_SP_Items')?['body/value']`

> **文字列補間を使用した動的 OData フィルター**: ランタイム値を挿入します
> `@{...}` 構文を使用して、`$filter` 文字列に直接入力します。
>```
> "$filter": "Title eq '@{outputs('ConfirmationCode')}'"  
> ```> 二重引用符内の単一引用符に注意してください — 正しい OData 文字列リテラル
> 構文。個別の変数アクションを回避します。

> **大きなリストのページネーション**: デフォルトでは、GetItems は `$top` で停止します。自動ページネーションするには
> さらに、アクションのページネーション ポリシーを有効にします。フロー定義ではこれ
> は次のように表示されます。
>```json
> "paginationPolicy": { "minimumItemCount": 10000 }
> ```> `minimumItemCount` を、予想される項目の最大数に設定します。コネクタは
> その数に達するかリストがなくなるまでページを取得し続けます。これがなければ、
> フローは、アイテム数が 5,000 を超えるリストに対して上限付きの結果をサイレントに返します。

---

### SharePoint — アイテムの取得 (ID による単一行)```json
"Get_SP_Item": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "GetItem"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList",
      "id": "@triggerBody()?['ID']"
    }
  }
}
```結果の参照: `@body('Get_SP_Item')?['FieldName']`

> すでに ID を持っている場合は、`GetItem` (フィルタ付きの `GetItems` ではない) を使用します。
> トリガー後に再フェッチすると、行の状態ではなく、**現在の** 行の状態が得られます。
> トリガー時にキャプチャされたスナップショット — 別のプロセスが
> フローの開始以降に項目が変更されました。

---

### SharePoint — アイテムの作成```json
"Create_SP_Item": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "PostItem"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList",
      "item/Title": "@variables('myTitle')",
      "item/Status": "Active"
    }
  }
}
```---

### SharePoint — アイテムの更新```json
"Update_SP_Item": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "PatchItem"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "MyList",
      "id": "@item()?['ID']",
      "item/Status": "Processed"
    }
  }
}
```---

### SharePoint — ファイルの更新/挿入 (ドキュメント ライブラリでの作成または上書き)

ファイルが既に存在する場合、SharePoint の `CreateFile` は失敗します。更新/挿入 (作成または上書き) するには
事前の存在チェックを行わずに、**成功と失敗**の両方で `GetFileMetadataByPath` を使用します。
`CreateFile` から — ファイルが存在するために作成が失敗した場合でも、メタデータ呼び出しは引き続き行われます
ID を返します。`UpdateFile` は上書きできます。```json
"Create_File": {
  "type": "OpenApiConnection",
  "inputs": {
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
              "connectionName": "<connectionName>", "operationId": "CreateFile" },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "folderPath": "/My Library/Subfolder",
      "name": "@{variables('filename')}",
      "body": "@outputs('Compose_File_Content')"
    }
  }
},
"Get_File_Metadata_By_Path": {
  "type": "OpenApiConnection",
  "runAfter": { "Create_File": ["Succeeded", "Failed"] },
  "inputs": {
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
              "connectionName": "<connectionName>", "operationId": "GetFileMetadataByPath" },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "path": "/My Library/Subfolder/@{variables('filename')}"
    }
  }
},
"Update_File": {
  "type": "OpenApiConnection",
  "runAfter": { "Get_File_Metadata_By_Path": ["Succeeded", "Skipped"] },
  "inputs": {
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
              "connectionName": "<connectionName>", "operationId": "UpdateFile" },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "id": "@outputs('Get_File_Metadata_By_Path')?['body/{Identifier}']",
      "body": "@outputs('Compose_File_Content')"
    }
  }
}
```> `Create_File` が成功すると、`Get_File_Metadata_By_Path` は `Skipped` と `Update_File` になります
> それでも起動し (`Skipped` を受け入れます)、作成したばかりのファイルを無害に上書きします。
> `Create_File` が失敗した場合 (ファイルが存在する場合)、メタデータ呼び出しによって既存のファイルの ID が取得されます。
> と `Update_File` で上書きされます。どちらの方法でも、最新のコンテンツで終わります。
>
> **ドキュメント ライブラリ システム プロパティ** — ファイル ライブラリの結果を反復するとき (例:
> `ListFolder` または `GetFilesV2` から)、中括弧のプロパティ名を使用してアクセスします
> SharePoint の組み込みファイル メタデータ。これらはリストのフィールド名とは異なります。
>```
> @item()?['{Name}']                  — filename without path (e.g. "report.csv")
> @item()?['{FilenameWithExtension}'] — same as {Name} in most connectors
> @item()?['{Identifier}']            — internal file ID for use in UpdateFile/DeleteFile
> @item()?['{FullPath}']              — full server-relative path
> @item()?['{IsFolder}']             — boolean, true for folder entries
> ```---

### SharePoint — GetItemChanges 列ゲート

SharePoint の「アイテムが変更されました」トリガーが起動しても、どれがどれであるかはわかりません
列が変更されました。 `GetItemChanges` を使用して列ごとの変更フラグを取得し、ゲートします
特定の列の下流ロジック:```json
"Get_Changes": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "GetItemChanges"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "table": "<list-guid>",
      "id": "@triggerBody()?['ID']",
      "since": "@triggerBody()?['Modified']",
      "includeDrafts": false
    }
  }
}
```特定の列のゲート:```json
"expression": {
  "and": [{
    "equals": [
      "@body('Get_Changes')?['Column']?['hasChanged']",
      true
    ]
  }]
}
```> **新しいアイテムの検出:** 最初の修正 (バージョン 1.0) では、
> `GetItemChanges` は以前のバージョンを報告しない可能性があります。チェックする
> `@equals(triggerBody()?['OData__UIVersionString'], '1.0')` を検出する
> 新しく作成された項目を選択し、それらの変更ゲート ロジックをスキップします。

---

### SharePoint — HttpRequest による REST MERGE

標準でサポートされていないクロスリスト更新または高度な操作の場合
アイテムの更新コネクタ (例: 別のサイトのリストの更新)、
`HttpRequest` 操作による SharePoint REST API:```json
"Update_Cross_List_Item": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "HttpRequest"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/target-site",
      "parameters/method": "POST",
      "parameters/uri": "/_api/web/lists(guid'<list-guid>')/items(@{variables('ItemId')})",
      "parameters/headers": {
        "Accept": "application/json;odata=nometadata",
        "Content-Type": "application/json;odata=nometadata",
        "X-HTTP-Method": "MERGE",
        "IF-MATCH": "*"
      },
      "parameters/body": "{ \"Title\": \"@{variables('NewTitle')}\", \"Status\": \"@{variables('NewStatus')}\" }"
    }
  }
}
```> **キーヘッダー:**
> - `X-HTTP-Method: MERGE` — SharePoint に部分更新を実行するように指示します (PATCH セマンティクス)
> - `IF-MATCH: *` — 現在の ETag に関係なく上書きします (競合チェックなし)
>
> `HttpRequest` 操作は既存の SharePoint 接続を再利用します - 余分なものはありません
>認証が必要です。標準の項目更新コネクタができない場合にこれを使用します。
> ターゲット リストに到達します (別のサイト コレクション、または生の REST コントロールが必要です)。

---

### SharePoint — JSON データベースとしてファイル (読み取り + 解析)

SharePoint ドキュメント ライブラリの JSON ファイルをクエリ可能な「データベース」として使用します。
最後に知られた状態のレコード。別のプロセス (Power BI データフローなど) が維持します。
ファイル。フローはそれをダウンロードし、前後の比較のためにフィルタリングします。```json
"Get_File": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connectionName": "<connectionName>",
      "operationId": "GetFileContent"
    },
    "parameters": {
      "dataset": "https://mytenant.sharepoint.com/sites/mysite",
      "id": "%252fShared%2bDocuments%252fdata.json",
      "inferContentType": false
    }
  }
},
"Parse_JSON_File": {
  "type": "Compose",
  "runAfter": { "Get_File": ["Succeeded"] },
  "inputs": "@json(decodeBase64(body('Get_File')?['$content']))"
},
"Find_Record": {
  "type": "Query",
  "runAfter": { "Parse_JSON_File": ["Succeeded"] },
  "inputs": {
    "from": "@outputs('Parse_JSON_File')",
    "where": "@equals(item()?['id'], variables('RecordId'))"
  }
}
```> **デコード チェーン:** `GetFileContent` は、base64 でエンコードされたコンテンツを返します
> @@コード1@@。 `decodeBase64()` を適用してから `json()` を適用して、
> 使用可能な配列。 `Filter Array` は WHERE 句として機能します。
>
> **使用する場合:** フィールドを検出するために軽量の「前」スナップショットが必要な場合
> Webhook ペイロードからの変更 (「後」の状態)。メンテナンスよりも簡単
> 完全な SharePoint リスト ミラー — 最大 10,000 件のレコードに適しています。
>
> **ファイル パスのエンコード:** `id` パラメーターで、SharePoint はパスを URL エンコードします
> 2回。スペースは `%2b` (プラス記号) になり、スラッシュは `%252f` になります。

---

## 展望

### Outlook — 電子メールの送信```json
"Send_Email": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connectionName": "<connectionName>",
      "operationId": "SendEmailV2"
    },
    "parameters": {
      "emailMessage/To": "recipient@contoso.com",
      "emailMessage/Subject": "Automated notification",
      "emailMessage/Body": "<p>@{outputs('Compose_Message')}</p>",
      "emailMessage/IsHtml": true
    }
  }
}
```---

### Outlook — 電子メールの取得 (フォルダーからテンプレートを読み取る)```json
"Get_Email_Template": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connectionName": "<connectionName>",
      "operationId": "GetEmailsV3"
    },
    "parameters": {
      "folderPath": "Id::<outlook-folder-id>",
      "fetchOnlyUnread": false,
      "includeAttachments": false,
      "top": 1,
      "importance": "Any",
      "fetchOnlyWithAttachment": false,
      "subjectFilter": "My Email Template Subject"
    }
  }
}
```件名と本文にアクセスします:```
@first(outputs('Get_Email_Template')?['body/value'])?['subject']
@first(outputs('Get_Email_Template')?['body/value'])?['body']
```> **Outlook-as-CMS パターン**: テンプレート電子メールを専用の Outlook フォルダーに保存します。
> `fetchOnlyUnread: false` を設定すると、最初の使用後にテンプレートが保持されます。
> 技術者以外のユーザーは、その電子メールを編集して件名と本文を更新できます —
> フローの変更は必要ありません。件名と本文を `SendEmailV2` に直接渡します。
>
> フォルダー ID を取得するには、Outlook on the web でフォルダーを右クリック→ [開く] をクリックします。
> 新しいタブ - フォルダーの GUID が URL に含まれています。 `folderPath` の先頭に `Id::` を付けます。

---

## チーム

### チーム — メッセージを投稿する```json
"Post_Teams_Message": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connectionName": "<connectionName>",
      "operationId": "PostMessageToConversation"
    },
    "parameters": {
      "poster": "Flow bot",
      "location": "Channel",
      "body/recipient": {
        "groupId": "<team-id>",
        "channelId": "<channel-id>"
      },
      "body/messageBody": "@outputs('Compose_Message')"
    }
  }
}
```#### バリエーション: グループ チャット (1 対 1 または複数人)

チャネルではなくグループ チャットに投稿するには、`"location": "Group chat"` を使用します
受信者としてのスレッド ID:```json
"Post_To_Group_Chat": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connectionName": "<connectionName>",
      "operationId": "PostMessageToConversation"
    },
    "parameters": {
      "poster": "Flow bot",
      "location": "Group chat",
      "body/recipient": "19:<thread-hash>@thread.v2",
      "body/messageBody": "@outputs('Compose_Message')"
    }
  }
}
```1:1 (「フローボットとチャット」) の場合は、`"location": "Chat with Flow bot"` を使用して設定します
`body/recipient` をユーザーの電子メール アドレスに送信します。

> **アクティブ ユーザー ゲート:** 通知をループで送信するときは、受信者の
> Azure AD アカウントは投稿前に有効化されます - 出発者への配達の失敗を回避します
> スタッフ:
>```json
> "Check_User_Active": {
>   "type": "OpenApiConnection",
>   "inputs": {
>     "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365users",
>               "operationId": "UserProfile_V2" },
>     "parameters": { "id": "@{item()?['Email']}" }
>   }
> }
> ```> 次にゲート: `@equals(body('Check_User_Active')?['accountEnabled'], true)`

---

## 承認

### 分割承認 (作成 → 待機)

標準の「開始して承認を待つ」は、単一のブロック アクションです。
より詳細な制御 (例: Teams への承認リンクの投稿、タイムアウトの追加)
スコープ))、それを 2 つのアクションに分割します: `CreateAnApproval` (fire-and-forget)、次に
`WaitForAnApproval` (Webhook の一時停止)。```json
"Create_Approval": {
  "type": "OpenApiConnection",
  "runAfter": {},
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_approvals",
      "connectionName": "<connectionName>",
      "operationId": "CreateAnApproval"
    },
    "parameters": {
      "approvalType": "CustomResponse/Result",
      "ApprovalCreationInput/title": "Review: @{variables('ItemTitle')}",
      "ApprovalCreationInput/assignedTo": "approver@contoso.com",
      "ApprovalCreationInput/details": "Please review and select an option.",
      "ApprovalCreationInput/responseOptions": ["Approve", "Reject", "Defer"],
      "ApprovalCreationInput/enableNotifications": true,
      "ApprovalCreationInput/enableReassignment": true
    }
  }
},
"Wait_For_Approval": {
  "type": "OpenApiConnectionWebhook",
  "runAfter": { "Create_Approval": ["Succeeded"] },
  "inputs": {
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_approvals",
      "connectionName": "<connectionName>",
      "operationId": "WaitForAnApproval"
    },
    "parameters": {
      "approvalName": "@body('Create_Approval')?['name']"
    }
  }
}
```> **`approvalType` オプション:**
> - `"Approve/Reject - First to respond"` — バイナリ、ファーストレスポンダが勝ち
> - `"Approve/Reject - Everyone must approve"` — すべての担当者が必要です
> - `"CustomResponse/Result"` — 独自の応答ボタンを定義します
>
> `Wait_For_Approval` の後に、結果を読み取ります。
>```
> @body('Wait_For_Approval')?['outcome']          → "Approve", "Reject", or custom
> @body('Wait_For_Approval')?['responses'][0]?['responder']?['displayName']
> @body('Wait_For_Approval')?['responses'][0]?['comments']
> ```>
> 分割パターンを使用すると、create と wait の間にアクションを挿入できます。例:
> Teams への承認リンクの投稿、タイムアウト スコープの開始、またはログ記録
> 追跡リストへの承認待ち。