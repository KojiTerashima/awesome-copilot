---
applyTo: '**/*.{cs,ts,java}'
description: ビジネスドメインコードでクリーンかつ保守しやすく堅牢なコードを実現するため Object Calisthenics の原則を適用する
---
# Object Calisthenics ルール

> ⚠️ **警告:** このファイルには、Object Calisthenics の元の 9 ルールが含まれています。追加ルールを加えてはならず、これらのルールを置き換えたり削除したりしてはいけません。
> 必要であれば後から例を追加しても構いません。

## 目的
このルールは、クリーンで保守しやすく堅牢なコードを実現するために、Object Calisthenics の原則を強制します。対象は **主にビジネスドメインコード** です。

## スコープと適用対象
- **主対象**: business domain class（aggregate、entity、value object、domain service）
- **副次対象**: application layer の service と use case handler
- **除外対象**:
  - DTO（Data Transfer Object）
  - API model / contract
  - configuration class
  - business logic を持たない単純な data container
  - 柔軟性が必要な infrastructure code

## 主要原則


1. **1 method あたり 1 レベルのインデント**:
   - method は単純に保ち、1 レベルを超えるインデントにしない。

   ```csharp
   // Bad Example - this method has multiple levels of indentation
   public void SendNewsletter() {
         foreach (var user in users) {
            if (user.IsActive) {
               // Do something
               mailer.Send(user.Email);
            }
         }
   }
   // Good Example - Extracted method to reduce indentation
   public void SendNewsletter() {
       foreach (var user in users) {
           SendEmail(user);
       }
   }
   private void SendEmail(User user) {
       if (user.IsActive) {
           mailer.Send(user.Email);
       }
   }

   // Good Example - Filtering users before sending emails
   public void SendNewsletter() {
       var activeUsers = users.Where(user => user.IsActive);

       foreach (var user in activeUsers) {
           mailer.Send(user.Email);
       }
   }
   ```
2. **`ELSE` キーワードを使わない**:

   - `else` キーワードは避け、複雑さを下げて可読性を高める。
   - 条件処理には early return を使う。
   - Fail Fast 原則を使う。
   - method 冒頭で input や条件を検証する Guard Clause を使う。

   ```csharp
   // Bad Example - Using else
   public void ProcessOrder(Order order) {
       if (order.IsValid) {
           // Process order
       } else {
           // Handle invalid order
       }
   }
   // Good Example - Avoiding else
   public void ProcessOrder(Order order) {
       if (!order.IsValid) return;
       // Process order
   }
   ```

   Fail fast principle の例:
   ```csharp
   public void ProcessOrder(Order order) {
       if (order == null) throw new ArgumentNullException(nameof(order));
       if (!order.IsValid) throw new InvalidOperationException("Invalid order");
       // Process order
   }
   ```

3. **すべての primitive と string をラップする**:
   - primitive type を直接使わない。
   - 文脈と振る舞いを持たせる class に包む。

   ```csharp
   // Bad Example - Using primitive types directly
   public class User {
       public string Name { get; set; }
       public int Age { get; set; }
   }
   // Good Example - Wrapping primitives
   public class User {
       private string name;
       private Age age;
       public User(string name, Age age) {
           this.name = name;
           this.age = age;
       }
   }
   public class Age {
       private int value;
       public Age(int value) {
           if (value < 0) throw new ArgumentOutOfRangeException(nameof(value), "Age cannot be negative");
           this.value = value;
       }
   }
   ```

4. **First Class Collection**:
   - 生の data structure を公開するのではなく、collection に data と振る舞いをカプセル化する。
First Class Collection: 配列や list を属性として持つ class は、それ以外の属性を持ってはならない

```csharp
   // Bad Example - Exposing raw collection
   public class Group {
      public int Id { get; private set; }
      public string Name { get; private set; }
      public List<User> Users { get; private set; }

      public int GetNumberOfUsersIsActive() {
         return Users
            .Where(user => user.IsActive)
            .Count();
      }
   }

   // Good Example - Encapsulating collection behavior
   public class Group {
      public int Id { get; private set; }
      public string Name { get; private set; }

      public GroupUserCollection userCollection { get; private set; } // The list of users is encapsulated in a class

      public int GetNumberOfUsersIsActive() {
         return userCollection
            .GetActiveUsers()
            .Count();
      }
   }
   ```

5. **1 行に 1 ドット**:
   - 1 行に 1 つの dot だけにし、Law of Demeter の違反を避ける。

   ```csharp
   // Bad Example - Multiple dots in a single line
   public void ProcessOrder(Order order) {
       var userEmail = order.User.GetEmail().ToUpper().Trim();
       // Do something with userEmail
   }
   // Good Example - One dot per line
   public class User {
     public NormalizedEmail GetEmail() {
       return NormalizedEmail.Create(/*...*/);
     }
   }
   public class Order {
     /*...*/
     public NormalizedEmail ConfirmationEmail() {
       return User.GetEmail();
     }
   }
   public void ProcessOrder(Order order) {
       var confirmationEmail = order.ConfirmationEmail();
       // Do something with confirmationEmail
   }
   ```

