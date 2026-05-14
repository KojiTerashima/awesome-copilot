---
name: qdrant-deployment-options
description: "Guides Qdrant deployment selection. Use when someone asks 'how to deploy Qdrant', 'Docker vs Cloud', 'local mode', 'embedded Qdrant', 'Qdrant EDGE', 'which deployment option', 'self-hosted vs cloud', or 'need lowest latency deployment'. Also use when choosing between deployment types for a new project."
---
# どの Qdrant デプロイメントが必要ですか?

必要なものから始めます: マネージド操作、それともフルコントロール?ネットワーク遅延は許容できるかどうか?量産か試作か？答えは 4 つの選択肢のいずれかに絞り込まれます。


## 開始またはプロトタイピング

プロトタイプの構築、テストの実行、CI/CD パイプライン、または Qdrant の学習時に使用します。

- ローカル モードを使用する (Python のみ): 依存関係なし、メモリ内またはディスク永続化、サーバーは不要 [ローカル モード](https://search.qdrant.tech/md/documentation/quickstart/)
- ローカル モードのデータ形式はサーバーと互換性がありません。本番環境やベンチマークには使用しないでください。
- ローカルの実サーバーの場合は、Docker [クイック スタート](https://search.qdrant.tech/md/documentation/quickstart/?s=download-and-run) を使用します。


## 本番環境への移行 (自己ホスト型)

インフラストラクチャ、データ常駐、またはカスタム構成を完全に制御する必要がある場合に使用します。

- Docker がデフォルトのデプロイメントです。完全な Qdrant オープンソース機能セット、最小限のセットアップ。 [クイックスタート](https://search.qdrant.tech/md/documentation/quickstart/?s=download-and-run)
- 自分の操作: アップグレード、バックアップ、スケーリング、監視
- マルチノード クラスターの場合は分散モードを手動で設定する必要があります [分散展開](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/)
- インフラストラクチャ上で Qdrant クラウド管理が必要な場合は、ハイブリッド クラウドを検討してください [ハイブリッド クラウド](https://search.qdrant.tech/md/documentation/hybrid-cloud/)


## 本番環境への移行 (ゼロオペレーション)

使用する場合: クラスターを自分で操作せずに、ダウンタイムなしの更新、自動バックアップ、および再シャーディングを備えたマネージド インフラストラクチャが必要な場合。

- Qdrant Cloud はアップグレード、スケーリング、バックアップ、モニタリングを処理します [Qdrant Cloud](https://search.qdrant.tech/md/documentation/cloud-quickstart/)
- マルチバージョンのアップグレードを自動的にサポート
- セルフホストでは利用できない機能を提供します: `/sys_metrics`、マネージド リシャーディング、事前設定されたアラート


## レイテンシーを可能な限り低くする必要がある

次の場合に使用します: サーバーへのネットワーク往復が許容されない。エッジ デバイス、インプロセス検索、または遅延が重要なアプリケーション。

- Qdrant EDGE: Qdrant シャードレベル関数へのインプロセス バインディング、ネットワーク オーバーヘッドなし [Qdrant EDGE](https://search.qdrant.tech/md/documentation/edge/edge-quickstart/)
- サーバーと同じデータ形式。シャードスナップショットを介してサーバーと同期できます。
- 単一ノードの機能セットのみ。分散モードはありません。


## してはいけないこと

- 運用またはベンチマークにはローカル モードを使用します (最適化されておらず、互換性のないデータ形式)
- 監視やバックアップ戦略を必要としないセルフホスト (データを失ったり、停止を見逃したりする可能性があります)
- 分散検索が必要な場合は EDGE を選択してください (単一ノードのみ)
- データ常駐要件がない限り、ハイブリッド クラウドを選択します (Qdrant クラウドが動作する場合、Kubernetes は不必要に複雑になります)。