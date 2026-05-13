---
description: "Gradle Kotlin DSL と version catalog を中心に、Spring Boot application を 3.x から 4.0 へ移行するための包括的ガイド"
applyTo: "**/*.java, **/*.kt, **/build.gradle.kts, **/build.gradle, **/settings.gradle.kts, **/gradle/libs.versions.toml, **/*.properties, **/*.yml, **/*.yaml"
---

# Spring Boot 3.x から 4.0 への移行ガイド

## Project Context

このガイドは、Spring Boot project を 3.x から 4.0 へ upgrade するための包括的な GitHub Copilot instruction を提供する。特に、Gradle Kotlin DSL、version catalog (`libs.versions.toml`)、および Kotlin 固有の考慮事項に重点を置いている。

**Spring Boot 4.0 の主なアーキテクチャ変更点:**
- より小さく焦点を絞った module による modular dependency structure
- Spring Framework 7.x 必須
- Jakarta EE 11 (Servlet 6.1 baseline)
- Jackson 3.x への移行 (package namespace の変更)
- Kotlin 2.2+ 必須
- 設定 property の大規模な再編成

## System Requirements

### Minimum Versions

- **Java**: 17+ (推奨は最新 LTS: Java 21 または 25)
- **Kotlin**: 2.2.0 以降
- **Spring Framework**: 7.x (Spring Boot 4.0 が管理)
- **Jakarta EE**: 11 (Servlet 6.1 baseline)
- **GraalVM** (native image 用): 25+
- **Gradle**: 8.5+ (Kotlin DSL と version catalog を使うため)
- **Gradle CycloneDX Plugin**: 3.0.0+

### Verify Compatibility

```bash
# 現在の version を確認
./gradlew --version
./gradlew dependencies --configuration runtimeClasspath
```

## Pre-Migration Steps

### 1. 最新の Spring Boot 3.5.x へ upgrade する

4.0 へ移行する前に、最新の 3.5.x release へ上げる:

```kotlin
// libs.versions.toml
[versions]
springBoot = "3.5.6" # 4.0 へ移行する前の最新 3.x
```

### 2. 非推奨 API を整理する

Spring Boot 3.x で deprecated になっている API の使用をすべて除去する。4.0 では compilation error になる。

```bash
# build して warning を確認
./gradlew clean build --warning-mode all
```

### 3. Dependency 変更を確認する

