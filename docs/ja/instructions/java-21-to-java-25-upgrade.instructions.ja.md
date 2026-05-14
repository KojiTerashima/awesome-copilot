---
applyTo: ['*']
description: "Java 21 のリリース以降に導入された Java 25 の新機能を採用するための包括的なベストプラクティス。"
---

# Java 21 から Java 25 へのアップグレードガイド

これらの instruction は、JDK 21 から JDK 25 へ Java project をアップグレードする開発者を GitHub Copilot が支援するためのものです。新しい language feature、API change、ベストプラクティスに焦点を当てています。

## JDK 22-25 における言語機能と API の変更

### Pattern Matching の強化 (JEP 455/488 - 23 では Preview)

**Pattern、`instanceof`、switch における primitive type**

pattern matching を扱うときは:
- switch expression や `instanceof` check で primitive type pattern を使うことを提案する
- 従来の switch からの upgrade 例:
```java
// Old approach (Java 21)
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    default -> "unknown status: " + x.getStatus();
}

// New approach (Java 25 Preview)
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    case int i -> "unknown status: " + i;
}
```

- preview feature は `--enable-preview` flag で有効化する
- より複雑な条件には guard pattern を提案する:
```java
switch (x.getYearlyFlights()) {
    case 0 -> ...;
    case int i when i >= 100 -> issueGoldCard();
    case int i -> ... // handle 1-99 range
}
```

### Class-File API (JEP 466/484 - 23 で Second Preview、25 で正式化)

**ASM を標準 API に置き換える**

bytecode manipulation や class file processing を検出した場合:
- ASM library から標準 Class-File API への移行を提案する
- `org.objectweb.asm` ではなく `java.lang.classfile` package を使う
- migration pattern の例:
```java
// Old ASM approach
ClassReader reader = new ClassReader(classBytes);
ClassWriter writer = new ClassWriter(reader, 0);
// ... ASM manipulation

// New Class-File API approach
ClassModel classModel = ClassFile.of().parse(classBytes);
byte[] newBytes = ClassFile.of().transform(classModel,
    ClassTransform.transformingMethods(methodTransform));
```

### Markdown Documentation Comment (JEP 467 - 23 で正式化)

**JavaDoc のモダナイズ**

JavaDoc comment を扱うときは:
- HTML に依存した JavaDoc を Markdown syntax に変換することを提案する
- Markdown documentation comment には `///` を使う
- 変換例:
```java
// Old HTML JavaDoc
/**
 * Returns the <b>absolute</b> value of an {@code int} value.
 * <p>
 * If the argument is not negative, return the argument.
 * If the argument is negative, return the negation of the argument.
 *
 * @param a the argument whose absolute value is to be determined
 * @return the absolute value of the argument
 */

// New Markdown JavaDoc
/// Returns the **absolute** value of an `int` value.
///
/// If the argument is not negative, return the argument.
/// If the argument is negative, return the negation of the argument.
///
/// @param a the argument whose absolute value is to be determined
/// @return the absolute value of the argument
```

### Derived Record Creation (JEP 468 - 23 では Preview)

**Record の拡張**

record を扱うときは:
- 派生 record を作るために `with` expression の使用を提案する
- derived record creation には preview feature を有効にする
- pattern の例:
```java
// Instead of manual record copying
public record Person(String name, int age, String email) {
    public Person withAge(int newAge) {
        return new Person(name, newAge, email);
    }
}

// Use derived record creation (Preview)
Person updated = person with { age = 30; };
```

### Stream Gatherer (JEP 473/485 - 23 で Second Preview、25 で正式化)

**強化された stream 処理**

複雑な stream 操作を扱うときは:
- custom intermediate operation には `Stream.gather()` の利用を提案する
- 組み込み gatherer には `java.util.stream.Gatherers` を import する
- 使用例:
```java
// Custom windowing operations
List<List<String>> windows = stream
    .gather(Gatherers.windowSliding(3))
    .toList();

// Custom filtering with state
List<Integer> filtered = numbers.stream()
    .gather(Gatherers.fold(0, (state, element) -> {
        // Custom stateful logic
        return state + element > threshold ? element : null;
    }))
    .filter(Objects::nonNull)
    .toList();
```

## 移行時の警告と非推奨事項

### `sun.misc.Unsafe` のメモリアクセス method (JEP 471 - 23 で非推奨)

`sun.misc.Unsafe` の利用を検出した場合:
- 非推奨の memory-access method であることを警告する
- 標準的な代替手段への移行を提案する:
```java
// Deprecated: sun.misc.Unsafe memory access
Unsafe unsafe = Unsafe.getUnsafe();
unsafe.getInt(object, offset);

// Preferred: VarHandle API
VarHandle vh = MethodHandles.lookup()
    .findVarHandle(MyClass.class, "fieldName", int.class);
int value = (int) vh.get(object);

// Or for off-heap: Foreign Function & Memory API
MemorySegment segment = MemorySegment.ofArray(new int[10]);
int value = segment.get(ValueLayout.JAVA_INT, offset);
```

