---
name: kotlin-springboot
description: 'Spring BootとKotlinでのアプリケーション開発におけるベストプラクティスを紹介します。'
---

# KotlinによるSpring Bootベストプラクティス

高品質でイディオマティックなSpring BootアプリケーションをKotlinで書くための支援をします。

## プロジェクトのセットアップと構成

- **ビルドツール:** Kotlinプラグイン（`kotlin-maven-plugin`や`org.jetbrains.kotlin.jvm`）を使ったMaven（`pom.xml`）またはGradle（`build.gradle`）を使用します。
- **Kotlinプラグイン:** JPAを使う場合は、エンティティクラスを自動的に`open`にする`kotlin-jpa`プラグインを有効にします。
- **スターター:** 通常通りSpring Bootスターター（例：`spring-boot-starter-web`、`spring-boot-starter-data-jpa`）を使用します。
- **パッケージ構成:** レイヤー別ではなく、機能やドメイン別（例：`com.example.app.order`、`com.example.app.user`）にコードを整理します。

## 依存性注入とコンポーネント

- **プライマリコンストラクタ:** 必須の依存性注入には常にプライマリコンストラクタを使います。Kotlinで最もイディオマティックかつ簡潔な方法です。
- **不変性:** 依存性はプライマリコンストラクタ内で`private val`として宣言します。どこでも`var`より`val`を優先し、不変性を促進します。
- **コンポーネントステレオタイプ:** Javaと同様に`@Service`、`@Repository`、`@RestController`アノテーションを使用します。

## 設定

- **外部設定:** 読みやすく階層構造を持つ`application.yml`を使用します。
- **型安全なプロパティ:** `@ConfigurationProperties`と`data class`を使い、不変で型安全な設定オブジェクトを作成します。
- **プロファイル:** Springプロファイル（`application-dev.yml`、`application-prod.yml`）を使って環境ごとの設定を管理します。
- **シークレット管理:** シークレットをハードコードしないでください。環境変数やHashiCorp Vault、AWS Secrets Managerなどの専用シークレット管理ツールを使用します。

## Webレイヤー（コントローラー）

- **RESTful API:** 明確で一貫性のあるRESTfulエンドポイントを設計します。
- **DTO用データクラス:** すべてのDTOにKotlinの`data class`を使います。これにより`equals()`、`hashCode()`、`toString()`、`copy()`が自動生成され、不変性が促進されます。
- **バリデーション:** DTOのデータクラスにJava Bean Validation（JSR 380）のアノテーション（`@Valid`、`@NotNull`、`@Size`）を使用します。
- **エラーハンドリング:** 一貫したエラーレスポンスのために`@ControllerAdvice`と`@ExceptionHandler`を使ったグローバル例外ハンドラーを実装します。

## サービスレイヤー

- **ビジネスロジック:** ビジネスロジックは`@Service`クラス内にカプセル化します。
- **ステートレス:** サービスはステートレスであるべきです。
- **トランザクション管理:** サービスメソッドに`@Transactional`を使用します。Kotlinではクラスレベルまたは関数レベルに適用可能です。

## データレイヤー（リポジトリ）

- **JPAエンティティ:** エンティティはクラスとして定義し、`open`である必要があります。これを自動化するために`kotlin-jpa`コンパイラプラグインの使用を強く推奨します。
- **ヌル安全:** Kotlinのヌル安全機能（`?`）を活用し、エンティティのフィールドが必須かオプションかを型レベルで明確にします。
- **Spring Data JPA:** `JpaRepository`や`CrudRepository`を継承してSpring Data JPAリポジトリを使用します。
- **コルーチン:** リアクティブアプリケーションの場合、データレイヤーでSpring BootのKotlinコルーチン対応を活用します。

## ロギング

- **コンパニオンオブジェクトのロガー:** ロガーはコンパニオンオブジェクト内で宣言するのがイディオマティックな方法です。
  ```kotlin
  companion object {
      private val logger = LoggerFactory.getLogger(MyClass::class.java)
  }
  ```
- **パラメータ化ロギング:** パフォーマンスと明瞭さのためにパラメータ化メッセージ（`logger.info("Processing user {}...", userId)`）を使用します。

## テスト

- **JUnit 5:** JUnit 5がデフォルトで、Kotlinとシームレスに動作します。
- **イディオマティックなテストライブラリ:** より流暢でイディオマティックなテストには、Kotlin向けに設計された**Kotest**（アサーション用）と**MockK**（モック用）を検討してください。より表現力豊かな構文を提供します。
- **テストスライス:** `@WebMvcTest`や`@DataJpaTest`などのテストスライスアノテーションを使い、アプリケーションの特定部分をテストします。
- **Testcontainers:** 実際のデータベースやメッセージブローカーなどを使った信頼性の高い統合テストにはTestcontainersを使用します。

## コルーチンと非同期プログラミング

- **`suspend`関数:** 非ブロッキングの非同期コードには、コントローラーやサービスで`suspend`関数を使います。Spring Bootはコルーチンを優れた形でサポートしています。
- **構造化並行性:** コルーチンのライフサイクル管理には`coroutineScope`や`supervisorScope`を使用します。
