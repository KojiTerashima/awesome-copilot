---
applyTo: ["*"]
description: "Java 11 のリリース以降に導入された Java 17 の新機能を採用するための包括的なベストプラクティス。"
---

# Java 11 から Java 17 へのアップグレードガイド

## プロジェクトコンテキスト

このガイドは、JDK 11 から JDK 17 への Java project のアップグレードに関する包括的な GitHub Copilot 向け instruction です。これらのバージョン間で統合された 47 の JEP に基づき、主要な language feature、API change、migration pattern を扱います。

## 言語機能と API の変更

### JEP 395: Record (Java 16)

**Migration Pattern**: data class を record に変換する

```java
// Old: Traditional data class
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() { return name; }
    public int age() { return age; }

    @Override
    public boolean equals(Object obj) { /* boilerplate */ }
    @Override
    public int hashCode() { /* boilerplate */ }
    @Override
    public String toString() { /* boilerplate */ }
}

// New: Record (Java 16+)
public record Person(String name, int age) {
    // Compact constructor for validation
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }

    // Custom methods can be added
    public boolean isAdult() {
        return age >= 18;
    }
}
```

### JEP 409: Sealed Class (Java 17)

**Migration Pattern**: 継承を制限するために sealed class を使う

```java
// New: Sealed class hierarchy
public sealed class Shape
    permits Circle, Rectangle, Triangle {

    public abstract double area();
}

public final class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public final class Rectangle extends Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}

public non-sealed class Triangle extends Shape {
    // Non-sealed allows further inheritance
    private final double base, height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double area() {
        return 0.5 * base * height;
    }
}
```

### JEP 394: `instanceof` の Pattern Matching (Java 16)

**Migration Pattern**: `instanceof` チェックを簡潔にする

```java
// Old: Traditional instanceof with casting
public String processObject(Object obj) {
    if (obj instanceof String) {
        String str = (String) obj;
        return str.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer num = (Integer) obj;
        return "Number: " + num;
    } else if (obj instanceof List<?>) {
        List<?> list = (List<?>) obj;
        return "List with " + list.size() + " elements";
    }
    return "Unknown type";
}

// New: Pattern matching for instanceof (Java 16+)
public String processObject(Object obj) {
    if (obj instanceof String str) {
        return str.toUpperCase();
    } else if (obj instanceof Integer num) {
        return "Number: " + num;
    } else if (obj instanceof List<?> list) {
        return "List with " + list.size() + " elements";
    }
    return "Unknown type";
}

// Works great with sealed classes
public String describeShape(Shape shape) {
    if (shape instanceof Circle circle) {
        return "Circle with radius " + circle.radius();
    } else if (shape instanceof Rectangle rect) {
        return "Rectangle " + rect.width() + "x" + rect.height();
    } else if (shape instanceof Triangle triangle) {
        return "Triangle with base " + triangle.base();
    }
    return "Unknown shape";
}
```

### JEP 361: Switch Expression (Java 14)

**Migration Pattern**: switch statement を expression に変換する

```java
// Old: Traditional switch statement
public String getDayType(DayOfWeek day) {
    String result;
    switch (day) {
        case MONDAY:
        case TUESDAY:
        case WEDNESDAY:
        case THURSDAY:
        case FRIDAY:
            result = "Workday";
            break;
        case SATURDAY:
        case SUNDAY:
            result = "Weekend";
            break;
        default:
            throw new IllegalArgumentException("Unknown day: " + day);
    }
    return result;
}

// New: Switch expression (Java 14+)
public String getDayType(DayOfWeek day) {
    return switch (day) {
        case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Workday";
        case SATURDAY, SUNDAY -> "Weekend";
    };
}

// With yield for complex logic
public int calculateScore(Grade grade) {
    return switch (grade) {
        case A -> 100;
        case B -> 85;
        case C -> 70;
        case D -> {
            System.out.println("Consider improvement");
            yield 55;
        }
        case F -> {
            System.out.println("Needs retake");
            yield 0;
        }
    };
}
```

### JEP 406: switch の Pattern Matching（Java 17 では Preview）

**Migration Pattern**: pattern を使った拡張 switch（Preview feature）

```java
// Requires --enable-preview flag
public String formatValue(Object obj) {
    return switch (obj) {
        case String s -> "String: " + s;
        case Integer i -> "Integer: " + i;
        case null -> "null value";
        case default -> "Unknown: " + obj.getClass().getSimpleName();
    };
}

// With guarded patterns
public String categorizeNumber(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Negative integer";
        case Integer i when i == 0 -> "Zero";
        case Integer i when i > 0 -> "Positive integer";
        case Double d when d.isNaN() -> "Not a number";
        case Number n -> "Other number: " + n;
        case null -> "null";
        case default -> "Not a number";
    };
}
```

