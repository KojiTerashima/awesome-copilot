# @WebMvcTest

集中的なスライス テストによる Spring MVC コントローラーのテスト。

## 基本構造```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
  
  @Autowired
  private MockMvcTester mvc;
  
  @MockitoBean
  private OrderService orderService;
  
  @MockitoBean
  private UserService userService;
}
```## ロードされるもの

- 指定されたコントローラー
- Spring MVC インフラストラクチャ (HandlerMapping、HandlerAdapter)
- Jackson ObjectMapper (JSON 用)
- 例外ハンドラー (@ControllerAdvice)
- Spring Security フィルター (クラスパス上の場合)
- 検証 (クラスパス上の場合)

## GET エンドポイントのテスト```java
@Test
void shouldReturnOrder() {
  var order = new Order(1L, "PENDING", BigDecimal.valueOf(99.99));
  given(orderService.findById(1L)).willReturn(order);
  
  assertThat(mvc.get().uri("/orders/1"))
    .hasStatusOk()
    .hasContentType(MediaType.APPLICATION_JSON)
    .bodyJson()
    .extractingPath("$.status")
    .isEqualTo("PENDING");
}
```## リクエスト本文を使用した POST のテスト

### テキスト ブロックの使用 (Java 25)```java
@Test
void shouldCreateOrder() {
  given(orderService.create(any(OrderRequest.class))).willReturn(1L);
  
  var json = """
    {
      "product": "Product A",
      "quantity": 2
    }
    """;
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content(json))
    .hasStatus(HttpStatus.CREATED)
    .hasHeader("Location", "/orders/1");
}
```### レコードの使用```java
record OrderRequest(String product, int quantity) {}

@Test
void shouldCreateOrderWithRecord() {
  var request = new OrderRequest("Product A", 2);
  given(orderService.create(any())).willReturn(1L);
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content(json.write(request).getJson()))
    .hasStatus(HttpStatus.CREATED);
}
```## 検証エラーのテスト```java
@Test
void shouldRejectInvalidOrder() {
  var invalidJson = """
    {
      "product": "",
      "quantity": -1
    }
    """;
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content(invalidJson))
    .hasStatus(HttpStatus.BAD_REQUEST)
    .bodyJson()
    .hasPath("$.errors");
}
```## クエリパラメータのテスト```java
@Test
void shouldFilterOrdersByStatus() {
  assertThat(mvc.get().uri("/orders?status=PENDING"))
    .hasStatusOk();
  
  verify(orderService).findByStatus(OrderStatus.PENDING);
}
```## パス変数のテスト```java
@Test
void shouldCancelOrder() {
  assertThat(mvc.put().uri("/orders/123/cancel"))
    .hasStatusOk();
  
  verify(orderService).cancel(123L);
}
```## セキュリティを備えたテスト```java
@Test
@WithMockUser(roles = "ADMIN")
void adminShouldDeleteOrder() {
  assertThat(mvc.delete().uri("/orders/1"))
    .hasStatus(HttpStatus.NO_CONTENT);
}

@Test
void anonymousUserShouldBeForbidden() {
  assertThat(mvc.delete().uri("/orders/1"))
    .hasStatus(HttpStatus.UNAUTHORIZED);
}
```## 複数のコントローラー```java
@WebMvcTest({OrderController.class, ProductController.class})
class WebLayerTest {
  // Tests multiple controllers in one slice
}
```## 自動構成を除く```java
@WebMvcTest(OrderController.class)
@AutoConfigureMockMvc(addFilters = false) // Skip security filters
class OrderControllerWithoutSecurityTest {
  // Tests without security filters
}
```## 重要なポイント

1. @MockitoBean を使用して常にサービスを模擬する
2. AssertJ スタイルのアサーションには MockMvcTester を使用する
3. HTTP セマンティクス (ステータス、ヘッダー、コンテンツ タイプ) をテストします。
4. 副作用が問題になる場合はサービス メソッド呼び出しを検証する
5. ここではビジネス ロジックをテストしないでください - それは単体テスト用です
6. JSON ペイロードに Java 25 テキスト ブロックを活用する