---
applyTo: ['*']
description: "Java 17 のリリース以降に導入された Java 21 の新機能を採用するための包括的なベストプラクティス。"
---

# Java 17 から Java 21 へのアップグレードガイド

これらの instruction は、JDK 17 から JDK 21 へ Java project をアップグレードする開発者を GitHub Copilot が支援するためのものです。新しい language feature、API change、ベストプラクティスに焦点を当てています。

## JDK 18-21 の主な言語機能

### switch の Pattern Matching (JEP 441 - 21 で正式化)

**強化された switch Expression と Statement**

switch 構文を扱うときは:
- 適切な箇所では、従来の switch を pattern matching に変換することを提案する
- 型チェックや分解には pattern matching を使う
- 次のような upgrade pattern を使う
```java
// Old approach (Java 17)
public String processObject(Object obj) {
    if (obj instanceof String) {
        String s = (String) obj;
        return s.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return i.toString();
    }
    return "unknown";
}

// New approach (Java 21)
public String processObject(Object obj) {
    return switch (obj) {
        case String s -> s.toUpperCase();
        case Integer i -> i.toString();
        case null -> "null";
        default -> "unknown";
    };
}
```

- guard 付き pattern もサポートする:
```java
switch (obj) {
    case String s when s.length() > 10 -> "Long string: " + s;
    case String s -> "Short string: " + s;
    case Integer i when i > 100 -> "Large number: " + i;
    case Integer i -> "Small number: " + i;
    default -> "Other";
}
```

### Record Pattern (JEP 440 - 21 で正式化)

**Pattern Matching における Record の分解**

record を扱うときは:
- 分解には record pattern の利用を提案する
- 強力な data processing のため switch expression と組み合わせる
- 使用例:
```java
public record Point(int x, int y) {}
public record ColoredPoint(Point point, Color color) {}

// Destructuring in switch
public String describe(Object obj) {
    return switch (obj) {
        case Point(var x, var y) -> "Point at (" + x + ", " + y + ")";
        case ColoredPoint(Point(var x, var y), var color) ->
            "Colored point at (" + x + ", " + y + ") in " + color;
        default -> "Unknown shape";
    };
}
```

- 複雑な pattern matching にも使う:
```java
// Nested record patterns
switch (shape) {
    case Rectangle(ColoredPoint(Point(var x1, var y1), var c1),
                   ColoredPoint(Point(var x2, var y2), var c2))
        when c1 == c2 -> "Monochrome rectangle";
    case Rectangle r -> "Multi-colored rectangle";
}
```

### Virtual Thread (JEP 444 - 21 で正式化)

**軽量 concurrency**

concurrency を扱うときは:
- 高スループットな並行 application には Virtual Thread を提案する
- virtual thread の生成には `Thread.ofVirtual()` を使う
- migration pattern の例:
```java
// Old platform thread approach
ExecutorService executor = Executors.newFixedThreadPool(100);
executor.submit(() -> {
    // blocking I/O operation
    httpClient.send(request);
});

// New virtual thread approach
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // blocking I/O operation - now scales to millions
        httpClient.send(request);
    });
}
```

- structured concurrency pattern を使う:
```java
// Structured concurrency (Preview)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser(userId));
    Future<String> order = scope.fork(() -> fetchOrder(orderId));

    scope.join();           // Join all subtasks
    scope.throwIfFailed();  // Propagate errors

    return processResults(user.resultNow(), order.resultNow());
}
```

### String Template (JEP 430 - 21 では Preview)

**安全な string interpolation**

string formatting を扱うときは:
- 安全な string interpolation のために String Template の利用を提案する（preview feature）
- preview feature は `--enable-preview` で有効化する
- 使用例:
```java
// Traditional concatenation
String message = "Hello, " + name + "! You have " + count + " messages.";

// String Templates (Preview)
String message = STR."Hello, \{name}! You have \{count} messages.";

// Safe HTML generation
String html = HTML."<p>User: \{username}</p>";

// Safe SQL queries
PreparedStatement stmt = SQL."SELECT * FROM users WHERE id = \{userId}";
```

### Sequenced Collection (JEP 431 - 21 で正式化)

**拡張された collection interface**

