---
description: 'Spring Boot JPA アプリケーションを変換して Spring Data Cosmos で Azure Cosmos DB を使用するためのステップバイステップガイド'
applyTo: '**/*.java,**/pom.xml,**/build.gradle,**/application*.properties'
---

# Spring JPA プロジェクトを Spring Data Cosmos に変換する

この一般化されたガイドは、JPA から Spring Data Cosmos DB への変換プロジェクトに適用されます。

## 上位計画

1. ビルドの依存関係を交換します (JPA を削除し、Cosmos + Identity を追加します)。
2. `cosmos` プロフィールとプロパティを追加します。
3. 適切な Azure ID 認証を使用して Cosmos 構成を追加します。
4. エンティティを変換します (ID → `String`、`@Container` と `@PartitionKey` を追加、JPA マッピングを削除、関係を調整)。
5. リポジトリを変換します (`JpaRepository` → `CosmosRepository`)。
6. **関係管理とテンプレートの互換性のためにサービスレイヤーを作成します**。
7. **重要**: 文字列 ID と Cosmos リポジトリで動作するようにすべてのテストファイルを更新します。
8. `CommandLineRunner` 経由でデータをシードします。
9. **重要**: ランタイム機能をテストし、テンプレートの互換性の問題を修正します。

## 段階的に

### ステップ 1 — 依存関係を構築する

- **Maven** (`pom.xml`):
  - 依存関係を削除 `spring-boot-starter-data-jpa`
  - 他の場所で必要な場合を除き、データベース固有の依存関係 (H2、MySQL、PostgreSQL) を削除します。
  - `com.azure:azure-spring-data-cosmos:5.17.0` (または互換性のある最新バージョン) を追加します
  - `com.azure:azure-identity:1.15.4` を追加します (DefaultAzureCredential に必須)
- **Gradle**: Gradle 構文に同じ依存関係の変更を適用します。
- テストコンテナと JPA 固有のテスト依存関係を削除する

### ステップ 2 — プロパティと構成

- `src/main/resources/application-cosmos.properties` を作成します:
```properties
  azure.cosmos.uri=${COSMOS_URI:https://localhost:8081}
  azure.cosmos.database=${COSMOS_DATABASE:petclinic}
  azure.cosmos.populate-query-metrics=false
  azure.cosmos.enable-multiple-write-locations=false
  ```
- `src/main/resources/application.properties` を更新:
```properties
  spring.profiles.active=cosmos
  ```

### ステップ 3 — Azure ID を使用した構成クラス

- `src/main/java/<rootpkg>/config/CosmosConfiguration.java` を作成します:
```java
  @Configuration
  @EnableCosmosRepositories(basePackages = "<rootpkg>")
  public class CosmosConfiguration extends AbstractCosmosConfiguration {

    @Value("${azure.cosmos.uri}")
    private String uri;

    @Value("${azure.cosmos.database}")
    private String dbName;

    @Bean
    public CosmosClientBuilder getCosmosClientBuilder() {
      return new CosmosClientBuilder().endpoint(uri).credential(new DefaultAzureCredentialBuilder().build());
    }

    @Override
    protected String getDatabaseName() {
      return dbName;
    }

    @Bean
    public CosmosConfig cosmosConfig() {
      return CosmosConfig.builder().enableQueryMetrics(false).build();
    }
  }

  ```
- **重要**: 本番環境のセキュリティにはキーベースの認証の代わりに `DefaultAzureCredentialBuilder().build()` を使用してください

### ステップ 4 — エンティティの変換

- JPA アノテーション (`@Entity`、`@MappedSuperclass`、`@Embeddable`) を持つすべてのクラスを対象とします。
- **基本エンティティの変更**:
  - `id` フィールドタイプを `Integer` から `String` に変更します
  - `@Id` および `@GeneratedValue` 注釈を追加する
  - `@PartitionKey` フィールドを追加します (通常は `String partitionKey`)
  - すべての `jakarta.persistence` インポートを削除します
- **重要 - Cosmos DB シリアル化要件**:
  - **Cosmos DB に永続化する必要があるフィールドからすべての `@JsonIgnore` 注釈を削除します**
  - **認証エンティティ (ユーザー、権限) は完全にシリアル化可能である必要があります** - パスワード、権限、またはその他の永続化フィールドに `@JsonIgnore` は使用できません
  - **JSON フィールド名を制御しながらデータを保持する必要がある場合は、`@JsonIgnore`** の代わりに `@JsonProperty` を使用してください
  - **一般的な認証シリアル化エラー**: `Cannot pass null or empty values to constructor` は通常、`@JsonIgnore` が必須フィールドのシリアル化をブロックしていることを意味します
- **エンティティ固有の変更**:
  - `@Entity` を `@Container(containerName = "<plural-entity-name>")` に置き換えます
  - `@Table`、`@Column`、`@JoinColumn` などを削除します。
  - 関係注釈の削除 (`@OneToMany`、`@ManyToOne`、`@ManyToMany`)
  - 人間関係の場合:
    - 1 対多のコレクションを埋め込む (例: Owner の `List<Pet> pets`)
    - 多対 1 の参照 ID を使用します (例: Pet の `String ownerId`)
    - **複雑な関係の場合**: ID を保存しますが、テンプレートの一時的なプロパティを追加します
  - パーティションキーを設定するコンストラクターを追加します: `setPartitionKey("entityType")`
- **重要 - 認証エンティティパターン**:
  - **Spring Security を使用するユーザーエンティティの場合**: 権限を `Set<Authority>` オブジェクトではなく `Set<String>` として保存します
  - **ユーザーエンティティ変換の例**:
```java
    @Container(containerName = "users")
    public class User {

      @Id
      private String id;

      @PartitionKey
      private String partitionKey = "user";

      private String login;
      private String password; // NO @JsonIgnore - must be serializable

      @JsonProperty("authorities") // Use @JsonProperty, not @JsonIgnore
      private Set<String> authorities = new HashSet<>(); // Store as strings

      // Add transient property for Spring Security compatibility if needed
      // @JsonIgnore - ONLY for transient properties not persisted to Cosmos
      private Set<Authority> authorityObjects = new HashSet<>();

      // Conversion methods between string authorities and Authority objects
      public void setAuthorityObjects(Set<Authority> authorities) {
        this.authorityObjects = authorities;
        this.authorities = authorities.stream().map(Authority::getName).collect(Collectors.toSet());
      }
    }

    ```
- **重要 - 関係変更に対するテンプレートの互換性**:
  - **関係を ID 参照に変換する場合は、テンプレートアクセスを保持します**
  - **例**: エンティティに `List<Specialty> specialties` がある場合 → 次のように変換します。
    - ストレージ: `List<String> specialtyIds` (Cosmos に永続化)
    - テンプレート: `@JsonIgnore private List<Specialty> specialties = new ArrayList<>()` (一時的)
    - 両方のプロパティにゲッター/セッターを追加します
  - **エンティティメソッド ロジックの更新**: `getNrOfSpecialties()` は一時リストを使用する必要があります
- **重要 - Thymeleaf/JSP アプリケーションのテンプレートの互換性**:
  - **テンプレートプロパティ アクセスの特定**: `.html` ファイル内の `${entity.relationshipProperty}` を検索します
  - **テンプレートでアクセスされる各関係プロパティ**:
    - **ストレージ**: ID ベースのストレージを保持します (例: `List<String> specialtyIds`)
    - **テンプレートアクセス**: `@JsonIgnore` を使用して一時プロパティを追加します (例: `private List<Specialty> specialties = new ArrayList<>()`)
    - **例**：

```java
      // Stored in Cosmos (persisted)
      private List<String> specialtyIds = new ArrayList<>();

      // For template access (transient)
      @JsonIgnore
      private List<Specialty> specialties = new ArrayList<>();

      // Getters/setters for both properties
      public List<String> getSpecialtyIds() {
        return specialtyIds;
      }

      public List<Specialty> getSpecialties() {
        return specialties;
      }

      ```

    - **カウント方法の更新**: `getNrOfSpecialties()` は ID リストではなく一時リストを使用する必要があります
- **重大 - メソッド署名の競合**:
  - **ID タイプを整数から文字列に変換する場合は、メソッドシグネチャの競合を確認してください**
  - **一般的な競合**: `getPet(String name)` 対 `getPet(String id)` - 両方とも同じ署名を持っています
  - **解決策**: メソッドの名前を具体的なものに変更します。
    - `getPet(String id)` (ID ベースの検索)
    - `getPetByName(String name)` (名前ベースの検索)
    - `getPetByName(String name, boolean ignoreNew)` 条件付き名前ベースの検索
  - コントローラーとテスト内の名前変更されたメソッドの **すべての呼び出し元を更新**
- **エンティティのメソッドの更新**:
  - `addVisit(Integer petId, Visit visit)` を `addVisit(String petId, Visit visit)` に更新します
  - すべての ID 比較ロジックが `==` ではなく `.equals()` を使用するようにします。

### ステップ 5 — リポジトリの変換

- すべてのリポジトリインターフェイスを変更します。
  - 送信者: `extends JpaRepository<Entity, Integer>`
  - 宛先: `extends CosmosRepository<Entity, String>`
- **クエリメソッドの更新**:
  - カスタムクエリからページネーションパラメータを削除する
  - `Page<Entity> findByX(String param, Pageable pageable)` を `List<Entity> findByX(String param)` に変更します
  - Cosmos SQL 構文を使用するように `@Query` 注釈を更新します
  - **カスタムメソッド名を置換**: `findPetTypes()` → `findAllOrderByName()`
  - **すべての参照を更新**して、コントローラーとフォーマッタの変更されたメソッド名を反映します

### ステップ 6 — 関係管理とテンプレートの互換性のための **サービスレイヤーの作成**

- **重要**: Cosmos ドキュメントストレージと既存のテンプレートの期待を橋渡しするサービスクラスを作成します。
- **目的**: 関係の母集団を処理し、テンプレートの互換性を維持します。
- **関係を持つ各エンティティのサービスパターン**:
```java
  @Service
  public class EntityService {

    private final EntityRepository entityRepository;
    private final RelatedRepository relatedRepository;

    public EntityService(EntityRepository entityRepository, RelatedRepository relatedRepository) {
      this.entityRepository = entityRepository;
      this.relatedRepository = relatedRepository;
    }

    public List<Entity> findAll() {
      List<Entity> entities = entityRepository.findAll();
      entities.forEach(this::populateRelationships);
      return entities;
    }

    public Optional<Entity> findById(String id) {
      Optional<Entity> entityOpt = entityRepository.findById(id);
      if (entityOpt.isPresent()) {
        Entity entity = entityOpt.get();
        populateRelationships(entity);
        return Optional.of(entity);
      }
      return Optional.empty();
    }

    private void populateRelationships(Entity entity) {
      if (entity.getRelatedIds() != null && !entity.getRelatedIds().isEmpty()) {
        List<Related> related = entity
          .getRelatedIds()
          .stream()
          .map(relatedRepository::findById)
          .filter(Optional::isPresent)
          .map(Optional::get)
          .collect(Collectors.toList());
        // Set transient property for template access
        entity.setRelated(related);
      }
    }
  }

  ```

### ステップ 6.5 — **Spring Security 統合** (認証に重要)

