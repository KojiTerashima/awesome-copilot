# @RestClientTest

MockRestServiceServer を使用して REST クライアントを分離してテストします。

## 概要

`@RestClientTest` は自動構成します:

- モックサーバーをサポートする RestTemplate/RestClient
- ジャクソンオブジェクトマッパー
- モックレストサービスサーバー

## 基本セットアップ```java
@RestClientTest(WeatherService.class)
class WeatherServiceTest {
  
  @Autowired
  private WeatherService weatherService;
  
  @Autowired
  private MockRestServiceServer server;
}
```## RestTemplate のテスト```java
@RestClientTest(WeatherService.class)
class WeatherServiceTest {
  
  @Autowired
  private WeatherService weatherService;
  
  @Autowired
  private MockRestServiceServer server;
  
  @Test
  void shouldFetchWeather() {
    // Given
    server.expect(requestTo("https://api.weather.com/v1/current"))
      .andExpect(method(HttpMethod.GET))
      .andExpect(queryParam("city", "Berlin"))
      .andRespond(withSuccess()
        .contentType(MediaType.APPLICATION_JSON)
        .body("{\"temperature\": 22, \"condition\": \"Sunny\"}"));
    
    // When
    Weather weather = weatherService.getCurrentWeather("Berlin");
    
    // Then
    assertThat(weather.getTemperature()).isEqualTo(22);
    assertThat(weather.getCondition()).isEqualTo("Sunny");
  }
}
```## RestClient のテスト (Spring 6.1 以降)```java
@RestClientTest(WeatherService.class)
class WeatherServiceTest {
  
  @Autowired
  private WeatherService weatherService;
  
  @Autowired
  private MockRestServiceServer server;
  
  @Test
  void shouldFetchWeatherWithRestClient() {
    server.expect(requestTo("https://api.weather.com/v1/current"))
      .andRespond(withSuccess()
        .body("{\"temperature\": 22}"));
    
    Weather weather = weatherService.getCurrentWeather("Berlin");
    
    assertThat(weather.getTemperature()).isEqualTo(22);
  }
}
```## リクエストのマッチング

### 正確な URL```java
server.expect(requestTo("https://api.example.com/users/1"))
  .andRespond(withSuccess());
```### URL パターン```java
server.expect(requestTo(matchesPattern("https://api.example.com/users/\\d+")))
  .andRespond(withSuccess());
```### HTTP メソッド```java
server.expect(ExpectedCount.once(), 
  requestTo("https://api.example.com/users"))
  .andExpect(method(HttpMethod.POST))
  .andRespond(withCreatedEntity(URI.create("/users/1")));
```### リクエスト本文```java
server.expect(requestTo("https://api.example.com/users"))
  .andExpect(content().contentType(MediaType.APPLICATION_JSON))
  .andExpect(content().json("{\"name\": \"John\"}"))
  .andRespond(withSuccess());
```### ヘッダー```java
server.expect(requestTo("https://api.example.com/users"))
  .andExpect(header("Authorization", "Bearer token123"))
  .andExpect(header("X-Api-Key", "secret"))
  .andRespond(withSuccess());
```## 応答タイプ

### 体を使って成功する```java
server.expect(requestTo("/users/1"))
  .andRespond(withSuccess()
    .contentType(MediaType.APPLICATION_JSON)
    .body("{\"id\": 1, \"name\": \"John\"}"));
```### リソースからの成功```java
server.expect(requestTo("/users/1"))
  .andRespond(withSuccess()
    .body(new ClassPathResource("user-response.json")));
```### 作成されました```java
server.expect(requestTo("/users"))
  .andExpect(method(HttpMethod.POST))
  .andRespond(withCreatedEntity(URI.create("/users/1")));
```### エラー応答```java
server.expect(requestTo("/users/999"))
  .andRespond(withResourceNotFound());

server.expect(requestTo("/users"))
  .andRespond(withServerError()
    .body("Internal Server Error"));

server.expect(requestTo("/users"))
  .andRespond(withStatus(HttpStatus.BAD_REQUEST)
    .body("{\"error\": \"Invalid input\"}"));
```## リクエストの検証```java
@Test
void shouldCallApi() {
  server.expect(ExpectedCount.once(), 
    requestTo("https://api.example.com/data"))
    .andRespond(withSuccess());
  
  service.fetchData();
  
  server.verify(); // Verify all expectations met
}
```## 余分なリクエストを無視する```java
@Test
void shouldHandleMultipleCalls() {
  server.expect(ExpectedCount.manyTimes(),
    requestTo(matchesPattern("/api/.*")))
    .andRespond(withSuccess());
  
  // Multiple calls allowed
  service.callApi();
  service.callApi();
  service.callApi();
}
```## テスト間でリセット```java
@BeforeEach
void setUp() {
  server.reset();
}
```## テストのタイムアウト```java
server.expect(requestTo("/slow-endpoint"))
  .andRespond(withSuccess()
    .body("{\"data\": \"test\"}")
    .delay(100, TimeUnit.MILLISECONDS));

// Test timeout handling
```## ベストプラクティス

1. テストの最後に `server.verify()` を必ず確認します
2. 大規模な JSON 応答にはリソース ファイルを使用する
3. リクエスト属性の最小限のセットに基づいて照合する
4. @BeforeEach でサーバーをリセットする
5. 成功だけでなくエラー応答をテストする
6. POST/PUT 呼び出しのリクエスト本文を確認する