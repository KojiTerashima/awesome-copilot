---
description: 'Spring Boot ベースアプリケーションを構築するためのガイドライン'
applyTo: '**/*.java, **/*.kt'
---

# Spring Boot 開発

## 一般的な指示

- コード変更をレビューする際は、高い確信が持てる提案だけを行うこと。
- 保守しやすさを意識してコードを書き、特定の設計判断を行った理由はコメントで補足すること。
- エッジケースに対応し、明確な例外処理を書くこと。
- ライブラリや外部依存を使う場合は、その用途と目的をコメントで説明すること。

## Spring Boot に関する指示

### 依存性注入

- 必須の依存関係にはすべてコンストラクターインジェクションを使う。
- 依存フィールドは `private final` で宣言する。

### 設定

- 外部化設定には YAML ファイル (`application.yml`) を使う。
- 環境プロファイル: 環境ごとの差分には Spring profile を使う (dev、test、prod)
- Configuration Properties: 型安全な設定バインディングには @ConfigurationProperties を使う
- シークレット管理: 環境変数またはシークレット管理システムでシークレットを外部化する

### コード構成

- パッケージ構成: レイヤー単位ではなく機能 / ドメイン単位で整理する
- 関心の分離: コントローラーは薄く、サービスは責務を絞り、リポジトリーはシンプルに保つ
- ユーティリティークラス: final にし、private コンストラクターを持たせる

### サービスレイヤー

- 業務ロジックは `@Service` を付与したクラスに置く。
- サービスはステートレスでテストしやすくする。
- リポジトリーはコンストラクター経由で注入する。
- サービスメソッドのシグネチャには、必要がない限りリポジトリーエンティティを直接公開せず、ドメイン ID または DTO を使う。

### ロギング

- すべてのロギングに SLF4J を使う (`private static final Logger logger = LoggerFactory.getLogger(MyClass.class);`)。
- 具体実装 (Logback、Log4j2) や `System.out.println()` を直接使わない。
- パラメーター化ログを使う: `logger.info("User {} logged in", userId);`。

### セキュリティと入力処理

- パラメーター化クエリを使う | SQL インジェクションを防ぐため、常に Spring Data JPA または `NamedParameterJdbcTemplate` を使う。
- リクエストボディーとパラメーターは JSR-380 (`@NotNull`、`@Size` など) アノテーションと `BindingResult` で検証する

## ビルドと検証

- コードを追加または変更した後は、プロジェクトが引き続き正常にビルドできることを確認する。
- プロジェクトが Maven を使っている場合は、`mvn clean package` を実行する。
- プロジェクトが Gradle を使っている場合は、`./gradlew build` (Windows では `gradlew.bat build`) を実行する。
- ビルドの一部として、すべてのテストが通ることを確認する。

## 便利なコマンド

| Gradle Command            | Maven Command                     | Description                         |
|:--------------------------|:----------------------------------|:------------------------------------|
| `./gradlew bootRun`       |`./mvnw spring-boot:run`           | アプリケーションを実行する。        |
| `./gradlew build`         |`./mvnw package`                   | アプリケーションをビルドする。      |
| `./gradlew test`          |`./mvnw test`                      | テストを実行する。                  |
| `./gradlew bootJar`       |`./mvnw spring-boot:repackage`     | アプリケーションを JAR としてパッケージ化する。 |
| `./gradlew bootBuildImage`|`./mvnw spring-boot:build-image`   | アプリケーションをコンテナーイメージとしてパッケージ化する。 |