- **UserDetailsS​​ervice 統合パターン**:
```java
  @Service
  @Transactional
  public class DomainUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;
    private final AuthorityRepository authorityRepository;

    @Override
    public UserDetails loadUserByUsername(String login) {
      log.debug("Authenticating user: {}", login);

      return userRepository
        .findOneByLogin(login)
        .map(user -> createSpringSecurityUser(login, user))
        .orElseThrow(() -> new UsernameNotFoundException("User " + login + " was not found"));
    }

    private org.springframework.security.core.userdetails.User createSpringSecurityUser(String lowercaseLogin, User user) {
      if (!user.isActivated()) {
        throw new UserNotActivatedException("User " + lowercaseLogin + " was not activated");
      }

      // Convert string authorities back to GrantedAuthority objects
      List<GrantedAuthority> grantedAuthorities = user
        .getAuthorities()
        .stream()
        .map(SimpleGrantedAuthority::new)
        .collect(Collectors.toList());

      return new org.springframework.security.core.userdetails.User(user.getLogin(), user.getPassword(), grantedAuthorities);
    }
  }

  ```
- **主要な認証要件**:
  - ユーザーエンティティは完全にシリアル化可能である必要があります (パスワード/権限に `@JsonIgnore` はありません)
  - Cosmos DB との互換性のために権限を `Set<String>` として保存します
  - UserDetailsS​​ervice の文字列権限と `GrantedAuthority` オブジェクトの間の変換
  - 認証フローをトレースするための包括的なデバッグログを追加します。
  - アクティブ化/非アクティブ化されたユーザー状態を適切に処理する

#### **テンプレート関係人口パターン**

テンプレートレンダリング用のエンティティを返す各サービスメソッドには、一時プロパティを設定する必要があります。

```java
private void populateRelationships(Entity entity) {
  // For each relationship used in templates
  if (entity.getRelatedIds() != null && !entity.getRelatedIds().isEmpty()) {
    List<Related> relatedObjects = entity
      .getRelatedIds()
      .stream()
      .map(relatedRepository::findById)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .collect(Collectors.toList());
    entity.setRelated(relatedObjects); // Set transient property
  }
}

```

#### **コントローラーでの重要なサービスの使用法**

- **すべての直接リポジトリ呼び出し**をコントローラーのサービス呼び出しに置き換えます
- **エンティティをリポジトリから直接テンプレートに返さないでください**
- **コントローラを更新**して、リポジトリを直接使用する代わりにサービス層を使用します
- **コントローラーパターンの変更**:

```java
  // OLD: Direct repository usage
  @Autowired
  private EntityRepository entityRepository;

  // NEW: Service layer usage
  @Autowired
  private EntityService entityService;
  // Update method calls
  // OLD: entityRepository.findAll()
  // NEW: entityService.findAll()

  ```

### ステップ 7 — データのシード処理

- `CommandLineRunner` を実装した `@Component` を作成します。
```java
  @Component
  public class DataSeeder implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
      if (ownerRepository.count() > 0) {
        return; // Data already exists
      }
      // Seed comprehensive test data with String IDs
      // Use meaningful ID patterns: "owner-1", "pet-1", "pettype-1", etc.
    }
  }

  ```
- **重大 - JDK 17+ での BigDecimal リフレクションの問題**:
  - **BigDecimal フィールドを使用している場合**、シード中にリフレクションエラーが発生する可能性があります
  - **エラーパターン**: `Unable to make field private final java.math.BigInteger java.math.BigDecimal.intVal accessible`
  - **解決策**:
    1. 通貨値には `BigDecimal` の代わりに `Double` または `String` を使用してください
    2. JVM 引数を追加: `--add-opens java.base/java.math=ALL-UNNAMED`
    3. BigDecimal 操作を try-catch でラップし、適切に処理します
  - **シードが失敗してもアプリケーションは正常に起動します** - シードエラーのログを確認してください

### ステップ 8 — ファイル変換のテスト (クリティカルセクション)

**このステップは見落とされがちですが、変換を成功させるためには不可欠です**

#### A. **コンパイルチェック戦略**

- **大きな変更を加えるたびに、`mvn test-compile` を実行して問題を早期に発見してください**
- **続行する前に体系的にコンパイルエラーを修正してください**
- **IDE に依存しないでください - Maven のコンパイルですべての問題が明らかになります**

#### B. **すべてのテストファイルを体系的に検索して更新します**

**検索ツールを使用して、すべての出現箇所を検索して更新します:**

- 検索: `int.*TEST.*ID` → 置換: `String.*TEST.*ID = "test-xyz-1"`
- 検索: `setId\(\d+\)` → 置換: `setId("test-id-X")`
- 検索: `findById\(\d+\)` → 置換: `findById("test-id-X")`
- 検索: `\.findPetTypes\(\)` → 置換: `.findAllOrderByName()`
- 検索: `\.findByLastNameStartingWith\(.*,.*Pageable` → ページネーションパラメータを削除

#### C. テストの注釈とインポートを更新する

- `@DataJpaTest` を `@SpringBootTest` または適切なスライステストに置き換えます。
- `@AutoConfigureTestDatabase` 注釈を削除する
- `@Transactional` をテストから削除します (単一パーティション操作を除く)
- `org.springframework.orm` パッケージからインポートを削除

#### D. すべてのテストファイルでのエンティティ ID の使用を修正する

**更新する必要がある重要なファイル (テストディレクトリ全体を検索):**

- `*ControllerTests.java` - パス変数、エンティティの作成、モックセットアップ
- `*ServiceTests.java` - リポジトリの操作、エンティティ ID
- `EntityUtils.java` - ID 処理のためのユーティリティメソッド
- `*FormatterTests.java` - リポジトリメソッド呼び出し
- `*ValidatorTests.java` - 文字列 ID を使用したエンティティの作成
- 統合テストクラス - テストデータのセットアップ

#### E. **リポジトリの変更によって影響を受けるコントローラーおよびサービスクラスを修正**

- **変更されたシグネチャを使用してリポジトリメソッドを呼び出すコントローラーを更新します**
- **リポジトリメソッドを使用するフォーマッタ/コンバータを更新します**
- **チェックすべき一般的なファイル**:
  - `PetTypeFormatter.java` - `findPetTypes()` メソッドを頻繁に呼び出します
  - `*Controller.java` - 削除するページネーションロジックがある可能性があります
  - リポジトリメソッドを使用するサービスクラス

