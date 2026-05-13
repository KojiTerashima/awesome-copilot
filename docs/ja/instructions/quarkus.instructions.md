---
applyTo: '*'
description: 'Quarkus の開発標準と instruction'
---

- Java 17 以降を使う高品質な Quarkus アプリケーションのための instruction。

## プロジェクト コンテキスト

- 最新の Quarkus version: 3.x
- Java version: 17 以降
- build 管理には Maven または Gradle を使う。
- clean architecture、保守性、パフォーマンスを重視する。

## 開発標準

  - 各 class、method、複雑なロジックには、明確で簡潔なコメントを書く。
  - public API と method には、利用者にとって明確になるよう Javadoc を使う。
  - Java の規約に従い、project 全体で一貫した coding style を保つ。
  - 最適なパフォーマンスと保守性のため、Quarkus の coding standard と best practice に従う。
  - package 構成が明確になるよう、Jarkarta EE と MicroProfile の規約に従う。
  - records や sealed classes など、適切な場面では Java 17 以降の機能を使う。


## 命名規則
  - class 名には PascalCase を使う (例: `ProductService`、`ProductResource`)。
  - method 名と変数名には camelCase を使う (例: `findProductById`、`isProductAvailable`)。
  - 定数には ALL_CAPS を使う (例: `DEFAULT_PAGE_SIZE`)。

##  Quarkus
  - 開発サイクルを高速化するため Quarkus Dev Mode を活用する。
  - Quarkus extension と best practice を使って build 時最適化を実装する。
  - 最適なパフォーマンスのため、GraalVM を用いた native build を構成する (例: quarkus-maven-plugin を使う)。
  - 一貫した logging のため、Quarkus の logging 機能 (JBoss、SL4J または JUL) を使う。

### Quarkus 固有パターン
- singleton bean には `@Singleton` ではなく `@ApplicationScoped` を使う
- dependency injection には `@Inject` を使う
- 従来の JPA repository より Panache repository を優先する
- データを変更する service method には `@Transactional` を使う
- REST endpoint path には説明的な `@Path` を適用する
- REST resource には `@Consumes(MediaType.APPLICATION_JSON)` と `@Produces(MediaType.APPLICATION_JSON)` を使う

### REST Resources
- 常に JAX-RS annotation (`@Path`、`@GET`、`@POST` など) を使う
- 適切な HTTP status code (200、201、400、404、500) を返す
- 複雑な response には `Response` class を使う
- try-catch block による適切なエラー処理を含める
- Bean Validation annotation で入力パラメーターを検証する
- public endpoint には rate limiting を実装する

### Data Access
- 従来の JPA より Panache entity (`PanacheEntity` を継承) を優先する
- 複雑な query には Panache repository (`PanacheRepository<T>`) を使う
- データ変更には常に `@Transactional` を使う
- 複雑な database 操作には named query を使う
- list endpoint には適切な pagination を実装する


### Configuration
- 単純な構成には `application.properties` または `application.yaml` を使う
- 型安全な configuration class には `@ConfigProperty` を使う
- 機密データには environment variable を優先する
- 環境ごと (dev、test、prod) に profile を使う


### Testing
- integration test には `@QuarkusTest` を使う
- unit test には JUnit 5 を使う
- native build test には `@QuarkusIntegrationTest` を使う
- 外部依存は `@QuarkusTestResource` を使って mock する
- REST endpoint test には RestAssured を使う (`@QuarkusTestResource`)
- database を変更する test には `@Transactional` を使う
- database integration test には test-containers を使う

### 使ってはいけないパターン
- test で field injection を使わない (constructor injection を使う)
- configuration value をハードコードしない
- 例外を無視しない


## 開発ワークフロー

### 新機能を作成するとき:
1. 適切な validation を備えた entity を作成する
2. custom query を持つ repository を作成する
3. business logic を持つ service を作成する
4. 適切な endpoint を持つ REST resource を作成する
5. 包括的な test を書く
6. 適切なエラー処理を追加する
7. ドキュメントを更新する

## セキュリティ上の考慮事項

### セキュリティを実装するとき:
- Quarkus Security extension (例: `quarkus-smallrye-jwt`、`quarkus-oidc`) を使う。
- MicroProfile JWT または OIDC を使って role-based access control (RBAC) を実装する。
- すべての入力パラメーターを検証する
