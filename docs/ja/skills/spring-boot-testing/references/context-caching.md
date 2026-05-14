# コンテキストのキャッシュ

コンテキスト キャッシュを通じて Spring Boot テスト スイートのパフォーマンスを最適化します。

## コンテキスト キャッシュの仕組み

Spring の TestContext フレームワークは、構成「キー」に基づいてアプリケーション コンテキストをキャッシュします。同一の構成のテストでは、同じコンテキストが再利用されます。

### キャッシュキーに影響するもの

- @ContextConfiguration
- @TestPropertySource
- @ActiveProfiles
- @WebAppConfiguration
- @MockitoBean の定義
- @TestConfiguration インポート

## キャッシュキーの例

### 同じキー (コンテキストの再利用)```java
@WebMvcTest(OrderController.class)
class OrderControllerTest1 {
  @MockitoBean private OrderService orderService;
}

@WebMvcTest(OrderController.class)
class OrderControllerTest2 {
  @MockitoBean private OrderService orderService;
}
// Same context reused
```### 異なるキー (新しいコンテキスト)```java
@WebMvcTest(OrderController.class)
@ActiveProfiles("test")
class OrderControllerTest1 { }

@WebMvcTest(OrderController.class)
@ActiveProfiles("integration")
class OrderControllerTest2 { }
// Different contexts loaded
```## キャッシュ統計の表示

### スプリングブーツアクチュエーター```yaml
management:
  endpoints:
    web:
      exposure:
        include: metrics
```アクセス：`GET /actuator/metrics/spring.test.context.cache`

### デバッグログ```properties
logging.level.org.springframework.test.context.cache=DEBUG
```## キャッシュ ヒット率の最適化

### 構成ごとにテストをグループ化する```
 tests/
   unit/           # No context
   web/            # @WebMvcTest
   repository/     # @DataJpaTest  
   integration/    # @SpringBootTest
```### @TestPropertySource のバリエーションを最小限に抑える

**悪い (複数のコンテキスト):**```java
@TestPropertySource(properties = "app.feature-x=true")
class FeatureXTest { }

@TestPropertySource(properties = "app.feature-y=true")
class FeatureYTest { }
```**より良い (グループ化):**```java
@TestPropertySource(properties = {"app.feature-x=true", "app.feature-y=true"})
class FeaturesTest { }
```### @DirtiesContext は慎重に使用してください

コンテキストの状態が実際に変化した場合のみ:```java
@Test
@DirtiesContext // Forces context rebuild after test
void testThatModifiesBeanDefinitions() { }
```## ベストプラクティス

1. **構成ごとにグループ化** - 同じ構成のテストをまとめて保持します
2. **プロパティの変動を制限する** - 個々のプロパティに対してプロファイルを使用する
3. **@DirtiesContext は避けてください** - テスト データのクリーンアップを優先します
4. **狭いスライスを使用する** - @WebMvcTest と @SpringBootTest
5. **キャッシュ ヒットを監視** - デバッグ ログを時々有効にします