### JEP 378: Text Block (Java 15)

**Migration Pattern**: 複数行 string には text block を使う

```java
// Old: Concatenated strings
String html = "<html>\n" +
              "  <body>\n" +
              "    <h1>Hello World</h1>\n" +
              "    <p>Welcome to Java 17!</p>\n" +
              "  </body>\n" +
              "</html>";

String sql = "SELECT p.id, p.name, p.email, " +
             "       a.street, a.city, a.state " +
             "FROM person p " +
             "JOIN address a ON p.address_id = a.id " +
             "WHERE p.active = true " +
             "ORDER BY p.name";

// New: Text blocks (Java 15+)
String html = """
              <html>
                <body>
                  <h1>Hello World</h1>
                  <p>Welcome to Java 17!</p>
                </body>
              </html>
              """;

String sql = """
             SELECT p.id, p.name, p.email,
                    a.street, a.city, a.state
             FROM person p
             JOIN address a ON p.address_id = a.id
             WHERE p.active = true
             ORDER BY p.name
             """;

// With string interpolation methods
String json = """
              {
                "name": "%s",
                "age": %d,
                "city": "%s"
              }
              """.formatted(name, age, city);
```

### JEP 358: Helpful NullPointerException (Java 14)

**Migration Guidance**: NPE のデバッグ改善（Java 17 ではデフォルト有効）

```java
// Old NPE message: "Exception in thread 'main' java.lang.NullPointerException"
// New NPE message shows exactly what was null:
// "Cannot invoke 'String.length()' because the return value of 'Person.getName()' is null"

public class PersonProcessor {
    public void processPersons(List<Person> persons) {
        // This will show exactly which person.getName() returned null
        persons.stream()
            .mapToInt(person -> person.getName().length())  // Clear NPE if getName() returns null
            .sum();
    }

    // Better error messages help with complex expressions
    public void complexExample(Map<String, List<Person>> groups) {
        // NPE will show exactly which part of the chain is null
        int totalNameLength = groups.get("admins")
                                  .get(0)
                                  .getName()
                                  .length();
    }
}
```

### JEP 371: Hidden Class (Java 15)

**Migration Pattern**: framework や proxy 生成に使う

```java
// For frameworks creating dynamic proxies
public class DynamicProxyExample {
    public static <T> T createProxy(Class<T> interfaceClass, InvocationHandler handler) {
        // Hidden classes provide better encapsulation for dynamically generated classes
        MethodHandles.Lookup lookup = MethodHandles.lookup();

        // Framework code would use hidden classes for better isolation
        // This is typically handled by frameworks, not application code
        return interfaceClass.cast(
            Proxy.newProxyInstance(
                interfaceClass.getClassLoader(),
                new Class<?>[]{interfaceClass},
                handler
            )
        );
    }
}
```

### JEP 334: JVM Constants API (Java 12)

**Migration Pattern**: compile-time constant に使う

```java
import java.lang.constant.*;

// For advanced metaprogramming and tooling
public class ConstantExample {
    // Use dynamic constants for computed values
    public static final DynamicConstantDesc<String> COMPUTED_CONSTANT =
        DynamicConstantDesc.of(
            ConstantDescs.BSM_INVOKE,
            "computeValue",
            ConstantDescs.CD_String
        );

    // Primarily used by compiler and framework developers
    public static String computeValue() {
        return "Computed at runtime, cached as constant";
    }
}
```

### JEP 415: Context-Specific Deserialization Filter (Java 17)

**Migration Pattern**: object deserialization のセキュリティ強化

```java
import java.io.*;

public class SecureDeserialization {
    // Set up deserialization filters for security
    public static void setupSerializationFilters() {
        // Global filter
        ObjectInputFilter globalFilter = ObjectInputFilter.Config.createFilter(
            "java.base/*;java.util.*;!*"
        );
        ObjectInputFilter.Config.setSerialFilter(globalFilter);
    }

    public <T> T deserializeSecurely(byte[] data, Class<T> expectedType) throws IOException, ClassNotFoundException {
        try (ByteArrayInputStream bis = new ByteArrayInputStream(data);
             ObjectInputStream ois = new ObjectInputStream(bis)) {

            // Context-specific filter
            ObjectInputFilter contextFilter = ObjectInputFilter.Config.createFilter(
                expectedType.getName() + ";java.lang.*;!*"
            );
            ois.setObjectInputFilter(contextFilter);

            return expectedType.cast(ois.readObject());
        }
    }
}
```

### JEP 356: Enhanced Pseudo-Random Number Generator (Java 17)

**Migration Pattern**: 新しい random generator interface を使う

