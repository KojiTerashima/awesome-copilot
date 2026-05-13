---
description: 'Spring Boot Cassandra アプリケーションを変換して Spring Data Cosmos で Azure Cosmos DB を使用するためのステップバイステップガイド'
applyTo: '**/*.java,**/pom.xml,**/build.gradle,**/application*.properties,**/application*.yml,**/application*.conf'
---

# 総合ガイド:Spring Boot Cassandra アプリケーションを変換して Spring Data Cosmos で Azure Cosmos DB を使用する (spring-data-cosmos)

## 適用性

このガイドは以下に適用されます。
- ✅ Spring Boot 2.x - 3.x アプリケーション (リアクティブと非リアクティブの両方)
- ✅ Maven および Gradle ベースのプロジェクト
- ✅ Spring Data Cassandra、Cassandra DAO、または DataStax ドライバーを使用するアプリケーション
- ✅ ロンボク島の有無にかかわらずプロジェクト
- ✅ UUID ベースまたは文字列ベースのエンティティ識別子
- ✅ 同期アプリケーションとリアクティブ (Spring WebFlux) アプリケーションの両方

このガイドでは以下については説明しません。
- ❌ Spring 以外のフレームワーク (Jakarta EE、Micronaut、Quarkus、プレーン Java)
- ❌ 複雑な Cassandra 機能 (マテリアライズドビュー、UDT、カウンター、カスタムタイプ)
- ❌ 一括データ移行 (コード変換のみ - データは個別に移行する必要があります)
- ❌ 軽量トランザクション (LWT) やパーティション間のバッチ操作などの Cassandra 固有の機能

## 概要

このガイドでは、Spring Data Cosmos を使用して、リアクティブ Spring Boot アプリケーションを Apache Cassandra から Azure Cosmos DB に変換するための手順を段階的に説明します。実際の変換経験に基づいて、遭遇したすべての主要な問題とその解決策を取り上げます。

## 前提条件

- Java 11 以降 (Spring Boot 3.x には Java 17 以降が必要)
- ローカル開発用に Azure CLI がインストールされ、認証されている (`az login`)
- Azure Portal で作成された Azure Cosmos DB アカウント
- Maven 3.6 以降または Gradle 6 以降 (プロジェクトに応じて)
- Spring Boot 3.x を使用した Gradle プロジェクトの場合: JAVA_HOME 環境変数が Java 17 以降を指していることを確認してください。
- アプリケーションのデータモデルとクエリパターンの基本的な理解

## Azure Cosmos DB のデータベースのセットアップ

**重要**: アプリケーションを実行する前に、Cosmos DB アカウントにデータベースが存在することを確認してください。

### オプション 1: 手動データベース作成 (初回実行に推奨)
1. Azure ポータル → Cosmos DB アカウントに移動します
2. 「データエクスプローラー」に移動します
3. 「新しいデータベース」をクリックします
4. アプリケーション構成に一致するデータベース名を入力します (構成されたデータベース名については `application.properties` または `application.yml` を確認してください)
5. スループット設定を選択します (ニーズに基づいて手動または自動スケール)
   - 開発/テストには手動 400 RU/秒から開始
   - トラフィックが変動する本番ワークロードには自動スケールを使用する
6. 「OK」をクリックします

### オプション 2: 自動作成
Spring Data Cosmos は最初の接続時にデータベースを自動作成できますが、これには以下が必要です。
- 適切な RBAC アクセス許可 (Cosmos DB 組み込みデータ共同作成者ロール)
- 権限が不十分な場合は失敗する可能性があります

### コンテナ（コレクション）の作成
コンテナーは、アプリケーションの起動時に、エンティティの `@Container` アノテーション設定を使用して Spring Data Cosmos によって自動作成されます。特定のスループットまたはインデックス作成ポリシーを構成する場合を除き、手動でコンテナを作成する必要はありません。

## Azure Cosmos DB による認証

### DefaultAzureCredential の使用 (推奨)
`DefaultAzureCredential` 認証方法は、開発と運用の両方で推奨されるアプローチです。

**仕組み**:
1. 複数の資格情報ソースを順番に試します。
   - 環境変数
   - Workload Identity (AKS 用)
   - マネージド ID (Azure VM/App Service 用)
   - Azure CLI (`az login`)
   - Azure PowerShell
   - Azure開発者CLI

**ローカル開発用のセットアップ**:
```bash
# Login via Azure CLI
az login

# The application will automatically use your CLI credentials
```

**構成** (キーは必要ありません):
```java
@Bean
public CosmosClientBuilder getCosmosClientBuilder() {
    return new CosmosClientBuilder()
        .endpoint(uri)
        .credential(new DefaultAzureCredentialBuilder().build());
}
```

**プロパティファイル** (application-cosmos.properties または application.properties):
```properties
azure.cosmos.uri=https://<your-cosmos-account-name>.documents.azure.com:443/
azure.cosmos.database=<your-database-name>
# No key property needed when using DefaultAzureCredential
azure.cosmos.populate-query-metrics=false
```

**注意**: `<your-cosmos-account-name>` と `<your-database-name>` を実際の値に置き換えます。

### RBAC 権限が必要です
DefaultAzureCredential を使用する場合、Azure ID には適切な RBAC アクセス許可が必要です。

**一般的な起動エラー**:
```
Request blocked by Auth: Request for Read DatabaseAccount is blocked because principal
[xxx] does not have required RBAC permissions to perform action
[Microsoft.DocumentDB/databaseAccounts/sqlDatabases/write] on any scope.
```

