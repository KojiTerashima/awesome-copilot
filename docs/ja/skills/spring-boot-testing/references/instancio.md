#インスタンス

複雑なテスト オブジェクトを自動的に生成します。エンティティ/DTO に 3 つ以上のプロパティがある場合に使用します。

## いつ使用するか

- **3 つ以上のプロパティ**を持つオブジェクト
- リポジトリ用のテストデータの設定
- コントローラーテスト用の DTO の作成
- 繰り返しのビルダー/セッター呼び出しの回避

## 依存関係```xml
<dependency>
  <groupId>org.instancio</groupId>
  <artifactId>instancio-junit</artifactId>
  <version>5.5.1</version>
  <scope>test</scope>
</dependency>
```## 基本的な使い方

### 単純なオブジェクト```java
final var order = Instancio.create(Order.class);
// All fields populated with random data
```### オブジェクトのリスト```java
final var orders = Instancio.ofList(Order.class).size(5).create();
// 5 orders with random data
```## 値のカスタマイズ

### 特定のフィールドを設定する```java
final var order = Instancio.of(Order.class)
  .set(field(Order::getStatus), "PENDING")
  .set(field(Order::getTotal), new BigDecimal("99.99"))
  .create();
```### 生成された値を提供する```java
final var order = Instancio.of(Order.class)
  .supply(field(Order::getEmail), () -> "user" + UUID.randomUUID() + "@test.com")
  .create();
```### フィールドを無視する```java
final var order = Instancio.of(Order.class)
  .ignore(field(Order::getId)) // Let DB generate
  .create();
```## 複雑なオブジェクト

### ネストされたオブジェクト```java
final var order = Instancio.of(Order.class)
  .set(field(Order::getCustomer), Instancio.create(Customer.class))
  .set(field(Order::getItems), Instancio.ofList(OrderItem.class).size(3).create())
  .create();
```### すべてのフィールドがランダム```java
// When you need fully random but valid data
final var randomOrder = Instancio.create(Order.class);
// Customer, items, addresses - all populated
```## Spring Boot の統合

### リポジトリのテスト設定```java
@DataJpaTest
@AutoConfigureTestDatabase
@Testcontainers
class OrderRepositoryTest {
  
  @Container
  @ServiceConnection
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
  
  @Autowired
  private OrderRepository orderRepository;
  
  @Test
  void shouldFindOrdersByStatus() {
    // Given: Create 10 random orders with PENDING status
    final var orders = Instancio.ofList(Order.class)
      .size(10)
      .set(field(Order::getStatus), "PENDING")
      .create();
    
    orderRepository.saveAll(orders);
    
    // When
    final var found = orderRepository.findByStatus("PENDING");
    
    // Then
    assertThat(found).hasSize(10);
  }
}
```### コントローラーテストのセットアップ```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
  
  @Autowired
  private MockMvcTester mvc;
  
  @MockitoBean
  private OrderService orderService;
  
  @Test
  void shouldReturnOrder() {
    // Given: Random order with specific ID
    Order order = Instancio.of(Order.class)
      .set(field(Order::getId), 1L)
      .create();
    
    given(orderService.findById(1L)).willReturn(order);
    
    // When/Then
    assertThat(mvc.get().uri("/orders/1"))
      .hasStatus(HttpStatus.OK)
      .bodyJson()
      .convertTo(OrderResponse.class)
      .satisfies(response -> {
        assertThat(response.getId()).isEqualTo(1L);
      });
  }
}
```## パターン

### ビルダーパターンの代替案```java
// Instead of:
Order order = Order.builder()
  .id(1L)
  .status("PENDING")
  .customer(Customer.builder().name("John").build())
  .items(List.of(
    OrderItem.builder().product("A").price(10).build(),
    OrderItem.builder().product("B").price(20).build()
  ))
  .build();

// Use:
Order order = Instancio.of(Order.class)
  .set(field(Order::getId), 1L)
  .set(field(Order::getStatus), "PENDING")
  .create();
// Customer and items auto-generated
```### シードされたデータ```java
// Consistent "random" data for reproducible tests
Order order = Instancio.of(Order.class)
  .withSeed(12345L)
  .create();
// Same data every test run with seed 12345
```## 一般的なパターン

### 電子メールの生成```java
String email = Instancio.gen().net().email();
```### 日付の生成```java
LocalDateTime createdAt = Instancio.gen().temporal()
  .localDateTime()
  .past()
  .create();
```### 文字列パターン```java
String phone = Instancio.gen().text().pattern("+1-###-###-####");
```## 比較

|アプローチ |コード行 |保守性 |
| -------- | ------------- | --------------- |
|マニュアルセッター | 10-20 |低い |
|ビルダーパターン | 5-10 |中 |
| **インスタンス** | 2-5 | **高** |

## ベストプラクティス

1. **3 つ以上のプロパティ オブジェクトに使用** - 単純なオブジェクトには価値がありません
2. **関連するものだけを設定** - 残りは Instantio に入力させます
3. **テストコンテナと併用** - データベースのシード処理に最適
4. **ID を明示的に設定** - 特定のシナリオをテストする場合
5. **自動生成フィールドを無視します** - createdAt、updatedAt など

## リンク

- [インスタンスのドキュメント](https://www.instancio.org/)
- [JUnit 5 拡張機能](https://www.instantio.org/user-guide/#junit-integration)