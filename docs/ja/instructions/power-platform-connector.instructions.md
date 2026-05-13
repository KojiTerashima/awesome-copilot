---
title: Power Platform Connectors Schema Development Instructions
description: 'JSON Schema 定義を使う Power Platform Custom Connector 向けの包括的な開発ガイドライン。API 定義 (Swagger 2.0)、API プロパティ、設定構成を Microsoft 拡張とともに扱います。'
applyTo: '**/*.{json,md}'
---

# Power Platform Connectors Schema Development Instructions

## プロジェクト概要
この workspace には、特に `paconn` (Power Apps Connector) tool 向けの、Power Platform Custom Connector 用 JSON Schema 定義が含まれています。これらの schema は、次の対象に対して validation と IntelliSense を提供します。

- **API 定義** (Swagger 2.0 形式)
- **API プロパティ** (connector metadata と構成)
- **設定** (environment と deployment の構成)

## ファイル構造の理解

### 1. apiDefinition.swagger.json
- **目的**: この file には、Power Platform 拡張を含む Swagger 2.0 API 定義が含まれます。
- **主な機能**:
  - info、paths、definitions などを含む標準 Swagger 2.0 property。
  - `x-ms-*` prefix で始まる Microsoft 固有の拡張。
  - `date-no-tz` や `html` のような、Power Platform 向けに特化した custom format type。
  - 実行時の柔軟性を提供する dynamic schema support。
  - OAuth2、API Key、Basic Auth 認証方式をサポートする security definition。

### 2. apiProperties.json
- **目的**: この file は、connector metadata、認証構成、policy 構成を定義します。
- **主な構成要素**:
  - **Connection Parameters**: OAuth、API Key、Gateway 構成を含むさまざまな認証 type をサポートします。
  - **Policy Template Instances**: connector のデータ変換および routing policy を処理します。
  - **Connector Metadata**: publisher 情報、capability、branding 要素を含みます。

### 3. settings.json
- **目的**: この file は、paconn tool 用の environment および deployment 構成設定を提供します。
- **構成オプション**:
  - 特定の Power Platform environment を対象にするための environment GUID。
  - connector asset と構成 file 用の file path mapping。
  - 本番環境およびテスト環境 (PROD / TIP1) 向け API endpoint URL。
  - Power Platform service との互換性を保つ API version 指定。

## 開発ガイドライン

### API 定義 (Swagger) を扱うとき
1. **常に Swagger 2.0 spec に対して検証する** - schema は厳密な Swagger 2.0 準拠を強制します

2. **Operation 向け Microsoft 拡張**:
   - `x-ms-summary`: ユーザーフレンドリーな表示名を提供するために使い、title case 形式を使うようにします。
   - `x-ms-visibility`: parameter の表示可視性を `important`、`advanced`、`internal` の値で制御するために使います。
   - `x-ms-trigger`: operation を trigger としてマークするために使い、値には `batch` または `single` を使います。
   - `x-ms-trigger-hint`: trigger を扱うときにユーザーを案内する helpful な hint text を提供するために使います。
   - `x-ms-trigger-metadata`: kind や mode property を含む trigger 構成設定を定義するために使います。
   - `x-ms-notification`: リアルタイム通知向けの webhook operation を構成するために使います。
   - `x-ms-pageable`: `nextLinkName` property を指定して pagination 機能を有効にするために使います。
   - `x-ms-safe-operation`: POST operation に副作用がない場合、それを安全としてマークするために使います。
   - `x-ms-no-generic-test`: 特定 operation の自動テストを無効にするために使います。
   - `x-ms-operation-context`: テスト目的の operation simulation 設定を構成するために使います。

