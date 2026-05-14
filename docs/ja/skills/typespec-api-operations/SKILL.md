---
name: typespec-api-operations
description: 'Add GET, POST, PATCH, and DELETE operations to a TypeSpec API plugin with proper routing, parameters, and adaptive cards'
---
# TypeSpec API オペレーションを追加

Microsoft 365 Copilot の既存の TypeSpec API プラグインに RESTful 操作を追加します。

## GET オペレーションの追加

### シンプルな GET - すべてのアイテムをリストする```typescript
/**
 * List all items.
 */
@route("/items")
@get op listItems(): Item[];
```### クエリ パラメータを使用した GET - 結果のフィルタリング```typescript
/**
 * List items filtered by criteria.
 * @param userId Optional user ID to filter items
 */
@route("/items")
@get op listItems(@query userId?: integer): Item[];
```### パスパラメータを使用した GET - 単一アイテムの取得```typescript
/**
 * Get a specific item by ID.
 * @param id The ID of the item to retrieve
 */
@route("/items/{id}")
@get op getItem(@path id: integer): Item;
```### アダプティブカードでGET```typescript
/**
 * List items with adaptive card visualization.
 */
@route("/items")
@card(#{
  dataPath: "$",
  title: "$.title",
  file: "item-card.json"
})
@get op listItems(): Item[];
```**アダプティブ カードを作成します** (`appPackage/item-card.json`):```json
{
  "type": "AdaptiveCard",
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.5",
  "body": [
    {
      "type": "Container",
      "$data": "${$root}",
      "items": [
        {
          "type": "TextBlock",
          "text": "**${if(title, title, 'N/A')}**",
          "wrap": true
        },
        {
          "type": "TextBlock",
          "text": "${if(description, description, 'N/A')}",
          "wrap": true
        }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "View Details",
      "url": "https://example.com/items/${id}"
    }
  ]
}
```## POST オペレーションの追加

### シンプルな POST - アイテムの作成```typescript
/**
 * Create a new item.
 * @param item The item to create
 */
@route("/items")
@post op createItem(@body item: CreateItemRequest): Item;

model CreateItemRequest {
  title: string;
  description?: string;
  userId: integer;
}
```### 確認付き POST```typescript
/**
 * Create a new item with confirmation.
 */
@route("/items")
@post
@capabilities(#{
  confirmation: #{
    type: "AdaptiveCard",
    title: "Create Item",
    body: """
    Are you sure you want to create this item?
      * **Title**: {{ function.parameters.item.title }}
      * **User ID**: {{ function.parameters.item.userId }}
    """
  }
})
op createItem(@body item: CreateItemRequest): Item;
```## PATCH オペレーションの追加

### シンプルなパッチ - アイテムの更新```typescript
/**
 * Update an existing item.
 * @param id The ID of the item to update
 * @param item The updated item data
 */
@route("/items/{id}")
@patch op updateItem(
  @path id: integer,
  @body item: UpdateItemRequest
): Item;

model UpdateItemRequest {
  title?: string;
  description?: string;
  status?: "active" | "completed" | "archived";
}
```### 確認付きパッチ```typescript
/**
 * Update an item with confirmation.
 */
@route("/items/{id}")
@patch
@capabilities(#{
  confirmation: #{
    type: "AdaptiveCard",
    title: "Update Item",
    body: """
    Updating item #{{ function.parameters.id }}:
      * **Title**: {{ function.parameters.item.title }}
      * **Status**: {{ function.parameters.item.status }}
    """
  }
})
op updateItem(
  @path id: integer,
  @body item: UpdateItemRequest
): Item;
```## DELETE オペレーションの追加

### 単純な削除```typescript
/**
 * Delete an item.
 * @param id The ID of the item to delete
 */
@route("/items/{id}")
@delete op deleteItem(@path id: integer): void;
```### 確認付き削除```typescript
/**
 * Delete an item with confirmation.
 */
@route("/items/{id}")
@delete
@capabilities(#{
  confirmation: #{
    type: "AdaptiveCard",
    title: "Delete Item",
    body: """
    ⚠️ Are you sure you want to delete item #{{ function.parameters.id }}?
    This action cannot be undone.
    """
  }
})
op deleteItem(@path id: integer): void;
```## 完全な CRUD の例

### サービスとモデルを定義する```typescript
@service
@server("https://api.example.com")
@actions(#{
  nameForHuman: "Items API",
  descriptionForHuman: "Manage items",
  descriptionForModel: "Read, create, update, and delete items"
})
namespace ItemsAPI {
  
  // Models
  model Item {
    @visibility(Lifecycle.Read)
    id: integer;
    
    userId: integer;
    title: string;
    description?: string;
    status: "active" | "completed" | "archived";
    
    @format("date-time")
    createdAt: utcDateTime;
    
    @format("date-time")
    updatedAt?: utcDateTime;
  }

  model CreateItemRequest {
    userId: integer;
    title: string;
    description?: string;
  }

  model UpdateItemRequest {
    title?: string;
    description?: string;
    status?: "active" | "completed" | "archived";
  }

