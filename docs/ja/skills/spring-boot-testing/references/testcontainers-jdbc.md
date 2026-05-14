# テストコンテナ JDBC

Testcontainer を使用した実際のデータベースでの JPA リポジトリのテスト。

## 概要

Testcontainers は、統合テスト用に実際のデータベース インスタンスを Docker コンテナ内に提供します。実稼働同等性に関しては H2 よりも信頼性が高くなります。

## PostgreSQL のセットアップ

### 依存関係```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-testcontainers</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>testcontainers-postgresql</artifactId>
  <scope>test</scope>
</dependency>
```### 基本テスト```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryPostgresTest {
  
  @Container
  @ServiceConnection
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
  
  @Autowired
  private OrderRepository orderRepository;
  
  @Autowired
  private TestEntityManager entityManager;
}
```## MySQL のセットアップ```xml
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>testcontainers-mysql</artifactId>
  <scope>test</scope>
</dependency>
```

```java
@Container
@ServiceConnection
static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.4");
```## 複数のデータベース```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class MultiDatabaseTest {
  
  @Container
  @ServiceConnection(name = "primary")
  static PostgreSQLContainer<?> primaryDb = new PostgreSQLContainer<>("postgres:18");
  
  @Container
  @ServiceConnection(name = "analytics")
  static PostgreSQLContainer<?> analyticsDb = new PostgreSQLContainer<>("postgres:18");
}
```## コンテナの再利用 (速度の最適化)

`~/.testcontainers.properties` に追加:```properties
testcontainers.reuse.enable=true
```次に、コード内での再利用を有効にします。```java
@Container
@ServiceConnection
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18")
  .withReuse(true);
```## データベースの初期化

### SQL スクリプトを使用する場合```java
@Container
@ServiceConnection
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18")
  .withInitScript("schema.sql");
```### フライウェイ付き```java
@SpringBootTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class MigrationTest {
  
  @Container
  @ServiceConnection
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
  
  @Autowired
  private Flyway flyway;
  
  @Test
  void shouldApplyMigrations() {
    flyway.migrate();
    // Test code
  }
}
```## 高度な構成

### カスタム データベース/スキーマ```java
@Container
@ServiceConnection
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18")
  .withDatabaseName("testdb")
  .withUsername("testuser")
  .withPassword("testpass")
  .withInitScript("init-schema.sql");
```### 待機戦略```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18")
  .waitingFor(Wait.forLogMessage(".*database system is ready.*", 1));
```## テスト例```java
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
  
  @Test
  void shouldFindOrdersByStatus() {
    // Given
    entityManager.persist(new Order("PENDING"));
    entityManager.persist(new Order("COMPLETED"));
    entityManager.flush();
    
    // When
    List<Order> pending = orderRepository.findByStatus("PENDING");
    
    // Then
    assertThat(pending).hasSize(1);
    assertThat(pending.get(0).getStatus()).isEqualTo("PENDING");
  }
  
  @Test
  void shouldSupportPostgresSpecificFeatures() {
    // Can use Postgres-specific features like:
    // - JSONB columns
    // - Array types
    // - Full-text search
  }
}
```## @DynamicPropertySource の代替

@ServiceConnection を使用しない場合:```java
@SpringBootTest
@Testcontainers
class OrderServiceTest {
  
  @Container
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18");
  
  @DynamicPropertySource
  static void configureProperties(DynamicPropertyRegistry registry) {
    registry.add("spring.datasource.url", postgres::getJdbcUrl);
    registry.add("spring.datasource.username", postgres::getUsername);
    registry.add("spring.datasource.password", postgres::getPassword);
  }
}
```## サポートされているデータベース

|データベース |コンテナクラス | Maven アーティファクト |
| -------- | --------------- | -------------- |
|ポストグレSQL | PostgreSQLコンテナ |テストコンテナ-postgresql |
| MySQL | MySQLコンテナ |テストコンテナ-mysql |
|マリアDB |マリアDBコンテナ | testcontainers-mariadb |
| SQLサーバー | MSSQLサーバーコンテナ |テストコンテナ-mssqlserver |
|オラクル | Oracleコンテナ | testcontainers-oracle-free |
|モンゴDB | MongoDBコンテナ |テストコンテナ-mongodb |

## ベストプラクティス

1. 可能な場合は @ServiceConnection を使用します (Spring Boot 3.1 以降)
2. ローカルビルドを高速化するためにコンテナーの再利用を有効にする
3. 最新ではない特定のバージョン (postgres:18) を使用する
4. コンテナー構成を静的フィールドに保持します
5. @DataJpaTest を AutoConfigureTestDatabase.Replace.NONE とともに使用する