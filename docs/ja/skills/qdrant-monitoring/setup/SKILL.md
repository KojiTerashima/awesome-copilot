---
name: qdrant-monitoring-setup
description: "Guides Qdrant monitoring setup including Prometheus scraping, health probes, Hybrid Cloud metrics, alerting, and log centralization. Use when someone asks 'how to set up monitoring', 'Prometheus config', 'Grafana dashboard', 'health check endpoints', 'how to scrape Hybrid Cloud', 'what alerts to set', 'how to centralize logs', or 'audit logging'."
---
# Qdrant モニタリングを設定する方法

最初に Prometheus スクレイピングを動作させ、次に正常性プローブを動作させ、次にアラートを発生させます。本番環境に移行する前に、監視のセットアップをスキップしないでください。


## プロメテウスのメトリクス

初めてメトリック収集をセットアップする場合、または新しいデプロイメントを追加する場合に使用します。

- `/metrics` エンドポイントのノード メトリック [モニタリング ドキュメント](https://search.qdrant.tech/md/documentation/operations/monitoring/)
- `/sys_metrics` のクラスター メトリック (Qdrant クラウドのみ)
- `service.metrics_prefix` config または `QDRANT__SERVICE__METRICS_PREFIX` 環境変数によるプレフィックスのカスタマイズ
- Prometheus + Grafana を使用したセルフホスト型セットアップの例 [prometheus-monitoring リポジトリ](https://github.com/qdrant/prometheus-monitoring)


## ハイブリッド クラウド スクレイピング

次の場合に使用します: Qdrant Hybrid Cloud を実行しており、クラスターレベルの可視性が必要です。

Qdrant ノードを単にスクレイピングしないでください。ハイブリッド クラウドでは、Kubernetes データ プレーンを管理します。クラスターの完全な可視性とオペレーターの状態を得るには、クラスター エクスポーターとオペレーター ポッドをスクレイピングする必要もあります。

- ハイブリッド クラウド Prometheus セットアップ チュートリアル [ハイブリッド クラウド Prometheus](https://search.qdrant.tech/md/documentation/tutorials-and-examples/hybrid-cloud-prometheus/)
- 公式 Grafana ダッシュボード [Grafana ダッシュボード リポジトリ](https://github.com/qdrant/qdrant-cloud-grafana-dashboard)


## Liveness プローブと Readiness プローブ

次の場合に使用します: Kubernetes ヘルスチェックを構成する場合。

- 基本ステータス、稼働状況、準備状況には `/healthz`、`/livez`、`/readyz` を使用します [Kubernetes 健全性エンドポイント](https://search.qdrant.tech/md/documentation/operations/monitoring/?s=kubernetes-health-endpoints)


## アラート中

次の場合に使用します: 運用環境またはハイブリッド クラウド展開のアラートを設定する場合。

- ハイブリッド クラウドは、すぐに使用できる最大 11 個の事前構成された Prometheus アラートを提供します [クラウド クラスターのモニタリング](https://search.qdrant.tech/md/documentation/cloud/cluster-monitoring/)
- AlertmanagerConfig を使用して、ラベルに基づいて Slack、PagerDuty、またはその他のターゲットにアラートをルーティングします
- 少なくとも、オプティマイザ エラー、ノードの準備ができていない、レプリケーション係数が目標を下回っている、ディスク使用率 > 80% について警告します。


## ログの一元化と監査ログ

次の場合に使用します: エンタープライズ コンプライアンスで一元化されたログまたは監査証跡が必要です。

- 構造化分析の JSON ログ形式を有効にする: config [Configuration](https://search.qdrant.tech/md/documentation/operations/configuration/) で `logger.format` を `json` に設定します。
- ログ集約には FluentD/OpenSearch を使用します
- 監査ログ (v1.17 以降) は、stdout ではなく、ローカル ファイルシステム (`/qdrant/storage/audit/`) に書き込みます。永続ボリュームをマウントし、サイドカー コンテナをデプロイしてこれらのファイルを stdout に追跡し、DaemonSet がそれらを取得できるようにします。 [監査ログ](https://search.qdrant.tech/md/documentation/operations/security/?s=audit-logging)


## してはいけないこと

- セルフホストで `/sys_metrics` をスクレイピング (Qdrant クラウドでのみ利用可能)
- ハイブリッド クラウド内の Qdrant ノードのみをスクレイピング (クラスター エクスポーターとオペレーターのメトリクスを欠落)
- 本番環境に入る前に監視設定をスキップします (後悔することになります)
- ページ キャッシュ メモリの使用状況に関するアラート (利用可能な RAM がいっぱいになるため、通常の OS の動作が想定されます)