3. **Parameter 向け Microsoft 拡張**:
   - `x-ms-dynamic-list`: API call から値を取得する dynamic dropdown list を有効にするために使います。
   - `x-ms-dynamic-values`: parameter option を埋める dynamic value source を構成するために使います。
   - `x-ms-dynamic-tree`: ネストした data structure 向けの階層 selector を作るために使います。
   - `x-ms-dynamic-schema`: ユーザー選択に応じて実行時に schema を変更できるようにするために使います。
   - `x-ms-dynamic-properties`: context に応じて変化する dynamic property 構成のために使います。
   - `x-ms-enum-values`: display name 付きの拡張 enum 定義を提供し、より良い user experience を実現するために使います。
   - `x-ms-test-value`: テスト用 sample 値を提供するために使いますが、secret や sensitive data は絶対に含めないでください。
   - `x-ms-trigger-value`: `value-collection` と `value-path` property を持つ trigger parameter 専用の値を指定するために使います。
   - `x-ms-url-encoding`: URL encoding style を `single` または `double` として指定するために使います (`single` が既定)。
   - `x-ms-parameter-location`: API 向け parameter location hint を提供するために使います (AutoRest 拡張 - Power Platform では無視されます)。
   - `x-ms-localizeDefaultValue`: default parameter 値の localization を有効にするために使います。
   - `x-ms-skip-url-encoding`: path parameter に対する URL encoding をスキップするために使います (AutoRest 拡張 - Power Platform では無視されます)。

4. **Schema 向け Microsoft 拡張**:
   - `x-ms-notification-url`: webhook 構成向けの通知 URL として schema property をマークするために使います。
   - `x-ms-media-kind`: content の media type を指定するために使い、サポートされる値は `image` または `audio` です。
   - `x-ms-enum`: 拡張 enum metadata を提供するために使います (AutoRest 拡張 - Power Platform では無視されます)。
   - 上記の parameter 拡張は schema property にも適用され、schema 定義内でも利用できます。

5. **Root-Level 拡張**:
   - `x-ms-capabilities`: file-picker や testConnection 機能など、connector capability を定義するために使います。
   - `x-ms-connector-metadata`: 標準 property 以外の connector metadata を提供するために使います。
   - `x-ms-docs`: connector の documentation 設定や参照先を構成するために使います。
   - `x-ms-deployment-version`: deployment 管理用の version 情報を追跡するために使います。
   - `x-ms-api-annotation`: 拡張機能のための API level annotation を追加するために使います。

6. **Path-Level 拡張**:
   - `x-ms-notification-content`: webhook path item 用の通知 content schema を定義するために使います。

7. **Operation-Level Capability**:
   - `x-ms-capabilities` (operation level): 大きな file 転送向けの `chunkTransfer` のような operation 固有 capability を有効にするために使います。

8. **セキュリティ上の考慮事項**:
   - 適切な認証を確保するため、API には適切な `securityDefinitions` を定義するべきです。
   - **複数の security definition を許可できます** - 最大 2 つの認証方式を定義できます (例: oauth2 + apiKey、basic + apiKey)。
   - **例外**: "None" 認証を使う場合、同じ connector 内に他の security definition を併存させることはできません。
   - 現代的な API には `oauth2`、単純な token 認証には `apiKey`、`basic` auth は internal / legacy system に限って検討するべきです。
   - 各 security definition は必ず 1 つの type でなければなりません (この制約は oneOf validation で強制されます)。

9. **Parameter のベスト プラクティス**:
   - 各 parameter の目的をユーザーが理解できるよう、説明的な `description` field を使うべきです。
   - より良い user experience のため、`x-ms-summary` を実装するべきです (title case が必要)。
   - 適切な validation を確保するため、required parameter は正しくマークしなければなりません。
   - 適切な data handling を可能にするため、適切な `format` 値 (Power Platform 拡張を含む) を使うべきです。
   - より良い user experience と data validation のため、dynamic 拡張を積極的に活用するべきです。

10. **Power Platform Format 拡張**:
   - `date-no-tz`: time-offset 情報を持たない date-time を表します。
   - `html`: 編集時は HTML editor、閲覧時は HTML viewer を client に使わせる format です。
   - 標準 format には次が含まれます: `int32`、`int64`、`float`、`double`、`byte`、`binary`、`date`、`date-time`、`password`、`email`、`uri`、`uuid`。

