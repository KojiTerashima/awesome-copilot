---
name: mcp-create-adaptive-cards
description: 'mcp-create-adaptive-cards.prompt.md から変換されたスキル'
---

````prompt
---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: 'Microsoft 365 Copilotでの視覚的データ表示のために、MCPベースのAPIプラグインにAdaptive Card応答テンプレートを追加する'
model: 'gpt-4.1'
tags: [mcp, adaptive-cards, m365-copilot, api-plugin, response-templates]
---

# MCPプラグイン用Adaptive Cardsの作成

MCPベースのAPIプラグインにAdaptive Card応答テンプレートを追加し、Microsoft 365 Copilotでのデータの視覚的な提示を強化します。

## Adaptive Cardの種類

### 静的応答テンプレート
APIが常に同じタイプのアイテムを返し、フォーマットがあまり変わらない場合に使用します。

`ai-plugin.json`の`response_semantics.static_template`で定義します：

```json
{
  "functions": [
    {
      "name": "GetBudgets",
      "description": "名前と利用可能資金を含む予算詳細を返します",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.name",
            "subtitle": "$.availableFunds"
          },
          "static_template": {
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
                    "text": "名前: ${if(name, name, 'N/A')}",
                    "wrap": true
                  },
                  {
                    "type": "TextBlock",
                    "text": "利用可能資金: ${if(availableFunds, formatNumber(availableFunds, 2), 'N/A')}",
                    "wrap": true
                  }
                ]
              }
            ]
          }
        }
      }
    }
  ]
}
```

### 動的応答テンプレート
APIが複数のタイプを返し、各アイテムに異なるテンプレートが必要な場合に使用します。

**ai-plugin.jsonの設定例：**
```json
{
  "name": "GetTransactions",
  "description": "動的テンプレートで取引詳細を返します",
  "capabilities": {
    "response_semantics": {
      "data_path": "$.transactions",
      "properties": {
        "template_selector": "$.displayTemplate"
      }
    }
  }
}
```

**テンプレートを埋め込んだAPIレスポンス例：**
```json
{
  "transactions": [
    {
      "budgetName": "Fourth Coffee ロビー改装",
      "amount": -2000,
      "description": "許可申請のための物件調査",
      "expenseCategory": "permits",
      "displayTemplate": "$.templates.debit"
    },
    {
      "budgetName": "Fourth Coffee ロビー改装",
      "amount": 5000,
      "description": "コスト超過を補う追加資金",
      "expenseCategory": null,
      "displayTemplate": "$.templates.credit"
    }
  ],
  "templates": {
    "debit": {
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "size": "medium",
          "weight": "bolder",
          "color": "attention",
          "text": "借方"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "予算",
              "value": "${budgetName}"
            },
            {
              "title": "金額",
              "value": "${formatNumber(amount, 2)}"
            },
            {
              "title": "カテゴリ",
              "value": "${if(expenseCategory, expenseCategory, 'N/A')}"
            },
            {
              "title": "説明",
              "value": "${if(description, description, 'N/A')}"
            }
          ]
        }
      ],
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json"
    },
    "credit": {
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "size": "medium",
          "weight": "bolder",
          "color": "good",
          "text": "貸方"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "予算",
              "value": "${budgetName}"
            },
            {
              "title": "金額",
              "value": "${formatNumber(amount, 2)}"
            },
            {
              "title": "説明",
              "value": "${if(description, description, 'N/A')}"
            }
          ]
        }
      ],
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json"
    }
  }
}
```

### 静的テンプレートと動的テンプレートの組み合わせ
アイテムに`template_selector`がない場合や値が解決できない場合は、静的テンプレートをデフォルトとして使用します。

```json
{
  "capabilities": {
    "response_semantics": {
      "data_path": "$.items",
      "properties": {
        "title": "$.name",
        "template_selector": "$.templateId"
      },
      "static_template": {
        "type": "AdaptiveCard",
        "version": "1.5",
        "body": [
          {
            "type": "TextBlock",
            "text": "デフォルト: ${name}",
            "wrap": true
          }
        ]
      }
    }
  }
}
```

## 応答セマンティクスのプロパティ

### data_path
APIレスポンス内のデータ位置を示すJSONPathクエリ：
```json
"data_path": "$"           // レスポンスのルート
"data_path": "$.results"   // resultsプロパティ内
"data_path": "$.data.items"// ネストされたパス
```

### properties
Copilotの引用に使うレスポンスフィールドのマッピング：
```json
"properties": {
  "title": "$.name",            // 引用タイトル
  "subtitle": "$.description",  // 引用サブタイトル
  "url": "$.link"               // 引用リンク
}
```

### template_selector
各アイテムが使用するテンプレートを示すプロパティ：
```json
"template_selector": "$.displayTemplate"
```

## Adaptive Cardテンプレート言語

### 条件付きレンダリング
```json
{
  "type": "TextBlock",
  "text": "${if(field, field, 'N/A')}"  // フィールドがあれば表示、なければ'N/A'
}
```

### 数値フォーマット
```json
{
  "type": "TextBlock",
  "text": "${formatNumber(amount, 2)}"  // 小数点以下2桁
}
```

### データバインディング
```json
{
  "type": "Container",
  "$data": "${$root}",  // ルートコンテキストに切り替え
  "items": [ ... ]
}
```

### 条件付き表示
```json
{
  "type": "Image",
  "url": "${imageUrl}",
  "$when": "${imageUrl != null}"  // imageUrlが存在する場合のみ表示
}
```

## カード要素

