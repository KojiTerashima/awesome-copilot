# Spring Boot 4.0 への移行

Spring Boot 3.x から 4.0 に移行する際の主要なテストの変更。

## 依存関係の変更

### モジュール式テストスターター

Spring Boot 4.0 では、モジュラー テスト スターターが導入されています。

**以前 (3.x):**```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```**(4.0) 以降 - WebMvc テスト:**```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webmvc-test</artifactId>
  <scope>test</scope>
</dependency>
```**(4.0) 後 - REST クライアント テスト:**```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-restclient-test</artifactId>
  <scope>test</scope>
</dependency>
```## アノテーションの移行

### @MockBean → @MockitoBean

**非推奨 (3.x):**```java
@MockBean
private OrderService orderService;
```**新機能 (4.0):**```java
@MockitoBean
private OrderService orderService;
```### @SpyBean → @MockitoSpyBean

**非推奨 (3.x):**```java
@SpyBean
private PaymentGatewayClient paymentClient;
```**新機能 (4.0):**```java
@MockitoSpyBean
private PaymentGatewayClient paymentClient;
```## 新しいテスト機能

### RestTestClient

TestRestTemplate を置き換えます (非推奨):```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class OrderIntegrationTest {
  
  @Autowired
  private RestTestClient restClient;
  
  @Test
  void shouldCreateOrder() {
    restClient
      .post()
      .uri("/orders")
      .body(new OrderRequest("Product", 2))
      .exchange()
      .expectStatus()
      .isCreated()
      .expectHeader()
      .location("/orders/1");
  }
}
```## JUnit 6 のサポート

Spring Boot 4.0 はデフォルトで JUnit 6 を使用します。

- JUnit 4 は非推奨になりました (一時的に JUnit Vintage を使用してください)
- JUnit 5 のすべての機能は引き続き動作します
- クリーンな移行のために JUnit 4 の依存関係を削除

## テストコンテナ 2.0

モジュールの名前が変更されました:

**以前 (1.x):**```xml
<artifactId>postgresql</artifactId>
```**(2.0) 以降:**```xml
<artifactId>testcontainers-postgresql</artifactId>
```## 非シングルトン Bean モッキング

Spring Framework 7 では、プロトタイプ スコープの Bean のモックが可能です。```java
@Component
@Scope("prototype")
public class OrderProcessor { }

@SpringBootTest
class OrderServiceTest {
  @MockitoBean
  private OrderProcessor orderProcessor; // Now works!
}
```## SpringExtension コンテキストの変更

拡張コンテキストはデフォルトでテストメソッドスコープになりました。

@Nested クラスでテストが失敗した場合:```java
@SpringExtensionConfig(useTestClassScopedExtensionContext = true)
@SpringBootTest
class OrderTest {
  // Use old behavior
}
```## 移行チェックリスト

- [ ] @MockBean を @MockitoBean に置き換えます
- [ ] @SpyBean を @MockitoSpyBean に置き換えます
- [ ] Testcontainers の依存関係を 2.0 の名前付けに更新します
- [ ] 必要に応じてモジュラー テスト スターターを追加します
- [ ] TestRestTemplate を RestTestClient に移行する
- [ ] JUnit 4 の依存関係を削除します。
- [ ] カスタム TestExecutionListener 実装を更新します
- [ ] @Nested クラスの動作をテストする

## 下位互換性

段階的な移行には「クラシック」スターターを使用します。```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test-classic</artifactId>
  <scope>test</scope>
</dependency>
```これにより、段階的に移行する際に古い動作が提供されます。