**解決策**:「Cosmos DB 組み込みデータ共同作成者」ロールを割り当てます。
```bash
# Get your user's object ID
PRINCIPAL_ID=$(az ad signed-in-user show --query id -o tsv)

# Assign the role (replace <resource-group> with your actual resource group)
az cosmosdb sql role assignment create \
  --account-name your-cosmos-account \
  --resource-group <resource-group> \
  --scope "/" \
  --principal-id $PRINCIPAL_ID \
  --role-definition-name "Cosmos DB Built-in Data Contributor"
```

**代替方法**: `az login` でログインしている場合、Cosmos DB アカウントの所有者/寄稿者であれば、アカウントにはすでにアクセス許可が付与されているはずです。

### キーベースの認証 (ローカルエミュレータのみ)
ローカルエミュレータ開発にはキーベースの認証のみを使用します。

```java
@Bean
public CosmosClientBuilder getCosmosClientBuilder() {
    // Only for local emulator
    if (key != null && !key.isEmpty()) {
        return new CosmosClientBuilder()
            .endpoint(uri)
            .key(key);
    }
    // Production: use DefaultAzureCredential
    return new CosmosClientBuilder()
        .endpoint(uri)
        .credential(new DefaultAzureCredentialBuilder().build());
}
```

## 学んだ重要な教訓

### Java バージョン要件 (Spring Boot 3.x)
**問題**: Spring Boot 3.0 以降には Java 17 以降が必要です。 Java 11 を使用するとビルドエラーが発生します。
**エラー**：
```
No matching variant of org.springframework.boot:spring-boot-gradle-plugin:3.0.5 was found.
Incompatible because this component declares a component compatible with Java 17
and the consumer needed a component compatible with Java 11
```

**解決**：
```bash
# Check Java version
java -version

# Set JAVA_HOME to Java 17+
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  # Linux
# or
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home  # macOS

# Verify
echo $JAVA_HOME
```

**Gradle プロジェクトの場合**、常に正しい JAVA_HOME で実行してください。
```bash
export JAVA_HOME=/path/to/java-17
./gradlew clean build
./gradlew bootRun
```

### Gradle 固有の問題

#### 問題 1: 古い設定ファイルの競合
**問題**: Cassandra 構成ファイルの名前を変更または置換すると、古いファイルがまだ存在し、コンパイルエラーが発生する可能性があります。
```
error: class CosmosConfiguration is public, should be declared in a file named CosmosConfiguration.java
```

**解決策**: 古い Cassandra 構成ファイルを明示的に削除します。
```bash
# Find and remove old Cassandra config files
find src/main/java -name "*CassandraConfig*.java" -o -name "*CassandraConfiguration*.java"
# Review the output, then delete if appropriate
rm src/main/java/<path-to-old-config>/CassandraConfig.java
```

#### 問題 2: リポジトリ findAllById が Iterable を返す
**問題**: CosmosRepository の `findAllById()` は `List<Entity>` ではなく `Iterable<Entity>` を返します。 `.stream()` を直接呼び出すと失敗します。
```
error: cannot find symbol
  symbol:   method stream()
  location: interface Iterable<YourEntity>
```

**解決策**: Iterable を適切に処理します。
```java
// WRONG - Iterable doesn't have stream() method
var entities = repository.findAllById(ids).stream()...

// CORRECT - Option 1: Use forEach to populate a collection
Iterable<YourEntity> entitiesIterable = repository.findAllById(ids);
Map<String, YourEntity> entityMap = new HashMap<>();
entitiesIterable.forEach(entity -> entityMap.put(entity.getId(), entity));

// CORRECT - Option 2: Convert to List first
List<YourEntity> entities = new ArrayList<>();
repository.findAllById(ids).forEach(entities::add);

// CORRECT - Option 3: Use StreamSupport (Java 8+)
List<YourEntity> entities = StreamSupport.stream(
    repository.findAllById(ids).spliterator(), false)
    .collect(Collectors.toList());
```

### package-info.java javax.annotation の問題
**問題**: `package-info.java` を `javax.annotation.ParametersAreNonnullByDefault` を使用すると、Java 11 以降でコンパイルエラーが発生します。
```
error: cannot find symbol
import javax.annotation.ParametersAreNonnullByDefault;
```

**解決策**: package-info.java ファイルを削除または簡略化します。
```java
// Simple version - just package declaration
package com.your.package;
```

### エンティティコンストラクターの問題
**問題**: Lombok `@NoArgsConstructor` を手動コンストラクターで使用すると、コンストラクターの重複コンパイルエラーが発生します。
**解決策**: アプローチを 1 つ選択してください:
- オプション 1: `@NoArgsConstructor` を削除し、手動コンストラクターを保持する
- オプション 2: 手動コンストラクターを削除し、Lombok アノテーションに依存する
- **ベストプラクティス**: 初期化ロジック (パーティションキーの設定など) を備えた Cosmos エンティティの場合は、`@NoArgsConstructor` を削除し、手動コンストラクターのみを使用します。

### ビジネスオブジェクトコンストラクターの削除
**問題**: `@AllArgsConstructor` またはカスタムコンストラクターをエンティティクラスから削除すると、それらのコンストラクターを使用する既存のコードが破損します。
**影響**: マッピングユーティリティ、データシーダー、およびテストファイルはコンパイルに失敗します。
**解決**：
- コンストラクターを削除または変更した後、すべてのファイルでそれらのエンティティへのコンストラクター呼び出しを検索します。
- デフォルトのコンストラクター + セッターパターンに置き換えます。
```java
  // Before - using all-args constructor
  MyEntity entity = new MyEntity(id, field1, field2, field3);

  // After - using default constructor + setters
  MyEntity entity = new MyEntity();
  entity.setId(id);
  entity.setField1(field1);
  entity.setField2(field2);
  entity.setField3(field3);
  ```