collection を扱うときは:
- 新しい `SequencedCollection`, `SequencedSet`, `SequencedMap` interface を使う
- collection type を問わず first / last element へ統一的にアクセスする
- 使用例:
```java
// New methods available on Lists, Deques, LinkedHashSet, etc.
List<String> list = List.of("first", "middle", "last");
String first = list.getFirst();  // "first"
String last = list.getLast();    // "last"
List<String> reversed = list.reversed(); // ["last", "middle", "first"]

// Works with any SequencedCollection
SequencedSet<String> set = new LinkedHashSet<>();
set.addFirst("start");
set.addLast("end");
String firstElement = set.getFirst();
```

### Unnamed Pattern と Variable (JEP 443 - 21 では Preview)

**簡潔な pattern matching**

pattern matching を扱うときは:
- 不要な値には unnamed pattern `_` を使う
- switch expression と record pattern を簡潔にする
- 使用例:
```java
// Ignore unused variables
switch (ball) {
    case RedBall(_) -> "Red ball";     // Don't care about size
    case BlueBall(var size) -> "Blue ball size " + size;
}

// Ignore parts of records
switch (point) {
    case Point(var x, _) -> "X coordinate: " + x; // Ignore Y
    case ColoredPoint(Point(_, var y), _) -> "Y coordinate: " + y;
}

// Exception handling with unnamed variables
try {
    riskyOperation();
} catch (IOException | SQLException _) {
    // Don't need exception details
    handleError();
}
```

### Scoped Value (JEP 446 - 21 では Preview)

**改善された context 伝播**

thread-local data を扱うときは:
- ThreadLocal のモダンな代替として Scoped Value を検討する
- virtual thread ではより高い performance と明確な意味論を提供する
- 使用例:
```java
// Define scoped value
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

// Set and use scoped value
ScopedValue.where(USER_ID, "user123")
    .run(() -> {
        processRequest(); // Can access USER_ID.get() anywhere in call chain
    });

// In nested method
public void processRequest() {
    String userId = USER_ID.get(); // "user123"
    // Process with user context
}
```

## API の強化と新機能

### デフォルト UTF-8 (JEP 400 - 18 で正式化)

file I/O を扱うときは:
- すべての platform で UTF-8 が default charset になっている
- 意図が UTF-8 なら、明示的な charset 指定を削除できる
- 単純化の例:
```java
// Old explicit UTF-8 specification
Files.readString(path, StandardCharsets.UTF_8);
Files.writeString(path, content, StandardCharsets.UTF_8);

// New default behavior (Java 18+)
Files.readString(path);  // Uses UTF-8 by default
Files.writeString(path, content);  // Uses UTF-8 by default
```

### Simple Web Server (JEP 408 - 18 で正式化)

基本的な HTTP server が必要な場合:
- 組み込みの `jwebserver` command または `com.sun.net.httpserver` の強化を使う
- test や development に向いている
- 使用例:
```java
// Command line
$ jwebserver -p 8080 -d /path/to/files

// Programmatic usage
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
server.createContext("/", new SimpleFileHandler(Path.of("/tmp")));
server.start();
```

### Internet-Address Resolution SPI (JEP 418 - 19 で正式化)

custom DNS resolution を扱うときは:
- custom address resolution には `InetAddressResolverProvider` を実装する
- service discovery や testing scenario に有用

### Key Encapsulation Mechanism API (JEP 452 - 21 で正式化)

post-quantum cryptography を扱うときは:
- key encapsulation mechanism には KEM API を使う
- 使用例:
```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("ML-KEM");
KeyPair kp = kpg.generateKeyPair();

KEM kem = KEM.getInstance("ML-KEM");
KEM.Encapsulator encapsulator = kem.newEncapsulator(kp.getPublic());
KEM.Encapsulated encapsulated = encapsulator.encapsulate();
```

## 非推奨事項と警告

### Finalization の非推奨化 (JEP 421 - 18 で非推奨)

`finalize()` method を見つけたら:
- finalize method を削除し、代替手段を使う
- Cleaner API または try-with-resources を提案する
- migration の例:
```java
// Deprecated finalize approach
@Override
protected void finalize() throws Throwable {
    cleanup();
}

// Modern approach with Cleaner
private static final Cleaner CLEANER = Cleaner.create();

public MyResource() {
    cleaner.register(this, new CleanupTask(nativeResource));
}

private static class CleanupTask implements Runnable {
    private final long nativeResource;

    CleanupTask(long nativeResource) {
        this.nativeResource = nativeResource;
    }

    public void run() {
        cleanup(nativeResource);
    }
}
```

