# AssertJ コレクション

コレクションの AssertJ アサーション: `List`、`Set`、`Map`、配列、およびストリーム。

## このリファレンスを使用する場合

- テスト対象の値が `List`、`Set`、`Map`、配列、または `Stream` である
- 複数の要素、その順序、またはその中の特定のフィールドをアサートする必要がある
- `extracting()`、`filteredOn()`、`containsExactly()`、または同様の収集方法を使用している
- 単一のスカラーまたは単一のオブジェクトをアサート → 代わりに [assertj-basics.md](assertj-basics.md) を使用してください

## 基本的なコレクション チェック```java
List<Order> orders = orderService.findAll();

assertThat(orders).isNotEmpty();
assertThat(orders).isEmpty();
assertThat(orders).hasSize(3);
assertThat(orders).hasSizeGreaterThan(0);
assertThat(orders).hasSizeLessThanOrEqualTo(10);
```## 封じ込めアサーション```java
// Contains (any order, allows extras)
assertThat(orders).contains(order1, order2);

// Contains exactly these elements in this order (no extras)
assertThat(statuses).containsExactly("NEW", "PENDING", "COMPLETED");

// Contains exactly these elements in any order (no extras)
assertThat(statuses).containsExactlyInAnyOrder("COMPLETED", "NEW", "PENDING");

// Contains any of these elements (at least one match required)
assertThat(statuses).containsAnyOf("NEW", "CANCELLED");

// Does not contain
assertThat(statuses).doesNotContain("DELETED");
```## フィールドの抽出

アサートする前に、各要素から 1 つのフィールドを抽出します。```java
assertThat(orders)
  .extracting(Order::getStatus)
  .containsExactly("NEW", "PENDING", "COMPLETED");
```複数のフィールドをタプルとして抽出します。```java
assertThat(orders)
  .extracting(Order::getId, Order::getStatus)
  .containsExactly(
    tuple(1L, "NEW"),
    tuple(2L, "PENDING"),
    tuple(3L, "COMPLETED")
  );
```## アサート前のフィルタリング```java
assertThat(orders)
  .filteredOn(order -> order.getStatus().equals("PENDING"))
  .hasSize(2)
  .extracting(Order::getId)
  .containsExactlyInAnyOrder(1L, 3L);

// Filter by field value
assertThat(orders)
  .filteredOn("status", "PENDING")
  .hasSize(2);
```## 述語チェック```java
assertThat(orders).allMatch(o -> o.getTotal().compareTo(BigDecimal.ZERO) > 0);
assertThat(orders).anyMatch(o -> o.getStatus().equals("COMPLETED"));
assertThat(orders).noneMatch(o -> o.getStatus().equals("DELETED"));

// With description for failure messages
assertThat(orders)
  .allSatisfy(o -> assertThat(o.getId()).isPositive());
```## 要素ごとに順序付けされたアサーション

各要素を個別の条件で順番にアサートします。```java
assertThat(orders).satisfiesExactly(
  first  -> assertThat(first.getStatus()).isEqualTo("NEW"),
  second -> assertThat(second.getStatus()).isEqualTo("PENDING"),
  third  -> {
    assertThat(third.getStatus()).isEqualTo("COMPLETED");
    assertThat(third.getTotal()).isGreaterThan(BigDecimal.ZERO);
  }
);
```## ネストされた/フラットなコレクション```java
// flatExtracting: flatten one level of nested collections
assertThat(orders)
  .flatExtracting(Order::getItems)
  .extracting(OrderItem::getProduct)
  .contains("Laptop", "Mouse");
```## 再帰的なフィールドの比較

オブジェクト ID ではなくフィールドによって要素を比較します。```java
assertThat(orders)
  .usingRecursiveFieldByFieldElementComparator()
  .containsExactlyInAnyOrder(expectedOrder1, expectedOrder2);

// Ignore specific fields (e.g. generated IDs or timestamps)
assertThat(orders)
  .usingRecursiveFieldByFieldElementComparatorIgnoringFields("id", "createdAt")
  .containsExactly(expectedOrder1, expectedOrder2);
```## マップアサーション```java
Map<String, Integer> stockByProduct = inventoryService.getStock();

assertThat(stockByProduct)
  .isNotEmpty()
  .hasSize(3)
  .containsKey("Laptop")
  .doesNotContainKey("Fax Machine")
  .containsEntry("Laptop", 10)
  .containsEntries(entry("Laptop", 10), entry("Mouse", 50));

assertThat(stockByProduct)
  .hasEntrySatisfying("Laptop", qty -> assertThat(qty).isGreaterThan(0));
```## 配列アサーション```java
String[] roles = user.getRoles();

assertThat(roles).hasSize(2);
assertThat(roles).contains("ADMIN");
assertThat(roles).containsExactlyInAnyOrder("USER", "ADMIN");
```## アサーションを設定する```java
Set<String> tags = product.getTags();

assertThat(tags).contains("electronics", "sale");
assertThat(tags).doesNotContain("expired");
assertThat(tags).hasSizeGreaterThanOrEqualTo(1);
```## 静的インポート```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.tuple;
import static org.assertj.core.api.Assertions.entry;
```## 重要なポイント

1. **`containsExactly` vs `containsExactlyInAnyOrder`** — 順序が重要な場合は前者を使用します
2. **`extracting()` 包含チェックの前** — ドメイン オブジェクトへの `equals()` の実装を回避します
3. **`filteredOn()` + `extracting()`** — コレクションのサブセットを正確にアサートするように構成します
4. **`satisfiesExactly()`** — 各要素が異なるアサーションを必要とする場合に使用します
5. **`usingRecursiveFieldByFieldElementComparator()`** — DTO およびレコードでは `equals()` よりも優先されます