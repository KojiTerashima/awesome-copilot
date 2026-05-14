---
name: breakdown-feature-implementation
description: 'Feature PRD に基づき、Epoch モノレポ構成に従って詳細な機能実装計画を作成するためのプロンプト。'
---

# 機能実装計画プロンプト

## 目標

大規模 SaaS 企業向けに高品質な機能を設計する、業界経験豊富なソフトウェアエンジニアとして振る舞ってください。Feature PRD に基づき、機能の詳細な技術実装計画を作成することに長けている前提です。
提供されたコンテキストを確認し、徹底的で包括的な実装計画を出力してください。
**注意:** 技術的な場面での疑似コードを除き、出力にコードを書かないでください。

## 出力形式

出力は Markdown 形式の完全な実装計画とし、`/docs/ways-of-work/plan/{epic-name}/{feature-name}/implementation-plan.md` に保存してください。

### ファイルシステム

Epoch のモノレポ構成に従った、フロントエンドおよびバックエンド両リポジトリのフォルダ・ファイル構成:

```
apps/
  [app-name]/
services/
  [service-name]/
packages/
  [package-name]/
```

### 実装計画

各機能について:

#### 目標

機能の目標を記述（3〜5 文）

#### 要件

- 詳細な機能要件（箇条書き）
- 実装計画の具体事項

#### 技術的考慮事項

##### システムアーキテクチャ概要

この機能がシステム全体へどのように統合されるかを示す、包括的なシステムアーキテクチャ図を Mermaid で作成してください。図には次を含めてください:

- **Frontend Layer**: ユーザーインターフェースコンポーネント、状態管理、クライアントサイドロジック
- **API Layer**: tRPC エンドポイント、認証ミドルウェア、入力バリデーション、リクエストルーティング
- **Business Logic Layer**: サービスクラス、ビジネスルール、ワークフローオーケストレーション、イベント処理
- **Data Layer**: データベース連携、キャッシュ機構、外部 API 統合
- **Infrastructure Layer**: Docker コンテナ、バックグラウンドサービス、デプロイ構成要素

これらのレイヤーを明確に整理するために subgraphs を使用してください。レイヤー間のデータフローは、リクエスト/レスポンスのパターン、データ変換、イベントフローを示すラベル付き矢印で表現してください。この実装に固有の機能別コンポーネント、サービス、データ構造も含めてください。

- **Technology Stack Selection**: 各レイヤーの選定理由を文書化
```

- **Technology Stack Selection**: 各レイヤーの選定理由を文書化
- **Integration Points**: 明確な境界と通信プロトコルを定義
- **Deployment Architecture**: Docker コンテナ化戦略
- **Scalability Considerations**: 水平・垂直スケーリングのアプローチ

##### データベーススキーマ設計

機能のデータモデルを示すエンティティ関連図を Mermaid で作成してください:

- **Table Specifications**: 型と制約を含む詳細なフィールド定義
- **Indexing Strategy**: パフォーマンス上重要なインデックスとその理由
- **Foreign Key Relationships**: データ整合性と参照制約
- **Database Migration Strategy**: バージョン管理とデプロイ方針

##### API 設計

- 完全仕様を含むエンドポイント
- TypeScript 型を用いたリクエスト/レスポンス形式
- Stack Auth を用いた認証・認可
- エラーハンドリング戦略とステータスコード
- レート制限およびキャッシュ戦略

##### フロントエンドアーキテクチャ

###### コンポーネント階層ドキュメント

コンポーネント構造は、一貫性とアクセシビリティのある基盤として `shadcn/ui` ライブラリを活用します。

**レイアウト構造:**

```
Recipe Library Page
├── Header Section (shadcn: Card)
│   ├── Title (shadcn: Typography `h1`)
│   ├── Add Recipe Button (shadcn: Button with DropdownMenu)
│   │   ├── Manual Entry (DropdownMenuItem)
│   │   ├── Import from URL (DropdownMenuItem)
│   │   └── Import from PDF (DropdownMenuItem)
│   └── Search Input (shadcn: Input with icon)
├── Main Content Area (flex container)
│   ├── Filter Sidebar (aside)
│   │   ├── Filter Title (shadcn: Typography `h4`)
│   │   ├── Category Filters (shadcn: Checkbox group)
│   │   ├── Cuisine Filters (shadcn: Checkbox group)
│   │   └── Difficulty Filters (shadcn: RadioGroup)
│   └── Recipe Grid (main)
│       └── Recipe Card (shadcn: Card)
│           ├── Recipe Image (img)
│           ├── Recipe Title (shadcn: Typography `h3`)
│           ├── Recipe Tags (shadcn: Badge)
│           └── Quick Actions (shadcn: Button - View, Edit)
```

- **State Flow Diagram**: Mermaid を用いたコンポーネント状態管理
- 再利用可能なコンポーネントライブラリ仕様
- Zustand/React Query を用いた状態管理パターン
- TypeScript のインターフェースと型

##### セキュリティとパフォーマンス

- 認証/認可要件
- データバリデーションとサニタイズ
- パフォーマンス最適化戦略
- キャッシュ機構

## コンテキストテンプレート

- **Feature PRD:** [Feature PRD markdown file の内容]

<system_reminder>
<sql_tables>現在テーブルは存在しません。SQL ツールを初めて使用する際に、デフォルトテーブル（todos, todo_deps）が自動作成されます。</sql_tables>
</system_reminder>

