---
name: create-spring-boot-kotlin-project
description: 'Spring Boot Kotlin Project Skeleton を作成します'
---

# Create Spring Boot Kotlin project prompt

- 次の software が system にインストールされていることを確認してください:

  - Java 21
  - Docker
  - Docker Compose

- project 名を変更したい場合は、[download-spring-boot-project-template](./create-spring-boot-kotlin-project.prompt.md#download-spring-boot-project-template) 内の `artifactId` と `packageName` を変更してください

- Spring Boot version を更新したい場合は、[download-spring-boot-project-template](./create-spring-boot-kotlin-project.prompt.md#download-spring-boot-project-template) 内の `bootVersion` を変更してください

## Check Java version

- terminal で次の command を実行し、Java の version を確認する

```shell
java -version
```

## Download Spring Boot project template

- terminal で次の command を実行し、Spring Boot project template をダウンロードする

```shell
curl https://start.spring.io/starter.zip \
  -d artifactId=${input:projectName:demo-kotlin} \
  -d bootVersion=3.4.5 \
  -d dependencies=configuration-processor,webflux,data-r2dbc,postgresql,data-redis-reactive,data-mongodb-reactive,validation,cache,testcontainers \
  -d javaVersion=21 \
  -d language=kotlin \
  -d packageName=com.example \
  -d packaging=jar \
  -d type=gradle-project-kotlin \
  -o starter.zip
```

## Unzip the downloaded file

- terminal で次の command を実行し、ダウンロードした file を展開する

```shell
unzip starter.zip -d ./${input:projectName:demo-kotlin}
```

## Remove the downloaded zip file

- terminal で次の command を実行し、ダウンロードした zip file を削除する

```shell
rm -f starter.zip
```

## Unzip the downloaded file

- terminal で次の command を実行し、ダウンロードした file を展開する

```shell
unzip starter.zip -d ./${input:projectName:demo-kotlin}
```

## Add additional dependencies

- `build.gradle.kts` file に `springdoc-openapi-starter-webmvc-ui` と `archunit-junit5` dependency を追加する

```gradle.kts
dependencies {
  implementation("org.springdoc:springdoc-openapi-starter-webflux-ui:2.8.6")
  testImplementation("com.tngtech.archunit:archunit-junit5:1.2.1")
}
```

- `application.properties` file に SpringDoc configuration を追加する

```properties
# SpringDoc configurations
springdoc.swagger-ui.doc-expansion=none
springdoc.swagger-ui.operations-sorter=alpha
springdoc.swagger-ui.tags-sorter=alpha
```

- `application.properties` file に Redis configuration を追加する

```properties
# Redis configurations
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=rootroot
```

- `application.properties` file に R2DBC configuration を追加する

```properties
# R2DBC configurations
spring.r2dbc.url=r2dbc:postgresql://localhost:5432/postgres
spring.r2dbc.username=postgres
spring.r2dbc.password=rootroot

spring.sql.init.mode=always
spring.sql.init.platform=postgres
spring.sql.init.continue-on-error=true
```

- `application.properties` file に MongoDB configuration を追加する

```properties
# MongoDB configurations
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.username=root
spring.data.mongodb.password=rootroot
spring.data.mongodb.database=test
```

- project root に `docker-compose.yaml` を作成し、`redis:6`、`postgresql:17`、`mongo:8` の service を追加する。

  - redis service は次を持つこと
    - password `rootroot`
    - port 6379 を 6379 に mapping
    - volume `./redis_data` を `/data` に mount
  - postgresql service は次を持つこと
    - password `rootroot`
    - port 5432 を 5432 に mapping
    - volume `./postgres_data` を `/var/lib/postgresql/data` に mount
  - mongo service は次を持つこと
    - initdb root username `root`
    - initdb root password `rootroot`
    - port 27017 を 27017 に mapping
    - volume `./mongo_data` を `/data/db` に mount

- `.gitignore` file に `redis_data`、`postgres_data`、`mongo_data` directory を追加する

- gradle clean test command を実行し、project が動作することを確認する

```shell
./gradlew clean test
```

- （任意）`docker-compose up -d` で service を起動し、`./gradlew spring-boot:run` で Spring Boot project を実行し、`docker-compose rm -sf` で service を停止する。

Let's do this step by step.