### データシーダー コンストラクターの呼び出し
**問題**: データシードまたは初期化コードでは、Cosmos アノテーションへのエンティティ変換後に存在しない可能性があるエンティティコンストラクターが使用されています。
**解決策**: セッターを使用するように、データシード コンポーネント内のすべてのエンティティのインスタンス化を更新します。
```java
// Before - constructor-based initialization
MyEntity entity1 = new MyEntity("entity-1", "value1", "value2");

// After - setter-based initialization
MyEntity entity1 = new MyEntity();
entity1.setId("entity-1");
entity1.setField1("value1");
entity1.setField2("value2");
```

**チェックすべき一般的なファイル**: DataSeeder、DatabaseInitializer、TestDataLoader、または `CommandLineRunner` を実装する `@Component`
```java
OwnerEntity owner1 = new OwnerEntity();
owner1.setId("owner-1");
```

### テストファイルの更新が必要です
**問題**: テストファイルは古い Cassandra DAO を参照し、UUID コンストラクターを使用します。
**更新する重要なファイル**:
1. `MockReactiveResultSet.java` を削除 (Cassandra 固有)
2. `*ReactiveServicesTest.java` を更新 - DAO 参照を Cosmos リポジトリに置き換えます
3. `*ReactiveControllerTest.java` を更新 - DAO 参照を Cosmos リポジトリに置き換えます
4. すべての `UUID.fromString()` を文字列 ID に置き換えます
5. コンストラクター呼び出しを置き換えます: `new Owner(UUID.fromString(...))` をセッターパターンに置き換えます

### アプリケーションの起動と DefaultAzureCredential の動作
**重要**: DefaultAzureCredential は複数の認証方法を順番に試行しますが、これは正常であり、予期されることです。

**予想される起動ログパターン**:
```
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential EnvironmentCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential WorkloadIdentityCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential ManagedIdentityCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential SharedTokenCacheCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential IntelliJCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential AzureCliCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential AzurePowerShellCredential is unavailable.
INFO c.azure.identity.ChainedTokenCredential : Azure Identity => Attempted credential AzureDeveloperCliCredential returns a token
```

**重要なポイント**:
- 「利用できません」メッセージは **正常** - 各資格情報ソースを順番に試行しています
- 機能するもの (AzureCliCredential や AzureDeveloperCliCredential など) が見つかると、それを使用します。
- **起動プロセスを中断しないでください** - 資格情報ソースを循環するのに 10 ～ 15 秒かかります
- 通常、アプリケーションが完全に起動して Cosmos DB に接続するまでに合計 30 ～ 60 秒かかります。

**成功の指標**:
```
INFO c.a.c.i.RxDocumentClientImpl : Initializing DocumentClient [1] with serviceEndpoint [https://your-account.documents.azure.com:443/]
INFO c.a.c.i.GlobalEndpointManager : db account retrieved {...}
INFO c.a.c.implementation.SessionContainer : Registering a new collection resourceId [...]
INFO o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat started on port(s): 8944 (http)
INFO com.your.app.Application : Started Application in X.XXX seconds
```

**起動失敗のトラブルシューティング**:

1. **すべての認証情報が「利用できない」場合**:
```bash
   # Re-authenticate with Azure CLI
   az login

   # Verify login
   az account show
   ```

2. **権限エラーが表示された場合**:
```
   Request blocked by Auth: principal [xxx] does not have required RBAC permissions
   ```
   - データベースが Cosmos DB アカウントに存在することを確認します (「データベースのセットアップ」セクションを参照)
   - RBAC 権限を確認します (「認証」セクションを参照)
   - 正しい Azure サブスクリプションにログインしていることを確認してください

3. **ポートはすでに使用されています**:
```bash
   # Find and kill the process
   lsof -ti:8944 | xargs kill -9

   # Or change the port in application.properties
   server.port=8945
   ```

### アプリケーション起動の忍耐力
**問題**: アプリケーションが完全に起動するまでに 30 ～ 60 秒かかります (コンパイル + Spring Boot + Cosmos DB 接続)。
**解決**：
- Gradle の場合: `./gradlew bootRun` (デフォルトではフォアグラウンドで実行)
- Maven の場合: `mvn spring-boot:run`
- 必要に応じてバックグラウンド実行を使用します: `nohup ./gradlew bootRun > app.log 2>&1 &`
- **重要**: 特に資格情報の認証中 (10 ～ 15 秒)、起動プロセスを中断しないでください。
- ログを監視します: `tail -f app.log` または「開始されたアプリケーション」メッセージを確認します。
- エンドポイントをテストする前に、Tomcat が起動してポート番号が表示されるまで待ちます。

### ポート構成
**問題**: アプリケーションはデフォルトのポート 8080 では実行できない可能性があります。
**解決**：
- 実際のポートを確認してください: `ss -tlnp | grep java`
- 接続のテスト: `curl http://localhost:<port>/petclinic/api/owners`
- 共通ポート: 8080、9966、9967

## 系統的なコンパイルエラー解決

この変換中に、100 を超えるコンパイルエラーが発生しました。これらを解決した体系的なアプローチは次のとおりです。

### ステップ 1: 残っている Cassandra ファイルを特定する
**問題**: 古い Cassandra 固有のファイルでは、依存関係が削除された後にコンパイルエラーが発生します。
**解決策**: Cassandra 固有のファイルをすべて体系的に削除します。

