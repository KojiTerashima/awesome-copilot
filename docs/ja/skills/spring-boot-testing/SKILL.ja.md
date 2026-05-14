---
name: spring-boot-testing
description: Expert Spring Boot 4 testing specialist that selects the best Spring Boot testing techniques for your situation with Junit 6 and AssertJ.
---
# Spring Boot テスト

このスキルは、最新のパターンとベスト プラクティスを使用して Spring Boot 4 アプリケーションをテストするための専門ガイドを提供します。

## 基本原則

1. **テスト ピラミッド**: ユニット (高速) > スライス (集中) > 統合 (完了)
2. **適切なツール**: 自信を与える最も狭いスライスを使用します
3. **AssertJ スタイル**: 冗長マッチャーを介した流暢で読みやすいアサーション
4. **最新の API**: 従来の代替案よりも MockMvcTester と RestTestClient を優先します

## どのテスト スライスですか?

|シナリオ |注釈 |参考資料 |
|----------|-----------|----------|
|コントローラー + HTTP セマンティクス | `@WebMvcTest` | [参照/webmvctest.md](参照/webmvctest.md) |
|リポジトリ + JPA クエリ | `@DataJpaTest` | [参照/datajpatest.md](参照/datajpatest.md) |
| REST クライアント + 外部 API | `@RestClientTest` | [参照/restclienttest.md](参照/restclienttest.md) |
| JSON (逆) シリアル化 | `@JsonTest` | [参照/テストスライス-概要.md](参照/テストスライス-概要.md) |
|完全なアプリケーション | `@SpringBootTest` | [参照/テストスライス-概要.md](参照/テストスライス-概要.md) |

## テスト スライスのリファレンス

- [references/test-slices-overview.md](references/test-slices-overview.md) - 意思決定マトリックスと比較
- [references/webmvctest.md](references/webmvctest.md) - MockMvc を使用した Web 層
- [references/datajpatest.md](references/datajpatest.md) - Testcontainers を使用したデータ層
- [references/restclienttest.md](references/restclienttest.md) - REST クライアントのテスト

## テストツールのリファレンス

- [references/mockmvc-tester.md](references/mockmvc-tester.md) - AssertJ スタイル MockMvc (3.2+)
- [references/mockmvc-classic.md](references/mockmvc-classic.md) - 従来の MockMvc (3.2 より前)
- [references/resttestclient.md](references/resttestclient.md) - Spring Boot 4+ REST クライアント
- [references/mockitobean.md](references/mockitobean.md) - 依存関係のモック化

## アサーション ライブラリ

- [references/assertj-basics.md](references/assertj-basics.md) - スカラー、文字列、ブール値、日付
- [references/assertj-collections.md](references/assertj-collections.md) - リスト、セット、マップ、配列

## テストコンテナ

- [references/testcontainers-jdbc.md](references/testcontainers-jdbc.md) - PostgreSQL、MySQL など

## テストデータの生成

- [references/instacio.md](references/instantio.md) - 複雑なテスト オブジェクト (3 つ以上のプロパティ) を生成します

## パフォーマンスと移行- [references/context-caching.md](references/context-caching.md) - テスト スイートの高速化
- [references/sb4-migration.md](references/sb4-migration.md) - Spring Boot 4.0 の変更点

## クイックデシジョンツリー```
Testing a controller endpoint?
  Yes → @WebMvcTest with MockMvcTester

Testing repository queries?
  Yes → @DataJpaTest with Testcontainers (real DB)

Testing business logic in service?
  Yes → Plain JUnit + Mockito (no Spring context)

Testing external API client?
  Yes → @RestClientTest with MockRestServiceServer

Testing JSON mapping?
  Yes → @JsonTest

Need full integration test?
  Yes → @SpringBootTest with minimal context config
```## Spring Boot 4 のハイライト

- **RestTestClient**: TestRestTemplate の最新の代替品
- **@MockitoBean**: @MockBean を置き換えます (非推奨)
- **MockMvcTester**: Web テスト用の AssertJ スタイルのアサーション
- **モジュール式スターター**: テクノロジー固有のテスト スターター
- **コンテキストの一時停止**: キャッシュされたコンテキストの自動一時停止 (Spring Framework 7)

## テストのベスト プラクティス

### コードの複雑さの評価

メソッドまたはクラスが複雑すぎて効果的にテストできない場合:

1. **複雑さを分析する** - 1 つのメソッドをカバーするために 5 ～ 7 を超えるテスト ケースが必要な場合は、複雑すぎる可能性があります。
2. **リファクタリングを推奨** - コードをより小さな、焦点を絞った関数に分割することを提案します。
3. **ユーザーの決定** - ユーザーがリファクタリングに同意した場合は、抽出ポイントの特定を支援します。
4. **必要に応じて続行** - ユーザーが複雑なコードを続行することにした場合は、困難にもかかわらずテストを実装します。

**リファクタリングの推奨例:**```java
// Before: Complex method hard to test
public Order processOrder(OrderRequest request) {
  // Validation, discount calculation, payment, inventory, notification...
  // 50+ lines of mixed concerns
}

// After: Refactored into testable units
public Order processOrder(OrderRequest request) {
  validateOrder(request);
  var order = createOrder(request);
  applyDiscount(order);
  processPayment(order);
  updateInventory(order);
  sendNotification(order);
  return order;
}
```### コードの冗長性を回避する

よく使用されるオブジェクトのヘルパー メソッドとモック セットアップを作成して、可読性と保守性を向上させます。

### @DisplayName を使用して組織をテストする

テストの意図を明確にするために、わかりやすい表示名を使用します。```java
@Test
@DisplayName("Should calculate discount for VIP customer")
void shouldCalculateDiscountForVip() { }

@Test
@DisplayName("Should reject order when customer has insufficient credit")
void shouldRejectOrderForInsufficientCredit() { }
```### テストカバレッジの順序

テストは常に次の順序で構成してください。

1. **メイン シナリオ** - ハッピー パス、最も一般的な使用例
2. **その他のパス** - 代替の有効なシナリオ、エッジケース
3. **例外/エラー** - 無効な入力、エラー状態、故障モード

### テスト運用シナリオ

実際の運用シナリオを念頭に置いてテストを作成します。これにより、テストがより関連付けられやすくなり、実際の運用ケースでのコードの動作を理解するのに役立ちます。

### テストカバレッジの目標

品質と労力の実際的なバランスとして、コード カバレッジ 80% を目指します。カバレッジが高いほど有益ですが、それだけが目標ではありません。

カバレッジのレポートと追跡には Jacoco Maven プラグインを使用します。


**適用ルール:**
- 最低 80% 以上のカバレッジ
- 実行だけでなく、意味のあるアサーションに焦点を当てる

**何を優先すべきか:**
1. ビジネスクリティカルなパス (支払い処理、注文の検証)
2. 複雑なアルゴリズム (価格設定、割引計算)
3. エラー処理 (例外、エッジケース)
4. 統合ポイント (外部 API、データベース)

## 依存関係 (Spring Boot 4)```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- For WebMvc tests -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webmvc-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- For Testcontainers -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-testcontainers</artifactId>
  <scope>test</scope>
</dependency>
```