6. **省略しない**:
   - class、method、variable には意味のある名前を使う。
   - 混乱を招く省略形は避ける。

   ```csharp
   // Bad Example - Abbreviated names
   public class U {
       public string N { get; set; }
   }
   // Good Example - Meaningful names
   public class User {
       public string Name { get; set; }
   }
   ```

7. **entity は小さく保つ（class、method、namespace、package）**:
   - class と method のサイズを制限し、可読性と保守性を高める。
   - 各 class は単一責任にし、できるだけ小さくする。

   制約:
   - 1 class あたり最大 10 method
   - 1 class あたり最大 50 行
   - 1 package / namespace あたり最大 10 class

   ```csharp
   // Bad Example - Large class with multiple responsibilities
   public class UserManager {
       public void CreateUser(string name) { /*...*/ }
       public void DeleteUser(int id) { /*...*/ }
       public void SendEmail(string email) { /*...*/ }
   }

   // Good Example - Small classes with single responsibility
   public class UserCreator {
       public void CreateUser(string name) { /*...*/ }
   }
   public class UserDeleter {
       public void DeleteUser(int id) { /*...*/ }
   }

   public class UserUpdater {
       public void UpdateUser(int id, string name) { /*...*/ }
   }
   ```


8. **instance variable を 3 つ以上持つ class を作らない**:
   - instance variable 数を制限して、class の単一責任を促す。
   - simplicity を保つため instance variable は 2 つまでにする。
   - ILogger やその他 logger は instance variable として数えない。

   ```csharp
   // Bad Example - Class with multiple instance variables
   public class UserCreateCommandHandler {
      // Bad: Too many instance variables
      private readonly IUserRepository userRepository;
      private readonly IEmailService emailService;
      private readonly ILogger logger;
      private readonly ISmsService smsService;

      public UserCreateCommandHandler(IUserRepository userRepository, IEmailService emailService, ILogger logger, ISmsService smsService) {
         this.userRepository = userRepository;
         this.emailService = emailService;
         this.logger = logger;
         this.smsService = smsService;
      }
   }

   // Good: Class with two instance variables
   public class UserCreateCommandHandler {
      private readonly IUserRepository userRepository;
      private readonly INotificationService notificationService;
      private readonly ILogger logger; // This is not counted as instance variable

      public UserCreateCommandHandler(IUserRepository userRepository, INotificationService notificationService, ILogger logger) {
         this.userRepository = userRepository;
         this.notificationService = notificationService;
         this.logger = logger;
      }
   }
   ```

9. **Domain Class に Getter / Setter を置かない**:
   - domain class の property を setter で公開しない。
   - object 作成には private constructor と static factory method を使う。
   - **注**: このルールは主に domain class に適用され、DTO や data transfer object には主に適用しない。

   ```csharp
   // Bad Example - Domain class with public setters
   public class User {  // Domain class
       public string Name { get; set; } // Avoid this in domain classes
   }

   // Good Example - Domain class with encapsulation
   public class User {  // Domain class
       private string name;
       private User(string name) { this.name = name; }
       public static User Create(string name) => new User(name);
   }

   // Acceptable Example - DTO with public setters
   public class UserDto {  // DTO - exemption applies
       public string Name { get; set; } // Acceptable for DTOs
   }
   ```

## 実装ガイドライン
- **Domain Class**:
  - instance 作成には private constructor と static factory method を使う。
  - property の setter は公開しない。
  - business domain code には 9 ルールを厳格に適用する。

- **Application Layer**:
  - use case handler と application service にもこれらのルールを適用する。
  - 単一責任とクリーンな抽象化を維持することに集中する。

- **DTO と Data Object**:
  - ルール 3（primitive のラップ）、8（instance variable 2 つまで）、9（getter / setter を置かない）は DTO では緩和してよい。
  - data transfer object では public property と getter / setter を許容する。

- **テスト**:
  - object の state ではなく behavior を検証する test にする。
  - test class は可読性と保守性のため、ルールを緩和してよい。

- **Code Review**:
  - domain code と application code では、review 時にこれらのルールを適用する。
  - infrastructure や DTO の code では実用性を優先する。

## 参考資料
- [Object Calisthenics - Original 9 Rules by Jeff Bay](https://www.cs.helsinki.fi/u/luontola/tdd-2009/ext/ObjectCalisthenics.pdf)
- [ThoughtWorks - Object Calisthenics](https://www.thoughtworks.com/insights/blog/object-calisthenics)
- [Clean Code: A Handbook of Agile Software Craftsmanship - Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
