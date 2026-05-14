#MockMvcTester

Spring MVC コントローラー (Spring Boot 3.2 以降) の AssertJ スタイルのテスト。

## 概要

MockMvcTester は、Web 層テスト用に流暢な AssertJ スタイルのアサーションを提供します。従来の MockMvc よりも読みやすく、タイプセーフです。

**推奨パターン**: JSON を実際のオブジェクトに変換し、AssertJ でアサートします。```java
assertThat(mvc.get().uri("/orders/1"))
  .hasStatus(HttpStatus.OK)
  .bodyJson()
  .convertTo(OrderResponse.class)
  .satisfies(response -> {
    assertThat(response.getTotalToPay()).isEqualTo(expectedAmount);
    assertThat(response.getItems()).isNotEmpty();
  });
```## 基本的な使い方```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
  
  @Autowired
  private MockMvcTester mvc;
  
  @MockitoBean
  private OrderService orderService;
}
```## 推奨: オブジェクト変換パターン

### 単一オブジェクトの応答```java
@Test
void shouldGetOrder() {
  given(orderService.findById(1L)).willReturn(new Order(1L, "PENDING", 99.99));
  
  assertThat(mvc.get().uri("/orders/1"))
    .hasStatus(HttpStatus.OK)
    .bodyJson()
    .convertTo(OrderResponse.class)
    .satisfies(response -> {
      assertThat(response.getId()).isEqualTo(1L);
      assertThat(response.getStatus()).isEqualTo("PENDING");
      assertThat(response.getTotalToPay()).isEqualTo(new BigDecimal("99.99"));
    });
}
```### リスト応答```java
@Test
void shouldGetAllOrders() {
  given(orderService.findAll()).willReturn(Arrays.asList(
    new Order(1L, "PENDING"),
    new Order(2L, "COMPLETED")
  ));
  
  assertThat(mvc.get().uri("/orders"))
    .hasStatus(HttpStatus.OK)
    .bodyJson()
    .convertTo(new TypeReference<List<OrderResponse>>() {})
    .satisfies(orders -> {
      assertThat(orders).hasSize(2);
      assertThat(orders.get(0).getStatus()).isEqualTo("PENDING");
      assertThat(orders.get(1).getStatus()).isEqualTo("COMPLETED");
    });
}
```### ネストされたオブジェクト```java
@Test
void shouldGetOrderWithCustomer() {
  assertThat(mvc.get().uri("/orders/1"))
    .hasStatus(HttpStatus.OK)
    .bodyJson()
    .convertTo(OrderResponse.class)
    .satisfies(response -> {
      assertThat(response.getCustomer()).isNotNull();
      assertThat(response.getCustomer().getName()).isEqualTo("John Doe");
      assertThat(response.getCustomer().getAddress().getCity()).isEqualTo("Berlin");
    });
}
```### 複雑なアサーション```java
@Test
void shouldCalculateOrderTotal() {
  assertThat(mvc.get().uri("/orders/1/calculate"))
    .hasStatus(HttpStatus.OK)
    .bodyJson()
    .convertTo(CalculationResponse.class)
    .satisfies(calc -> {
      assertThat(calc.getSubtotal()).isEqualTo(new BigDecimal("100.00"));
      assertThat(calc.getTax()).isEqualTo(new BigDecimal("19.00"));
      assertThat(calc.getTotalToPay()).isEqualTo(new BigDecimal("119.00"));
      assertThat(calc.getItems()).allMatch(item -> item.getPrice().compareTo(BigDecimal.ZERO) > 0);
    });
}
```## HTTP メソッド

### リクエスト本文を含む POST```java
@Test
void shouldCreateOrder() {
  given(orderService.create(any())).willReturn(1L);
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content("{\"product\": \"Laptop\", \"quantity\": 2}"))
    .hasStatus(HttpStatus.CREATED)
    .hasHeader("Location", "/orders/1");
}
```### PUT リクエスト```java
@Test
void shouldUpdateOrder() {
  assertThat(mvc.put().uri("/orders/1")
    .contentType(MediaType.APPLICATION_JSON)
    .content("{\"status\": \"COMPLETED\"}"))
    .hasStatus(HttpStatus.OK);
}
```### 削除リクエスト```java
@Test
void shouldDeleteOrder() {
  assertThat(mvc.delete().uri("/orders/1"))
    .hasStatus(HttpStatus.NO_CONTENT);
}
```## ステータス アサーション```java
assertThat(mvc.get().uri("/orders/1"))
  .hasStatusOk()                    // 200
  .hasStatus(HttpStatus.OK)         // 200
  .hasStatus2xxSuccessful()         // 2xx
  .hasStatusBadRequest()            // 400
  .hasStatusNotFound()              // 404
  .hasStatusUnauthorized()          // 401
  .hasStatusForbidden()             // 403
  .hasStatus(HttpStatus.CREATED);   // 201
