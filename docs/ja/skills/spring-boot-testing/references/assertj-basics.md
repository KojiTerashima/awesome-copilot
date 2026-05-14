# AssertJ の基本

読みやすく保守しやすいテストのための流暢なアサーション。

## 基本的なアサーション

### オブジェクトの等価性```java
assertThat(order.getStatus()).isEqualTo("PENDING");
assertThat(order.getId()).isNotEqualTo(0);
assertThat(order).isEqualTo(expectedOrder);
assertThat(order).isNotNull();
assertThat(nullOrder).isNull();
```### 文字列アサーション```java
assertThat(order.getDescription())
  .isEqualTo("Test Order")
  .startsWith("Test")
  .endsWith("Order")
  .contains("Test")
  .hasSize(10)
  .matches("[A-Za-z ]+");
```### 数値アサーション```java
assertThat(order.getAmount())
  .isEqualTo(99.99)
  .isGreaterThan(50)
  .isLessThan(100)
  .isBetween(50, 100)
  .isPositive()
  .isNotZero();
```### ブール値アサーション```java
assertThat(order.isActive()).isTrue();
assertThat(order.isDeleted()).isFalse();
```## 日付/時刻アサーション```java
assertThat(order.getCreatedAt())
  .isEqualTo(LocalDateTime.of(2024, 1, 15, 10, 30))
  .isBefore(LocalDateTime.now())
  .isAfter(LocalDateTime.of(2024, 1, 1))
  .isCloseTo(LocalDateTime.now(), within(5, ChronoUnit.SECONDS));
```## オプションのアサーション```java
Optional<Order> maybeOrder = orderService.findById(1L);

assertThat(maybeOrder)
  .isPresent()
  .hasValueSatisfying(order -> {
    assertThat(order.getId()).isEqualTo(1L);
  });

assertThat(orderService.findById(999L)).isEmpty();
```## 例外アサーション

### JUnit 5 例外処理```java
@Test
void shouldThrowException() {
  OrderService service = new OrderService();
  
  assertThatThrownBy(() -> service.findById(999L))
    .isInstanceOf(OrderNotFoundException.class)
    .hasMessage("Order 999 not found")
    .hasMessageContaining("999");
}
```### AssertJ 例外処理```java
@Test
void shouldThrowExceptionWithCause() {
  assertThatExceptionOfType(OrderProcessingException.class)
    .isThrownBy(() -> service.processOrder(invalidOrder))
    .withCauseInstanceOf(ValidationException.class);
}
```## カスタム アサーション

再利用可能なテスト コード用にドメイン固有のアサーションを作成します。```java
public class OrderAssert extends AbstractAssert<OrderAssert, Order> {
  
  public static OrderAssert assertThat(Order actual) {
    return new OrderAssert(actual);
  }
  
  private OrderAssert(Order actual) {
    super(actual, OrderAssert.class);
  }
  
  public OrderAssert isPending() {
    isNotNull();
    if (!"PENDING".equals(actual.getStatus())) {
      failWithMessage("Expected order status to be PENDING but was %s", actual.getStatus());
    }
    return this;
  }
  
  public OrderAssert hasTotal(BigDecimal expected) {
    isNotNull();
    if (!expected.equals(actual.getTotal())) {
      failWithMessage("Expected total %s but was %s", expected, actual.getTotal());
    }
    return this;
  }
}
```使用法：```java
OrderAssert.assertThat(order)
  .isPending()
  .hasTotal(new BigDecimal("99.99"));
```## ソフト アサーション

失敗する前に複数の失敗を収集します。```java
@Test
void shouldValidateOrder() {
  Order order = orderService.findById(1L);
  
  SoftAssertions.assertSoftly(softly -> {
    softly.assertThat(order.getId()).isEqualTo(1L);
    softly.assertThat(order.getStatus()).isEqualTo("PENDING");
    softly.assertThat(order.getItems()).isNotEmpty();
  });
}
```## パターンを満たす```java
assertThat(order)
  .satisfies(o -> {
    assertThat(o.getId()).isPositive();
    assertThat(o.getStatus()).isNotBlank();
    assertThat(o.getCreatedAt()).isNotNull();
  });
```## Spring での使用```java
import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
class OrderServiceTest {
  
  @Autowired
  private OrderService orderService;
  
  @Test
  void shouldCreateOrder() {
    Order order = orderService.create(new OrderRequest("Product", 2));
    
    assertThat(order)
      .isNotNull()
      .extracting(Order::getId, Order::getStatus)
      .containsExactly(1L, "PENDING");
  }
}
```## 静的インポート

クリーンなアサーションを得るには、常に静的インポートを使用してください。```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.assertj.core.api.Assertions.catchThrowable;
```## 主な利点

1. **読みやすい**: 文章のような構造
2. **タイプセーフ**: IDE オートコンプリートが機能します
3. **豊富な API**: 多くの組み込みアサーション
4. **拡張可能**: ドメインのカスタム アサーション
5. **エラーの改善**: 失敗メッセージをクリアする