  // Operations
  @route("/items")
  @card(#{ dataPath: "$", title: "$.title", file: "item-card.json" })
  @get op listItems(@query userId?: integer): Item[];

  @route("/items/{id}")
  @card(#{ dataPath: "$", title: "$.title", file: "item-card.json" })
  @get op getItem(@path id: integer): Item;

  @route("/items")
  @post
  @capabilities(#{
    confirmation: #{
      type: "AdaptiveCard",
      title: "Create Item",
      body: "Creating: **{{ function.parameters.item.title }}**"
    }
  })
  op createItem(@body item: CreateItemRequest): Item;

  @route("/items/{id}")
  @patch
  @capabilities(#{
    confirmation: #{
      type: "AdaptiveCard",
      title: "Update Item",
      body: "Updating item #{{ function.parameters.id }}"
    }
  })
  op updateItem(@path id: integer, @body item: UpdateItemRequest): Item;

  @route("/items/{id}")
  @delete
  @capabilities(#{
    confirmation: #{
      type: "AdaptiveCard",
      title: "Delete Item",
      body: "⚠️ Delete item #{{ function.parameters.id }}?"
    }
  })
  op deleteItem(@path id: integer): void;
}
```## 高度な機能

### 複数のクエリパラメータ```typescript
@route("/items")
@get op listItems(
  @query userId?: integer,
  @query status?: "active" | "completed" | "archived",
  @query limit?: integer,
  @query offset?: integer
): ItemList;

model ItemList {
  items: Item[];
  total: integer;
  hasMore: boolean;
}
```### ヘッダーパラメータ```typescript
@route("/items")
@get op listItems(
  @header("X-API-Version") apiVersion?: string,
  @query userId?: integer
): Item[];
```### カスタム応答モデル```typescript
@route("/items/{id}")
@delete op deleteItem(@path id: integer): DeleteResponse;

model DeleteResponse {
  success: boolean;
  message: string;
  deletedId: integer;
}
```### エラー応答```typescript
model ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: string[];
  };
}

@route("/items/{id}")
@get op getItem(@path id: integer): Item | ErrorResponse;
```## プロンプトのテスト

操作を追加した後、次のプロンプトを使用してテストします。

**GET オペレーション:**
- 「すべての項目をリストして表に表示」
- 「ユーザーID 1のアイテムを表示」
- 「アイテム 42 の詳細を取得する」

**POST 操作:**
- 「ユーザー 1 用に、タイトルが「マイ タスク」の新しいアイテムを作成します。」
- 「項目を追加します: タイトル「新機能」、説明「ログインの追加」」

**パッチ操作:**
- 「項目 10 を「更新されたタイトル」というタイトルで更新します」
- 「項目 5 のステータスを完了に変更します」

**削除操作:**
- 「項目99を削除」
- 「ID 15 のアイテムを削除」

## ベストプラクティス

### パラメータの命名
- わかりやすいパラメータ名を使用します: `uid` ではなく `userId`
- オペレーション全体で一貫性を保つ
- フィルターにはオプションのパラメーター (`?`) を使用します

### ドキュメント
- すべての操作に JSDoc コメントを追加します
- 各パラメータの機能の説明
- 予想される応答を文書化する

### モデル
- `id` のような読み取り専用フィールドには `@visibility(Lifecycle.Read)` を使用します
- 日付フィールドには `@format("date-time")` を使用します
- 列挙型には共用体型を使用します: `"active" | "completed"`
- `?` を使用してオプションのフィールドを明示的にする

### 確認
- 破壊的な操作 (DELETE、PATCH) には必ず確認を追加します。
- 確認本文に重要な詳細を表示します
- 取り消しできないアクションには警告絵文字 (⚠️) を使用してください

### アダプティブ カード
- カードをシンプルかつ集中的に保つ
- `${if(..., ..., 'N/A')}` で条件付きレンダリングを使用する
- 一般的な次のステップのためのアクション ボタンを含めます
- 実際の API 応答を使用してデータ バインディングをテストする

### ルーティング
- RESTful 規約を使用します。
  - `GET /items` - リスト
  - `GET /items/{id}` - 1 つ入手してください
  - `POST /items` - 作成
  - `PATCH /items/{id}` - 更新
  - `DELETE /items/{id}` - 削除
- 関連する操作を同じ名前空間にグループ化する
- 階層リソースにネストされたルートを使用する

## よくある問題

### 問題: Copilot にパラメータが表示されない
**解決策**: パラメーターが `@query`、`@path`、または `@body` で適切に修飾されていることを確認してください。

### 問題: アダプティブ カードがレンダリングされない
**解決策**: `@card` デコレータでファイル パスを確認し、JSON 構文を確認してください。

### 問題: 確認が表示されない
**解決策**: `@capabilities` デコレータが確認オブジェクトで適切にフォーマットされていることを確認してください

### 問題: モデルのプロパティが応答に表示されない
**解決策**: プロパティに `@visibility(Lifecycle.Read)` が必要かどうかを確認するか、書き込み可能である必要がある場合は削除してください。