#### F. テストでのリポジトリモックの更新

- リポジトリモックからページネーションを削除します。
  - `given(repository.findByX(param, pageable)).willReturn(pageResult)`
  - →`given(repository.findByX(param)).willReturn(listResult)`
- モック内のメソッド名を更新します。
  - `given(petTypeRepository.findPetTypes()).willReturn(types)`
  - →`given(petTypeRepository.findAllOrderByName()).willReturn(types)`

#### G. テストで使用されるユーティリティクラスを修正する

- `EntityUtils.java` などを更新します。
  - JPA 固有の例外インポートを削除 (`ObjectRetrievalFailureException`)
  - メソッドのシグネチャを `int id` から `String id` に変更します
  - 更新ID比較ロジック：`entity.getId() == entityId` → `entity.getId().equals(entityId)`
  - JPA 例外を標準例外に置き換えます (`IllegalArgumentException`)

#### H. 文字列 ID のアサーションを更新する

- 変更 ID アサーション:
  - `assertThat(entity.getId()).isNotZero()` → `assertThat(entity.getId()).isNotEmpty()`
  - `assertThat(entity.getId()).isEqualTo(1)` → `assertThat(entity.getId()).isEqualTo("test-id-1")`
  - JSON パスアサーション: `jsonPath("$.id").value(1)` → `jsonPath("$.id").value("test-id-1")`

### ステップ 8 — ファイル変換のテスト (クリティカルセクション)

**このステップは見落とされがちですが、変換を成功させるためには不可欠です**

#### A. **コンパイルチェック戦略**

- **大きな変更を加えるたびに、`mvn test-compile` を実行して問題を早期に発見してください**
- **続行する前に体系的にコンパイルエラーを修正してください**
- **IDE に依存しないでください - Maven のコンパイルですべての問題が明らかになります**

#### B. **すべてのテストファイルを体系的に検索して更新します**

**検索ツールを使用して、すべての出現箇所を検索して更新します:**

- 検索: `setId\(\d+\)` → 置換: `setId("test-id-X")`
- 検索: `findById\(\d+\)` → 置換: `findById("test-id-X")`
- 検索: `\.findPetTypes\(\)` → 置換: `.findAllOrderByName()`
- 検索: `\.findByLastNameStartingWith\(.*,.*Pageable` → ページネーションパラメータを削除

#### C. テストの注釈とインポートを更新する

- `@DataJpaTest` を `@SpringBootTest` または適切なスライステストに置き換えます。
- `@AutoConfigureTestDatabase` 注釈を削除する
- `@Transactional` をテストから削除します (単一パーティション操作を除く)
- `org.springframework.orm` パッケージからインポートを削除

#### D. すべてのテストファイルでのエンティティ ID の使用を修正する

**更新する必要がある重要なファイル (テストディレクトリ全体を検索):**

- `*ControllerTests.java` - パス変数、エンティティの作成、モックセットアップ
- `*ServiceTests.java` - リポジトリの操作、エンティティ ID
- `EntityUtils.java` - ID 処理のためのユーティリティメソッド
- `*FormatterTests.java` - リポジトリメソッド呼び出し
- `*ValidatorTests.java` - 文字列 ID を使用したエンティティの作成
- 統合テストクラス - テストデータのセットアップ

#### E. **リポジトリの変更によって影響を受けるコントローラーおよびサービスクラスを修正**

- **変更されたシグネチャを使用してリポジトリメソッドを呼び出すコントローラーを更新します**
- **リポジトリメソッドを使用するフォーマッタ/コンバータを更新します**
- **チェックすべき一般的なファイル**:
  - `PetTypeFormatter.java` - `findPetTypes()` メソッドを頻繁に呼び出します
  - `*Controller.java` - 削除するページネーションロジックがある可能性があります
  - リポジトリメソッドを使用するサービスクラス

#### F. テストでのリポジトリモックの更新

- リポジトリモックからページネーションを削除します。
  - `given(repository.findByX(param, pageable)).willReturn(pageResult)`
  - →`given(repository.findByX(param)).willReturn(listResult)`
- モック内のメソッド名を更新します。
  - `given(petTypeRepository.findPetTypes()).willReturn(types)`
  - →`given(petTypeRepository.findAllOrderByName()).willReturn(types)`

#### G. テストで使用されるユーティリティクラスを修正する

- `EntityUtils.java` などを更新します。
  - JPA 固有の例外インポートを削除 (`ObjectRetrievalFailureException`)
  - メソッドのシグネチャを `int id` から `String id` に変更します
  - 更新ID比較ロジック：`entity.getId() == entityId` → `entity.getId().equals(entityId)`
  - JPA 例外を標準例外に置き換えます (`IllegalArgumentException`)

#### H. 文字列 ID のアサーションを更新する

- 変更 ID アサーション:
  - `assertThat(entity.getId()).isNotZero()` → `assertThat(entity.getId()).isNotEmpty()`
  - `assertThat(entity.getId()).isEqualTo(1)` → `assertThat(entity.getId()).isEqualTo("test-id-1")`
  - JSON パスアサーション: `jsonPath("$.id").value(1)` → `jsonPath("$.id").value("test-id-1")`

### ステップ 9 — **実行時テストとテンプレートの互換性**

#### **重要**: コンパイルが成功した後、実行中のアプリケーションをテストします。

- **アプリケーションを起動**: `mvn spring-boot:run`
- Web インターフェイスの **すべてのページに移動**して、ランタイムエラーを特定します
- **変換後の一般的なランタイムの問題**:
  - テンプレートがもう存在しないプロパティにアクセスしようとしています (例: `vet.specialties`)
  - サービス層に一時的な関係プロパティが設定されていない
  - コントローラーがリレーションシップの読み込みにサービス層を使用しない

#### **テンプレート互換性の修正**:

- **テンプレートが関係プロパティにアクセスする場合** (例: `entity.relatedObjects`):
  - 適切なゲッター/セッターを使用して一時プロパティがエンティティに存在することを確認する
  - サービス層にこれらの一時プロパティが設定されていることを確認します
  - ID リストの代わりに一時リストを使用するように `getNrOfXXX()` メソッドを更新します。
- **ログで SpEL (Spring Expression Language) エラーを確認します**:
  - `Property or field 'xxx' cannot be found` → 欠落している一時プロパティを追加
  - `EL1008E` エラー → サービス層にリレーションシップが設定されていない

#### **サービス層の検証**:

- **すべてのコントローラーが直接リポジトリアクセスではなくサービスレイヤーを使用していることを確認します**
- **エンティティを返す前に、サービスメソッドがリレーションシップを設定していることを確認します**
- **Web インターフェイスを介してすべての CRUD 操作をテスト**

### ステップ 9.5 — **テンプレートのランタイム検証** (重要)

#### **体系的なテンプレートテスト プロセス**

コンパイルが成功し、アプリケーションが起動したら、次のようにします。

1. **アプリケーション内のすべてのページに体系的に移動**
2. **エンティティデータを表示する各テンプレートをテストします**:
   - リストページ (例: `/vets`、`/owners`)
   - 詳細ページ (例: `/owners/{id}`、`/vets/{id}`)
   - フォームと編集ページ
3. **特定のテンプレートエラーを探します**:
   - `Property or field 'relationshipName' cannot be found on object of type 'EntityName'`
   - `EL1008E` Spring 式言語のエラー
   - 関係が表示されるはずのデータが空であるか欠落している

#### **テンプレートエラー解決チェックリスト**

テンプレートエラーが発生した場合:

- [ ] **エラーメッセージから欠落しているプロパティを特定します**
- [ ] **プロパティがエンティティ内の一時フィールドとして存在するかどうかを確認します**
- [ ] **エンティティを返す前に、サービスレイヤーがプロパティに値を設定していることを確認します**
- [ ] **コントローラーが直接リポジトリアクセスではなく、サービスレイヤーを使用していることを確認します**
- [ ] 修正後、**特定のページを再度テストします**

#### **一般的なテンプレートエラー パターン**

- `Property or field 'specialties' cannot be found` → `@JsonIgnore private List<Specialty> specialties` を獣医エンティティに追加
- `Property or field 'pets' cannot be found` → `@JsonIgnore private List<Pet> pets` を所有者エンティティに追加
- 空の関係データが表示される → サービスが一時的なプロパティを設定しない

### ステップ 10 — **体系的なエラー解決プロセス**

#### コンパイルが失敗した場合:

1. **最初に `mvn compile` を実行します** - テスト前に主要なソースの問題を修正します
2. **`mvn test-compile`** を実行 - 各テストのコンパイルエラーを体系的に修正します
3. **最も頻繁に発生するエラーパターンに焦点を当てます**:
   - `int cannot be converted to String` → テスト定数とエンティティセッターを変更する
   - `method X cannot be applied to given types` → ページネーションパラメータを削除
   - `cannot find symbol: method Y()` → 新しいリポジトリメソッド名に更新
   - メソッド署名の競合 → 競合するメソッドの名前を変更

### ステップ 10 — **体系的なエラー解決プロセス**

#### コンパイルが失敗した場合:

1. **最初に `mvn compile` を実行します** - テスト前に主要なソースの問題を修正します
2. **`mvn test-compile`** を実行 - 各テストのコンパイルエラーを体系的に修正します
3. **最も頻繁に発生するエラーパターンに焦点を当てます**:
   - `int cannot be converted to String` → テスト定数とエンティティセッターを変更する
   - `method X cannot be applied to given types` → ページネーションパラメータを削除
   - `cannot find symbol: method Y()` → 新しいリポジトリメソッド名に更新
   - メソッド署名の競合 → 競合するメソッドの名前を変更
#### ランタイムが失敗した場合:

1. **特定のエラーメッセージについてはアプリケーションログを確認してください**
2. **テンプレート/SpEL エラーを探します**:
   - `Property or field 'xxx' cannot be found` → エンティティに一時プロパティを追加
   - 関係データが欠落している → サービス層に関係が設定されていない
3. コントローラーでの **サービスレイヤーの使用状況の確認**
4. **すべてのアプリケーションページのナビゲーションをテストします**

#### 一般的なエラーのパターンと解決策:

- **`method findByLastNameStartingWith cannot be applied`** → `Pageable` パラメータを削除
- **`cannot find symbol: method findPetTypes()`** → `findAllOrderByName()` に変更
- **`incompatible types: int cannot be converted to String`** → テスト ID 定数を更新
- **`method getPet(String) is already defined`** → 1 つのメソッドの名前を変更 (例: `getPetByName`)
- **`cannot find symbol: method isNotZero()`** → 文字列 ID の場合は `isNotEmpty()` に変更します
- **`Property or field 'specialties' cannot be found`** → 一時的なプロパティを追加し、サービスに設定します
- **`ClassCastException: reactor.core.publisher.BlockingIterable cannot be cast to java.util.List`** → StreamSupport を使用するようにリポジトリ `findAllWithEagerRelationships()` メソッドを修正
- **`Unable to make field...BigDecimal.intVal accessible`** → アプリケーション全体で BigDecimal を Double に置き換えます
- **ヘルスチェック データベースの障害** → ヘルスチェックの準備構成から「db」を削除します

#### **テンプレート固有のランタイムエラー**

- **`Property or field 'XXX' cannot be found on object of type 'YYY'`**:

  - 根本原因: ID ストレージに変換された関係プロパティにアクセスするテンプレート
  - 解決策: 一時プロパティをエンティティに追加し、サービス層に設定します
  - 予防策: リレーションシップを変換する前に、テンプレートの使用状況を必ず確認してください。

