---
description: 'Databricks の style guide に従った、Scala 2.12 / 2.13 の関数型 programming、型安全性、本番 code 品質のためのコーディング規約とベストプラクティス。'
applyTo: '**/*.scala, **/build.sbt, **/build.sc'
---

# Scala ベストプラクティス

[Databricks Scala Style Guide](https://github.com/databricks/scala-style-guide) に基づく。

## 中核原則

### シンプルな code を書く
code は 1 回書かれても、何度も読まれ、修正される。シンプルな code を書いて、長期的な可読性と保守性を最適化すること。

### デフォルトで不変にする
- 常に `var` より `val` を優先する
- `scala.collection.immutable` の不変 collection を使う
- case class の constructor parameter を mutable にしてはならない
- 値を変更したい場合は copy constructor を使う

```scala
// 良い例 - immutable な case class
case class Person(name: String, age: Int)

// 悪い例 - mutable な case class
case class Person(name: String, var age: Int)

// 値を変更するには copy constructor を使う
val p1 = Person("Peter", 15)
val p2 = p1.copy(age = 16)

// 良い例 - immutable な collection
val users = List(User("Alice", 30), User("Bob", 25))
val updatedUsers = users.map(u => u.copy(age = u.age + 1))
```

### 純粋関数
- function は決定的で副作用がないべきである
- 純粋なロジックと effect を分離する
- effect を持つ method には明示的な型を使う

```scala
// 良い例 - 純粋関数
def calculateTotal(items: List[Item]): BigDecimal =
  items.map(_.price).sum

// 悪い例 - 副作用を持つ impure function
def calculateTotal(items: List[Item]): BigDecimal = {
  println(s"${items.size} 件に対して合計を計算中")  // 副作用
  val total = items.map(_.price).sum
  saveToDatabase(total)  // 副作用
  total
}
```

## 命名規則

### Class と Object

```scala
// class、trait、object - PascalCase
class ClusterManager
trait Expression
object Configuration

// package - すべて小文字 ASCII
package com.databricks.resourcemanager

// method / function - camelCase
def getUserById(id: Long): Option[User]
def processData(input: String): Result

// 定数 - companion object では大文字
object Configuration {
  val DEFAULT_PORT = 10000
  val MAX_RETRIES = 3
  val TIMEOUT_MS = 5000L
}
```

### 変数と Parameter

```scala
// 変数 - camelCase、自己説明的な名前
val serverPort = 1000
val clientPort = 2000
val maxRetryAttempts = 3

// 小さく局所的な scope なら 1 文字名も可
for (i <- 0 until 10) {
  // ...
}

// "l" (Larry) は使わない - "1"、"|"、"I" に見える
```

### Enumeration

```scala
// Enumeration object - PascalCase
// 値 - UPPER_CASE と underscore
private object ParseState extends Enumeration {
  type ParseState = Value

  val PREFIX,
      TRIM_BEFORE_SIGN,
      SIGN,
      VALUE,
      UNIT_BEGIN,
      UNIT_END = Value
}
```

## 構文スタイル

### 行長と Spacing

```scala
// 行長は 100 文字以内に抑える
// 演算子の前後は 1 スペース
def add(int1: Int, int2: Int): Int = int1 + int2

// カンマの後は 1 スペース
val list = List("a", "b", "c")

// コロンの後は 1 スペース
def getConf(key: String, defaultValue: String): String = {
  // code
}

// インデントは 2 スペース
if (true) {
  println("Wow!")
}

// 長い parameter list は 4 スペースでインデント
def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
    path: String,
    fClass: Class[F],
    kClass: Class[K],
    vClass: Class[V],
    conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
  // method body
}

// parameter が長い class
class Foo(
    val param1: String,  // 4 スペース indent
    val param2: String,
    val param3: Array[Byte])
  extends FooInterface  // 2 スペース indent
  with Logging {

  def firstMethod(): Unit = { ... }  // 上に空行を入れる
}
```

### Rule of 30

- method は 30 行未満にする
- class は 30 method 未満にする

### 波かっこ

```scala
// 複数行 block では常に波かっこを使う
if (true) {
  println("Wow!")
}

// 例外: 1 行の ternary 相当 (副作用なし)
val result = if (condition) value1 else value2

// try-catch では必ず波かっこを使う
try {
  foo()
} catch {
  case e: Exception => handle(e)
}
```

### Long Literal

```scala
// long literal には大文字 L を使う
val longValue = 5432L  // これが正しい
val badValue = 5432l   // 見分けにくいので避ける
```

### 括弧

```scala
// 副作用を持つ method - 括弧を使う
class Job {
  def killJob(): Unit = { ... }  // 正しい - 状態を変える
  def getStatus: JobStatus = { ... }  // 正しい - 副作用なし
}

// 呼び出し側も宣言に合わせる
new Job().killJob()  // 正しい
new Job().getStatus  // 正しい
```

### Import

```scala
// wildcard import は 6 個以上の entity を import する場合以外は避ける
import scala.collection.mutable.{Map, HashMap, ArrayBuffer}

// implicits または 6 個以上なら wildcard でもよい
import scala.collection.JavaConverters._
import java.util.{Map, HashMap, List, ArrayList, Set, HashSet}

// 常に絶対 path を使う
import scala.util.Random  // 良い
// import util.Random     // relative は使わない

// import 順序 (空行を入れる):
import java.io.File
import javax.servlet.http.HttpServlet

import scala.collection.mutable.HashMap
import scala.util.Random

import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD

import com.databricks.MyClass
```

### Pattern Matching

```scala
// method 全体が pattern match なら、同じ行に match を置く
def test(msg: Message): Unit = msg match {
  case TextMessage(text) => handleText(text)
  case ImageMessage(url) => handleImage(url)
}

// 単一 case の closure - 同じ行
list.zipWithIndex.map { case (elem, i) =>
  // 処理
}

// 複数 case - 改行して indent
list.map {
  case a: Foo => processFoo(a)
  case b: Bar => processBar(b)
  case _ => handleDefault()
}

// 型だけで match する場合 - 全 arg を展開しない
case class Pokemon(name: String, weight: Int, hp: Int, attack: Int, defense: Int)

// 悪い例 - field 変更に弱い
targets.foreach {
  case Pokemon(_, _, hp, _, defense) =>
    // 壊れやすい
}

// 良い例 - 型で match
targets.foreach {
  case p: Pokemon =>
    val loss = math.min(0, myAttack - p.defense)
    p.copy(hp = p.hp - loss)
}
```

### Anonymous Function

```scala
// 過剰な括弧は避ける
// 正しい
list.map { item =>
  transform(item)
}

// 正しい
list.map(item => transform(item))

// 誤り - 不要な波かっこ
list.map(item => {
  transform(item)
})

// 誤り - 過剰なネスト
list.map({ item => ... })
```

### Infix Method

```scala
// 記号以外の method に infix は避ける
list.map(func)  // 正しい
list map func   // 誤り

// 演算子なら可
arrayBuffer += elem
```

## Language Feature

### Class で apply() を避ける

```scala
// class 上の apply は避ける - 追跡しにくい
class TreeNode {
  def apply(name: String): TreeNode = { ... }  // こうしない
}

// companion object 上の factory としてなら可
object TreeNode {
  def apply(name: String): TreeNode = new TreeNode(name)  // OK
}
```

### override Modifier

```scala
// abstract method に対しても常に override を使う
trait Parent {
  def hello(data: Map[String, String]): Unit
}

class Child extends Parent {
  // override がないと、実際には override していない可能性がある
  override def hello(data: Map[String, String]): Unit = {
    println(data)
  }
}
```

### Constructor で Destructuring を避ける

```scala
// constructor で destructuring bind を使わない
class MyClass {
  // 悪い例 - non-transient な Tuple2 を作る
  @transient private val (a, b) = someFuncThatReturnsTuple2()

  // 良い例
  @transient private val tuple = someFuncThatReturnsTuple2()
  @transient private val a = tuple._1
  @transient private val b = tuple._2
}
```

### Call-by-Name を避ける

```scala
// call-by-name parameter は避ける
// 悪い例 - 呼び出し側から 1 回なのか複数回なのか分からない
def print(value: => Int): Unit = {
  println(value)
  println(value + 1)
}

// 良い例 - 明示的な function type
def print(value: () => Int): Unit = {
  println(value())
  println(value() + 1)
}
```

### 複数 Parameter List を避ける

```scala
// 複数 parameter list は避ける (implicit を除く)
// 悪い例
case class Person(name: String, age: Int)(secret: String)

// 良い例
case class Person(name: String, age: Int, secret: String)

// 例外: implicit 用の別 list (ただし implicit 自体も避けたい)
def foo(x: Int)(implicit ec: ExecutionContext): Future[Int]
```

### Symbolic Method

```scala
// 算術演算子にだけ使う
class Vector {
  def +(other: Vector): Vector = { ... }  // OK
  def -(other: Vector): Vector = { ... }  // OK
}

// その他の method には使わない
// 悪い例
channel ! msg
stream1 >>= stream2

// 良い例
channel.send(msg)
stream1.join(stream2)
```

### Type Inference

```scala
// public method には常に型を付ける
def getUserById(id: Long): Option[User] = { ... }

// implicit method にも常に型を付ける
implicit def stringToInt(s: String): Int = s.toInt

// 明白でない variable には型を付ける (3 秒ルール)
val user: User = complexComputation()

// 明白なら省略してよい
val count = 5
val name = "Alice"
```

### return Statement

```scala
// closure 内で return を避ける - 内部的には exception を使う
def receive(rpc: WebSocketRPC): Option[Response] = {
  tableFut.onComplete { table =>
    if (table.isFailure) {
      return None  // こうしない - thread も誤る
    }
  }
}

// 制御 flow を単純化する guard として return を使うのは可
def doSomething(obj: Any): Any = {
  if (obj eq null) {
    return null
  }
  // do something
}

// ループを早期終了する return も可
while (true) {
  if (cond) {
    return
  }
}
```

### Recursion と Tail Recursion

```scala
// 自然に再帰的な構造 (tree、graph) 以外では recursion を避ける
// tail-recursive な method には @tailrec を使う
@scala.annotation.tailrec
def max0(data: Array[Int], pos: Int, max: Int): Int = {
  if (pos == data.length) {
    max
  } else {
    max0(data, pos + 1, if (data(pos) > max) data(pos) else max)
  }
}

// 明確さのため、通常は明示的 loop を優先する
def max(data: Array[Int]): Int = {
  var max = Int.MinValue
  for (v <- data) {
    if (v > max) {
      max = v
    }
  }
  max
}
```

### Implicit

```scala
// implicit は次の場合以外避ける:
// 1. DSL を構築するとき
// 2. implicit な type parameter (ClassTag、TypeTag)
// 3. 自分の class 内だけで使う private な型変換

// どうしても使うなら overload しない
object ImplicitHolder {
  // 悪い例 - 選択的 import ができない
  def toRdd(seq: Seq[Int]): RDD[Int] = { ... }
  def toRdd(seq: Seq[Long]): RDD[Long] = { ... }
}

// 良い例 - distinct な名前
object ImplicitHolder {
  def intSeqToRdd(seq: Seq[Int]): RDD[Int] = { ... }
  def longSeqToRdd(seq: Seq[Long]): RDD[Long] = { ... }
}
```

## 型安全性

### Algebraic Data Type

```scala
// Sum type - sealed trait と case class
sealed trait PaymentMethod
case class CreditCard(number: String, cvv: String) extends PaymentMethod
case class PayPal(email: String) extends PaymentMethod
case class BankTransfer(account: String, routing: String) extends PaymentMethod

def processPayment(payment: PaymentMethod): Either[Error, Receipt] = payment match {
  case CreditCard(number, cvv) => chargeCreditCard(number, cvv)
  case PayPal(email) => chargePayPal(email)
  case BankTransfer(account, routing) => chargeBankAccount(account, routing)
}

// Product type - case class
case class User(id: Long, name: String, email: String, age: Int)
case class Order(id: Long, userId: Long, items: List[Item], total: BigDecimal)
```

### null より Option

```scala
// null の代わりに Option を使う
def findUserById(id: Long): Option[User] = {
  database.query(id)
}

// null に備えるには Option() を使う
def myMethod1(input: String): Option[String] = Option(transform(input))

// Some() は null を保護しないので使わない
def myMethod2(input: String): Option[String] = Some(transform(input)) // 悪い例

// Option に対する pattern matching
def processUser(id: Long): String = findUserById(id) match {
  case Some(user) => s"Found: ${user.name}"
  case None => "User not found"
}

// 確信がない限り get() は呼ばない
val user = findUserById(123).get  // 危険

// 代わりに getOrElse、map、flatMap、fold を使う
val name = findUserById(123).map(_.name).getOrElse("Unknown")
```

### Either による Error Handling

```scala
sealed trait ValidationError
case class InvalidEmail(email: String) extends ValidationError
case class InvalidAge(age: Int) extends ValidationError
case class MissingField(field: String) extends ValidationError

def validateUser(data: Map[String, String]): Either[ValidationError, User] = {
  for {
    name <- data.get("name").toRight(MissingField("name"))
    email <- data.get("email").toRight(MissingField("email"))
    validEmail <- validateEmail(email)
    ageStr <- data.get("age").toRight(MissingField("age"))
    age <- ageStr.toIntOption.toRight(InvalidAge(-1))
  } yield User(name, validEmail, age)
}
```

### Try と Exception

```scala
// API から Try を返さない
// 悪い例
def getUser(id: Long): Try[User]

// 良い例 - 明示的 throws
@throws(classOf[DatabaseConnectionException])
def getUser(id: Long): Option[User]

// exception を catch するときは NonFatal を使う
import scala.util.control.NonFatal

try {
  dangerousOperation()
} catch {
  case NonFatal(e) =>
    logger.error("Operation failed", e)
  case e: InterruptedException =>
    // interruption を処理
}
```

## Collection

### Immutable Collection を優先

```scala
import scala.collection.immutable._

// 良い例
val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(_ * 2)
val evens = numbers.filter(_ % 2 == 0)

val userMap = Map(
  1L -> "Alice",
  2L -> "Bob"
)
val updated = userMap + (3L -> "Charlie")

// 遅延 sequence には Stream (Scala 2.12) または LazyList (Scala 2.13) を使う
val fibonacci: LazyList[BigInt] =
  BigInt(0) #:: BigInt(1) #:: fibonacci.zip(fibonacci.tail).map { case (a, b) => a + b }

val first10 = fibonacci.take(10).toList
```

### Monadic Chaining

```scala
// 3 つを超える chaining は避ける
// flatMap の後で分ける
// if-else block と一緒に chaining しない

// 悪い例 - 複雑すぎる
database.get(name).flatMap { elem =>
  elem.data.get("address").flatMap(Option.apply)
}

// 良い例 - より読みやすい
def getAddress(name: String): Option[String] = {
  if (!database.contains(name)) {
    return None
  }

  database(name).data.get("address") match {
    case Some(null) => None
    case Some(addr) => Option(addr)
    case None => None
  }
}

// if-else と chaining を組み合わせない
// 悪い例
if (condition) {
  Seq(1, 2, 3)
} else {
  Seq(1, 2, 3)
}.map(_ + 1)

// 良い例
val seq = if (condition) Seq(1, 2, 3) else Seq(4, 5, 6)
seq.map(_ + 1)
```

## パフォーマンス

### while Loop を使う

```scala
// performance 重要 code では for / map より while を使う
val arr = Array.fill(1000)(Random.nextInt())

// 遅い
val newArr = arr.zipWithIndex.map { case (elem, i) =>
  if (i % 2 == 0) 0 else elem
}

// 速い
val newArr = new Array[Int](arr.length)
var i = 0
while (i < arr.length) {
  newArr(i) = if (i % 2 == 0) 0 else arr(i)
  i += 1
}
```

### Option と null

```scala
// performance 重要 code では Option より null を優先する
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### private[this] を使う

```scala
// private[this] は accessor method ではなく field を生成する
class MyClass {
  private val field1 = ...        // accessor を使う可能性がある
  private[this] val field2 = ...  // 直接 field access

  def perfSensitiveMethod(): Unit = {
    var i = 0
    while (i < 1000000) {
      field2  // field access が保証される
      i += 1
    }
  }
}
```

### Java Collection

```scala
// performance のため Java collection を優先する
import java.util.{ArrayList, HashMap}

val list = new ArrayList[String]()
val map = new HashMap[String, Int]()
```

## 並行性

### ConcurrentHashMap を優先

```scala
// java.util.concurrent.ConcurrentHashMap を使う
private[this] val map = new java.util.concurrent.ConcurrentHashMap[String, String]

// contention が低いなら synchronized map でもよい
private[this] val map = java.util.Collections.synchronizedMap(
  new java.util.HashMap[String, String]
)
```

### 明示的 Synchronization

```scala
class Manager {
  private[this] var count = 0
  private[this] val map = new java.util.HashMap[String, String]

  def update(key: String, value: String): Unit = synchronized {
    map.put(key, value)
    count += 1
  }

  def getCount: Int = synchronized { count }
}
```

### Atomic Variable

```scala
import java.util.concurrent.atomic._

// @volatile より Atomic を優先する
val initialized = new AtomicBoolean(false)

// 1 回だけ実行することを明確に表す
if (!initialized.getAndSet(true)) {
  initialize()
}
```

## テスト

### 具体的な Exception を Intercept する

```scala
import org.scalatest._

// 悪い例 - 広すぎる
intercept[Exception] {
  thingThatThrows()
}

// 良い例 - 具体的な型
intercept[IllegalArgumentException] {
  thingThatThrows()
}
```

## SBT 設定

```scala
// build.sbt
ThisBuild / version := "0.1.0-SNAPSHOT"
ThisBuild / scalaVersion := "2.13.12"
ThisBuild / organization := "com.example"

lazy val root = (project in file("."))
  .settings(
    name := "my-application",

    libraryDependencies ++= Seq(
      "org.typelevel" %% "cats-core" % "2.10.0",
      "org.typelevel" %% "cats-effect" % "3.5.2",

      // テスト
      "org.scalatest" %% "scalatest" % "3.2.17" % Test,
      "org.scalatestplus" %% "scalacheck-1-17" % "3.2.17.0" % Test
    ),

    scalacOptions ++= Seq(
      "-encoding", "UTF-8",
      "-feature",
      "-unchecked",
      "-deprecation",
      "-Xfatal-warnings"
    )
  )
```

## その他

### nanoTime を使う

```scala
// duration の計測には currentTimeMillis ではなく nanoTime を使う
val start = System.nanoTime()
doWork()
val elapsed = System.nanoTime() - start

import java.util.concurrent.TimeUnit
val elapsedMs = TimeUnit.NANOSECONDS.toMillis(elapsed)
```

### URL より URI

```scala
// URL ではなく URI を使う (URL.equals は DNS lookup を行う)
val uri = new java.net.URI("http://example.com")
// Not: val url = new java.net.URL("http://example.com")
```

## まとめ

1. **シンプルな code を書く** - 可読性と保守性を優先する
2. **不変 data を使う** - val、不変 collection、case class
3. **language feature を抑制する** - implicit を制限し、symbolic method を避ける
4. **public API には型を付ける** - method と field には明示型を使う
5. **implicit より explicit** - 短さより明確さを優先する
6. **標準 library を使う** - 車輪の再発明をしない
7. **命名規則に従う** - PascalCase、camelCase、UPPER_CASE
8. **method は小さく保つ** - Rule of 30
9. **error を明示的に扱う** - Option、Either、@throws 付き exception
10. **最適化前に profile する** - 推測ではなく計測する

完全な詳細は [Databricks Scala Style Guide](https://github.com/databricks/scala-style-guide) を参照する。