### API Properties を扱うとき
1. **Connection Parameters**:
   - `string`、`securestring`、`oauthSetting` のような適切な parameter type を選ぶべきです。
   - OAuth 設定は正しい identity provider で構成するべきです。
   - dropdown option が適切な場合は `allowedValues` を使うべきです。
   - 条件付き parameter が必要な場合は parameter dependency を実装するべきです。

2. **Policy Templates**:
   - 異なる API endpoint へ backend routing するには `routerequesttoendpoint` を使うべきです。
   - query parameter の既定値設定には `setqueryparameter` を実装するべきです。
   - pagination scenario では paging を正しく処理するために `updatenextlink` を使うべきです。
   - polling が必要な trigger operation には `pollingtrigger` を適用するべきです。

3. **Branding と Metadata**:
   - すべての connector で必須 property であるため、`iconBrandColor` は必ず指定しなければなりません。
   - connector が action または trigger をサポートするかを示すため、適切な `capabilities` を定義するべきです。
   - connector の ownership を特定するため、意味のある `publisher` と `stackOwner` 値を設定するべきです。

### Settings を扱うとき
1. **Environment 構成**:
   - `environment` には validation pattern に一致する適切な GUID format を使うべきです。
   - 対象 environment に対して正しい `powerAppsUrl` と `flowUrl` を設定するべきです。
   - API version は具体的な要件に合わせて選ぶべきです。

2. **File 参照**:
   - `apiProperties.json` と `apiDefinition.swagger.json` という既定値に合わせて、一貫した file 名を保つべきです。
   - local development environment では相対 path を使うべきです。
   - icon file が存在し、設定内で正しく参照されていることを確認するべきです。

## Schema Validation ルール

### 必須 Property
- **API Definition**: `swagger: "2.0"`、`info` (`title` と `version` を含む)、`paths`
- **API Properties**: `iconBrandColor` を含む `properties`
- **Settings**: 必須 property なし (すべて optional で default あり)

### Pattern Validation
- **Vendor Extensions**: Microsoft 以外の拡張は `^x-(?!ms-)` pattern に一致する必要があります
- **Path Items**: API path は `/` で始まる必要があります
- **Environment GUID**: UUID format pattern `^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$` に一致する必要があります
- **URLs**: endpoint 構成には有効な URI を使う必要があります
- **Host Pattern**: `^[^{}/ :\\]+(?::\\d+)?$` に一致する必要があります (空白、protocol、path は不可)

### 型制約
- **Security Definitions**:
  - `securityDefinitions` object では最大 2 つの security definition を許可
  - 各 security definition はちょうど 1 つの type でなければならない (oneOf validation: `basic`、`apiKey`、`oauth2`)
  - **例外**: "None" 認証は他の security definition と共存できない
- **Parameter Types**: 特定の enum 値 (`string`、`number`、`integer`、`boolean`、`array`、`file`) に限定
- **Policy Templates**: type ごとの parameter 要件
- **Format Values**: Power Platform format を含む拡張セット
- **Visibility Values**: `important`、`advanced`、`internal` のいずれかでなければならない
- **Trigger Types**: `batch` または `single` でなければならない

### 追加の Validation ルール
- **$ref References**: `#/definitions/`、`#/parameters/`、`#/responses/` のいずれかだけを指すべきです
- **Path Parameters**: `required: true` としてマークする必要があります
- **Info Object**: description は title と異なるべきです
- **Contact Object**: email は有効な email format、URL は有効な URI でなければなりません
- **License Object**: name は必須で、URL は指定するなら有効な URI でなければなりません
- **External Docs**: URL は必須で、有効な URI でなければなりません
- **Tags**: array 内で name は一意でなければなりません
- **Schemes**: 有効な HTTP scheme (`http`、`https`、`ws`、`wss`) でなければなりません
- **MIME Types**: `consumes` と `produces` では有効な MIME type format に従う必要があります

