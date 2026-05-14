# @DataJpaTest

分離されたデータ層スライスを使用して JPA リポジトリをテストします。

## 基本構造```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryTest {
  
  @Container
  @ServiceConnection
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
  
  @Autowired
  private OrderRepository orderRepository;
  
  @Autowired
  private TestEntityManager entityManager;
}
```## ロードされるもの

- リポジトリ Bean
- EntityManager / TestEntityManager
- データソース
- トランザクションマネージャー
- Web レイヤー、サービス、コントローラーなし

## カスタムクエリのテスト```java
@Test
void shouldFindOrdersByStatus() {
  // Given - Using var for cleaner code
  var pending = new Order("PENDING");
  var completed = new Order("COMPLETED");
  entityManager.persist(pending);
  entityManager.persist(completed);
  entityManager.flush();
  
  // When
  var pendingOrders = orderRepository.findByStatus("PENDING");
  
  // Then - Using sequenced collection methods
  assertThat(pendingOrders).hasSize(1);
  assertThat(pendingOrders.getFirst().getStatus()).isEqualTo("PENDING");
}
```## ネイティブ クエリのテスト```java
@Test
void shouldExecuteNativeQuery() {
  entityManager.persist(new Order("PENDING", BigDecimal.valueOf(100)));
  entityManager.persist(new Order("PENDING", BigDecimal.valueOf(200)));
  entityManager.flush();
  
  var total = orderRepository.calculatePendingTotal();
  
  assertThat(total).isEqualTo(new BigDecimal("300.00"));
}
```## ページネーションのテスト```java
@Test
void shouldReturnPagedResults() {
  // Insert 20 orders using IntStream
  IntStream.range(0, 20).forEach(i -> {
    entityManager.persist(new Order("PENDING"));
  });
  entityManager.flush();
  
  var page = orderRepository.findByStatus("PENDING", PageRequest.of(0, 10));
  
  assertThat(page.getContent()).hasSize(10);
  assertThat(page.getTotalElements()).isEqualTo(20);
  assertThat(page.getContent().getFirst().getStatus()).isEqualTo("PENDING");
}
```## 遅延読み込みのテスト```java
@Test
void shouldLazyLoadOrderItems() {
  var order = new Order("PENDING");
  order.addItem(new OrderItem("Product", 2));
  entityManager.persist(order);
  entityManager.flush();
  entityManager.clear(); // Detach from persistence context
  
  var found = orderRepository.findById(order.getId());
  
  assertThat(found).isPresent();
  // This will trigger lazy loading
  assertThat(found.get().getItems()).hasSize(1);
  assertThat(found.get().getItems().getFirst().getProduct()).isEqualTo("Product");
}
```## カスケードテスト```java
@Test
void shouldCascadeDelete() {
  var order = new Order("PENDING");
  order.addItem(new OrderItem("Product", 2));
  entityManager.persist(order);
  entityManager.flush();
  
  orderRepository.delete(order);
  entityManager.flush();
  
  assertThat(entityManager.find(OrderItem.class, order.getItems().getFirst().getId()))
    .isNull();
}
```## @Query メソッドのテスト```java
@Query("SELECT o FROM Order o WHERE o.createdAt > :date AND o.status = :status")
List<Order> findRecentByStatus(@Param("date") LocalDateTime date, 
                               @Param("status") String status);

@Test
void shouldFindRecentOrders() {
  var old = new Order("PENDING");
  old.setCreatedAt(LocalDateTime.now().minusDays(10));
  var recent = new Order("PENDING");
  recent.setCreatedAt(LocalDateTime.now().minusHours(1));
  
  entityManager.persist(old);
  entityManager.persist(recent);
  entityManager.flush();
  
  var recentOrders = orderRepository.findRecentByStatus(
    LocalDateTime.now().minusDays(1), "PENDING");
  
  assertThat(recentOrders).hasSize(1);
  assertThat(recentOrders.getFirst().getId()).isEqualTo(recent.getId());
}
```## H2 とリアル データベースの使用

### H2 (デフォルト - 運用パリティには推奨されません)```java
@DataJpaTest // Uses embedded H2 by default
class OrderRepositoryH2Test {
  // Fast but may miss DB-specific issues
}
```### テストコンテナ (推奨)```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryPostgresTest {
  @Container
  @ServiceConnection
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
}
```## トランザクションの動作

テストはデフォルトで @Transactional であり、各テスト後にロールバックされます。```java
@Test
@Rollback(false) // Don't roll back (rarely needed)
void shouldPersistData() {
  orderRepository.save(new Order("PENDING"));
  // Data will remain in database after test
}
```## 重要なポイント

1. 設定データに TestEntityManager を使用する
2. SQL をトリガーするには、persist() の後に常に flash() を実行します
3. エンティティ マネージャーを Clear() して遅延読み込みをテストします。
4. 正確な結果を得るために実際のデータベース (Testcontainers) を使用する
5. 成功ケースと失敗ケースの両方をテストする
6. Java 25 var キーワードを活用してよりクリーンな変数宣言を行う
7. 順序付けされたコレクション メソッド (getFirst()、getLast()、reversed()) を使用する