### TextBlock
```json
{
  "type": "TextBlock",
  "text": "テキスト内容",
  "size": "medium",      // small, default, medium, large, extraLarge
  "weight": "bolder",    // lighter, default, bolder
  "color": "attention",  // default, dark, light, accent, good, warning, attention
  "wrap": true
}
```

### FactSet
```json
{
  "type": "FactSet",
  "facts": [
    {
      "title": "ラベル",
      "value": "値"
    }
  ]
}
```

### Image
```json
{
  "type": "Image",
  "url": "https://example.com/image.png",
  "size": "medium",  // auto, stretch, small, medium, large
  "style": "default" // default, person
}
```

### Container
```json
{
  "type": "Container",
  "$data": "${items}",  // 配列を反復処理
  "items": [
    {
      "type": "TextBlock",
      "text": "${name}"
    }
  ]
}
```

### ColumnSet
```json
{
  "type": "ColumnSet",
  "columns": [
    {
      "type": "Column",
      "width": "auto",
      "items": [ ... ]
    },
    {
      "type": "Column",
      "width": "stretch",
      "items": [ ... ]
    }
  ]
}
```

### Actions
```json
{
  "type": "Action.OpenUrl",
  "title": "詳細を見る",
  "url": "https://example.com/item/${id}"
}
```

## レスポンシブデザインのベストプラクティス

### シングルカラムレイアウト
- 狭いビューポートでは単一カラムを使用
- 可能な限りマルチカラムレイアウトを避ける
- カードが最小ビューポート幅で機能することを確認

### 柔軟な幅設定
- 要素に固定幅を割り当てない
- 幅プロパティには「auto」または「stretch」を使用
- ビューポートに合わせて要素がリサイズ可能にする
- アイコンやアバターのみ固定幅を許容

### テキストと画像
- テキストと画像を同じ行に配置しない
- 例外：小さなアイコンやアバター
- テキストコンテンツには「wrap": true」を使用
- さまざまなビューポート幅でテスト

### ハブ間でのテスト
以下でカードを検証：
- Teams（デスクトップおよびモバイル）
- Word
- PowerPoint
- さまざまなビューポート幅（UIの収縮・拡張）

## 完全な例

**ai-plugin.json:**
```json
{
  "functions": [
    {
      "name": "SearchProjects",
      "description": "ステータスと詳細を含むプロジェクトを検索します",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.projects",
          "properties": {
            "title": "$.name",
            "subtitle": "$.status",
            "url": "$.projectUrl"
          },
          "static_template": {
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
                    "size": "medium",
                    "weight": "bolder",
                    "text": "${if(name, name, '無題のプロジェクト')}",
                    "wrap": true
                  },
                  {
                    "type": "FactSet",
                    "facts": [
                      {
                        "title": "ステータス",
                        "value": "${status}"
                      },
                      {
                        "title": "担当者",
                        "value": "${if(owner, owner, '未割当')}"
                      },
                      {
                        "title": "期限",
                        "value": "${if(dueDate, dueDate, '未設定')}"
                      },
                      {
                        "title": "予算",
                        "value": "${if(budget, formatNumber(budget, 2), 'N/A')}"
                      }
                    ]
                  },
                  {
                    "type": "TextBlock",
                    "text": "${if(description, description, '説明なし')}",
                    "wrap": true,
                    "separator": true
                  }
                ]
              }
            ],
            "actions": [
              {
                "type": "Action.OpenUrl",
                "title": "プロジェクトを見る",
                "url": "${projectUrl}"
              }
            ]
          }
        }
      }
    }
  ]
}
```

## ワークフロー

ユーザーに質問：
1. APIはどのタイプのデータを返しますか？
2. すべてのアイテムは同じタイプ（静的）ですか、それとも異なるタイプ（動的）ですか？
3. カードに表示すべきフィールドは何ですか？
4. アクション（例：「詳細を見る」）は必要ですか？
5. 複数の状態やカテゴリがあり、異なるテンプレートが必要ですか？

その後、生成するもの：
- 適切なresponse_semantics設定
- 静的テンプレート、動的テンプレート、または両方
- 条件付きレンダリングを含む適切なデータバインディング
- レスポンシブなシングルカラムレイアウト
- 検証用のテストシナリオ

## リソース

- [Adaptive Card Designer](https://adaptivecards.microsoft.com/designer) - ビジュアルデザインツール
- [Adaptive Card Schema](https://adaptivecards.io/schemas/adaptive-card.json) - 完全なスキーマリファレンス
- [Template Language](https://learn.microsoft.com/en-us/adaptive-cards/templating/language) - バインディング構文ガイド
- [JSONPath](https://www.rfc-editor.org/rfc/rfc9535) - パスクエリ構文

## よく使われるパターン

### 画像付きリスト
```json
{
  "type": "Container",
  "$data": "${items}",
  "items": [
    {
      "type": "ColumnSet",
      "columns": [
        {
          "type": "Column",
          "width": "auto",
          "items": [
            {
              "type": "Image",
              "url": "${thumbnailUrl}",
              "size": "small",
              "$when": "${thumbnailUrl != null}"
            }
          ]
        },
        {
          "type": "Column",
          "width": "stretch",
          "items": [
            {
              "type": "TextBlock",
              "text": "${title}",
              "weight": "bolder",
              "wrap": true
            }
          ]
        }
      ]
    }
  ]
}
```

### ステータスインジケーター
```json
{
  "type": "TextBlock",
  "text": "${status}",
  "color": "${if(status == 'Completed', 'good', if(status == 'In Progress', 'attention', 'default'))}"
}
```

### 通貨フォーマット
```json
{
  "type": "TextBlock",
  "text": "$${formatNumber(amount, 2)}"
}
```

````