```java
import java.util.random.*;

// Old: Limited Random class
Random oldRandom = new Random();
int oldValue = oldRandom.nextInt(100);

// New: Enhanced random generators (Java 17+)
RandomGenerator generator = RandomGeneratorFactory
    .of("Xoshiro256PlusPlus")
    .create(System.nanoTime());

RandomGenerator.SplittableGenerator splittableGenerator =
    RandomGeneratorFactory.of("L64X128MixRandom").create();

// Better for parallel processing
splittableGenerator.splits(4)
    .parallel()
    .mapToInt(rng -> rng.nextInt(1000))
    .forEach(System.out::println);

// Streamable random values
generator.ints(10, 1, 101)
    .forEach(System.out::println);
```

## I/O と Networking の改善

### JEP 380: Unix-Domain Socket Channel (Java 16)

**Migration Pattern**: ローカル IPC には Unix domain socket を使う

```java
import java.net.UnixDomainSocketAddress;
import java.nio.channels.*;

// Old: TCP sockets for local communication
// ServerSocketChannel server = ServerSocketChannel.open();
// server.bind(new InetSocketAddress("localhost", 8080));

// New: Unix domain sockets (Java 16+)
public class UnixSocketExample {
    public void createUnixDomainServer() throws IOException {
        Path socketPath = Path.of("/tmp/my-app.socket");
        UnixDomainSocketAddress address = UnixDomainSocketAddress.of(socketPath);

        try (ServerSocketChannel server = ServerSocketChannel.open(StandardProtocolFamily.UNIX)) {
            server.bind(address);

            while (true) {
                try (SocketChannel client = server.accept()) {
                    // Handle client connection
                    handleClient(client);
                }
            }
        }
    }

    public void connectToUnixSocket() throws IOException {
        Path socketPath = Path.of("/tmp/my-app.socket");
        UnixDomainSocketAddress address = UnixDomainSocketAddress.of(socketPath);

        try (SocketChannel client = SocketChannel.open(address)) {
            // Communicate with server
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            client.read(buffer);
        }
    }

    private void handleClient(SocketChannel client) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        int bytesRead = client.read(buffer);
        // Process client data
    }
}
```

### JEP 352: Non-Volatile Mapped Byte Buffer (Java 14)

**Migration Pattern**: 永続メモリ操作に使う

```java
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.StandardOpenOption;

public class PersistentMemoryExample {
    public void usePersistentMemory() throws IOException {
        Path nvmFile = Path.of("/mnt/pmem/data.bin");

        try (FileChannel channel = FileChannel.open(nvmFile,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE,
                StandardOpenOption.CREATE)) {

            // Map as persistent memory
            MappedByteBuffer buffer = channel.map(
                FileChannel.MapMode.READ_WRITE, 0, 1024,
                ExtendedMapMode.READ_WRITE_SYNC
            );

            // Write data that persists across crashes
            buffer.putLong(0, System.currentTimeMillis());
            buffer.putInt(8, 12345);

            // Force write to persistent storage
            buffer.force();
        }
    }
}
```

## Build System 設定

### Maven 設定

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <maven.compiler.release>17</maven.compiler.release>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <release>17</release>
                <!-- Enable preview features if using JEP 406 -->
                <compilerArgs>
                    <arg>--enable-preview</arg>
                </compilerArgs>
            </configuration>
        </plugin>

        <!-- For running tests with preview features -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <argLine>--enable-preview</argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle 設定

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

tasks.withType<JavaCompile> {
    options.release.set(17)
    // Enable preview features if needed
    options.compilerArgs.addAll(listOf("--enable-preview"))
}

tasks.withType<Test> {
    useJUnitPlatform()
    // Enable preview features for tests
    jvmArgs("--enable-preview")
}
```

## 非推奨化と削除

### JEP 411: Security Manager の削除予定に向けた非推奨化

**Migration Pattern**: Security Manager 依存を除去する

```java
// Old: Using Security Manager
SecurityManager sm = System.getSecurityManager();
if (sm != null) {
    sm.checkPermission(new RuntimePermission("shutdownHooks"));
}

// New: Alternative security approaches
// Use application-level security, containers, or process isolation
// Most applications don't need Security Manager functionality
```

### JEP 398: Applet API の削除予定に向けた非推奨化

**Migration Pattern**: Applet からモダンな web 技術へ移行する

```java
// Old: Java Applet (deprecated)
public class MyApplet extends Applet {
    @Override
    public void start() {
        // Applet code
    }
}

// New: Modern alternatives
// 1. Convert to standalone Java application
public class MyApplication extends JFrame {
    public MyApplication() {
        setTitle("My Application");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Application code
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new MyApplication().setVisible(true);
        });
    }
}

// 2. Use Java Web Start alternative (jlink)
// 3. Convert to web application using modern frameworks
```

### JEP 372: Nashorn JavaScript Engine の削除

**Migration Pattern**: 別の JavaScript engine を使う

```java
// Old: Nashorn (removed in Java 17)
// ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