### Dynamic Agent Loading (JEP 451 - 21 で警告)

agent や instrumentation を扱うときは:
- 必要なら警告を抑えるため `-XX:+EnableDynamicAgentLoading` を追加する
- agent は動的に読み込むより起動時にロードすることを検討する
- tooling は起動時 agent loading に更新する

## Build 設定の更新

### Preview Feature

preview feature を使う project では:
- compiler と runtime の両方に `--enable-preview` を追加する
- Maven 設定:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <release>21</release>
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
        languageVersion = JavaLanguageVersion.of(21)
    }
}

tasks.withType<JavaCompile> {
    options.compilerArgs.add("--enable-preview")
}

tasks.withType<Test> {
    jvmArgs("--enable-preview")
}
```

### Virtual Thread の設定

Virtual Thread を使う application では:
- 追加の JVM flag は不要（21 で正式機能）
- debugging 用に次の system property を検討する:
```bash
-Djdk.virtualThreadScheduler.parallelism=N  # Set carrier thread count
-Djdk.virtualThreadScheduler.maxPoolSize=N  # Set max pool size
```

## Runtime と GC の改善

### Generational ZGC (JEP 439 - 21 で利用可能)

garbage collection を設定するときは:
- performance 向上のため Generational ZGC を試す
- 有効化: `-XX:+UseZGC -XX:+ZGenerational`
- allocation pattern と GC 挙動を監視する

## 移行戦略

### 段階的なアップグレード手順

1. **Build Tool を更新する**: Maven / Gradle が JDK 21 をサポートしていることを確認する
2. **言語機能を採用する**:
   - まずは正式機能である switch の pattern matching から始める
   - 効果がある箇所で record pattern を追加する
   - I/O が重い application では Virtual Thread を検討する
3. **Preview Feature**: 特定 use case に必要な場合にのみ有効化する
4. **Testing**: 特に concurrency 変更に対して包括的にテストする
5. **Performance**: 新しい GC option で benchmark を取る

### Code Review チェックリスト

Java 21 へのアップグレード用に code を review する際は:
- [ ] 適切な `instanceof` chain を switch expression に変換している
- [ ] data の分解に record pattern を使っている
- [ ] 適切な箇所で ThreadLocal を ScopedValue に置き換えている
- [ ] 高 concurrency scenario で Virtual Thread を検討している
- [ ] 明示的な UTF-8 charset 指定を削除している
- [ ] `finalize()` method を Cleaner または try-with-resources に置き換えている
- [ ] first / last access pattern に SequencedCollection method を使っている
- [ ] 実際に使っている preview feature に対してのみ preview flag を追加している

### よくある移行パターン

1. **Switch の強化**:
   ```java
   // From instanceof chains to switch expressions
   if (obj instanceof String s) return processString(s);
   else if (obj instanceof Integer i) return processInt(i);
   // becomes:
   return switch (obj) {
       case String s -> processString(s);
       case Integer i -> processInt(i);
       default -> processDefault(obj);
   };
   ```

2. **Virtual Thread の採用**:
   ```java
   // From platform threads to virtual threads
   Executors.newFixedThreadPool(200)
   // becomes:
   Executors.newVirtualThreadPerTaskExecutor()
   ```

3. **Record Pattern の活用**:
   ```java
   // From manual destructuring to record patterns
   if (point instanceof Point p) {
       int x = p.x();
       int y = p.y();
   }
   // becomes:
   if (point instanceof Point(var x, var y)) {
       // use x and y directly
   }
   ```

## パフォーマンスに関する考慮事項

- Virtual Thread は blocking I/O で特に有効だが、CPU 集約タスクでは効果が薄い場合がある
- Generational ZGC は多くの application で GC オーバーヘッドを下げられる
- switch の pattern matching は一般に `instanceof` chain より効率的
- SequencedCollection method により first / last への O(1) access ができる
- Scoped Value は Virtual Thread で ThreadLocal より低オーバーヘッド

## テストの推奨事項

- Virtual Thread を使う application は高 concurrency 下でテストする
- pattern matching がすべての想定 case を網羅しているか確認する
- Generational ZGC と他 collector を比較して performance test を行う
- 異なる platform 間で UTF-8 default behavior を検証する
- preview feature は本番投入前に十分テストする

preview feature は本当に必要なときだけ有効にし、本番環境へ deploy する前に staging 環境で十分にテストしてください。
