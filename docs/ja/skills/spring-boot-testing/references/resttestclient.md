#RestTestClient

Spring Boot 4+ を使用した最新の REST クライアント テスト (TestRestTemplate を置き換えます)。

## 概要

RestTestClient は、Spring Boot 4.0 以降の TestRestTemplate の最新の代替手段です。 REST エンドポイントをテストするための流暢でリアクティブな API を提供します。

## セットアップ

### 依存関係 (Spring Boot 4+)```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-restclient-test</artifactId>
  <scope>test</scope>
</dependency>
```### 基本構成```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class OrderIntegrationTest {
  
  @Autowired
  private RestTestClient restClient;
}
```## HTTP メソッド

### GETリクエスト```java
@Test
void shouldGetOrder() {
  restClient
    .get()
    .uri("/orders/1")
    .exchange()
    .expectStatus()
    .isOk()
    .expectBody(Order.class)
    .value(order -> {
      assertThat(order.getId()).isEqualTo(1L);
      assertThat(order.getStatus()).isEqualTo("PENDING");
    });
}
```### POSTリクエスト```java
@Test
void shouldCreateOrder() {
  OrderRequest request = new OrderRequest("Laptop", 2);
  
  restClient
    .post()
    .uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .body(request)
    .exchange()
    .expectStatus()
    .isCreated()
    .expectHeader()
    .location("/orders/1")
    .expectBody(Long.class)
    .isEqualTo(1L);
}
```### PUT リクエスト```java
@Test
void shouldUpdateOrder() {
  restClient
    .put()
    .uri("/orders/1")
    .body(new OrderUpdate("COMPLETED"))
    .exchange()
    .expectStatus()
    .isOk();
}
```### 削除リクエスト```java
@Test
void shouldDeleteOrder() {
  restClient
    .delete()
    .uri("/orders/1")
    .exchange()
    .expectStatus()
    .isNoContent();
}
```## 応答アサーション

### ステータスコード```java
restClient
  .get()
  .uri("/orders/1")
  .exchange()
  .expectStatus()
  .isOk()           // 200
  .isCreated()      // 201
  .isNoContent()    // 204
  .isBadRequest()   // 400
  .isNotFound()     // 404
  .is5xxServerError() // 5xx
  .isEqualTo(200);  // Specific code
```### ヘッダー```java
restClient
  .post()
  .uri("/orders")
  .exchange()
  .expectHeader()
  .location("/orders/1")
  .contentType(MediaType.APPLICATION_JSON)
  .exists("X-Request-Id")
  .valueEquals("X-Api-Version", "v1");
```### 本文のアサーション```java
restClient
  .get()
  .uri("/orders/1")
  .exchange()
  .expectBody(Order.class)
  .value(order -> assertThat(order.getId()).isEqualTo(1L))
  .returnResult();
```### JSON パス```java
restClient
  .get()
  .uri("/orders")
  .exchange()
  .expectBody()
  .jsonPath("$.content[0].id").isEqualTo(1)
  .jsonPath("$.content[0].status").isEqualTo("PENDING")
  .jsonPath("$.totalElements").isNumber();
```## 構成のリクエスト

### ヘッダー```java
restClient
  .get()
  .uri("/orders/1")
  .header("Authorization", "Bearer token")
  .header("X-Api-Key", "secret")
  .exchange();
```### クエリパラメータ```java
restClient
  .get()
  .uri(uriBuilder -> uriBuilder
    .path("/orders")
    .queryParam("status", "PENDING")
    .queryParam("page", 0)
    .queryParam("size", 10)
    .build())
  .exchange();
```### パス変数```java
restClient
  .get()
  .uri("/orders/{id}", 1L)
  .exchange();
```## MockMvc を使用する

RestTestClient は MockMvc とも連携できます (サーバーは起動しません)。```java
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestTestClient
class OrderMockMvcTest {
  
  @Autowired
  private RestTestClient restClient;
  
  @Test
  void shouldWorkWithMockMvc() {
    // Uses MockMvc under the hood - no server startup
    restClient
      .get()
      .uri("/orders/1")
      .exchange()
      .expectStatus()
      .isOk();
  }
}
```## 比較: RestTestClient と TestRestTemplate

|特集 |テストクライアント |テストレストテンプレート |
| ------- | -------------- | ---------------- |
|スタイル |流暢/反応的 |命令的 |
|スプリングブーツ | 4.0+ |すべてのバージョン (4 で非推奨) |
|アサーション |内蔵 |マニュアル |
| MockMvc のサポート |はい |いいえ |
|非同期 |ネイティブ |追加の処理が必要 |

## TestRestTemplate からの移行

### 以前 (非推奨)```java
@Autowired
private TestRestTemplate restTemplate;

@Test
void shouldGetOrder() {
  ResponseEntity<Order> response = restTemplate
    .getForEntity("/orders/1", Order.class);
  
  assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
  assertThat(response.getBody().getId()).isEqualTo(1L);
}
```### 後 (RestTestClient)```java
@Autowired
private RestTestClient restClient;

@Test
void shouldGetOrder() {
  restClient
    .get()
    .uri("/orders/1")
    .exchange()
    .expectStatus()
    .isOk()
    .expectBody(Order.class)
    .value(order -> assertThat(order.getId()).isEqualTo(1L));
}
```## ベストプラクティス

1. 実際の HTTP には @SpringBootTest(WebEnvironment.RANDOM_PORT) とともに使用します
2. @AutoConfigureMockMvc と組み合わせて使用すると、サーバーなしでテストを高速化できます。
3. 読みやすさのために流暢なアサーションを活用する
4. 成功シナリオとエラーシナリオの両方をテストする
5. セキュリティ/API バージョン管理のヘッダーを確認する