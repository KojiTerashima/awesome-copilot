---
name: qdrant-monitoring
description: "Guides Qdrant monitoring and observability setup. Use when someone asks 'how to monitor Qdrant', 'what metrics to track', 'is Qdrant healthy', 'optimizer stuck', 'why is memory growing', 'requests are slow', or needs to set up Prometheus, Grafana, or health checks. Also use when debugging production issues that require metric analysis."
allowed-tools:
  - Read
  - Grep
  - Glob
---
# Qdrant モニタリング

Qdrant モニタリングを使用すると、デプロイメントのパフォーマンスと健全性を追跡し、障害が発生する前に問題を特定できます。まず、監視を設定する必要があるか、進行中の問題を診断する必要があるかを判断します。

- 利用可能なメトリクスを理解する [モニタリング ドキュメント](https://search.qdrant.tech/md/documentation/operations/monitoring/)


## モニタリングのセットアップ

Prometheus スクレイピング、正常性プローブ、ハイブリッド クラウドの詳細、アラート、ログの一元化。 [モニタリング設定](setup/SKILL.md)


## メトリクスを使用したデバッグ

オプティマイザがスタックし、メモリが増加し、リクエストが遅くなります。メトリクスを使用してアクティブな運用上の問題を診断します。 [メトリクスを使用したデバッグ](debugging/SKILL.md)