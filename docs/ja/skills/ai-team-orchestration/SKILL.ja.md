---
name: ai-team-orchestration
description: 'マルチエージェント AI 開発チームをブートストラップして実行します。 AI エージェントによる新しいソフトウェア プロジェクトの開始、並行開発/QA チームのセットアップ、スプリント計画の作成、個別のエージェントの声によるブレインストーミング プロンプトの作成、プロジェクト ワークフローの回復、またはスプリントの計画の場合に使用します。'
---

# AI チーム オーケストレーション

## いつ使用するか
- 計画、開発、テスト、展開が必要な新しいプロジェクトの開始
- 並列 AI エージェント チームのセットアップ (開発、QA、DevOps)
- 実際の議論を生み出すブレインストーミング プロンプトを作成する (一般的な出力ではない)
- クロスチャット コンテキスト サバイバルを使用したスプリント プランの作成
- スプリント中のコンテキスト オーバーフローからの回復

## チームの役割

|エージェント |名前 |役割 |フォーカス |
|------|------|------|------|
|プロデューサー | **レミ** |スプリントの計画、調整、PR の結合 |範囲制御、ハンドオフ、問題のトリアージ |
|プロダクトデザイナー | **キラ** | UX、仕組み、ユーザーエクスペリエンス |楽しい要素、ユーザー フロー、機能設計 |
|ビジュアル/アートディレクター | **マイロ** | CSS、アニメーション、ビジュアルアイデンティティ |デザインシステム、磨き、アクセシビリティ |
|フロントエンドエンジニア | **ノヴァ** | UI フレームワーク、状態管理、コンポーネント | React/Vue/Svelte、クライアント側ロジック |
|バックエンドエンジニア | **セージ** | API、データベース、認証、セキュリティ |サーバー側ロジック、インフラストラクチャ |
| DevOpsエンジニア | **ダッシュ** | CI/CD、クラウド展開、パイプライン | GitHub アクション、Azure/AWS/GCP |
| QAエンジニア | **アイビー** | E2E テスト、自動化、プレイテスト | Playwright/Cypress、バグ報告、サインオフ |

プロジェクトの名前と役割をカスタマイズします。すべてのプロジェクトにすべての役割が必要なわけではありません。

## チャットのアーキテクチャ

人間 (CEO) は、並列チャット間のメッセージ バスです。
```
┌────────────────────────────────────────┐
│  @ai-team-producer — Plans, merges     │
│  NEVER writes code                     │
└────────────────┬───────────────────────┘
                 │ Human carries messages
      ┌──────────┼──────────┐
      ▼          ▼          ▼
┌──────────┐ ┌────────┐ ┌────────┐
│@ai-team  │ │@ai-team│ │DevOps  │
│-dev      │ │-qa     │ │(on     │
│          │ │        │ │demand) │
│ Nova     │ │ Ivy    │ │        │
│ Sage     │ │        │ │        │
│ Milo     │ │        │ │        │
│          │ │feature/│ │feature/│
│ feature/ │ │qa-N    │ │devops-N│
│ sprint-N │ └────────┘ └────────┘
└──────────┘
```

各チームは、独自のクローンを使用して **別の VS Code ウィンドウ**で作業します。```bash
git clone <repo> project-dev    # Dev team
git clone <repo> project-qa     # QA
git clone <repo> project-devops # DevOps (only when needed)
```

## プロジェクトブートストラップ

### 1.PROJECT_BRIEF.mdを作成する

すべてのチャットにわたる唯一の真実の情報源。 [プロジェクト概要テンプレート](./references/project-brief-template.md) を参照してください。

**必須セクション (省略しないでください):**
1. プロジェクト概要
2. コンセプト・商品説明
3. 技術スタック
4. アーキテクチャ (ASCII 図)
5. キーファイルマップ
6. チームの役割
7. スプリントステータス (スプリントごとに更新)
8. 現在の状態 (スプリントごとに書き換えられる)
9. セキュリティルール
10. ローカルで実行する方法
11. 導入方法
12. **クロスチャットハンドオフプロトコル** — チャット間でコンテキストがどのように存続するか
13. **バグと修正の追跡** — 唯一の信頼できる情報源としての GitHub の問題
14. **マルチリポジトリのセットアップ** — 個別のクローン、ブランチ戦略、マージ ルール

### 2. ブレインストーミングを実行する

[ブレインストーミングの形式](./references/brainstorm-format.md) を参照してください。キー: 明確な個性と視点を持って各エージェントに明示的に名前を付けます。集団思考を防ぐには、少なくとも 2 つの本物の意見の相違が必要です。

### 3. スプリント計画を作成する

[スプリント計画テンプレート](./references/sprint-plan-template.md) を参照してください。すべてのスプリントで以下が得られます:
- `docs/sprint-N/plan.md` — 優先順位付けされたタスク、成功基準
- `docs/sprint-N/progress.md` — ライブ トラッカー、回復を可能にする
- `docs/sprint-N/done.md` — スプリント終了時に書かれたハンドオフドキュメント

### 4. スプリントの実行
```
Read PROJECT_BRIEF.md, then read docs/sprint-N/plan.md. Execute Sprint N.

First: git pull origin main && git checkout -b feature/sprint-N

Close GitHub Issues in commits: "fix: description (Fixes #NN)"
Update docs/sprint-N/progress.md after each phase.
When done, push and create PR: git push origin feature/sprint-N
Follow Sections 12-14 of PROJECT_BRIEF.md.
```

### 5. QA 承認

開発者がマージした後、QA は完全なプレイスルーを実行します。```
Read PROJECT_BRIEF.md. You are Ivy (QA).
Sprint N is merged to main. Do full playthrough.
File bugs as GitHub Issues. Write docs/qa/sprint-N-signoff.md.
```

## コンテキストの回復

チャットが長くなった場合 (メッセージが 100 件を超える)、状態を保存して新たに開始します。

**閉店前に:**
1. `docs/sprint-N/progress.md` を現在のステータスで更新します
2. `PROJECT_BRIEF.md` セクション 7+8 を更新
3. `docs/sprint-N/done.md`と書いてください

**コールド スタート プロンプト:**```
Read PROJECT_BRIEF.md and docs/sprint-N/progress.md.
Continue from where it left off.
```

## アンチパターン

完全なリストについては、[アンチパターン リファレンス](./references/anti-patterns.md) を参照してください。トップ5:

|しないでください |代わりに行う |
|------|-----------|
|機能ブランチをリベースする |マージ (リベースによりコミットが失われる) |
|プロデューサーがコードを書く |プロデューサーのみが計画、マージ、ファイルの問題を作成する |
|バッチ「すべてを修正」コミット |問題のリファレンス付きの修正ごとに 1 つのコミット |
|漠然としたブレインストーミングのプロンプト |それぞれのエージェントに明確な観点から名前を付けます |
|チャット内でのみバグを保持する |ファイル GitHub の問題 (チャット コンテキストが停止する) |

## より良い結果のためのヒント

- **プロンプトで「ゆっくり時間をかけて、正しくやってください」** と急ぐよりも良い出力が生成されます
- **マージ前のテスト** — プレイテスト、ファイルの問題、開発修正を行ってからマージします。
- **主要なスプリントの前にチームコンシリ​​ウムを実行** - 各エージェントがそれぞれの観点から計画をレビューします
- Error 500 (Server Error)!!1500.That’s an error.There was an error. Please try again later.That’s all we know.