// New: Alternative approaches
// 1. Use GraalVM JavaScript engine
ScriptEngine engine = new ScriptEngineManager().getEngineByName("graal.js");

// 2. Use external JavaScript execution
ProcessBuilder pb = new ProcessBuilder("node", "script.js");
Process process = pb.start();

// 3. Use web-based approach or embedded browser
```

## JVM とパフォーマンスの改善

### JEP 377: ZGC - A Scalable Low-Latency Garbage Collector (Java 15)

**Migration Pattern**: 低レイテンシ application では ZGC を有効にする

```bash
# Enable ZGC
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions  # Not needed in Java 17

# Monitor ZGC performance
-XX:+LogVMOutput
-XX:LogFile=gc.log
```

### JEP 379: Shenandoah - A Low-Pause-Time Garbage Collector (Java 15)

**Migration Pattern**: 一貫したレイテンシのため Shenandoah を有効にする

```bash
# Enable Shenandoah
-XX:+UseShenandoahGC
-XX:+UnlockExperimentalVMOptions  # Not needed in Java 17

# Shenandoah tuning
-XX:ShenandoahGCHeuristics=adaptive
```

### JEP 341: Default CDS Archive (Java 12) と JEP 350: Dynamic CDS Archive (Java 13)

**Migration Pattern**: 起動性能を改善する

```bash
# CDS is enabled by default, but you can create custom archives
# Create custom CDS archive
java -XX:DumpLoadedClassList=classes.lst -cp myapp.jar com.example.Main
java -Xshare:dump -XX:SharedClassListFile=classes.lst -XX:SharedArchiveFile=myapp.jsa -cp myapp.jar

# Use custom CDS archive
java -XX:SharedArchiveFile=myapp.jsa -cp myapp.jar com.example.Main
```

## テストと移行戦略

### Phase 1: 基盤整備（1〜2 週目）

1. **build system を更新する**

   - Maven / Gradle 設定を Java 17 向けに変更する
   - CI / CD pipeline を更新する
   - dependency の互換性を確認する

2. **削除事項と非推奨事項に対応する**
   - Nashorn JavaScript engine の利用を除去する
   - 非推奨 Applet API を置き換える
   - Security Manager 利用を更新する

### Phase 2: 言語機能（3〜4 週目）

1. **Record を導入する**

   - data class を record に変換する
   - compact constructor に validation を追加する
   - serialization 互換性をテストする

2. **Pattern Matching を追加する**
   - `instanceof` chain を変換する
   - type-safe な cast pattern を実装する

### Phase 3: 高度な機能（5〜6 週目）

1. **Switch Expression**

   - switch statement を expression に変換する
   - 新しい arrow syntax を使う
   - 複雑な `yield` ロジックを実装する

2. **Text Block**
   - 複数行 string の連結を置き換える
   - SQL や HTML 生成を更新する
   - formatting method を使う

### Phase 4: Sealed Class（7〜8 週目）

1. **sealed hierarchy を設計する**

   - 継承制約が必要な箇所を特定する
   - sealed class pattern を実装する
   - pattern matching と組み合わせる

2. **テストと検証**
   - 包括的な test coverage
   - performance benchmark
   - 互換性検証

## パフォーマンスに関する考慮事項

### Record と従来 class の比較

- Record はメモリ効率がよい
- 生成と equality check が高速
- 自動的に serialization をサポートする
- DTO には特に適している

### Pattern Matching のパフォーマンス

- 重複した型チェックを排除できる
- cast のオーバーヘッドを減らせる
- JVM 最適化の余地が増える
- exhaustiveness が取れるため sealed class と相性がよい

### Switch Expression の最適化

- より効率的な bytecode を生成できる
- constant folding が効きやすい
- branch prediction が改善される
- 複雑な条件分岐には特に有効

## ベストプラクティス

1. **data class には Record を使う**

   - immutable な data container
   - API の data transfer object
   - configuration object

2. **Pattern Matching を戦略的に適用する**

   - `instanceof` chain を置き換える
   - sealed class と組み合わせる
   - switch expression と合わせて使う

3. **複数行コンテンツには Text Block を採用する**

   - SQL query
   - JSON template
   - HTML content
   - configuration file

4. **Sealed Class を前提に設計する**

   - domain modeling
   - state machine
   - algebraic data type
   - API の進化制御

5. **拡張 random generator を活用する**
   - 並列処理シナリオ
   - 高品質な乱数
   - 統計用途
   - ゲームや simulation

この包括的ガイドにより、GitHub Copilot は Java 11 project を Java 17 へアップグレードする際に、言語強化、API 改善、モダンな Java 開発実践に基づいた適切な提案を行えるようになります。