```bash
# Identify and delete old DAOs
find . -name "*Dao.java" -o -name "*DAO.java"
# Delete: OwnerReactiveDao, PetReactiveDao, VetReactiveDao, VisitReactiveDao

# Identify and delete Cassandra mappers
find . -name "*Mapper.java" -o -name "*EntityToOwnerMapper.java"
# Delete: EntityToOwnerMapper, EntityToPetMapper, EntityToVetMapper, EntityToVisitMapper

# Identify and delete old configuration
find . -name "*CassandraConfig.java" -o -name "CassandraConfiguration.java"
# Delete: CassandraConfiguration.java

# Identify test utilities for Cassandra
find . -name "MockReactiveResultSet.java"
# Delete: MockReactiveResultSet.java (Cassandra-specific test utility)
```

### ステップ 2: 増分コンパイルチェックを実行する
**アプローチ**: 大きな変更を加えるたびにコンパイルして、残っている問題を特定します。

```bash
# After deleting old files
mvn compile 2>&1 | grep -E "(ERROR|error)" | wc -l
# Expected: Number decreases with each fix

# After updating entity constructors
mvn compile 2>&1 | grep "constructor"
# Identify constructor-related compilation errors

# After fixing business object constructors
mvn compile 2>&1 | grep -E "(new Owner|new Pet|new Vet|new Visit)"
# Identify remaining constructor calls that need fixing
```

### ステップ 3: コンストラクター関連のエラーを体系的に修正する
**パターン**: 特定のファイルタイプ内のすべてのコンストラクター呼び出しを検索します。

```bash
# Find all constructor calls in MappingUtils
grep -n "new Owner\|new Pet\|new Vet\|new Visit" src/main/java/**/MappingUtils.java

# Find all constructor calls in DataSeeder
grep -n "new OwnerEntity\|new PetEntity\|new VetEntity\|new VisitEntity" src/main/java/**/DataSeeder.java

# Find all constructor calls in test files
grep -rn "new Owner\|new Pet\|new Vet\|new Visit" src/test/java/
```

### ステップ 4: 最後にテストを更新する
**根拠**: すべての問題を明確に確認するには、コードをテストする前にアプリケーションコードを修正します。

1. 最初: テストリポジトリ モックを更新します (DAO → Cosmos リポジトリ)
2. 2 番目: テストデータ内の UUID → 文字列変換を修正
3. 3 番目: テストセットアップでのコンストラクター呼び出しを更新する
4. 最後に: テストを実行して検証します: `mvn test`

### ステップ 5: コンパイルエラーがないことを確認する
**最終チェック**:
```bash
# Clean and full compile
mvn clean compile

# Should see: BUILD SUCCESS
# Should NOT see any ERROR messages

# Verify test compilation
mvn test-compile

# Run tests
mvn test
```

**成功指標**:
- `mvn compile`: 構築の成功
- `mvn test`: すべてのテストに合格します (一部がスキップされた場合でも)
- 出力にエラーメッセージが表示されない
- 「シンボルが見つかりません」エラーがない
- 「コンストラクターを適用できません」エラーが発生しない

## 変換手順

### 1. Maven の依存関係を更新する

#### Cassandra の依存関係を削除する
```xml
<!-- REMOVE these Cassandra dependencies -->
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-core</artifactId>
</dependency>
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-query-builder</artifactId>
</dependency>
```

#### Azure Cosmos の依存関係を追加する
```xml
<!-- Azure Spring Data Cosmos (Java 11 compatible) -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-cosmos</artifactId>
    <version>3.46.0</version>
</dependency>

<!-- Azure Identity for DefaultAzureCredential authentication -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.11.4</version>
</dependency>
```

#### 重要: 互換性のためにバージョン管理を追加する
Spring Boot 2.3.x には、Azure ライブラリとバージョンの競合があります。これを `<dependencyManagement>` セクションに追加します。

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override reactor-netty version to fix compatibility with azure-spring-data-cosmos -->
        <dependency>
            <groupId>io.projectreactor.netty</groupId>
            <artifactId>reactor-netty</artifactId>
            <version>1.0.40</version>
        </dependency>
        <dependency>
            <groupId>io.projectreactor.netty</groupId>
            <artifactId>reactor-netty-http</artifactId>
            <version>1.0.40</version>
        </dependency>
        <dependency>
            <groupId>io.projectreactor.netty</groupId>
            <artifactId>reactor-netty-core</artifactId>
            <version>1.0.40</version>
        </dependency>

        <!-- Override reactor-core version to support Sinks API required by azure-identity -->
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <version>3.4.32</version>
        </dependency>

        <!-- Override Netty versions to fix compatibility with Azure Cosmos Client -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-bom</artifactId>
            <version>4.1.101.Final</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <!-- Override netty-tcnative to match Netty version -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-tcnative-boringssl-static</artifactId>
            <version>2.0.62.Final</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2. 構成のセットアップ

#### Cosmos 構成クラスの作成
Cassandra 構成を次のものに置き換えます。

```java
@Configuration
@EnableCosmosRepositories  // Required for non-reactive repositories
@EnableReactiveCosmosRepositories  // CRITICAL: Required for reactive repositories
public class CosmosConfiguration extends AbstractCosmosConfiguration {

    @Value("${azure.cosmos.uri}")
    private String uri;

    @Value("${azure.cosmos.database}")
    private String database;

    @Bean
    public CosmosClientBuilder getCosmosClientBuilder() {
        return new CosmosClientBuilder()
            .endpoint(uri)
            .credential(new DefaultAzureCredential());
    }

    @Bean
    public CosmosAsyncClient cosmosAsyncClient(CosmosClientBuilder cosmosClientBuilder) {
        return cosmosClientBuilder.buildAsyncClient();
    }

    @Bean
    public CosmosClientBuilderFactory cosmosFactory(CosmosAsyncClient cosmosAsyncClient) {
        return new CosmosClientBuilderFactory(cosmosAsyncClient);
    }

    @Bean
    public ReactiveCosmosTemplate reactiveCosmosTemplate(CosmosClientBuilderFactory cosmosClientBuilderFactory) {
        return new ReactiveCosmosTemplate(cosmosClientBuilderFactory, database);
    }

    @Override
    protected String getDatabaseName() {
        return database;
    }
}
```