## よくあるパターンと例

### API 定義の例

#### Microsoft 拡張を使う基本 Operation
```json
{
  "get": {
    "operationId": "GetItems",
    "summary": "Get items",
    "x-ms-summary": "Get Items",
    "x-ms-visibility": "important",
    "description": "Retrieves a list of items from the API",
    "parameters": [
      {
        "name": "category",
        "in": "query",
        "type": "string",
        "x-ms-summary": "Category",
        "x-ms-visibility": "important",
        "x-ms-dynamic-values": {
          "operationId": "GetCategories",
          "value-path": "id",
          "value-title": "name"
        }
      }
    ],
    "responses": {
      "200": {
        "description": "Success",
        "x-ms-summary": "Success",
        "schema": {
          "type": "object",
          "properties": {
            "items": {
              "type": "array",
              "x-ms-summary": "Items",
              "items": {
                "$ref": "#/definitions/Item"
              }
            }
          }
        }
      }
    }
  }
}
```

#### Trigger Operation の構成
```json
{
  "get": {
    "operationId": "WhenItemCreated",
    "x-ms-summary": "When an Item is Created",
    "x-ms-trigger": "batch",
    "x-ms-trigger-hint": "To see it work now, create an item",
    "x-ms-trigger-metadata": {
      "kind": "query",
      "mode": "polling"
    },
    "x-ms-pageable": {
      "nextLinkName": "@odata.nextLink"
    }
  }
}
```

#### Dynamic Schema の例
```json
{
  "name": "dynamicSchema",
  "in": "body",
  "schema": {
    "x-ms-dynamic-schema": {
      "operationId": "GetSchema",
      "parameters": {
        "table": {
          "parameter": "table"
        }
      },
      "value-path": "schema"
    }
  }
}
```

#### File Picker Capability
```json
{
  "x-ms-capabilities": {
    "file-picker": {
      "open": {
        "operationId": "OneDriveFilePickerOpen",
        "parameters": {
          "dataset": {
            "value-property": "dataset"
          }
        }
      },
      "browse": {
        "operationId": "OneDriveFilePickerBrowse",
        "parameters": {
          "dataset": {
            "value-property": "dataset"
          }
        }
      },
      "value-title": "DisplayName",
      "value-collection": "value",
      "value-folder-property": "IsFolder",
      "value-media-property": "MediaType"
    }
  }
}
```

#### Test Connection Capability (注: Custom Connector では未サポート)
```json
{
  "x-ms-capabilities": {
    "testConnection": {
      "operationId": "TestConnection",
      "parameters": {
        "param1": "literal-value"
      }
    }
  }
}
```

#### シミュレーション用 Operation Context
```json
{
  "x-ms-operation-context": {
    "simulate": {
      "operationId": "SimulateOperation",
      "parameters": {
        "param1": {
          "parameter": "inputParam"
        }
      }
    }
  }
}
```

### 基本的な OAuth 構成
```json
{
  "type": "oauthSetting",
  "oAuthSettings": {
    "identityProvider": "oauth2",
    "clientId": "your-client-id",
    "scopes": ["scope1", "scope2"],
    "redirectMode": "Global"
  }
}
```

#### 複数 Security Definition の例
```json
{
  "securityDefinitions": {
    "oauth2": {
      "type": "oauth2",
      "flow": "accessCode",
      "authorizationUrl": "https://api.example.com/oauth/authorize",
      "tokenUrl": "https://api.example.com/oauth/token",
      "scopes": {
        "read": "Read access",
        "write": "Write access"
      }
    },
    "apiKey": {
      "type": "apiKey",
      "name": "X-API-Key",
      "in": "header"
    }
  }
}
```

**注**: 共存できる security definition は最大 2 つですが、"None" 認証は他の方法と組み合わせられません。

