# テストスライスの概要

適切な Spring Boot テスト スライスを選択するためのクイック リファレンス。

## 意思決定マトリックス

|注釈 |いつ使用する |負荷 |スピード |
| ---------- | -------- | ----- | ----- |
| **なし** (プレーン JUnit) |純粋なビジネス ロジックのテスト |何も |最速 |
| `@WebMvcTest` |コントローラー + HTTP レイヤー |コントローラー、MVC、ジャクソン |速い |
| `@DataJpaTest` |リポジトリのクエリ |リポジトリ、JPA、データソース |速い |
| `@RestClientTest` | REST クライアント コード | RestTemplate/RestClient、ジャクソン |速い |
| `@JsonTest` | JSON シリアル化 | ObjectMapper のみ |最速のスライス |
| `@WebFluxTest` |リアクティブコントローラー |コントローラー、WebFlux |速い |
| `@DataJdbcTest` | JDBC リポジトリ |リポジトリ、JDBC |速い |
| `@DataMongoTest` | MongoDB リポジトリ |リポジトリ、MongoDB |速い |
| `@DataRedisTest` | Redis リポジトリ |リポジトリ、Redis |速い |
| `@SpringBootTest` |完全な統合 |アプリケーション全体 |遅い |

## 選択ガイド

### アノテーションを使用しない (単純な単体テスト)```java
class PriceCalculatorTest {
  private PriceCalculator calculator = new PriceCalculator();
  
  @Test
  void shouldApplyDiscount() {
    var result = calculator.applyDiscount(100, 0.1);
    assertThat(result).isEqualTo(new BigDecimal("90.00"));
  }
}
```**いつ**: 純粋なビジネス ロジック。依存関係がない、またはコンストラクター インジェクションによって模擬可能な単純な依存関係。

### @WebMvcTest を使用する```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
  @Autowired private MockMvcTester mvc;
  @MockitoBean private OrderService orderService;
}
```**いつ**: リクエストのマッピング、検証、JSON マッピング、セキュリティ、フィルターをテストします。

**得られるもの**: MockMvc、ObjectMapper、Spring Security (存在する場合)、例外ハンドラー。

### @DataJpaTest を使用する```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryTest {
  @Container
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
}
```**いつ**: カスタム JPA クエリ、エンティティ マッピング、トランザクション動作、カスケード操作をテストします。

**得られるもの**: リポジトリ Bean、EntityManager、TestEntityManager、トランザクション サポート。

### @RestClientTest を使用する```java
@RestClientTest(WeatherService.class)
class WeatherServiceTest {
  @Autowired private WeatherService weatherService;
  @Autowired private MockRestServiceServer server;
}
```**いつ**: 外部 API を呼び出す REST クライアントをテストします。

**得られるもの**: HTTP 応答をスタブするための MockRestServiceServer。

### @JsonTest を使用する```java
@JsonTest
class OrderJsonTest {
  @Autowired private JacksonTester<Order> json;
}
```**時期**: カスタム シリアライザー/デシリアライザー、複雑な JSON マッピングをテストします。

### @SpringBootTest を使用する```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class OrderIntegrationTest {
  @Autowired private RestTestClient restClient;
}
```**時期**: 完全なリクエスト フロー、セキュリティ フィルター、データベース インタラクションを一緒にテストします。

**得られるもの**: 完全なアプリケーション コンテキスト、組み込みサーバー (オプション)、実際の Bean。

## よくある間違い

1. **すべてに @SpringBootTest を使用する** - テスト スイートが不必要に遅くなります
2. **サービスをモックしない @WebMvcTest** - コンテキストの読み込みエラーが発生する
3. **@DataJpaTest と @MockBean** - 目的を果たせません (実際のリポジトリが必要です)
4. **1 つのテスト内の複数のスライス** - 各スライスは個別のテスト クラスです。

## テストにおける Java 25 の機能

### テストデータの記録```java
record OrderRequest(String product, int quantity) {}
record OrderResponse(Long id, String status, BigDecimal total) {}
```### テストでのパターン マッチング```java
@Test
void shouldHandleDifferentOrderTypes() {
  var order = orderService.create(new OrderRequest("Product", 2));
  
  switch (order) {
    case PhysicalOrder po -> assertThat(po.getShippingAddress()).isNotNull();
    case DigitalOrder do_ -> assertThat(do_.getDownloadLink()).isNotNull();
    default -> throw new IllegalStateException("Unknown order type");
  }
}
```### JSON のテキスト ブロック```java
@Test
void shouldParseComplexJson() {
  var json = """
    {
      "id": 1,
      "status": "PENDING",
      "items": [
        {"product": "Laptop", "price": 999.99},
        {"product": "Mouse", "price": 29.99}
      ]
    }
    """;
  
  assertThat(mvc.post().uri("/orders")
    .contentType(APPLICATION_JSON)
    .content(json))
    .hasStatus(CREATED);
}
```### シーケンスされたコレクション```java
@Test
void shouldReturnOrdersInSequence() {
  var orders = orderRepository.findAll();
  
  assertThat(orders.getFirst().getStatus()).isEqualTo("NEW");
  assertThat(orders.getLast().getStatus()).isEqualTo("COMPLETED");
  assertThat(orders.reversed().getFirst().getStatus()).isEqualTo("COMPLETED");
}
```## スライスごとの依存関係```xml
<!-- WebMvcTest -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webmvc-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- DataJpaTest -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- RestClientTest -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-restclient-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- Testcontainers -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-testcontainers</artifactId>
  <scope>test</scope>
</dependency>
```