**重要な注意事項:**
- **両方のアノテーションが必要です**: @EnableCosmosRepositories と @EnableReactiveCosmosRepositories
- @EnableReactiveCosmosRepositories が存在しないと、リアクティブリポジトリに対して「適格な Bean がありません」エラーが発生します

#### アプリケーションのプロパティ
コスモスプロファイル構成を追加します。

```properties
# application-cosmos.properties
azure.cosmos.uri=https://your-cosmos-account.documents.azure.com:443/
azure.cosmos.database=your-database-name
```

### 3. エンティティの変換

#### Cassandra から Cosmos アノテーションへの変換

**前 (カサンドラ):**
```java
@Table(value = "entity_table")
public class EntityName {
    @PartitionKey
    private UUID id;

    @ClusteringColumn
    private String fieldName;

    @Column("column_name")
    private String anotherField;
}
```

**後 (コスモス):**
```java
@Container(containerName = "entities")
public class EntityName {
    @Id
    private String id;  // Changed from UUID to String

    @PartitionKey
    private String fieldName;  // Choose appropriate partition key

    private String anotherField;

    // Generate String IDs
    public EntityName() {
        this.id = UUID.randomUUID().toString();
    }
}
```

#### 主な変更点:
- `@Table` を `@Container(containerName = "...")` に置き換えます
- `@PartitionKey` を Cosmos パーティションキー戦略に変更します
- すべての ID を `UUID` から `String` に変換します
- `@Column` 注釈を削除します (Cosmos はフィールド名を使用します)
- `@ClusteringColumn` を削除します (Cosmos では適用されません)

### 4. リポジトリの変換

#### Cassandra データアクセス レイヤーを Cosmos リポジトリに置き換える

**アプリケーションが DAO またはカスタムデータアクセスクラスを使用している場合:**

**前 (Cassandra DAO パターン):**
```java
@Repository
public class EntityReactiveDao {
    // Custom Cassandra query methods
}
```

**後 (Cosmos リポジトリ):**
```java
@Repository
public interface EntityCosmosRepository extends ReactiveCosmosRepository<EntityName, String> {

    @Query("SELECT * FROM entities e WHERE e.fieldName = @fieldName")
    Flux<EntityName> findByFieldName(@Param("fieldName") String fieldName);

    @Query("SELECT * FROM entities e WHERE e.id = @id")
    Mono<EntityName> findEntityById(@Param("id") String id);
}
```

**アプリケーションが Spring Data Cassandra リポジトリを使用している場合:**

**前に：**
```java
@Repository
public interface EntityCassandraRepository extends ReactiveCassandraRepository<EntityName, UUID> {
    // Cassandra-specific methods
}
```

**後：**
```java
@Repository
public interface EntityCosmosRepository extends ReactiveCosmosRepository<EntityName, String> {
    // Convert existing methods to Cosmos queries
}
```

**アプリケーションが直接 CqlSession または Cassandra ドライバーを使用している場合:**
- ドライバーの直接呼び出しをリポジトリパターンで置き換える
- CQL クエリを Cosmos SQL 構文に変換する
- 上に示したようにリポジトリインターフェイスを実装します。

#### 重要なポイント:
- **重要**: リアクティブプログラミングには `ReactiveCosmosRepository<Entity, String>` を使用します (CosmosRepository ではありません)
- 非リアクティブなアプリケーションには `CosmosRepository<Entity, String>` を使用してください
- **リポジトリインターフェイスの変更**: 既存の Cassandra リポジトリ/DAO から変換する場合は、すべてのリポジトリインターフェイスが ReactiveCosmosRepository を拡張していることを確認してください。
- **一般的なエラー**: 「ReactiveCosmosRepository タイプの対象となる Bean がありません」 = @EnableReactiveCosmosRepositories がありません
- **カスタムデータアクセスクラスを使用している場合**: 統合を改善するためにリポジトリパターンに変換します
- **既に Spring Data を使用している場合**: インターフェイス拡張機能を ReactiveCassandraRepository から ReactiveCosmosRepository に変更します。
- SQL のような構文 (CQL ではない) を使用して、`@Query` アノテーションを含むカスタムクエリを実装します。
- すべてのクエリパラメータは `@Param` アノテーションを使用する必要があります

### 5. サービス層の更新

#### リアクティブプログラミングのサービスクラスを更新する (該当する場合)

**アプリケーションにサービス層がある場合:**

**重要**: サービスメソッドは Iterable/Optional ではなく Flux/Mono を返す必要があります

```java
@Service
public class EntityReactiveServices {
    private final EntityCosmosRepository repository;

    public EntityReactiveServices(EntityCosmosRepository repository) {
        this.repository = repository;
    }

    // CORRECT: Returns Flux<EntityName>
    public Flux<EntityName> findAll() {
        return repository.findAll();
    }

    // CORRECT: Returns Mono<EntityName>
    public Mono<EntityName> findById(String id) {
        return repository.findById(id);
    }

    // CORRECT: Returns Mono<EntityName>
    public Mono<EntityName> save(EntityName entity) {
        return repository.save(entity);
    }

    // Custom queries - MUST return Flux/Mono
    public Flux<EntityName> findByFieldName(String fieldName) {
        return repository.findByFieldName(fieldName);
    }

    // WRONG PATTERNS TO AVOID:
    // public Iterable<EntityName> findAll() - Will cause compilation errors
    // public Optional<EntityName> findById() - Will cause compilation errors
    // repository.findAll().collectList() - Unnecessary blocking
}
```