### Dynamic Parameter の設定
```json
{
  "x-ms-dynamic-values": {
    "operationId": "GetItems",
    "value-path": "id",
    "value-title": "name"
  }
}
```

### Routing 用 Policy Template
```json
{
  "templateId": "routerequesttoendpoint",
  "title": "Route to backend",
  "parameters": {
    "x-ms-apimTemplate-operationName": ["GetData"],
    "x-ms-apimTemplateParameter.newPath": "/api/v2/data"
  }
}
```

## ベスト プラクティス

1. **IntelliSense を使う**: これらの schema は豊富な autocomplete と validation 機能を提供し、開発時に役立ちます。
2. **命名規則に従う**: operation と parameter には説明的な名前を付け、code の可読性を高めます。
3. **エラー処理を実装する**: failure scenario を適切に扱えるよう、適切な response schema と error code を定義します。
4. **十分にテストする**: development process の早い段階で問題を見つけるため、deployment 前に schema を検証します。
5. **拡張を文書化する**: team の理解と将来の保守のため、Microsoft 固有拡張には comment を付けます。
6. **Version 管理**: API info には semantic versioning を使い、変更と互換性を追跡します。
7. **Security First**: API endpoint を保護するため、常に適切な認証方式を実装します。

## トラブルシューティング

### よくある Schema 違反
- **必須 property の不足**: `swagger: "2.0"`、`info.title`、`info.version`、`paths`
- **無効な pattern format**:
  - GUID は正確な format `^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$` に一致する必要があります
  - URL は適切な scheme を持つ有効な URI でなければなりません
  - path は `/` で始まる必要があります
  - host には protocol、path、空白を含めてはいけません
- **Vendor 拡張名の誤り**: Microsoft 拡張には `x-ms-*`、その他には `^x-(?!ms-)` を使います
- **Security Definition Type の不一致**: 各 security definition はちょうど 1 つの type でなければなりません
- **無効な enum 値**: `x-ms-visibility`、`x-ms-trigger`、parameter type の許可値を確認してください
- **無効な場所を指す $ref**: `#/definitions/`、`#/parameters/`、`#/responses/` のいずれかを指す必要があります
- **required 指定されていない Path Parameter**: すべての path parameter に `required: true` が必要です
- **誤った文脈での type `file`**: `formData` parameter でのみ許可され、schema 内では許可されません

### API Definition 固有の問題
- **Dynamic schema の競合**: `x-ms-dynamic-schema` を固定 schema property と併用できません
- **Trigger 構成エラー**: `x-ms-trigger-metadata` には `kind` と `mode` の両方が必要です
- **Pagination 設定**: `x-ms-pageable` には `nextLinkName` property が必要です
- **File picker の誤構成**: `open` operation と必須 property の両方を含める必要があります
- **Capability の競合**: 一部の capability は特定 parameter type と競合する場合があります
- **Test value の security**: `x-ms-test-value` には secret や PII を絶対に含めないでください
- **Operation context の設定**: `x-ms-operation-context` には、`operationId` を含む `simulate` object が必要です
- **Notification content schema**: path level の `x-ms-notification-content` では適切な schema 構造を定義する必要があります
- **Media kind の制限**: `x-ms-media-kind` は `image` または `audio` のみをサポートします
- **Trigger value 構成**: `x-ms-trigger-value` には最低 1 つの property (`value-collection` または `value-path`) が必要です

### Validation Tool
- JSON Schema validator を使って schema 定義の準拠性を確認します。
- VS Code 組み込みの schema validation を活用して、開発中に error を検出します。
- deployment 前に `paconn validate --api-def apiDefinition.swagger.json` を使って paconn CLI でテストします。
- Power Platform connector 要件に対して検証し、互換性を確認します。
- 対象 environment での validation と testing には Power Platform Connector portal を使います。
- 実行時 error を防ぐため、operation response が期待 schema と一致することを確認します。

覚えておくべきこと: これらの schema により、Power Platform connector は適切に整形され、Power Platform ecosystem 内で正しく動作します。
