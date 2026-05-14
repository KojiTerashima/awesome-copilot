---
name: qdrant-clients-sdk
description: "Qdrant provides client SDKs for various programming languages, allowing easy integration with Qdrant deployments."
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---
# Qdrant クライアント SDK

Qdrant には、次の公式にサポートされているクライアント SDK があります。

- Python — [qdrant-client](https://github.com/qdrant/qdrant-client) · インストール: `pip install qdrant-client[fastembed]`
- JavaScript / TypeScript — [qdrant-js](https://github.com/qdrant/qdrant-js) · インストール: `npm install @qdrant/js-client-rest`
- Rust — [rust-client](https://github.com/qdrant/rust-client) · インストール: `cargo add qdrant-client`
- Go — [go-client](https://github.com/qdrant/go-client) · インストール: `go get github.com/qdrant/go-client`
- .NET — [qdrant-dotnet](https://github.com/qdrant/qdrant-dotnet) · インストール: `dotnet add package Qdrant.Client`
- Java — [java-client](https://github.com/qdrant/java-client) · Maven Central で利用可能: https://central.sonatype.com/artifact/io.qdrant/client


## API リファレンス

Qdrant とのすべての対話は、REST API または gRPC API を通じて行うことができます。 Qdrant を初めて使用する場合、またはプロトタイプに取り組んでいる場合は、REST API を使用することをお勧めします。

* REST API - [OpenAPI リファレンス](https://api.qdrant.tech/api-reference) - [GitHub](https://github.com/qdrant/qdrant/blob/master/docs/redoc/master/openapi.json)
* gRPC API - [gRPC protobuf 定義](https://github.com/qdrant/qdrant/tree/master/lib/api/src/grpc/proto)

## コード例

特定のクライアントおよびユースケースのコード例を取得するには、Qdrant クライアント用に厳選されたコード スニペットのライブラリに検索リクエストを送信します。「」バッシュ
カール -X GET "https://snippets.qdrant.tech/search? language=python&query=how+to+upload+points"
「」利用可能な言語: `python`、`typescript`、`rust`、`java`、`go`、`csharp`


応答例:```マークダウン

## スニペット 1

*qdrant-client* (vlatest) — https://search.qdrant.tech/md/documentation/manage-data/points/

Python qdrant_client (PointStruct) を使用して、ID、ペイロード (色など)、および類似性検索用の 3D 類似ベクトルを使用して、複数のベクトル埋め込みポイントを Qdrant コレクションにアップロードします。並列アップロード (Parallel=4) と堅牢なインデックス作成のための再試行ポリシー (max_retries=3) をサポートしています。この操作はべき等です。同じ ID で再アップロードすると、既存のポイントが上書きされます。 ID が指定されていない場合、Qdrant は UUID を自動生成します。

client.upload_points(
    コレクション名="{コレクション名}",
    ポイント=[
        models.PointStruct(
            ID=1、
            ペイロード={
                "色": "赤",
            }、
            ベクトル=[0.9, 0.1, 0.1],
        ）、
        models.PointStruct(
            ID=2、
            ペイロード={
                "色": "緑",
            }、
            ベクトル=[0.1, 0.9, 0.1],
        ）、
    ]、
    平行=4、
    max_retries=3、
）
「」デフォルトの応答形式はマークダウンです。JSON 形式でのスニペット出力が必要な場合は、クエリ文字列に `&format=json` を追加できます。