- **`EL1008E` Spring 式言語エラー**:

  - 根本原因: サービス層に一時的なプロパティが設定されていない
  - 解決策: `populateRelationships()` メソッドが呼び出され、動作していることを確認します。
  - 予防策: サービス層の実装後にすべてのテンプレートナビゲーションをテストします。

- **テンプレート内の空/null の関係データ**:
  - 根本原因: コントローラーがサービス層をバイパスしているか、サービスがリレーションシップを設定していません。
  - 解決策: すべてのコントローラーメソッドでエンティティの取得にサービス層を使用するようにします。
  - 予防策: リポジトリの結果をテンプレートに直接返さないでください。

### ステップ 11 — 検証チェックリスト

変換後、次のことを確認します。

- [ ] **メインアプリケーションのコンパイル**: `mvn compile` は成功します
- [ ] **すべてのテストファイルがコンパイルされます**: `mvn test-compile` は成功しました
- [ ] **コンパイルエラーなし**: あらゆるコンパイルエラーに対処します
- [ ] **アプリケーションは正常に開始します**: `mvn spring-boot:run` エラーなし
- [ ] **すべての Web ページの読み込み**: 実行時エラーなしですべてのアプリケーションページを移動します。
- [ ] **サービス層がリレーションシップを設定します**: 一時的なプロパティは正しく設定されています
- [ ] **すべてのテンプレートページはエラーなしで表示されます**: アプリケーション全体をナビゲートします
- [ ] **関係データが正しく表示される**: リスト、カウント、および関連オブジェクトが正しく表示されます。
- [ ] **ログに SpEL テンプレートエラーはありません**: ナビゲーション中にアプリケーションログを確認してください
- [ ] **一時プロパティには @JsonIgnore アノテーションが付けられています**: JSON シリアル化の問題を防止します
- [ ] **サービス層は一貫して使用されます**: テンプレートのレンダリングのためにコントローラーでリポジトリに直接アクセスすることはできません
- [ ] `jakarta.persistence` インポートは残りません
- [ ] すべてのエンティティ ID は一貫して `String` タイプです
- [ ] すべてのリポジトリインターフェイスは `CosmosRepository<Entity, String>` を拡張します
- [ ] 構成では認証に `DefaultAzureCredential` を使用します
- [ ] データシーディング コンポーネントが存在し、機能します
- [ ] テストファイルは一貫して文字列 ID を使用します
- [ ] Cosmos メソッド用にリポジトリモックが更新されました
- [ ] **エンティティクラスでメソッドシグネチャの競合はありません**
- [ ] 呼び出し元 (コントローラー、テスト、フォーマッタ) の **名前変更されたすべてのメソッドが更新されました**

### 避けるべきよくある落とし穴

1. **コンパイルを頻繁にチェックしない** - 大きな変更が行われるたびに `mvn test-compile` を実行します
2. **メソッド署名の競合** - ID タイプを変換する際のメソッドのオーバーロードの問題
3. **メソッド呼び出し元の更新を忘れている** - メソッドの名前を変更するときは、すべての呼び出し元を更新します
4. **リポジトリメソッドの名前変更がありません** - カスタムリポジトリ メソッドは、呼び出されたすべての場所で更新する必要があります
5. **キーベースの認証の使用** - 代わりに `DefaultAzureCredential` を使用してください
6. **整数 ID と文字列 ID の混合** - どこでも (特にテストでは) 文字列 ID と一貫性を保つようにしてください。
7. **コントローラーのページネーションロジックが更新されていません** - リポジトリが変更されたときにコントローラーからページネーションを削除します
8. **JPA 固有のテストアノテーションを残す** - Cosmos 互換の代替アノテーションに置き換えます
9. **不完全なテストファイルの更新** - 明らかなファイルだけでなく、テストディレクトリ全体を検索します
10. **実行時テストのスキップ** - コンパイルだけでなく、実行中のアプリケーションを常にテストします。
11. **サービス層が欠落しています** - コントローラーから直接リポジトリにアクセスしないでください
12. **一時的なプロパティの忘れ** - テンプレートは関係データへのアクセスを必要とする場合があります
13. **テンプレートナビゲーションをテストしていません** - コンパイルが成功しても、テンプレートが機能することを意味するわけではありません
14. **テンプレートの一時プロパティが欠落しています** - テンプレートには ID だけでなくオブジェクトへのアクセスが必要です
15. **サービス層のバイパス** - コントローラーはサービスを使用する必要があり、決してリポジトリに直接アクセスしないでください。
16. **不完全なリレーションシップの入力** - サービスメソッドは、テンプレートで使用されるすべての一時的なプロパティを入力する必要があります
17. **一時プロパティの @JsonIgnore を忘れる** - シリアル化の問題を防止します
18. **永続フィールドでは @JsonIgnore** - **重要**: Cosmos DB に保存する必要があるフィールドでは `@JsonIgnore` を使用しないでください
19. **認証シリアル化エラー** - ユーザー/権限エンティティは、`@JsonIgnore` が必須フィールドをブロックすることなく完全にシリアル化可能である必要があります
20. **BigDecimal リフレクションの問題** - JDK 17 以降との互換性のために代替データ型または JVM 引数を使用する
21. **リポジトリのリアクティブ型キャスト** - `findAll()` を `List` に直接キャストせず、`StreamSupport.stream().collect(Collectors.toList())` を使用してください。
22. **ヘルスチェック データベース参照** - JPA の削除後に Spring Boot ヘルスチェックからデータベースの依存関係を削除します。
23. **コレクションタイプの不一致** - 文字列コレクションとオブジェクトコレクションを一貫して処理できるようにサービスメソッドを更新します。

### コンパイルの問題を体系的にデバッグする

変換後にコンパイルが失敗した場合:

1. **メインのコンパイルから開始**: `mvn compile` - 最初にエンティティとコントローラーの問題を修正します
2. **次にコンパイルをテストします**: `mvn test-compile` - 各エラーを系統的に修正します
3. **コードベース全体で残りの `jakarta.persistence` インポートを確認します**
4. **すべてのテスト定数が文字列 ID を使用していることを確認します** - `int.*TEST.*ID` を検索します
5. **リポジトリメソッドの署名が一致することを確認** 新しい Cosmos インターフェイス
6. **エンティティ関係およびテストでの整数/文字列 ID の使用が混在していないか確認します**
7. **すべてのモックが正しいメソッド名を使用していることを検証します** (`findPetTypes()` ではなく `findAllOrderByName()`)
8. **メソッドシグネチャの競合を探します** - 競合するメソッドの名前を変更して解決します
9. **アサーションメソッドが文字列 ID で機能することを確認します** (`isNotZero()` ではなく `isNotEmpty()`)

### ランタイムの問題を系統的にデバッグする

コンパイルが成功した後にランタイムが失敗した場合:

1. **アプリケーションの起動ログを確認してください** 初期化エラーがないかどうか
2. **すべてのページに移動**して、テンプレート/コントローラーの問題を特定します
3. **ログで SpEL テンプレートエラーを探します**:
   - `Property or field 'xxx' cannot be found` → 一時的なプロパティがありません
   - `EL1008E` → サービス層にリレーションシップが設定されていない
4. **直接リポジトリアクセスではなく、サービスレイヤーが使用されていることを確認します**
5. **サービスメソッドに一時プロパティが設定されていることを確認します**
6. **Web インターフェイスを介してすべての CRUD 操作をテスト**
7. **データシーディングが正しく機能していることを確認**し、関係が維持されている
8. **認証固有のデバッグ**:
   - `Cannot pass null or empty values to constructor` → 必須フィールドの `@JsonIgnore` にチェックを入れてください
   - `BadCredentialsException` → ユーザーエンティティのシリアル化とパスワードフィールドのアクセス可能性を確認する
   - 「DomainUserDetailsS​​ervice」デバッグ出力のログを確認して、認証フローをトレースします。

### **成功のためのプロのヒント**

- **早期かつ頻繁にコンパイルします** - エラーが蓄積しないようにします
- **グローバル検索と置換を使用します** - 更新するパターンの出現をすべて検索します
- **体系的に行う** - 次のファイルに進む前に、すべてのファイルにわたる 1 つのタイプのエラーを修正してください。
- **メソッドの名前変更は慎重にテストしてください** - すべての呼び出し元が更新されていることを確認してください
- **意味のある文字列 ID を使用します** - ランダムな文字列の代わりに「owner-1」、「pet-1」
- **コントローラークラスを確認する** - コントローラークラスは、署名を変更するリポジトリメソッドを呼び出すことがよくあります。
- **常にランタイムをテストしてください** - コンパイルの成功は、機能するテンプレートを保証するものではありません
- **サービス層は重要です** - ドキュメントストレージとテンプレートの期待の間の橋渡しをします

### **認証トラブルシューティングガイド** (重要)

#### **一般的な認証シリアル化エラー**:

1. **`Cannot pass null or empty values to constructor`**:

   - **根本原因**: `@JsonIgnore` が Cosmos DB への必須フィールドのシリアル化を妨げています
   - **解決策**: すべての永続フィールド (パスワード、権限など) から `@JsonIgnore` を削除します。
   - **検証**: ユーザーエンティティに保存されたフィールドに `@JsonIgnore` がないことを確認してください

2. **`BadCredentialsException` ログイン中**:

   - **根本原因**: 認証中にパスワードフィールドにアクセスできません
   - **解決策**: パスワードフィールドがシリアル化可能で、UserDetailsS​​ervice でアクセスできることを確認します。
   - **検証**: `loadUserByUsername` メソッドにデバッグログを追加します

3. **当局が正しく読み込まれていません**:

   - **根本原因**: 権限オブジェクトが文字列ではなく複雑なエンティティとして保存されている
   - **解決策**: 権限を `Set<String>` として保存し、UserDetailsS​​ervice で `GrantedAuthority` に変換します
   - **パターン**：

```java
     // In User entity - stored in Cosmos
     @JsonProperty("authorities")
     private Set<String> authorities = new HashSet<>();

     // In UserDetailsService - convert for Spring Security
     List<GrantedAuthority> grantedAuthorities = user
       .getAuthorities()
       .stream()
       .map(SimpleGrantedAuthority::new)
       .collect(Collectors.toList());

     ```

4. **認証中にユーザーエンティティが見つかりません**:
   - **根本原因**: リポジトリクエリ メソッドが文字列 ID で機能しない
   - **解決策**: Cosmos DB と連携するようにリポジトリ `findOneByLogin` メソッドを更新します。
   - **検証**: リポジトリメソッドを個別にテストします。

#### **認証デバッグチェックリスト**:

- [ ] ユーザーエンティティは完全にシリアル化可能 (永続化フィールドに `@JsonIgnore` はありません)
- [ ] アクセス可能なパスワードフィールド、null ではない
- [ ] 権限は `Set<String>` として保存されます
- [ ] UserDetailsS​​ervice は文字列権限を `GrantedAuthority` に変換します
- [ ] リポジトリメソッドは文字列 ID で動作します
- [ ] 認証サービスでデバッグログが有効になっています
- [ ] ユーザーのアクティブ化ステータスが適切にチェックされました
- [ ] 既知の資格情報 (admin/admin) を使用してログインをテストします。

### **一般的なランタイムの問題と解決策**

#### **問題 1: リポジトリのリアクティブ型キャストエラー**

**エラー**: `ClassCastException: reactor.core.publisher.BlockingIterable cannot be cast to java.util.List`

**根本原因**: Cosmos リポジトリはリアクティブ型 (`Iterable`) を返しますが、従来の JPA コードは `List` を想定しています。

**解決策**: リポジトリメソッドでリアクティブ型を適切に変換します。