```## コンテンツ タイプ アサーション```java
assertThat(mvc.get().uri("/orders/1"))
  .hasContentType(MediaType.APPLICATION_JSON)
  .hasContentTypeCompatibleWith(MediaType.APPLICATION_JSON);
```## ヘッダー アサーション```java
assertThat(mvc.post().uri("/orders"))
  .hasHeader("Location", "/orders/123")
  .hasHeader("X-Request-Id", matchesPattern("[a-z0-9-]+"));
```## 代替: JSON パス (慎重に使用)

型付きオブジェクトに変換できない場合にのみ使用してください。```java
assertThat(mvc.get().uri("/orders/1"))
  .hasStatusOk()
  .bodyJson()
  .extractingPath("$.customer.address.city")
  .asString()
  .isEqualTo("Berlin");
```## リクエストパラメータ```java
// Query parameters
assertThat(mvc.get().uri("/orders?status=PENDING&page=0"))
  .hasStatusOk();

// Path parameters
assertThat(mvc.get().uri("/orders/{id}", 1L))
  .hasStatusOk();

// Headers
assertThat(mvc.get().uri("/orders/1")
  .header("X-Api-Key", "secret"))
  .hasStatusOk();
```## JacksonTester を使用したリクエスト本文```java
@Autowired
private JacksonTester<OrderRequest> json;

@Test
void shouldCreateOrder() {
  OrderRequest request = new OrderRequest("Laptop", 2);
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content(json.write(request).getJson()))
    .hasStatus(HttpStatus.CREATED);
}
```## エラー応答```java
@Test
void shouldReturnValidationErrors() {
  given(orderService.findById(999L))
    .willThrow(new OrderNotFoundException(999L));
  
  assertThat(mvc.get().uri("/orders/999"))
    .hasStatus(HttpStatus.NOT_FOUND)
    .bodyJson()
    .convertTo(ErrorResponse.class)
    .satisfies(error -> {
      assertThat(error.getMessage()).isEqualTo("Order 999 not found");
      assertThat(error.getCode()).isEqualTo("ORDER_NOT_FOUND");
    });
}
```## 検証エラーのテスト```java
@Test
void shouldRejectInvalidOrder() {
  OrderRequest invalidRequest = new OrderRequest("", -1);
  
  assertThat(mvc.post().uri("/orders")
    .contentType(MediaType.APPLICATION_JSON)
    .content(json.write(invalidRequest).getJson()))
    .hasStatus(HttpStatus.BAD_REQUEST)
    .bodyJson()
    .convertTo(ValidationErrorResponse.class)
    .satisfies(errors -> {
      assertThat(errors.getFieldErrors()).hasSize(2);
      assertThat(errors.getFieldErrors())
        .extracting("field")
        .contains("product", "quantity");
    });
}
```## 比較: MockMvcTester と Classic MockMvc

|特集 |モックMvcテスター |クラシックモックMVC |
| ------- | ------------- | --------------- |
|スタイル | AssertJ 流暢 | MockMvc マッチャー |
|可読性 |高 |中 |
|タイプセーフティ |より良い |少ない |
| IDEサポート |素晴らしい |良い |
|オブジェクト変換 |ネイティブ |マニュアル |

## Classic MockMvc からの移行

### 前 (クラシック)```java
mvc.perform(get("/orders/1"))
  .andExpect(status().isOk())
  .andExpect(jsonPath("$.status").value("PENDING"))
  .andExpect(jsonPath("$.totalToPay").value(99.99));
```### 後 (オブジェクト変換を使用したテスター)```java
assertThat(mvc.get().uri("/orders/1"))
  .hasStatus(HttpStatus.OK)
  .bodyJson()
  .convertTo(OrderResponse.class)
  .satisfies(response -> {
    assertThat(response.getStatus()).isEqualTo("PENDING");
    assertThat(response.getTotalToPay()).isEqualTo(new BigDecimal("99.99"));
  });
```## 重要なポイント

1. **`extractingPath()`** よりも `convertTo()` を優先 - タイプセーフでリファクタリング可能
2. **複数のアサーションには `satisfies()` を使用します** - テストを読みやすく保ちます
3. **静的 `org.assertj.core.api.Assertions.assertThat`** をインポートします
4. **`TypeReference`** を介してジェネリックで動作します - `List<T>` 応答の場合
5. **IDE リファクタリングに優しい** - フィールド名の変更、IDE のテストの更新