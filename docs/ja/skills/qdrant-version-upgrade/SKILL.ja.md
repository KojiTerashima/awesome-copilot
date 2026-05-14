---
name: qdrant-version-upgrade
description: "Guidance on how to upgrade your Qdrant version without interrupting the availability of your application and ensuring data integrity."
---
# Qdrant バージョンのアップグレード

Qdrant では、バージョンの互換性について次の保証を行っています。

- Qdrant と SDK のメジャー バージョンとマイナー バージョンは一致することが期待されます。たとえば、Qdrant 1.17.x は SDK 1.17.x と互換性があります。

- Qdrant は、マイナー バージョン間の下位互換性についてテストされています。たとえば、Qdrant 1.17.x は SDK 1.16.x と互換性がある必要があります。 Qdrant サーバー 1.16.x も SDK 1.17.x と互換性があることが期待されていますが、これは 1.16.x で利用可能だった機能のサブセットのみです。

- 次のマイナー バージョンに移行する場合は、まず SDK を次のマイナー バージョンにアップグレードしてから、Qdrant サーバーをアップグレードすることをお勧めします。

- ストレージの互換性は 1 つのマイナー バージョンに対してのみ保証されます。たとえば、Qdrant 1.16.x で保存されたデータは Qdrant 1.17.x と互換性があることが期待されます。複数のマイナー バージョンを移行する必要がある場合は、一度に 1 つのマイナー バージョンずつ、段階的にアップグレードを実行する必要があります。たとえば、1.15.x から 1.17.x に移行するには、まず 1.16.x にアップグレードしてから、1.17.x にアップグレードする必要があります。注: Qdrant Cloud はこのプロセスを自動化するため、中間手順なしで 1.15.x から 1.17.x に直接アップグレードできます。

- レプリケーション係数が 2 以上の Qdrant クラスターは、ローリング アップグレードを実行することで、ダウンタイムなしでアップグレードできます。これは、他のノードがリクエストを処理し続けている間、一度に 1 つのノードをアップグレードできることを意味します。これにより、アップグレード プロセス中にアプリケーションの可用性を維持できます。レプリケーション係数の詳細: [レプリケーション係数](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/?s=replication-factor)

Qdrant Cloud で Qdrant バージョンのアップグレードを管理するには、[qcloud](https://github.com/qdrant/qcloud-cli) CLI ツールを使用できます。