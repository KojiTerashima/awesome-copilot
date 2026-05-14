# @MockitoBean

Spring Boot テストでの依存関係のモック (Spring Boot 4 以降では非推奨の @MockBean を置き換えます)。

## 概要

`@MockitoBean` は、Spring Boot 4.0 以降で非推奨となった `@MockBean` アノテーションを置き換えます。 Mockito モックを作成して Spring コンテキストに登録し、同じタイプの既存の Bean を置き換えます。

## 基本的な使い方```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
  
  @MockitoBean
  private OrderService orderService;
  
  @MockitoBean
  private UserService userService;
}
```## サポートされているテスト スライス

- `@WebMvcTest` - サービス/リポジトリの依存関係をモックする
- `@WebFluxTest` - リアクティブ サービスの依存関係を模擬する
- `@SpringBootTest` - 本物の Bean をモックに置き換えます

## スタブ化メソッド

### 基本スタブ```java
@Test
void shouldReturnOrder() {
  Order order = new Order(1L, "PENDING");
  given(orderService.findById(1L)).willReturn(order);
  
  // Test code
}
```### 複数の返品```java
given(orderService.findById(anyLong()))
  .willReturn(new Order(1L, "PENDING"))
  .willReturn(new Order(2L, "COMPLETED"));
```### 例外のスロー```java
given(orderService.findById(999L))
  .willThrow(new OrderNotFoundException(999L));
```### 引数のマッチング```java
given(orderService.create(argThat(req -> req.getQuantity() > 0)))
  .willReturn(1L);

given(orderService.findByStatus(eq("PENDING")))
  .willReturn(List.of(new Order()));
```## インタラクションの検証

### 呼び出されたメソッドの検証```java
verify(orderService).findById(1L);
```### 電話がかかっていないことを確認する```java
verify(orderService, never()).delete(any());
```### カウントの検証```java
verify(orderService, times(2)).findById(anyLong());
verify(orderService, atLeastOnce()).findByStatus(anyString());
```### 注文の確認```java
InOrder inOrder = inOrder(orderService, userService);
inOrder.verify(orderService).findById(1L);
inOrder.verify(userService).getUser(any());
```## モックのリセット

モックはテスト間で自動的にリセットされます。テスト中にリセットするには:```java
Mockito.reset(orderService);
```## 部分モッキング用の @MockitoSpyBean

`@MockitoSpyBean` を使用して、実際の Bean を Mockito でラップします。```java
@SpringBootTest
class OrderServiceIntegrationTest {
  
  @MockitoSpyBean
  private PaymentGatewayClient paymentClient;
  
  @Test
  void shouldProcessOrder() {
    doReturn(true).when(paymentClient).processPayment(any());
    
    // Test with real service but mocked payment client
  }
}
```## カスタム テスト Bean の @TestBean

カスタム Bean インスタンスをテスト コンテキストに登録します。```java
@SpringBootTest
class OrderServiceTest {
  
  @TestBean
  private PaymentGatewayClient paymentClient() {
    return new FakePaymentClient();
  }
}
```## スコーピング: シングルトンとプロトタイプ

Spring Framework 7+ (Spring Boot 4+) は、非シングルトン Bean のモックをサポートしています。```java
@Component
@Scope("prototype")
public class OrderProcessor {
  public String process() { return "real"; }
}

@SpringBootTest
class OrderServiceTest {
  @MockitoBean
  private OrderProcessor orderProcessor;
  
  @Test
  void shouldWorkWithPrototype() {
    given(orderProcessor.process()).willReturn("mocked");
    // Test code
  }
}
```## 一般的なパターン

### サービステストでのリポジトリのモック化```java
@SpringBootTest
class OrderServiceTest {
  @MockitoBean
  private OrderRepository orderRepository;
  
  @Autowired
  private OrderService orderService;
  
  @Test
  void shouldCreateOrder() {
    given(orderRepository.save(any())).willReturn(new Order(1L));
    
    Long id = orderService.createOrder(new OrderRequest());
    
    assertThat(id).isEqualTo(1L);
    verify(orderRepository).save(any(Order.class));
  }
}
```### 同じタイプの複数のモック

Bean 名を使用します。```java
@MockitoBean(name = "primaryDataSource")
private DataSource primaryDataSource;

@MockitoBean(name = "secondaryDataSource")
private DataSource secondaryDataSource;
```## @MockBean からの移行

### 以前 (非推奨)```java
@MockBean
private OrderService orderService;
```### 後 (Spring Boot 4+)```java
@MockitoBean
private OrderService orderService;
```## Mockito @Mock との主な違い

|特集 | @MockitoBean @モック |
| ------- | ------------ | ----- |
|コンテキストの統合 |はい |いいえ |
|春のライフサイクル |参加 |なし |
| @Autowired と連携します |はい |いいえ |
|テストスライスのサポート |はい |限定 |

## ベストプラクティス

1. Spring コンテキストが関係する場合にのみ `@MockitoBean` を使用します
2. 純粋な単体テストの場合は、Mockito の `@Mock` または `Mockito.mock()` を使用します。
3. 副作用のある相互作用を常に検証する
4. 単純なクエリを検証しない (スタブ化で十分です)
5. テストによって共有モック状態が変更された場合はモックをリセットする