**アプリケーションがコントローラーで直接リポジトリインジェクションを使用している場合:**
- 懸念事項をより適切に分離するためにサービス層の追加を検討してください
- 新しい Cosmos リポジトリを使用するようにコントローラーの依存関係を更新する
- 呼び出しチェーン全体で適切なリアクティブ型処理を保証する

**一般的な問題:**
- **コンパイルエラー**: Iterable 戻り値の型を使用する場合は「メソッドを解決できません」
- **実行時エラー**: .collectList() または .block() を不必要に呼び出そうとしています
- **パフォーマンス**: リアクティブストリームをブロックすると、リアクティブプログラミングの目的が無効になります。

### 6. コントローラーのアップデート (該当する場合)

#### 文字列 ID の REST コントローラーを更新する

**アプリケーションに REST コントローラーがある場合:**

**前に：**
```java
@GetMapping("/entities/{entityId}")
public Mono<EntityDto> getEntity(@PathVariable UUID entityId) {
    return entityService.findById(entityId);
}
```

**後：**
```java
@GetMapping("/entities/{entityId}")
public Mono<EntityDto> getEntity(@PathVariable String entityId) {
    return entityService.findById(entityId);
}
```

**アプリケーションがコントローラーを使用しない場合:**
- 同じ UUID → 文字列変換原則をデータアクセス レイヤーに適用します。
- エンティティ ID を受け入れる/返す外部 API またはインターフェイスを更新します。

### 7. データマッピング ユーティリティ (該当する場合)

#### ドメインオブジェクトとエンティティ間のマッピングの更新

**アプリケーションがマッピングユーティリティまたはコンバータを使用している場合:**

```java
public class MappingUtils {

    // Convert domain object to entity
    public static EntityName toEntity(DomainObject domain) {
        EntityName entity = new EntityName();
        entity.setId(domain.getId()); // Now String instead of UUID
        entity.setFieldName(domain.getFieldName());
        entity.setAnotherField(domain.getAnotherField());
        // ... other fields
        return entity;
    }

    // Convert entity to domain object
    public static DomainObject toDomain(EntityName entity) {
        DomainObject domain = new DomainObject();
        domain.setId(entity.getId());
        domain.setFieldName(entity.getFieldName());
        domain.setAnotherField(entity.getAnotherField());
        // ... other fields
        return domain;
    }
}
```

**アプリケーションが明示的なマッピングを使用しない場合:**
- コードベース全体で一貫した ID タイプの使用を保証します
- 文字列 ID を処理するためにオブジェクトの構築またはコピーロジックを更新します。

### 8. テストの更新

#### テストクラスを更新する

**重要**: 文字列 ID と Cosmos リポジトリで動作するには、すべてのテストファイルを更新する必要があります。

```java
**If your application has unit tests:**

```java
@ExtendWith(MockitoExtension.class)
クラス EntityReactiveServicesTest {

@モック
プライベートEntityCosmosRepositoryエンティティリポジトリ; // Cosmos リポジトリに更新されました

@InjectMocks
プライベートEntityReactiveServicesエンティティサービス;

@テスト
void testFindById() {
文字列エンティティ ID = "テストエンティティ ID"; // UUID から文字列に変更
EntityName モックエンティティ = new EntityName();
モックエンティティ.setId(エンティティID);

when(entityRepository.findById(entityId)).thenReturn(Mono.just(mockEntity));

StepVerifier.create(entityService.findById(entityId))
.expectNext(モックエンティティ)
.verifyComplete();
}
}
```

**If your application has integration tests:**
- Update test data setup to use String IDs
- Replace Cassandra test containers with Cosmos DB emulator (if available)
- Update test queries to use Cosmos SQL syntax instead of CQL

**If your application doesn't have tests:**
- Consider adding basic tests to verify the conversion works correctly
- Focus on testing ID conversion and basic CRUD operations
```

### 9. 一般的な問題と解決策

#### 問題 1:actor.core.publisher.Sinks による NoClassDefFoundError
**問題**: Azure Identity ライブラリには新しい Reactor Core バージョンが必要です
**エラー**: `java.lang.NoClassDefFoundError: reactor/core/publisher/Sinks`
**根本原因**: Spring Boot 2.3.x は、シンク API を持たない古いリアクターコアを使用しています。
**解決策**: dependencyManagement にリアクターコア バージョンのオーバーライドを追加します (ステップ 1 を参照)

#### 問題 2: Netty Epoll メソッドでの NoSuchMethodError
**問題**: Spring Boot Netty 要件と Azure Cosmos 要件の間のバージョンの不一致
**エラー**: `java.lang.NoSuchMethodError: 'boolean io.netty.channel.epoll.Epoll.isTcpFastOpenClientSideAvailable()'`
**根本原因**: Spring Boot 2.3.x は Netty 4.1.51.Final を使用しており、Azure には新しいメソッドが必要です
**解決策**: netty-bom バージョンオーバーライドを追加します (ステップ 1 を参照)

#### 問題 3: SSL コンテキストでの NoSuchMethodError
**問題**: Netty TLS ネイティブライブラリのバージョンが一致しません
**エラー**: `java.lang.NoSuchMethodError: 'boolean io.netty.internal.tcnative.SSLContext.setCurvesList(long, java.lang.String[])'`
**根本原因**: netty-tcnative バージョンがアップグレードされた Netty と互換性がない
**解決策**: netty-tcnative-boringssl-static バージョンオーバーライドを追加します (ステップ 1 を参照)