dependency を次と比較する:
- [Spring Boot 3.5.x Dependency Versions](https://docs.spring.io/spring-boot/3.5/appendix/dependency-versions/coordinates.html)
- [Spring Boot 4.0.x Dependency Versions](https://docs.spring.io/spring-boot/4.0/appendix/dependency-versions/coordinates.html)

## Module Restructuring and Starter Changes

### Critical: Modular Architecture

Spring Boot 4.0 は、大きな monolithic jar を置き換える **より小さく焦点を絞った module** を導入する。このため、多くの project で dependency の更新が必要になる。

**Library Author 向け重要事項:** modular 化と package 再編成により、**同一 artifact で Spring Boot 3 と Spring Boot 4 の両方を support することは強く非推奨** である。runtime conflict を避け、dependency 管理を明確にするため、major version ごとに別 artifact を publish すべきである。

### 移行戦略: 1 つ選ぶ

#### Option 1: Technology-Specific Starters (本番向け推奨)

Spring Boot が扱う大半の technology には、現在 **専用の test starter companion** がある。これにより、より細かい制御ができる。

**Complete Starter Reference:** 利用可能なすべての starter (Core、Web、Database、Spring Data、Messaging、Security、Templating、Production-Ready など) と test companion の完全な table は、[official Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide#starters) を参照。

**libs.versions.toml:**
```toml
[versions]
springBoot = "4.0.0"

[libraries]
# 専用 test module を持つ core starter
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-webmvc", version.ref = "springBoot" }
spring-boot-starter-webmvc-test = { module = "org.springframework.boot:spring-boot-starter-webmvc-test", version.ref = "springBoot" }

spring-boot-starter-data-jpa = { module = "org.springframework.boot:spring-boot-starter-data-jpa", version.ref = "springBoot" }
spring-boot-starter-data-jpa-test = { module = "org.springframework.boot:spring-boot-starter-data-jpa-test", version.ref = "springBoot" }

spring-boot-starter-security = { module = "org.springframework.boot:spring-boot-starter-security", version.ref = "springBoot" }
spring-boot-starter-security-test = { module = "org.springframework.boot:spring-boot-starter-security-test", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    implementation(libs.spring.boot.starter.webmvc)
    implementation(libs.spring.boot.starter.data.jpa)
    implementation(libs.spring.boot.starter.security)

    testImplementation(libs.spring.boot.starter.webmvc.test)
    testImplementation(libs.spring.boot.starter.data.jpa.test)
    testImplementation(libs.spring.boot.starter.security.test)
}
```

#### Option 2: Classic Starters (簡易移行向け、deprecated)

素早く移行したい場合は、Spring Boot 3.x のようにすべての auto-configuration を束ねる **classic starter** を使える:

**libs.versions.toml:**
```toml
[libraries]
spring-boot-starter-classic = { module = "org.springframework.boot:spring-boot-starter-classic", version.ref = "springBoot" }
spring-boot-starter-test-classic = { module = "org.springframework.boot:spring-boot-starter-test-classic", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    implementation(libs.spring.boot.starter.classic)
    testImplementation(libs.spring.boot.starter.test.classic)
}
```

**Warning**: classic starter は **deprecated** であり、将来の release で削除される。technology-specific starter への移行を計画すること。

#### Option 3: Direct Module Dependencies (上級者向け)

transitive dependency を明示的に制御したい場合:

**libs.versions.toml:**
```toml
[libraries]
spring-boot-webmvc = { module = "org.springframework.boot:spring-boot-webmvc", version.ref = "springBoot" }
spring-boot-webmvc-test = { module = "org.springframework.boot:spring-boot-webmvc-test", version.ref = "springBoot" }
```

### Renamed Starters (Breaking Changes)

`libs.versions.toml` 内の starter 名を次のように更新する:

| Spring Boot 3.x | Spring Boot 4.0 | Notes |
|----------------|-----------------|-------|
| `spring-boot-starter-web` | `spring-boot-starter-webmvc` | より明示的な naming |
| `spring-boot-starter-web-services` | `spring-boot-starter-webservices` | hyphen が削除 |
| `spring-boot-starter-aop` | `spring-boot-starter-aspectj` | `org.aspectj.lang.annotation` を使う場合のみ必要 |
| `spring-boot-starter-oauth2-authorization-server` | `spring-boot-starter-security-oauth2-authorization-server` | Security namespace |
| `spring-boot-starter-oauth2-client` | `spring-boot-starter-security-oauth2-client` | Security namespace |
| `spring-boot-starter-oauth2-resource-server` | `spring-boot-starter-security-oauth2-resource-server` | Security namespace |

**Migration Example (libs.versions.toml):**
```toml
[libraries]
# Old (Spring Boot 3.x)
# spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "springBoot" }
# spring-boot-starter-oauth2-client = { module = "org.springframework.boot:spring-boot-starter-oauth2-client", version.ref = "springBoot" }

# New (Spring Boot 4.0)
spring-boot-starter-webmvc = { module = "org.springframework.boot:spring-boot-starter-webmvc", version.ref = "springBoot" }
spring-boot-starter-security-oauth2-client = { module = "org.springframework.boot:spring-boot-starter-security-oauth2-client", version.ref = "springBoot" }
```

### AspectJ Starter Clarification

`spring-boot-starter-aspectj` は **実際に AspectJ annotation を使っている場合のみ** 含める:

```kotlin
// code が org.aspectj.lang.annotation package を使う場合のみ必要
import org.aspectj.lang.annotation.Aspect
import org.aspectj.lang.annotation.Before

@Aspect
class MyAspect {
    @Before("execution(* com.example..*(..))")
    fun beforeAdvice() { }
}
```

AspectJ を使っていないなら dependency を削除する。

## Removed Features and Alternatives

### Embedded Servers

#### Undertow Removed

**Undertow は完全に削除された**。Servlet 6.1 baseline と互換性がない。

**Migration:**
- **Tomcat** (default) または **Jetty** を使う
- Spring Boot 4.0 app を Servlet 6.1 非対応 container に deploy **しない**

**libs.versions.toml:**
```toml
[libraries]
# Undertow を削除
# spring-boot-starter-undertow = { module = "org.springframework.boot:spring-boot-starter-undertow", version.ref = "springBoot" }

# Tomcat (default) または Jetty を使う
spring-boot-starter-jetty = { module = "org.springframework.boot:spring-boot-starter-jetty", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    implementation(libs.spring.boot.starter.webmvc) {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-tomcat")
    }
    implementation(libs.spring.boot.starter.jetty) // Tomcat の代替
}
```

### Session Management

#### Spring Session Hazelcast と MongoDB が削除

**それぞれの team によって管理されるため**、Spring Boot dependency management から外れた。

**Migration (libs.versions.toml):**
```toml
[versions]
hazelcast-spring-session = "3.x.x" # Hazelcast documentation を確認
mongodb-spring-session = "4.x.x"   # MongoDB documentation を確認

[libraries]
# 明示 version が必要
spring-session-hazelcast = { module = "com.hazelcast:spring-session-hazelcast", version.ref = "hazelcast-spring-session" }
spring-session-mongodb = { module = "org.springframework.session:spring-session-data-mongodb", version.ref = "mongodb-spring-session" }
```

### Reactive Messaging

#### Pulsar Reactive Removed

Spring Pulsar は Reactor support を廃止し、reactive Pulsar client が削除された。

**Migration:**
- imperative Pulsar client を使う
- または別の reactive messaging (Kafka、RabbitMQ) へ移行する

### Testing

#### Spock Framework Removed

**Spock はまだ Groovy 5 を support していない** (Spring Boot 4.0 に必要)。

**Migration:**
- Kotlin と JUnit 5 を使う
- または Spock が Groovy 5 互換になるまで待つ

### Build Features

#### Executable Jar Launch Scripts Removed

"fully executable" jar 用の embedded launch script は削除された (Unix 固有で、用途が限定的)。

**build.gradle.kts (削除):**
```kotlin
// この設定を削除
tasks.bootJar {
    launchScript() // もはや support されない
}
```

**Alternatives:**
- `java -jar app.jar` を直接使う
- native launcher には Gradle Application Plugin を使う
- systemd service file を使う

#### Classic Uber-Jar Loader Removed

classic uber-jar loader は削除された。build から loader implementation の設定を削除する。

**Maven (pom.xml) - 削除:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <loaderImplementation>CLASSIC</loaderImplementation> <!-- REMOVE THIS -->
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Gradle (build.gradle.kts) - 削除:**
```kotlin
tasks.bootJar {
    loaderImplementation = org.springframework.boot.loader.tools.LoaderImplementation.CLASSIC // REMOVE THIS
}
```

## Jackson 3 Migration

### Major Breaking Change: Package Namespace

Jackson 3 では **group ID と package 名** が変わる:

| Component | Old (Jackson 2) | New (Jackson 3) |
|-----------|----------------|-----------------|
| Group ID | `com.fasterxml.jackson` | `tools.jackson` |
| Packages | `com.fasterxml.jackson.*` | `tools.jackson.*` |
| Exception | `jackson-annotations` | 引き続き `com.fasterxml.jackson.core` group を使う |

**libs.versions.toml:**
```toml
[versions]
jackson = "3.0.1" # Spring Boot 4.0 が管理

[libraries]
# Jackson 3 は新しい group ID を使う
jackson-databind = { module = "tools.jackson.core:jackson-databind", version.ref = "jackson" }
jackson-module-kotlin = { module = "tools.jackson.module:jackson-module-kotlin", version.ref = "jackson" }

# 例外: annotations は旧 group のまま
jackson-annotations = { module = "com.fasterxml.jackson.core:jackson-annotations", version.ref = "jackson" }
```

### Class と Annotation の Rename

import と annotation を更新する:

| Spring Boot 3.x | Spring Boot 4.0 |
|----------------|-----------------|
| `Jackson2ObjectMapperBuilderCustomizer` | `JsonMapperBuilderCustomizer` |
| `JsonObjectSerializer` | `ObjectValueSerializer` |
| `JsonValueDeserializer` | `ObjectValueDeserializer` |
| `@JsonComponent` | `@JacksonComponent` |
| `@JsonMixin` | `@JacksonMixin` |

**Migration Example:**
```kotlin
// Old (Spring Boot 3.x)
import com.fasterxml.jackson.databind.ObjectMapper
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer
import org.springframework.boot.jackson.JsonComponent

@JsonComponent
class CustomSerializer : JsonSerializer<MyType>() { }

@Configuration
class JacksonConfig {
    @Bean
    fun customizer(): Jackson2ObjectMapperBuilderCustomizer {
        return Jackson2ObjectMapperBuilderCustomizer { builder ->
            builder.simpleDateFormat("yyyy-MM-dd")
        }
    }
}

// New (Spring Boot 4.0)
import tools.jackson.databind.ObjectMapper
import org.springframework.boot.autoconfigure.jackson.JsonMapperBuilderCustomizer
import org.springframework.boot.jackson.JacksonComponent

@JacksonComponent
class CustomSerializer : JsonSerializer<MyType>() { }

@Configuration
class JacksonConfig {
    @Bean
    fun customizer(): JsonMapperBuilderCustomizer {
        return JsonMapperBuilderCustomizer { builder ->
            builder.simpleDateFormat("yyyy-MM-dd")
        }
    }
}
```

### Configuration Property Changes

**application.yml migration:**
```yaml
# Old (Spring Boot 3.x)
spring:
  jackson:
    read:
      enums-using-to-string: true
    write:
      dates-as-timestamps: false

# New (Spring Boot 4.0)
spring:
  jackson:
    json:
      read:
        enums-using-to-string: true
      write:
        dates-as-timestamps: false
```

### Jackson 2 Compatibility Module (一時的)

段階的に移行するには **一時的 compatibility module** を使う (deprecated、将来削除予定):

**libs.versions.toml:**
```toml
[libraries]
spring-boot-jackson2 = { module = "org.springframework.boot:spring-boot-jackson2", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    implementation(libs.spring.boot.jackson2)
}
```

**application.yml:**
```yaml
spring:
  jackson:
    use-jackson2-defaults: true # Jackson 2 の挙動を使う
```

compatibility module を使う場合、property は `spring.jackson2.*` namespace 配下になる。

**この module からの脱却を計画すること**。将来 version で削除される。

## Core Framework Changes

### Nullability Annotations: JSpecify

Spring Boot 4.0 は codebase 全体に **JSpecify nullability annotation** を追加した。

**Impact:**
- Kotlin の null-safety により新しい warning / error が出る可能性がある
- Null checker (SpotBugs、NullAway) が新しい issue を報告する可能性がある
- **`body()` などの RestClient method は明示的に nullable とマークされるようになった**。常に null check するか `Objects.requireNonNull()` を使う

**Migration for Kotlin:**
```kotlin
// nullable 型を明示する必要がある場合がある
fun processUser(id: String?): User? {
    return userRepository.findById(id) // 明示的に nullable になっている可能性がある
}

// RestClient body() は null を返し得る
val body: String? = restClient.get()
    .uri("https://api.example.com/data")
    .retrieve()
    .body(String::class.java) // Nullable - 適切に処理する

if (body != null) {
    println(body.length)
}
```

**Actuator endpoint parameter:**
- `javax.annotations.NonNull` や `org.springframework.lang.Nullable` は使えない
- 代わりに `org.jspecify.annotations.Nullable` を使う

**libs.versions.toml:**
```toml
[libraries]
jspecify = { module = "org.jspecify:jspecify", version = "1.0.0" }
```

### Package Relocations

#### BootstrapRegistry

**Old import:**
```kotlin
import org.springframework.boot.BootstrapRegistry
```

**New import:**
```kotlin
import org.springframework.boot.bootstrap.BootstrapRegistry
```

#### EnvironmentPostProcessor

**Old import:**
```kotlin
import org.springframework.boot.env.EnvironmentPostProcessor
```

**New import:**
```kotlin
import org.springframework.boot.EnvironmentPostProcessor
```

**`META-INF/spring.factories` を更新:**
```properties
# Old
org.springframework.boot.env.EnvironmentPostProcessor=com.example.MyPostProcessor

# New
org.springframework.boot.EnvironmentPostProcessor=com.example.MyPostProcessor
```

**Note:** deprecated 形式は一時的に使えるが、将来削除される。

#### Entity Scan

**Old import:**
```kotlin
import org.springframework.boot.autoconfigure.domain.EntityScan
```

**New import:**
```kotlin
import org.springframework.boot.persistence.autoconfigure.EntityScan
```

### Logging Changes

#### Logback Default Charset

log file は既定で **UTF-8** になる (Log4j2 と統一)。

**logback-spring.xml (明示設定):**
```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>app.log</file>
        <encoder>
            <charset>UTF-8</charset> <!-- これが既定になった -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>
        </encoder>
    </appender>
</configuration>
```

**Console logging:** `Console#charset()` が利用可能ならそれを使い (Java 17+)、そうでなければ UTF-8 に fallback する。これにより、一貫した encoding を保ちつつ platform 互換性が向上する。

### DevTools Changes

#### Live Reload Disabled by Default

**application.yml:**
```yaml
spring:
  devtools:
    livereload:
      enabled: true # 4.0 では明示的に有効化が必要
```

**libs.versions.toml:**
```toml
[libraries]
spring-boot-devtools = { module = "org.springframework.boot:spring-boot-devtools", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    developmentOnly(libs.spring.boot.devtools)
}
```

### PropertyMapper API Behavioral Change

**Breaking change:** source が `null` のとき、既定では adapter / predicate method を呼ばなくなった。

**Migration pattern:**
```kotlin
// Old behavior (Spring Boot 3.x)
map.from(source::method).to(destination::method)
// source が null を返すと destination.method(null) を呼ぶ

// New behavior (Spring Boot 4.0)
map.from(source::method).to(destination::method)
// source が null を返すと呼び出し自体を skip する

// 明示的な null handling (新方式)
map.from(source::method).always().to(destination::method)
// 値が null でも destination.method(value) を常に呼ぶ
```

**Removed method:** `alwaysApplyingNotNull()` - 代わりに `always()` を使う。

**Migration example:** [Spring Boot commit 239f384ac0](https://github.com/spring-projects/spring-boot/commit/239f384ac0893d151b89f204886874c6adb00001) を見て、Spring Boot 本体が新 API にどう対応したか確認するとよい。

## Dependency and Build Changes

### Gradle Plugin Updates

**build.gradle.kts:**
```kotlin
plugins {
    kotlin("jvm") version "2.2.0" // 最低 2.2.0
    kotlin("plugin.spring") version "2.2.0"
    id("org.springframework.boot") version "4.0.0"
    id("io.spring.dependency-management") version "1.1.7"
    id("org.cyclonedx.bom") version "3.0.0" // 最低 3.0.0
}
```

### Optional Dependencies in Gradle

optional dependency は **既定では uber jar に含まれなくなった**。

**build.gradle.kts (必要なら明示的に含める):**
```kotlin
tasks.bootJar {
    includeOptional = true // 必要な場合のみ
}
```

### Spring Retry → Spring Framework Core Retry

Spring Boot 4.0 は **Spring Retry** の dependency management を削除した (portfolio が Spring Framework 7.0 core retry へ移行中のため)。

**Migration Option 1: Spring Framework Core Retry を使う (推奨)**

```kotlin
// 組み込みの Spring Framework retry を使う
import org.springframework.core.retry.RetryTemplate
import org.springframework.core.retry.support.RetryTemplateBuilder

@Configuration
class RetryConfig {
    @Bean
    fun retryTemplate(): RetryTemplate {
        return RetryTemplateBuilder()
            .maxAttempts(3)
            .fixedBackoff(1000)
            .build()
    }
}
```

**Migration Option 2: 明示的な Spring Retry Version (一時対応)**

**libs.versions.toml:**
```toml
[versions]
spring-retry = "2.0.5" # 明示 version が必要

[libraries]
spring-retry = { module = "org.springframework.retry:spring-retry", version.ref = "spring-retry" }
```

**Spring Framework core retry への移行を計画すること。**

### Spring Authorization Server

現在は Spring Security の一部となり、明示的な version 管理が不要になった。

**libs.versions.toml (before - Spring Boot 3.x):**
```toml
[versions]
spring-authorization-server = "1.3.0" # もはや機能しない

[libraries]
spring-security-oauth2-authorization-server = { module = "org.springframework.security:spring-security-oauth2-authorization-server", version.ref = "spring-authorization-server" }
```

**Migration (Spring Boot 4.0):**
```toml
[versions]
spring-security = "7.0.0" # Spring Security version を使う

[libraries]
# 別 version ではなく spring-security.version property で管理される
spring-security-oauth2-authorization-server = { module = "org.springframework.security:spring-security-oauth2-authorization-server", version.ref = "spring-security" }
```

または Spring Boot dependency management に任せる (推奨):
```kotlin
dependencies {
    implementation("org.springframework.security:spring-security-oauth2-authorization-server")
    // Version は Spring Boot 4.0 が管理
}
```

### Elasticsearch Client Changes

#### Low-Level Client Replacement

**deprecated な low-level `RestClient` → 新しい `Rest5Client`:**

**Note:** 上位 client (`ElasticsearchClient` と Spring Data の `ReactiveElasticsearchClient`) は **変更されず**、内部的に新 low-level client を使うよう更新されている。

**Imports:**
```kotlin
// Old (Spring Boot 3.x)
import org.elasticsearch.client.RestClient
import org.elasticsearch.client.RestClientBuilder
import org.springframework.boot.autoconfigure.elasticsearch.RestClientBuilderCustomizer

// New (Spring Boot 4.0)
import co.elastic.clients.transport.rest_client.Rest5Client
import co.elastic.clients.transport.rest_client.Rest5ClientBuilder
import org.springframework.boot.autoconfigure.elasticsearch.Rest5ClientBuilderCustomizer
```

**Configuration:**
```kotlin
@Configuration
class ElasticsearchConfig {

    // Old
    // @Bean
    // fun restClientCustomizer(): RestClientBuilderCustomizer {
    //     return RestClientBuilderCustomizer { builder ->
    //         builder.setRequestConfigCallback { config ->
    //             config.setConnectTimeout(5000)
    //         }
    //     }
    // }

    // New
    @Bean
    fun rest5ClientCustomizer(): Rest5ClientBuilderCustomizer {
        return Rest5ClientBuilderCustomizer { builder ->
            builder.setRequestConfigCallback { config ->
                config.setConnectTimeout(5000)
            }
        }
    }
}
```

**Dependency Consolidation:**

Sniffer は `co.elastic.clients:elasticsearch-java` module に統合された。

**libs.versions.toml:**
```toml
[libraries]
# これらを削除 - もはや管理されない
# elasticsearch-rest-client = { module = "org.elasticsearch.client:elasticsearch-rest-client", version = "..." }
# elasticsearch-rest-client-sniffer = { module = "org.elasticsearch.client:elasticsearch-rest-client-sniffer", version = "..." }

# 単一 dependency を使う (sniffer を含む)
elasticsearch-java = { module = "co.elastic.clients:elasticsearch-java", version = "8.x.x" }
```

### Hibernate Dependency Changes

**libs.versions.toml:**
```toml
[libraries]
# Rename された module (hibernate-jpamodelgen は hibernate-processor に置き換え)
hibernate-processor = { module = "org.hibernate.orm:hibernate-processor", version.ref = "hibernate" }

# これらの artifact は Hibernate によって公開されなくなった:
# hibernate-proxool - Hibernate project 側で廃止
# hibernate-vibur - Hibernate project 側で廃止
# これらへの dependency は削除する
```

**Note:** `hibernate-jpamodelgen` artifact はまだ存在するが deprecated。今後は `hibernate-processor` を使う。

## Configuration Property Changes

### MongoDB Property Restructuring

**大きな再編成:** Spring Data 固有ではない property は `spring.mongodb.*` へ移動した:

**application.yml migration:**
```yaml
# Old (Spring Boot 3.x)
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb
      database: mydb
      host: localhost
      port: 27017
      username: user
      password: pass
      authentication-database: admin
      replica-set-name: rs0
      additional-hosts:
        - host1:27017
        - host2:27017
      ssl:
        enabled: true
        bundle: my-bundle
      representation:
        uuid: STANDARD

management:
  health:
    mongo:
      enabled: true
  metrics:
    mongo:
      command:
        enabled: true
      connectionpool:
        enabled: true

# New (Spring Boot 4.0)
spring:
  mongodb:
    uri: mongodb://localhost:27017/mydb
    database: mydb
    host: localhost
    port: 27017
    username: user
    password: pass
    authentication-database: admin
    replica-set-name: rs0
    additional-hosts:
      - host1:27017
      - host2:27017
    ssl:
      enabled: true
      bundle: my-bundle
    representation:
      uuid: STANDARD # いまは明示設定が必須

  data:
    mongodb:
      # Spring Data 固有の property はここに残る
      auto-index-creation: true
      field-naming-strategy: org.springframework.data.mapping.model.SnakeCaseFieldNamingStrategy
      gridfs:
        bucket: fs
        database: gridfs-db
      repositories:
        type: auto
      representation:
        big-decimal: DECIMAL128 # いまは明示設定が必須

management:
  health:
    mongodb: # "mongo" から rename
      enabled: true
  metrics:
    mongodb: # "mongo" から rename
      command:
        enabled: true
      connectionpool:
        enabled: true
```

**Key changes:**
- **UUID representation**: **必須** - default がないため、`spring.mongodb.representation.uuid` を明示設定しなければならない (`STANDARD`、`JAVA_LEGACY`、`PYTHON_LEGACY`、`C_SHARP_LEGACY` など)
- **BigDecimal representation**: **必須** - default がないため、`spring.data.mongodb.representation.big-decimal` を明示設定しなければならない (`DECIMAL128`、`STRING` など)
- **Management property**: `mongo` → `mongodb`
- **これらを設定しないと、UUID や BigDecimal を永続化するとき runtime error になる**

### Spring Session Property Renames

**application.yml migration:**
```yaml
# Old (Spring Boot 3.x)
spring:
  session:
    redis:
      namespace: myapp:session
      flush-mode: on-save
    mongodb:
      collection-name: sessions

# New (Spring Boot 4.0)
spring:
  session:
    data:
      redis:
        namespace: myapp:session
        flush-mode: on-save
      mongodb:
        collection-name: sessions
```

### Persistence Module Property Change

**application.yml migration:**
```yaml
# Old (Spring Boot 3.x)
spring:
  dao:
    exceptiontranslation:
      enabled: true

# New (Spring Boot 4.0)
spring:
  persistence:
    exceptiontranslation:
      enabled: true
```

## Web Framework Changes

### Static Resource Locations

`PathRequest#toStaticResources()` は既定で `/fonts/**` も含むようになった。

**Security configuration (必要なら font を除外):**
```kotlin
import org.springframework.boot.autoconfigure.security.servlet.PathRequest
import org.springframework.boot.autoconfigure.security.StaticResourceLocation

@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            authorizeHttpRequests {
                // 必要なら font を除外
                authorize(PathRequest.toStaticResources()
                    .atCommonLocations()
                    .excluding(StaticResourceLocation.FONTS), permitAll)
                authorize(anyRequest, authenticated)
            }
        }
        return http.build()
    }
}
```

### HttpMessageConverters Deprecation

`HttpMessageConverters` は framework 改善により deprecated になった (client / server converter が混同されていたため)。

**Migration:**
```kotlin
// Old (Spring Boot 3.x)
import org.springframework.boot.autoconfigure.http.HttpMessageConverters
import org.springframework.context.annotation.Bean

@Configuration
class WebConfig {
    @Bean
    fun customConverters(): HttpMessageConverters {
        return HttpMessageConverters(MyCustomConverter())
    }
}

// New (Spring Boot 4.0)
import org.springframework.boot.autoconfigure.http.client.ClientHttpMessageConvertersCustomizer
import org.springframework.boot.autoconfigure.http.server.ServerHttpMessageConvertersCustomizer

@Configuration
class WebConfig {

    // client と server の converter を分離する
    @Bean
    fun clientConvertersCustomizer(): ClientHttpMessageConvertersCustomizer {
        return ClientHttpMessageConvertersCustomizer { converters ->
            converters.add(MyCustomClientConverter())
        }
    }

    @Bean
    fun serverConvertersCustomizer(): ServerHttpMessageConvertersCustomizer {
        return ServerHttpMessageConvertersCustomizer { converters ->
            converters.add(MyCustomServerConverter())
        }
    }
}
```

### Jersey and Jackson 3 Incompatibility

**Jersey 4.0 の制約:** Spring Boot 4.0 は Jersey 4.0 を support するが、**まだ Jackson 3 を support していない**。

**Solution:** `spring-boot-jackson2` compatibility module を `spring-boot-jackson` の **代わりに、または併用で** 使う:

**libs.versions.toml:**
```toml
[libraries]
spring-boot-starter-jersey = { module = "org.springframework.boot:spring-boot-starter-jersey", version.ref = "springBoot" }
spring-boot-jackson2 = { module = "org.springframework.boot:spring-boot-jackson2", version.ref = "springBoot" }
# 任意: application の Jersey 以外の部分では Jackson 3 を維持する
spring-boot-jackson = { module = "org.springframework.boot:spring-boot-jackson", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    implementation(libs.spring.boot.starter.jersey)
    implementation(libs.spring.boot.jackson2) // Jersey の JSON 処理に必須
    // 任意: application の他部分では Jackson 3 を使う
    // implementation(libs.spring.boot.jackson)
}
```

**Note:** application が Jersey だけを使うなら、Jackson 3 を完全に Jackson 2 compatibility module に置き換えてよい。

## Messaging Framework Changes

### Kafka Streams Customizer Replacement

**deprecated な `StreamBuilderFactoryBeanCustomizer` → `StreamsBuilderFactoryBeanConfigurer`:**

```kotlin
// Old (Spring Boot 3.x)
import org.springframework.boot.autoconfigure.kafka.StreamsBuilderFactoryBeanCustomizer

@Configuration
class KafkaStreamsConfig {
    @Bean
    fun streamsCustomizer(): StreamBuilderFactoryBeanCustomizer {
        return StreamBuilderFactoryBeanCustomizer { factoryBean ->
            factoryBean.setKafkaStreamsCustomizer { streams ->
                // Custom config
            }
        }
    }
}

// New (Spring Boot 4.0)
import org.springframework.kafka.config.StreamsBuilderFactoryBeanConfigurer

@Configuration
class KafkaStreamsConfig {
    @Bean
    fun streamsConfigurer(): StreamsBuilderFactoryBeanConfigurer {
        return StreamsBuilderFactoryBeanConfigurer { factoryBean ->
            factoryBean.setKafkaStreamsCustomizer { streams ->
                // Custom config
            }
        }
    }
}
```

**Note:** 新しい configurer は `Ordered` を実装し、default 値は `0`。

### Kafka Retry Property Change

**application.yml migration:**
```yaml
# Old (Spring Boot 3.x)
spring:
  kafka:
    retry:
      topic:
        backoff:
          random: true

# New (Spring Boot 4.0)
spring:
  kafka:
    retry:
      topic:
        backoff:
          jitter: 0.5 # boolean より柔軟
```

### RabbitMQ Retry Customizer Split

**Spring AMQP は Spring Retry から Spring Framework core retry へ移行し、customizer も分割された:**

```kotlin
// Old (Spring Boot 3.x)
import org.springframework.boot.autoconfigure.amqp.RabbitRetryTemplateCustomizer

@Configuration
class RabbitConfig {
    @Bean
    fun retryCustomizer(): RabbitRetryTemplateCustomizer {
        return RabbitRetryTemplateCustomizer { template ->
            // RabbitTemplate と listener の両方に適用
        }
    }
}

// New (Spring Boot 4.0)
import org.springframework.boot.autoconfigure.amqp.RabbitTemplateRetrySettingsCustomizer
import org.springframework.boot.autoconfigure.amqp.RabbitListenerRetrySettingsCustomizer

@Configuration
class RabbitConfig {

    // RabbitTemplate operation 用
    @Bean
    fun templateRetryCustomizer(): RabbitTemplateRetrySettingsCustomizer {
        return RabbitTemplateRetrySettingsCustomizer { settings ->
            settings.maxAttempts = 5
        }
    }

    // message listener 用
    @Bean
    fun listenerRetryCustomizer(): RabbitListenerRetrySettingsCustomizer {
        return RabbitListenerRetrySettingsCustomizer { settings ->
            settings.maxAttempts = 3
        }
    }
}
```

## Testing Framework Changes

### Mockito Integration Removed

`MockitoTestExecutionListener` は削除された (3.4 で deprecated)。

**MockitoExtension への移行:**
```kotlin
// Old (Spring Boot 3.x)
import org.springframework.boot.test.context.SpringBootTest
import org.mockito.Mock
import org.mockito.Captor

@SpringBootTest
class MyServiceTest {
    @Mock
    private lateinit var repository: MyRepository

    @Captor
    private lateinit var captor: ArgumentCaptor<String>
}

// New (Spring Boot 4.0)
import org.springframework.boot.test.context.SpringBootTest
import org.mockito.Mock
import org.mockito.Captor
import org.mockito.junit.jupiter.MockitoExtension
import org.junit.jupiter.api.extension.ExtendWith

@SpringBootTest
@ExtendWith(MockitoExtension::class) // 明示的 extension が必要
class MyServiceTest {
    @Mock
    private lateinit var repository: MyRepository

    @Captor
    private lateinit var captor: ArgumentCaptor<String>
}
```

### @SpringBootTest Changes

`@SpringBootTest` は **MockMVC**、**WebTestClient**、**TestRestTemplate** を自動提供しなくなった。

#### MockMVC Configuration

```kotlin
// Old (Spring Boot 3.x)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ControllerTest {
    @Autowired
    private lateinit var mockMvc: MockMvc // 自動で利用可能だった
}

// New (Spring Boot 4.0)
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc
import org.springframework.boot.test.autoconfigure.web.servlet.HtmlUnit

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc // 明示 annotation が必要
class ControllerTest {
    @Autowired
    private lateinit var mockMvc: MockMvc
}

// HtmlUnit 設定は annotation attribute に移動
@AutoConfigureMockMvc(
    htmlUnit = HtmlUnit(webClient = false, webDriver = false)
)
```

#### WebTestClient Configuration

```kotlin
// Old (Spring Boot 3.x)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class WebFluxTest {
    @Autowired
    private lateinit var webTestClient: WebTestClient // 自動で利用可能だった
}

// New (Spring Boot 4.0)
import org.springframework.boot.test.autoconfigure.web.reactive.AutoConfigureWebTestClient

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWebTestClient // 明示 annotation が必要
class WebFluxTest {
    @Autowired
    private lateinit var webTestClient: WebTestClient
}
```

#### TestRestTemplate → RestTestClient (推奨)

**Spring Boot 4.0 は `TestRestTemplate` の modern な代替として `RestTestClient` を導入した。**

```kotlin
// Old approach (annotation 付きならまだ動く)
import org.springframework.boot.test.autoconfigure.web.client.AutoConfigureTestRestTemplate
import org.springframework.boot.test.web.client.TestRestTemplate

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestRestTemplate // 4.0 では必須
class RestApiTest {
    @Autowired
    private lateinit var testRestTemplate: TestRestTemplate
}

// New recommended approach
import org.springframework.boot.test.autoconfigure.web.client.AutoConfigureRestTestClient
import org.springframework.boot.resttestclient.RestTestClient

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient // 新しい annotation
class RestApiTest {
    @Autowired
    private lateinit var restTestClient: RestTestClient

    @Test
    fun testEndpoint() {
        val response = restTestClient.get()
            .uri("/api/users")
            .retrieve()
            .toEntity<List<User>>()

        assertThat(response.statusCode).isEqualTo(HttpStatus.OK)
    }
}
```

**TestRestTemplate の package 変更 (使い続ける場合):**

**IMPORTANT:** `TestRestTemplate` を継続利用する場合は次が必要:
1. `spring-boot-resttestclient` の test dependency を追加する
2. **package import を更新する** (class が新 package へ移動した)

**libs.versions.toml:**
```toml
[libraries]
spring-boot-resttestclient = { module = "org.springframework.boot:spring-boot-resttestclient", version.ref = "springBoot" }
```

**build.gradle.kts:**
```kotlin
dependencies {
    testImplementation(libs.spring.boot.resttestclient)
}
```

**Update package import (必須):**
```kotlin
// Old package import - compilation error になる
// import org.springframework.boot.test.web.client.TestRestTemplate

// New package import - Spring Boot 4.0 では必須
import org.springframework.boot.resttestclient.TestRestTemplate
```

### @PropertyMapping Annotation Relocation

```kotlin
// Old (Spring Boot 3.x)
import org.springframework.boot.test.autoconfigure.properties.PropertyMapping
import org.springframework.boot.test.autoconfigure.properties.Skip

// New (Spring Boot 4.0)
import org.springframework.boot.test.context.PropertyMapping
import org.springframework.boot.test.context.PropertyMapping.Skip
```

## Production-Ready Features and Modules

### Health, Metrics, and Observability Modules

Spring Boot 4.0 は production-ready 機能を、より焦点を絞った module に分割した:

**libs.versions.toml:**
```toml
[libraries]
# Health monitoring
spring-boot-health = { module = "org.springframework.boot:spring-boot-health", version.ref = "springBoot" }

# Micrometer metrics
spring-boot-micrometer-metrics = { module = "org.springframework.boot:spring-boot-micrometer-metrics", version.ref = "springBoot" }
spring-boot-micrometer-metrics-test = { module = "org.springframework.boot:spring-boot-micrometer-metrics-test", version.ref = "springBoot" }

# Micrometer observation
spring-boot-micrometer-observation = { module = "org.springframework.boot:spring-boot-micrometer-observation", version.ref = "springBoot" }

# Distributed tracing
spring-boot-micrometer-tracing = { module = "org.springframework.boot:spring-boot-micrometer-tracing", version.ref = "springBoot" }
spring-boot-micrometer-tracing-test = { module = "org.springframework.boot:spring-boot-micrometer-tracing-test", version.ref = "springBoot" }
spring-boot-micrometer-tracing-brave = { module = "org.springframework.boot:spring-boot-micrometer-tracing-brave", version.ref = "springBoot" }
spring-boot-micrometer-tracing-opentelemetry = { module = "org.springframework.boot:spring-boot-micrometer-tracing-opentelemetry", version.ref = "springBoot" }

# OpenTelemetry integration
spring-boot-opentelemetry = { module = "org.springframework.boot:spring-boot-opentelemetry", version.ref = "springBoot" }

# Zipkin reporter
spring-boot-zipkin = { module = "org.springframework.boot:spring-boot-zipkin", version.ref = "springBoot" }
```

**build.gradle.kts (observability stack の例):**
```kotlin
dependencies {
    // metrics と tracing を含む Actuator
    implementation(libs.spring.boot.starter.actuator)
    implementation(libs.spring.boot.micrometer.observation)
    implementation(libs.spring.boot.micrometer.tracing.opentelemetry)
    implementation(libs.spring.boot.opentelemetry)

    // test support
    testImplementation(libs.spring.boot.micrometer.metrics.test)
    testImplementation(libs.spring.boot.micrometer.tracing.test)
}
```

**Note:** starter (例: `spring-boot-starter-actuator`) を使っている大半の application では、これらの module を直接宣言する必要はない。細かく制御したい場合のみ direct module dependency を使う。

## Actuator Changes

### Health Probes Enabled by Default

liveness / readiness probe は **既定で有効** になった。

**application.yml (必要なら無効化):**
```yaml
management:
  endpoint:
    health:
      probes:
        enabled: false # Kubernetes probe を使わないなら無効化
```

**自動で公開される endpoint:**
- `/actuator/health/liveness`
- `/actuator/health/readiness`

## Build Configuration

### Kotlin Compiler Configuration

**build.gradle.kts:**
```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    kotlin("jvm") version "2.2.0" // 最低 2.2.0
    kotlin("plugin.spring") version "2.2.0"
    kotlin("plugin.jpa") version "2.2.0"
    id("org.springframework.boot") version "4.0.0"
    id("io.spring.dependency-management") version "1.1.7"
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21) // または 17、25
    }
}

kotlin {
    compilerOptions {
        freeCompilerArgs.addAll(
            "-Xjsr305=strict", // strict な null-safety
            "-Xemit-jvm-type-annotations" // type annotation を出力
        )
    }
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        jvmTarget = "21" // Java toolchain に合わせる
    }
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### Java Preview Features (Java 25 を使う場合)

**build.gradle.kts:**
```kotlin
tasks.withType<JavaCompile> {
    options.compilerArgs.add("--enable-preview")
}

tasks.withType<Test> {
    jvmArgs("--enable-preview")
}

tasks.withType<JavaExec> {
    jvmArgs("--enable-preview")
}
```

## Migration Checklist

### Pre-Migration

- [ ] 最新の Spring Boot 3.5.x に upgrade する
- [ ] すべての deprecation warning を確認し、修正する
- [ ] 現在の dependency version を記録する
- [ ] test suite 全体を実行し、green build を確認する
- [ ] [Spring Boot 3.5.x → 4.0 dependency changes](https://docs.spring.io/spring-boot/4.0/appendix/dependency-versions/coordinates.html) を確認する

### Core Migration

- [ ] `libs.versions.toml` を Spring Boot 4.0.0 に更新する
- [ ] Kotlin version を 2.2.0+ に更新する
- [ ] starter を rename する: `spring-boot-starter-web` → `spring-boot-starter-webmvc` など
- [ ] technology-specific test starter を追加する (または一時的に classic starter を使う)
- [ ] Undertow dependency があれば削除する (Tomcat / Jetty へ切り替える)
- [ ] `spring-session-hazelcast` / `spring-session-mongodb` を削除するか、明示 version を追加する

### Jackson 3 Migration

- [ ] import を更新する: `com.fasterxml.jackson` → `tools.jackson`
- [ ] 例外に注意: `jackson-annotations` は引き続き `com.fasterxml.jackson.core` を使う
- [ ] `@JsonComponent` → `@JacksonComponent` に rename する
- [ ] `Jackson2ObjectMapperBuilderCustomizer` → `JsonMapperBuilderCustomizer` に rename する
- [ ] property を更新する: `spring.jackson.read.*` → `spring.jackson.json.read.*`
- [ ] 必要なら一時的に `spring-boot-jackson2` module を検討する

### Property Updates

- [ ] MongoDB: `spring.data.mongodb.*` → `spring.mongodb.*` (Spring Data 固有でない property)
- [ ] Session: `spring.session.redis.*` → `spring.session.data.redis.*`
- [ ] Persistence: `spring.dao.exceptiontranslation` → `spring.persistence.exceptiontranslation`
- [ ] Kafka retry: `backoff.random` → `backoff.jitter`

### Code Updates

- [ ] package を更新する: `BootstrapRegistry` → `org.springframework.boot.bootstrap.BootstrapRegistry`
- [ ] package を更新する: `EnvironmentPostProcessor` → `org.springframework.boot.EnvironmentPostProcessor`
- [ ] package を更新する: `EntityScan` → `org.springframework.boot.persistence.autoconfigure.EntityScan`
- [ ] `RestClient` → `Rest5Client` に更新する (Elasticsearch)
- [ ] `StreamBuilderFactoryBeanCustomizer` → `StreamsBuilderFactoryBeanConfigurer` に更新する (Kafka)
- [ ] `RabbitRetryTemplateCustomizer` を `RabbitTemplateRetrySettingsCustomizer` / `RabbitListenerRetrySettingsCustomizer` に分割する
- [ ] `HttpMessageConverters` を `ClientHttpMessageConvertersCustomizer` / `ServerHttpMessageConvertersCustomizer` に置き換える
- [ ] null handling が必要な箇所で `PropertyMapper` に `.always()` を使うよう更新する

### Testing Updates

- [ ] `@Mock` / `@Captor` を使う test に `@ExtendWith(MockitoExtension::class)` を追加する
- [ ] `MockMvc` を使う test に `@AutoConfigureMockMvc` を追加する
- [ ] `WebTestClient` を使う test に `@AutoConfigureWebTestClient` を追加する
- [ ] `TestRestTemplate` を `RestTestClient` へ移行する (または `@AutoConfigureTestRestTemplate` を追加する)
- [ ] `@PropertyMapping` の import を `org.springframework.boot.test.context` へ更新する

### Build Configuration

- [ ] Gradle を 8.5+ に更新する
- [ ] Gradle CycloneDX plugin を 3.0.0+ に更新する
- [ ] uber jar に optional dependency を含める設定を確認する
- [ ] `loaderImplementation = CLASSIC` があれば削除する
- [ ] `launchScript()` 設定があれば削除する

### Verification

- [ ] `./gradlew clean build` を実行する
- [ ] full test suite を実行する
- [ ] TestContainers を使う integration test を確認する
- [ ] Kotlin の新しい null-safety warning を確認する
- [ ] Spring Boot Actuator endpoint を test する
- [ ] health probe (`/actuator/health/liveness`, `/actuator/health/readiness`) を確認する
- [ ] 新 default で performance test を行う

### Post-Migration

- [ ] 追加機能について Spring Boot 4.0 release note を確認する
- [ ] Spring Framework 7.0 の新機能採用を検討する
- [ ] classic starter を使っているなら脱却計画を立てる
- [ ] `spring-boot-jackson2` module を使っているなら脱却計画を立てる
- [ ] Java 17+ 必須に合わせて CI / CD pipeline を更新する
- [ ] deployment manifest を更新する (Servlet 6.1 container)

## Common Pitfalls

1. **Classic starter**: deprecated であることを忘れない。technology-specific starter への移行を計画する
2. **Undertow**: 完全削除。回避策はなく、Tomcat か Jetty を使う必要がある
3. **Jackson 3 package**: `jackson-annotations` だけ旧 group ID のままで見落としやすい
4. **MongoDB property**: 多くが `spring.mongodb.*` に移動したが、一部は `spring.data.mongodb.*` に残る
5. **Test configuration**: `@SpringBootTest` は MockMVC / WebTestClient / TestRestTemplate を自動設定しない
6. **Kotlin 2.2**: 最低要件。古い version では動かない
7. **Null-safety**: JSpecify annotation により Kotlin で新しい warning が表面化することがある
8. **PropertyMapper**: null handling の挙動変更に注意して usage を確認する
9. **Jersey + Jackson 3**: 互換性がない。`spring-boot-jackson2` module を使う
10. **Health probe**: 既定で有効になったため、非 Kubernetes deployment に影響する場合がある

## Performance Considerations

- **Modular starter**: technology-specific starter を使うことで JAR が小さくなり、startup も高速化しやすい
- **Spring Framework 7**: core framework に performance 改善がある
- **Jackson 3**: JSON 処理 performance が向上している
- **Virtual thread**: Java 21+ では有効化を検討する (`spring.threads.virtual.enabled=true`)

## Resources

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/releases)
- [Spring Framework 7.0 Documentation](https://docs.spring.io/spring-framework/reference/)
- [Jackson 3 Migration Guide](https://github.com/FasterXML/jackson/wiki/Jackson-3.0-Migration-Guide)
- [Kotlin 2.2 Release Notes](https://kotlinlang.org/docs/whatsnew22.html)

---
