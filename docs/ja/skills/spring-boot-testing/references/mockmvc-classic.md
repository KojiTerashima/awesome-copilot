#MockMvc クラシック

Spring MVC コントローラー テスト用の従来の MockMvc API (Spring Boot 3.2 以前または従来のコードベース)。

## このリファレンスを使用する場合

- プロジェクトは Spring Boot < 3.2 を使用しています (`MockMvcTester` は利用できません)
- 既存のテストは `mvc.perform(...)` を使用しており、それらを維持または拡張しています
- クラシック MockMvc テストを `MockMvcTester` に移行する必要があります (下記の移行セクションを参照)
- ユーザーが `ResultActions`、`andExpect()`、または Hamcrest スタイルの Web アサーションについて明示的に質問する

Spring Boot 3.2 以降の新しいテストの場合は、代わりに [mockmvc-tester.md](mockmvc-tester.md) を優先してください。

＃＃ 設定```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

  @Autowired
  private MockMvc mvc;

  @MockBean
  private OrderService orderService;
}
```## 基本的な GET リクエスト```java
@Test
void shouldReturnOrder() throws Exception {
  given(orderService.findById(1L)).willReturn(new Order(1L, "PENDING", 99.99));

  mvc.perform(get("/orders/1"))
    .andExpect(status().isOk())
    .andExpect(content().contentType(MediaType.APPLICATION_JSON))
    .andExpect(jsonPath("$.id").value(1))
    .andExpect(jsonPath("$.status").value("PENDING"))
    .andExpect(jsonPath("$.totalToPay").value(99.99));
}
```## リクエスト本文を含む POST```java
@Test
void shouldCreateOrder() throws Exception {
  given(orderService.create(any(OrderRequest.class))).willReturn(1L);

  mvc.perform(post("/orders")
      .contentType(MediaType.APPLICATION_JSON)
      .content("{\"product\": \"Laptop\", \"quantity\": 2}"))
    .andExpect(status().isCreated())
    .andExpect(header().string("Location", "/orders/1"));
}
```## PUT リクエスト```java
@Test
void shouldUpdateOrder() throws Exception {
  mvc.perform(put("/orders/1")
      .contentType(MediaType.APPLICATION_JSON)
      .content("{\"status\": \"COMPLETED\"}"))
    .andExpect(status().isOk());
}
```## 削除リクエスト```java
@Test
void shouldDeleteOrder() throws Exception {
  mvc.perform(delete("/orders/1"))
    .andExpect(status().isNoContent());
}
```## ステータスマッチャー```java
.andExpect(status().isOk())           // 200
.andExpect(status().isCreated())      // 201
.andExpect(status().isNoContent())    // 204
.andExpect(status().isBadRequest())   // 400
.andExpect(status().isUnauthorized()) // 401
.andExpect(status().isForbidden())    // 403
.andExpect(status().isNotFound())     // 404
.andExpect(status().is(422))          // arbitrary code
```## JSON パス アサーション```java
// Exact value
.andExpect(jsonPath("$.status").value("PENDING"))

// Existence
.andExpect(jsonPath("$.id").exists())
.andExpect(jsonPath("$.deletedAt").doesNotExist())

// Array size
.andExpect(jsonPath("$.items").isArray())
.andExpect(jsonPath("$.items", hasSize(3)))

// Nested field
.andExpect(jsonPath("$.customer.name").value("John Doe"))
.andExpect(jsonPath("$.customer.address.city").value("Berlin"))

// With Hamcrest matchers
.andExpect(jsonPath("$.total", greaterThan(0.0)))
.andExpect(jsonPath("$.description", containsString("order")))
```## コンテンツ アサーション```java
.andExpect(content().contentType(MediaType.APPLICATION_JSON))
.andExpect(content().contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
.andExpect(content().string(containsString("PENDING")))
.andExpect(content().json("{\"status\":\"PENDING\"}"))
```## ヘッダー アサーション```java
.andExpect(header().string("Location", "/orders/1"))
.andExpect(header().string("Content-Type", containsString("application/json")))
.andExpect(header().exists("X-Request-Id"))
.andExpect(header().doesNotExist("X-Deprecated"))
```## リクエストのパラメータとヘッダー```java
// Query parameters
mvc.perform(get("/orders").param("status", "PENDING").param("page", "0"))
  .andExpect(status().isOk());

// Path variables
mvc.perform(get("/orders/{id}", 1L))
  .andExpect(status().isOk());

// Request headers
mvc.perform(get("/orders/1").header("X-Api-Key", "secret"))
  .andExpect(status().isOk());
```## 応答のキャプチャ```java
@Test
void shouldReturnCreatedId() throws Exception {
  given(orderService.create(any())).willReturn(42L);

  MvcResult result = mvc.perform(post("/orders")
      .contentType(MediaType.APPLICATION_JSON)
      .content("{\"product\": \"Laptop\", \"quantity\": 1}"))
    .andExpect(status().isCreated())
    .andReturn();

  String location = result.getResponse().getHeader("Location");
  assertThat(location).isEqualTo("/orders/42");
}
```## andDo によるチェーン```java
mvc.perform(get("/orders/1"))
  .andDo(print())              // prints request/response to console (debug)
  .andExpect(status().isOk());
```## 静的インポート```java
import org.springframework.boot.test.mock.mockito.MockBean;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.hamcrest.Matchers.*;
```## MockMvcTester への移行

|クラシックモックMVC | MockMvcTester (推奨) |
| --- | --- |
| `@Autowired MockMvc mvc` | `@Autowired MockMvcTester mvc` |
| `mvc.perform(get("/orders/1"))` | `mvc.get().uri("/orders/1")` |
| `.andExpect(status().isOk())` | `.hasStatusOk()` |
| `.andExpect(jsonPath("$.status").value("X"))` | `.bodyJson().convertTo(T.class)` + AssertJ |
|すべてのメソッドの `throws Exception` |チェックされた例外はありません |
|ハムクレストマッチャー | AssertJ の流暢なアサーション |

完全な最新 API については、[mockmvc-tester.md](mockmvc-tester.md) を参照してください。

## 重要なポイント

1. **すべてのテスト メソッドは `throws Exception`** を宣言する必要があります — `perform()` はチェックされた例外をスローします
2. **デバッグ中に `andDo(print())` を使用します** — コミットする前に削除してください
3. **`content().string()`** よりも `jsonPath()` を優先します — より正確なフィールドレベルのアサーション
4. **静的インポートが必要です** — IDE は静的インポートを自動追加できます
5. Spring Boot 3.2 以降にアップグレードする場合は、可読性を高めるために **MockMvcTester** に移行してください。