#### 問題 4: ReactiveCosmosRepository Bean が作成されない
**問題**: @EnableReactiveCosmosRepositories アノテーションがありません
**エラー**: `No qualifying bean of type 'ReactiveCosmosRepository' available`
**根本原因**: @EnableCosmosRepositories のみがリアクティブリポジトリ Bean を作成しません
**解決策**: @EnableCosmosRepositories と @EnableReactiveCosmosRepositories の両方を構成に追加します

#### 問題 5: リポジトリインターフェイスのコンパイルエラー
**問題**: ReactiveCosmosRepository の代わりに CosmosRepository を使用する
**エラー**: `Cannot resolve method 'findAll()' in 'CosmosRepository'`
**根本原因**: CosmosRepository は Flux ではなく Iterable を返します
**解決策**: ReactiveCosmosRepository<Entity, String> を拡張するようにすべてのリポジトリインターフェイスを変更します。

#### 問題 6: サービス層のリアクティブタイプの不一致
**問題**: サービスメソッドが Flux/Mono ではなく Iterable/Optional を返す
**エラー**: `Required type: Flux<Entity> Provided: Iterable<Entity>`
**根本原因**: リポジトリメソッドはリアクティブタイプを返すため、サービスは一致する必要があります。
**解決策**: Flux/Mono を返すようにすべてのサービスメソッド シグネチャを更新します。

#### 問題 7: DefaultAzureCredential による認証の失敗
**問題**: DefaultAzureCredential で資格情報が見つからない
**エラー**: `All credentials in the chain are unavailable` または特定の認証情報が利用できないメッセージ
**根本原因**: 有効な Azure 資格情報ソースが利用できません

**解決策**:
1. **ローカル開発の場合**: Azure CLI ログインを確認する
```bash
   az login
   # Verify login
   az account show
   ```

2. **Azure でホストされるアプリケーションの場合**: マネージド ID が有効であり、適切な RBAC 権限があることを確認してください。

3. **資格情報チェーンの順序を確認してください**: DefaultAzureCredential は次の順序で試行します。
   - 環境変数 → Workload Identity → マネージド ID → Azure CLI → PowerShell → 開発者 CLI

#### 問題 8: データベースが見つからないエラー
**問題**: データベースが見つからないエラーでアプリケーションが起動できない
**エラー**: `Database 'your-database-name' not found` または `Resource Not Found`
**根本原因**:Cosmos DB アカウントにデータベースが存在しません。

**解決策**: 最初の実行前にデータベースを作成します (「データベースのセットアップ」セクションを参照)。
```bash
# Via Azure CLI
az cosmosdb sql database create \
  --account-name your-cosmos-account \
  --name your-database-name \
  --resource-group your-resource-group

# Or via Azure Portal (recommended for first-time setup)
# Portal → Cosmos DB → Data Explorer → New Database
```

**注意**: コンテナー (コレクション) はエンティティ `@Container` アノテーションから自動作成されますが、RBAC 権限に応じて最初にデータベース自体が存在する必要がある場合があります。

#### 問題 9: RBAC 権限エラー
**問題**: アプリケーションがアクセス許可拒否エラーで失敗する
**エラー**：
```
Request blocked by Auth: principal [xxx] does not have required RBAC permissions
to perform action [Microsoft.DocumentDB/databaseAccounts/sqlDatabases/write]
```

**根本原因**: Azure ID に必要な Cosmos DB アクセス許可がありません

**解決策**:「Cosmos DB 組み込みデータ共同作成者」ロールを割り当てます。
```bash
# Get resource group
RESOURCE_GROUP=$(az cosmosdb show --name your-cosmos-account --query resourceGroup -o tsv 2>/dev/null)

# If the above fails, list all Cosmos accounts to find it
az cosmosdb list --query "[?name=='your-cosmos-account'].{name:name, resourceGroup:resourceGroup}" -o table

# Assign role
az cosmosdb sql role assignment create \
  --account-name your-cosmos-account \
  --resource-group $RESOURCE_GROUP \
  --scope "/" \
  --principal-id $(az ad signed-in-user show --query id -o tsv) \
  --role-definition-name "Cosmos DB Built-in Data Contributor"
```

**代替**: ポータル → Cosmos DB → アクセス制御 (IAM) → ロール割り当ての追加 → 「Cosmos DB 組み込みデータ共同作成者」

#### 問題 10: パーティションキー戦略の違い
**問題**: Cassandra クラスタリングキーが Cosmos パーティションキーに直接マップされない
**エラー**: パーティション間のクエリまたはパフォーマンスの低下
**根本原因**: データ分散戦略の違い
**解決策**: クエリパターン (通常は最も頻繁にクエリされるフィールド) に基づいて適切なパーティションキーを選択します。

#### 問題 10: UUID から文字列への変換の問題
**問題**: テストファイルとコントローラーがまだ UUID タイプを使用している
**エラー**: `Cannot convert UUID to String` またはタイプの不一致エラー
**根本原因**: UUID のすべてが文字列に変換されたわけではありません
**解決策**: すべての UUID 参照を体系的に検索して文字列に置き換えます。

### 10. データシーディング (該当する場合)

#### データ入力の実装

**アプリケーションに初期データが必要な場合:**