```java
// WRONG - Direct casting fails
default List<Entity> customFindMethod() {
    return (List<Entity>) this.findAll(); // ClassCastException!
}

// CORRECT - Convert Iterable to List
default List<Entity> customFindMethod() {
    return StreamSupport.stream(this.findAll().spliterator(), false)
            .collect(Collectors.toList());
}
```

**チェックするファイル**:

- カスタムのデフォルトメソッドを備えたすべてのリポジトリインターフェイス
- Cosmos リポジトリ呼び出しから `List<Entity>` を返すメソッド
- `java.util.stream.StreamSupport` と `java.util.stream.Collectors` をインポートします

#### **問題 2: Java 17 以降の BigDecimal リフレクションの問題**

**エラー**: `Unable to make field private final java.math.BigInteger java.math.BigDecimal.intVal accessible`

**根本原因**: Java 17 以降のモジュールシステムにより、シリアル化中の BigDecimal 内部フィールドへのリフレクションアクセスが制限されます。

**解決策**:

1. **単純な場合は Double に置き換えます**:

```java
   // Before: BigDecimal fields
   private BigDecimal amount;

   // After: Double fields (if precision requirements allow)
   private Double amount;

   ```

2. **高精度要件には文字列を使用します**:

```java
   // Store as String, convert as needed
   private String amount; // Store "1500.00"

   public BigDecimal getAmountAsBigDecimal() {
     return new BigDecimal(amount);
   }

   ```

3. **JVM 引数を追加** (BigDecimal を保持する必要がある場合):
```
   --add-opens java.base/java.math=ALL-UNNAMED
   ```

#### **問題 3: ヘルスチェック データベースの依存関係**

**エラー**: アプリケーションは、削除されたデータベースコンポーネントを探すヘルスチェックに失敗します。

**根本原因**: Spring Boot ヘルスチェックは、削除後も JPA/データベースの依存関係を参照します。

**解決策**: ヘルスチェック構成を更新します。

```yaml
# In application.yml - Remove database from health checks
management:
  health:
    readiness:
      include: 'ping,diskSpace' # Remove 'db' if present
```

**チェックするファイル**:

- すべての `application*.yml` 設定ファイル
- データベース固有の健全性インジケーターをすべて削除します。
- アクチュエータのエンドポイント構成を確認する

#### **問題 4: サービス内のコレクションタイプの不一致**

**エラー**: エンティティの関係を文字列ベースのストレージに変換するときに型不一致エラーが発生する

**根本原因**: エンティティ変換後に異なるコレクションタイプを期待するサービスメソッド

**解決策**: 新しいエンティティ構造を処理できるようにサービスメソッドを更新します。

```java
// Before: Entity relationships
public Set<RelatedEntity> getRelatedEntities() {
    return entity.getRelatedEntities(); // Direct entity references
}

// After: String-based relationships with conversion
public Set<RelatedEntity> getRelatedEntities() {
    return entity.getRelatedEntityIds()
        .stream()
        .map(relatedRepository::findById)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .collect(Collectors.toSet());
}

### **Enhanced Error Resolution Process**

#### **Common Error Patterns and Solutions**:

1. **Reactive Type Casting Errors**:
   - **Pattern**: `cannot be cast to java.util.List`
   - **Fix**: Use `StreamSupport.stream().collect(Collectors.toList())`
   - **Files**: Repository interfaces with custom default methods

2. **BigDecimal Serialization Errors**:
   - **Pattern**: `Unable to make field...BigDecimal.intVal accessible`
   - **Fix**: Replace with Double, String, or add JVM module opens
   - **Files**: Entity classes, DTOs, data initialization classes

3. **Health Check Database Errors**:
   - **Pattern**: Health check fails looking for database
   - **Fix**: Remove database references from health check configuration
   - **Files**: application.yml configuration files

4. **Collection Type Conversion Errors**:
   - **Pattern**: Type mismatch in entity relationship handling
   - **Fix**: Update service methods to handle String-based entity references
   - **Files**: Service classes, DTOs, entity relationship methods

#### **Enhanced Validation Checklist**:
- [ ] **Repository reactive casting handled**: No ClassCastException on collection returns
- [ ] **BigDecimal compatibility resolved**: Java 17+ serialization works
- [ ] **Health checks updated**: No database dependencies in health configuration
- [ ] **Service layer collection handling**: String-based entity references work correctly
- [ ] **Data seeding completes**: "Data seeding completed" message appears in logs
- [ ] **Application starts fully**: Both frontend and backend accessible
- [ ] **Authentication works**: Can sign in without serialization errors
- [ ] **CRUD operations functional**: All entity operations work through UI

## **Quick Reference: Common Post-Migration Fixes**

### **Top Runtime Issues to Check**

1. **Repository Collection Casting**:
   ```ジャワ
// コレクションを返すリポジトリメソッドを修正します。
デフォルト List<Entity> CustomFindMethod() {
return StreamSupport.stream(this.findAll().spliterator(), false)
.collect(Collectors.toList());
}

2. **BigDecimal の互換性 (Java 17 以降)**:

```java
   // Replace BigDecimal fields with alternatives:
   private Double amount; // Or String for high precision

   ```

3. **ヘルスチェック構成**:
```yaml
   # Remove database dependencies from health checks:
   management:
     health:
       readiness:
         include: 'ping,diskSpace'
   ```

### **認証変換パターン**

- **Cosmos DB の永続性が必要なフィールドから `@JsonIgnore` を削除します**
- **複雑なオブジェクトを単純なタイプとして保存** (例: 権限を `Set<String>` として)
- **サービス/リポジトリ層での**単純型と複合型間の変換**

### **テンプレート/UI 互換性パターン**

- 関連データへの UI アクセス用に `@JsonIgnore` を使用して **一時プロパティを追加**
- **サービスレイヤーを使用**して、レンダリング前に一時的な関係を設定します。
- **リポジトリの結果をリレーションシップの追加なしでテンプレートに直接返さないでください**