### JNI 利用の警告 (JEP 472 - 24 で警告)

JNI の使用を検出した場合:
- 今後の JNI 制限について警告する
- JNI を使う application には `--enable-native-access` flag の追加を提案する
- 可能なら Foreign Function & Memory API への移行を推奨する
- native access 用に module-info.java の entry を追加する:
```java
module com.example.app {
    requires jdk.unsupported; // for remaining JNI usage
}
```

## Garbage Collection の更新

### ZGC の generational mode (JEP 474 - 23 でデフォルト)

garbage collection を設定するときは:
- default の ZGC は now generational mode を使う
- 非 generational ZGC を明示している場合は JVM flag を更新する:
```bash
# Explicit non-generational mode (will show deprecation warning)
-XX:+UseZGC -XX:-ZGenerational

# Default generational mode
-XX:+UseZGC
```

### G1 の改善 (JEP 475 - 24 で実装)

G1GC を使うときは:
- code change は不要 - JVM 内部最適化である
- C2 compiler で compilation performance が改善する可能性がある

## Vector API (JEP 469 - 25 で 8 回目の Incubator)

数値計算を扱うときは:
- SIMD operation には Vector API の利用を提案する（依然 incubating）
- `--add-modules jdk.incubator.vector` を追加する
- 使用例:
```java
import jdk.incubator.vector.*;

// Traditional scalar computation
for (int i = 0; i < a.length; i++) {
    c[i] = a[i] + b[i];
}

// Vectorized computation
var species = IntVector.SPECIES_PREFERRED;
for (int i = 0; i < a.length; i += species.length()) {
    var va = IntVector.fromArray(species, a, i);
    var vb = IntVector.fromArray(species, b, i);
    var vc = va.add(vb);
    vc.intoArray(c, i);
}
```

## コンパイルと Build 設定

### Preview Feature

preview feature を使う project では:
- compiler argument に `--enable-preview` を追加する
- runtime argument に `--enable-preview` を追加する
- Maven 設定:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <release>25</release>
        <compilerArgs>
            <arg>--enable-preview</arg>
        </compilerArgs>
    </configuration>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>--enable-preview</argLine>
    </configuration>
</plugin>
```

- Gradle 設定:
```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}

tasks.withType<JavaCompile> {
    options.compilerArgs.add("--enable-preview")
}

tasks.withType<Test> {
    jvmArgs("--enable-preview")
}
```

## 移行戦略

### 段階的なアップグレード手順

1. **Build Tool を更新する**: Maven / Gradle が JDK 25 をサポートしていることを確認する
2. **Dependency を更新する**: JDK 25 との互換性を確認する
3. **警告に対応する**: JEP 471 / 472 の非推奨警告に対処する
4. **Preview Feature を有効化する**: pattern matching などの preview feature を使う場合のみ有効にする
5. **十分にテストする**: 特に JNI や `sun.misc.Unsafe` を使う application を重点的に確認する
6. **Performance Test**: 新しい ZGC default 挙動で GC を検証する

### Code Review チェックリスト

Java 25 へのアップグレード用に code を review する際は:
- [ ] ASM の利用を Class-File API に置き換えている
- [ ] 複雑な HTML JavaDoc を Markdown に変換している
- [ ] 適切な箇所で switch expression に primitive pattern を使っている
- [ ] `sun.misc.Unsafe` を VarHandle または FFM API に置き換えている
- [ ] JNI 利用のため native-access permission を追加している
- [ ] 複雑な stream operation には Stream gatherer を使っている
- [ ] preview feature 用の build 設定を更新している

### テスト時の考慮事項

- preview feature は `--enable-preview` flag 付きでテストする
- native access warning が出る JNI application の動作を確認する
- 新しい ZGC generational mode で performance test を行う
- Markdown documentation comment による JavaDoc 生成を検証する

## よくある落とし穴

1. **Preview Feature への依存**: library code では、明確な文書化なしに preview feature を使わない
2. **Native Access**: JNI を直接または間接に使う application では `--enable-native-access` 設定が必要になる可能性がある
3. **Unsafe の移行遅延**: `sun.misc.Unsafe` からの移行を先送りしない。非推奨警告は将来的な削除の前兆である
4. **Pattern Matching の適用範囲**: primitive pattern は `int` だけでなくすべての primitive type で使える
5. **Record の拡張**: derived record creation は Java 23 では preview flag が必要

## パフォーマンスに関する考慮事項

- ZGC generational mode は多くの workload で performance を改善する可能性がある
- Class-File API は ASM 関連のオーバーヘッドを下げる
- Stream gatherer は複雑な stream operation でより高い performance を提供する
- G1GC の改善により JIT compilation overhead が減る

Java 25 へのアップグレードを本番投入する前に、staging 環境で十分にテストしてください。