```java
@Component
public class DataSeeder implements CommandLineRunner {

    private final EntityCosmosRepository entityRepository;

    @Override
    public void run(String... args) throws Exception {
        if (entityRepository.count().block() == 0) {
            // Seed initial data
            EntityName entity = new EntityName();
            entity.setFieldName("Sample Value");
            entity.setAnotherField("Sample Data");

            entityRepository.save(entity).block();
        }
    }
}
```

**アプリケーションに既存のデータ移行のニーズがある場合:**
- Cassandra からエクスポートして Cosmos DB にインポートする移行スクリプトを作成する
- データ変換のニーズを考慮する (UUID から文字列への変換)
- Cassandra データモデルと Cosmos データモデル間のスキーマの違いを計画する

**アプリケーションにデータシードが必要ない場合:**
- このステップをスキップして検証に進みます

### 11. アプリケーションプロファイル

#### Cosmos プロファイルの application.yml を更新する
```yaml
spring:
  profiles:
    active: cosmos

---
spring:
  profiles: cosmos

azure:
  cosmos:
    uri: ${COSMOS_URI:https://your-account.documents.azure.com:443/}
    database: ${COSMOS_DATABASE:your-database}
```

## 検証手順

1. **コンパイルチェック**: `mvn compile` はエラーなしで成功するはずです
2. **テストチェック**: `mvn test` は更新されたテストケースで合格するはずです
3. **実行時チェック**: アプリケーションはバージョンの競合なしで起動する必要があります。
4. **接続チェック**: アプリケーションは Cosmos DB に正常に接続する必要があります
5. **データチェック**: CRUD 操作は API を通じて機能する必要があります
6. **UI チェック**: フロントエンドは Cosmos DB からのデータを表示する必要があります

## ベストプラクティス

1. **ID 戦略**: Cosmos DB には UUID ではなく常に文字列 ID を使用します。
2. **パーティションキー**: クエリパターンとデータ分散に基づいてパーティションキーを選択します
3. **クエリの設計**: メソッドの命名規則の代わりに、カスタムクエリに @Query アノテーションを使用します。
4. **リアクティブプログラミング**: サービス層全体で Flux/Mono パターンに固執する
5. **バージョン管理**: Spring Boot 2.x プロジェクトの依存関係バージョンのオーバーライドを常に含めます。
6. **テスト**: 文字列 ID とモック Cosmos リポジトリを使用するようにすべてのテストファイルを更新します。
7. **認証**: 実稼働対応の認証には DefaultAzureCredential を使用します

## トラブルシューティングコマンド

```bash
# Check dependencies and version conflicts
mvn dependency:tree | grep -E "(reactor|netty|cosmos)"

# Verify specific problematic dependencies
mvn dependency:tree | grep "reactor-core"
mvn dependency:tree | grep "reactor-netty"
mvn dependency:tree | grep "netty-tcnative"

# Test connection
curl http://localhost:8080/api/entities

# Check Azure login status
az account show

# Clean and rebuild (often fixes dependency issues)
mvn clean compile

# Run with debug logging for dependency resolution
mvn dependency:resolve -X

# Check for compilation errors specifically
mvn compile 2>&1 | grep -E "(ERROR|error)"

# Run with debug for runtime issues
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"

# Check application logs for version conflicts
grep -E "(NoSuchMethodError|NoClassDefFoundError|reactor|netty)" application.log
```

## 一般的なエラーシーケンスと解決策

実際の変換経験に基づくと、次の順序でエラーが発生する可能性があります。

### **フェーズ 1: コンパイルエラー**
1. **不足している依存関係** → azure-spring-data-cosmos と azure-identity を追加
2. **構成クラスエラー** → CosmosConfiguration を作成します (まだ存在しない場合)
3. **エンティティアノテーション エラー** → @Table を @Container に変換するなど。
4. **リポジトリインターフェイス エラー** → ReactiveCosmosRepository に変更します (リポジトリパターンを使用している場合)

### **フェーズ 2: Bean 作成エラー**
5. **「ReactiveCosmosRepository タイプの適格な Bean がありません」** → @EnableReactiveCosmosRepositories を追加
6. **サービスレイヤーのタイプの不一致** → Iterable を Flux に、Optional を Mono に変更します (サービスレイヤーを使用している場合)

### **フェーズ 3: ランタイムバージョンの競合** (最も複雑)
7. **NoClassDefFoundError:actor.core.publisher.Sinks** →actor-core 3.4.32 オーバーライドを追加
8. **NoSuchMethodError: Epoll.isTcpFastOpenClientSideAvailable** → netty-bom 4.1.101.Final オーバーライドを追加
9. **NoSuchMethodError: SSLContext.setCurvesList** → netty-tcnative-boringssl-static 2.0.62.Final オーバーライドを追加

### **フェーズ 4: 認証と接続**
10. **ManagedIdentityCredential 認証は利用できません** → `az login --use-device-code` を実行
11. **アプリケーションが正常に起動します** → Cosmos DB に接続されました。

**重要**: これらに順番に対処してください。先へスキップしないでください。各フェーズは、次のフェーズが表示される前に解決する必要があります。

## パフォーマンスに関する考慮事項

1. **パーティション戦略**: 負荷を均等に分散するようにパーティションキーを設計します。
2. **クエリの最適化**: インデックスを使用し、可能な場合はパーティション間のクエリを回避します。
3. **接続プーリング**: Cosmos クライアントが接続を自動的に管理します
4. **リクエストユニット**: RU 消費量を監視し、必要に応じてスループットを調整します
5. **一括操作**: 複数のドキュメントの更新にバッチ操作を使用します。

このガイドでは、実際のシナリオで発生するすべてのバージョンの競合や認証の問題など、Cassandra から Cosmos DB への変換に関する主要な側面をすべて説明します。
