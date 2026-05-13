---
name: java-springboot
description: 'Spring Bootでのアプリケーション開発におけるベストプラクティスを紹介します。'
---

# Spring Bootベストプラクティス

高品質なSpring Bootアプリケーションを作成するために、確立されたベストプラクティスに従うことを目標とします。

## プロジェクトのセットアップと構成

- **ビルドツール:** 依存関係管理にはMaven（`pom.xml`）またはGradle（`build.gradle`）を使用します。
- **スターター:** Spring Bootスターター（例：`spring-boot-starter-web`、`spring-boot-starter-data-jpa`）を使って依存関係管理を簡素化します。
- **パッケージ構成:** レイヤー別（例：`com.example.app.controller`、`com.example.app.service`）ではなく、機能やドメイン別（例：`com.example.app.order`、`com.example.app.user`）にコードを整理します。

## 依存性注入とコンポーネント

- **コンストラクタ注入:** 必須の依存関係には常にコンストラクタベースの注入を使用します。これによりコンポーネントのテストが容易になり、依存関係が明示的になります。
- **不変性:** 依存関係のフィールドは`private final`として宣言します。
- **コンポーネントステレオタイプ:** `@Component`、`@Service`、`@Repository`、`@Controller`/`@RestController`アノテーションを適切に使い、Beanを定義します。

## 設定

- **外部化された設定:** 設定には`application.yml`（または`application.properties`）を使用します。YAMLは可読性と階層構造のために好まれます。
- **型安全なプロパティ:** `@ConfigurationProperties`を使って設定を型安全なJavaオブジェクトにバインドします。
- **プロファイル:** Springプロファイル（`application-dev.yml`、`application-prod.yml`）を使い環境ごとの設定を管理します。
- **シークレット管理:** シークレットをハードコードしないでください。環境変数やHashiCorp Vault、AWS Secrets Managerなどの専用シークレット管理ツールを使用します。

## Webレイヤー（コントローラー）

- **RESTful API:** 明確で一貫性のあるRESTfulエンドポイントを設計します。
- **DTO（データ転送オブジェクト）:** APIレイヤーでデータを公開・受信する際はDTOを使用し、JPAエンティティを直接クライアントに公開しないでください。
- **バリデーション:** Java Bean Validation（JSR 380）を使い、DTOに`@Valid`、`@NotNull`、`@Size`などのアノテーションを付けてリクエストペイロードを検証します。
- **エラーハンドリング:** `@ControllerAdvice`と`@ExceptionHandler`を使ってグローバル例外ハンドラーを実装し、一貫したエラー応答を提供します。

## サービスレイヤー

- **ビジネスロジック:** すべてのビジネスロジックは`@Service`クラス内にカプセル化します。
- **ステートレス:** サービスはステートレスであるべきです。
- **トランザクション管理:** データベーストランザクションは`@Transactional`をサービスメソッドに付与して宣言的に管理します。必要な最小単位で適用してください。

## データレイヤー（リポジトリ）

- **Spring Data JPA:** 標準的なデータベース操作には`JpaRepository`や`CrudRepository`を拡張したSpring Data JPAリポジトリを使用します。
- **カスタムクエリ:** 複雑なクエリには`@Query`やJPA Criteria APIを使用します。
- **プロジェクション:** 必要なデータのみを取得するためにDTOプロジェクションを使用します。

## ロギング

- **SLF4J:** ロギングにはSLF4J APIを使用します。
- **ロガー宣言:** `private static final Logger logger = LoggerFactory.getLogger(MyClass.class);`
- **パラメータ化ロギング:** 文字列連結の代わりにパラメータ化メッセージ（`logger.info("Processing user {}...", userId);`）を使いパフォーマンスを向上させます。

## テスト

- **ユニットテスト:** JUnit 5とMockitoなどのモックフレームワークを使い、サービスやコンポーネントのユニットテストを書きます。
- **統合テスト:** Springアプリケーションコンテキストをロードする統合テストには`@SpringBootTest`を使用します。
- **テストスライス:** `@WebMvcTest`（コントローラー用）や`@DataJpaTest`（リポジトリ用）などのテストスライスアノテーションを使い、特定部分を単独でテストします。
- **Testcontainers:** 実際のデータベースやメッセージブローカーなどを使った信頼性の高い統合テストにはTestcontainersの利用を検討してください。

## セキュリティ

- **Spring Security:** 認証と認可にはSpring Securityを使用します。
- **パスワードエンコード:** パスワードは常にBCryptなどの強力なハッシュアルゴリズムでエンコードします。
- **入力サニタイズ:** SQLインジェクションを防ぐためにSpring Data JPAやパラメータ化クエリを使用し、クロスサイトスクリプティング（XSS）を防ぐために出力を適切にエンコードします。
