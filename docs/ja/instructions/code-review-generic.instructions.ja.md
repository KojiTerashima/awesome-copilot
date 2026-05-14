---
description: 'GitHub Copilot を使用して任意のプロジェクトに合わせてカスタマイズできる一般的なコードレビュー手順'
applyTo: '**'
excludeAgent: ["coding-agent"]
---

# 一般的なコードレビュー手順

あらゆるプロジェクトに適用できる、GitHub Copilot の包括的なコードレビュー ガイドライン。これらの手順は、プロンプトエンジニアリングのベストプラクティスに従っており、コードの品質、セキュリティ、テスト、アーキテクチャレビューに対する構造化されたアプローチを提供します。

## 言語を確認する

コードレビューを実行するときは、**英語** (または希望の言語を指定) で回答してください。

> **カスタマイズのヒント**: 「英語」を「ポルトガル語 (ブラジル)」、「スペイン語」、「フランス語」などに置き換えて、お好みの言語に変更します。

## 優先順位を確認する

コードレビューを実行するときは、次の順序で問題に優先順位を付けます。

### 🔴 クリティカル (ブロックマージ)
- **セキュリティ**: 脆弱性、漏洩した秘密、認証/認可の問題
- **正確性**: 論理エラー、データ破損のリスク、競合状態
- **重大な変更**: バージョン管理を行わない API コントラクトの変更
- **データ損失**: データ損失または破損のリスク

### 🟡 重要 (話し合いが必要)
- **コードの品質**: SOLID 原則の重大な違反、過剰な重複
- **テストカバレッジ**: クリティカルパスまたは新機能のテストが欠落しています
- **パフォーマンス**: 明らかなパフォーマンスのボトルネック (N+1 クエリ、メモリリーク)
- **アーキテクチャ**: 確立されたパターンからの大幅な逸脱

### 🟢 提案 (ブロックしない改善)
- **可読性**: 名前付けが不十分で、ロジックが複雑なので簡略化できる可能性があります。
- **最適化**: 機能に影響を与えずにパフォーマンスを向上
- **ベストプラクティス**: 慣例からのわずかな逸脱
- **ドキュメント**: コメント/ドキュメントが欠落しているか不完全です

## 一般的なレビュー原則

コードレビューを実行するときは、次の原則に従ってください。

1. **具体的に**: 正確な行、ファイルを参照し、具体的な例を提供します
2. **コンテキストを提供します**: 何かが問題となる理由と潜在的な影響を説明します
3. **解決策の提案**: 何が間違っているかだけでなく、該当する場合は修正されたコードを表示します
4. **建設的であること**: 作成者を批判するのではなく、コードを改善することに集中してください。
5. **グッドプラクティスを認識する**: よく書かれたコードとスマートなソリューションを認識する
6. **現実的であれ**: すべての提案が直ちに実装される必要があるわけではありません
7. **グループ関連のコメント**: 同じトピックに関する複数のコメントは避けてください。

## コードの品質基準

コードレビューを実行するときは、次の点を確認してください。

### クリーンなコード
- 変数、関数、クラスの説明的で意味のある名前
- 単一責任の原則: 各関数/クラスは 1 つのことをうまく実行します。
- DRY (Don't Reply Yourself): コードの重複はありません
- 関数は小さく、焦点を絞ったものにする必要があります (理想的には 20 ～ 30 行未満)
- 深くネストされたコードを避ける (最大 3 ～ 4 レベル)
- マジックナンバーや文字列は避けてください (定数を使用してください)
- コードは自己文書化されている必要があります。必要な場合のみコメントする

### 例
```javascript
// ❌ BAD: Poor naming and magic numbers
function calc(x, y) {
    if (x > 100) return y * 0.15;
    return y * 0.10;
}

// ✅ GOOD: Clear naming and constants
const PREMIUM_THRESHOLD = 100;
const PREMIUM_DISCOUNT_RATE = 0.15;
const STANDARD_DISCOUNT_RATE = 0.10;

function calculateDiscount(orderTotal, itemPrice) {
    const isPremiumOrder = orderTotal > PREMIUM_THRESHOLD;
    const discountRate = isPremiumOrder ? PREMIUM_DISCOUNT_RATE : STANDARD_DISCOUNT_RATE;
    return itemPrice * discountRate;
}
```

### エラー処理
- 適切なレベルでの適切なエラー処理
- 意味のあるエラーメッセージ
- サイレントエラーや無視された例外はありません
- フェイルファスト: 入力を早期に検証します
- 適切なエラータイプ/例外を使用する

### 例
```python
# ❌ BAD: Silent failure and generic error
def process_user(user_id):
    try:
        user = db.get(user_id)
        user.process()
    except:
        pass

# ✅ GOOD: Explicit error handling
def process_user(user_id):
    if not user_id or user_id <= 0:
        raise ValueError(f"Invalid user_id: {user_id}")

    try:
        user = db.get(user_id)
    except UserNotFoundError:
        raise UserNotFoundError(f"User {user_id} not found in database")
    except DatabaseError as e:
        raise ProcessingError(f"Failed to retrieve user {user_id}: {e}")

    return user.process()
```

## セキュリティレビュー

コードレビューを実行するときは、セキュリティ上の問題を確認してください。

- **機密データ**: コードやログにパスワード、API キー、トークン、PII は含まれていません
- **入力検証**: すべてのユーザー入力が検証され、サニタイズされます。
- **SQL インジェクション**: パラメータ化されたクエリを使用し、文字列連結は使用しないでください。
- **認証**: リソースにアクセスする前の適切な認証チェック
- **権限**: ユーザーがアクションを実行する権限を持っていることを確認します
- **暗号化**: 確立されたライブラリを使用し、独自の暗号化をロールしないでください。
- **依存関係のセキュリティ**: 依存関係の既知の脆弱性を確認します。

### 例
```java
// ❌ BAD: SQL injection vulnerability
String query = "SELECT * FROM users WHERE email = '" + email + "'";

// ✅ GOOD: Parameterized query
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE email = ?"
);
stmt.setString(1, email);
```

```javascript
// ❌ BAD: Exposed secret in code
const API_KEY = "sk_live_abc123xyz789";

// ✅ GOOD: Use environment variables
const API_KEY = process.env.API_KEY;
```

## 試験基準

コードレビューを実行するときは、テストの品質を検証します。

- **対象範囲**: クリティカルパスと新機能にはテストが必要です
- **テスト名**: 何がテストされているかを説明するわかりやすい名前
- **テスト構造**: 明確な Arrange-Act-Assert または Given-When-Then パターン
- **独立性**: テストは相互に依存したり、外部状態に依存してはなりません。
- **アサーション**: 特定のアサーションを使用し、一般的なassertTrue/assertFalseを避けてください。
- **エッジケース**: 境界条件、NULL 値、空のコレクションをテストします。
- **適切にモック**: ドメインロジックではなく、外部の依存関係をモックします。

### 例
```typescript
// ❌ BAD: Vague name and assertion
test('test1', () => {
    const result = calc(5, 10);
    expect(result).toBeTruthy();
});

// ✅ GOOD: Descriptive name and specific assertion
test('should calculate 10% discount for orders under $100', () => {
    const orderTotal = 50;
    const itemPrice = 20;

    const discount = calculateDiscount(orderTotal, itemPrice);

    expect(discount).toBe(2.00);
});
```

## パフォーマンスに関する考慮事項

コードレビューを実行するときは、パフォーマンスの問題がないか確認してください。

- **データベースクエリ**: N+1 クエリを避け、適切なインデックスを使用します。
- **アルゴリズム**: ユースケースに適した時間/空間の複雑さ
- **キャッシュ**: コストのかかる操作や繰り返しの操作にはキャッシュを利用します。
- **リソース管理**: 接続、ファイル、ストリームの適切なクリーンアップ
- **ページネーション**: 大きな結果セットはページネーションする必要があります
- **遅延読み込み**: 必要な場合にのみデータを読み込みます

### 例
```python
# ❌ BAD: N+1 query problem
users = User.query.all()
for user in users:
    orders = Order.query.filter_by(user_id=user.id).all()  # N+1!

# ✅ GOOD: Use JOIN or eager loading
users = User.query.options(joinedload(User.orders)).all()
for user in users:
    orders = user.orders
```

## 建築とデザイン

コードレビューを実行するときは、アーキテクチャの原則を確認してください。

- **懸念事項の分離**: レイヤー/モジュール間の明確な境界
- **依存関係の方向**: 高レベルのモジュールは低レベルの詳細に依存しません。
- **インターフェイスの分離**: 小規模で焦点を絞ったインターフェイスを好む
- **疎結合**: コンポーネントは独立してテスト可能である必要があります
- **高い凝集性**: 関連する機能がグループ化されています
- **一貫したパターン**: コードベースで確立されたパターンに従います。

## 文書化基準

コードレビューを実行するときは、ドキュメントを確認してください。

- **API ドキュメント**: パブリック API はドキュメント化する必要があります (目的、パラメータ、戻り値)
- **複雑なロジック**: 自明ではないロジックには説明的なコメントが必要です
- **README の更新**: 機能の追加または設定の変更時に README を更新します。
- **重大な変更**: 重大な変更を明確に文書化します。
- **例**: 複雑な機能の使用例を提供します。

## コメント形式テンプレート

コードレビューを実行するときは、コメントに次の形式を使用します。

```markdown
**[PRIORITY] Category: Brief title**

Detailed description of the issue or suggestion.

**Why this matters:**
Explanation of the impact or reason for the suggestion.

**Suggested fix:**
[code example if applicable]

**Reference:** [link to relevant documentation or standard]
```

### コメントの例

#### 重大な問題
````markdown
**🔴 CRITICAL - Security: SQL Injection Vulnerability**

The query on line 45 concatenates user input directly into the SQL string,
creating a SQL injection vulnerability.

**Why this matters:**
An attacker could manipulate the email parameter to execute arbitrary SQL commands,
potentially exposing or deleting all database data.

**Suggested fix:**
```SQL
 - の代わりに：
クエリ = "SELECT * FROM ユーザー WHERE 電子メール = '" + 電子メール + ""

 - 使用：
PreparedStatement stmt = conn.prepareStatement(
「SELECT * FROM users WHERE email = ?」
);
stmt.setString(1, 電子メール);
```

**Reference:** OWASP SQL Injection Prevention Cheat Sheet
````

#### 重要な問題
````markdown
**🟡 IMPORTANT - Testing: Missing test coverage for critical path**

The `processPayment()` function handles financial transactions but has no tests
for the refund scenario.

**Why this matters:**
Refunds involve money movement and should be thoroughly tested to prevent
financial errors or data inconsistencies.

**Suggested fix:**
Add test case:
```javascript
test('注文がキャンセルされた場合は全額返金を処理する必要があります', () => {
const order = createOrder({ total: 100, status: 'キャンセル' });

const result = processPayment(order, { type: 'refund' });

Expect(result.refundAmount).toBe(100);
Expect(result.status).toBe('返金');
});
```
````

#### 提案
````markdown
**🟢 SUGGESTION - Readability: Simplify nested conditionals**

The nested if statements on lines 30-40 make the logic hard to follow.

**Why this matters:**
Simpler code is easier to maintain, debug, and test.

**Suggested fix:**
```javascript
// ネストされた if の代わりに:
if (ユーザー) {
if (user.isActive) {
if (user.hasPermission('write')) {
// 何かをする
}
}
}

// ガード句を考慮します。
if (!user || !user.isActive || !user.hasPermission('write')) {
戻る;
}
// 何かをする
```
````

## チェックリストを確認する

コードレビューを実行するときは、次のことを体系的に検証してください。

### コードの品質
- [ ] コードは一貫したスタイルと規則に従っています
- [ ] 名前はわかりやすいものであり、命名規則に従っています。
- [ ] 関数/メソッドが小さく、焦点が絞られている
- [ ] コードの重複なし
- [ ] 複雑なロジックはより単純な部分に分割されます
- [ ] エラー処理は適切です
- [ ] コメントアウトされたコードやチケットのない TODO はありません

### 安全
- [ ] コードまたはログに機密データが含まれていない
- [ ] すべてのユーザー入力に対する入力検証
- [ ] SQL インジェクションの脆弱性はありません
- [ ] 認証と認可が適切に実装されている
- [ ] 依存関係は最新で安全です

### テスト
- [ ] 新しいコードには適切なテストカバレッジがある
- [ ] テストには適切な名前が付けられ、焦点が絞られています
- [ ] テストはエッジケースとエラーシナリオをカバーします
- [ ] テストは独立していて決定的です
- [ ] 常に合格するかコメントアウトされるテストはありません

### パフォーマンス
- [ ] 明らかなパフォーマンスの問題なし (N+1、メモリリーク)
- [ ] キャッシュの適切な使用
- [ ] 効率的なアルゴリズムとデータ構造
- [ ] 適切なリソースのクリーンアップ

### 建築
- [ ] 確立されたパターンと慣例に従います
- [ ] 関心事の適切な分離
- [ ] アーキテクチャ違反はありません
- [ ] 依存関係は正しい方向に流れます

### ドキュメント
- [ ] パブリック API は文書化されています
- [ ] 複雑なロジックには説明コメントが付いています
- [ ] README は必要に応じて更新されます
- [ ] 重大な変更が文書化されています

## プロジェクト固有のカスタマイズ

このテンプレートをプロジェクトに合わせてカスタマイズするには、次のセクションを追加します。

1. **言語/フレームワーク固有のチェック**
   - 例: 「コードレビューを実行するときは、React フックがフックのルールに従っていることを確認してください」
   - 例: 「コードレビューを実行するときは、Spring Boot コントローラーが適切なアノテーションを使用していることを確認してください」

2. **構築と展開**
   - 例: 「コードレビューを実行するときは、CI/CD パイプライン構成が正しいことを確認してください」
   - 例: 「コードレビューを実行するときは、データベースの移行が元に戻せるかどうかを確認してください」

3. **ビジネスロジック ルール**
   - 例: 「コードレビューを実行するときは、価格計算に該当する税金がすべて含まれていることを確認してください」
   - 例: 「コードレビューを実行するときは、データ処理の前にユーザーの同意が得られていることを確認してください」

4. **チーム規約**
   - 例: 「コードレビューを実行するときは、コミットメッセージが従来のコミット形式に従っていることを確認してください」
   - 例: 「コードレビューを実行するときは、ブランチ名がパターンに従っていることを確認してください: type/ticket-description」

## 追加リソース

効果的なコードレビューと GitHub Copilot のカスタマイズの詳細については、次を参照してください。

- [GitHub コパイロットプロンプト エンジニアリング](https://docs.github.com/en/copilot/concepts/prompting/prompt-engineering)
- [GitHub Copilot のカスタム手順](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [素晴らしい GitHub Copilot リポジトリ](https://github.com/github/awesome-copilot)
- [GitHub コードレビューのガイドライン](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests)
- [Google エンジニアリングプラクティス - コードレビュー](https://google.github.io/eng-practices/review/)
- [OWASP セキュリティガイドライン](https://owasp.org/)

## 迅速なエンジニアリングのヒント

コードレビューを実行するときは、[GitHub コパイロットのドキュメント](https://docs.github.com/en/copilot/concepts/prompting/prompt-engineering) から次のプロンプトエンジニアリング原則を適用してください。

1. **一般から始めて、次に具体的に**: 高レベルのアーキテクチャのレビューから始めて、実装の詳細を掘り下げます
2. **例を示します**: 変更を提案するときに、コードベース内の同様のパターンを参照します。
3. **複雑なタスクの解消**: 大規模な PR を論理的なチャンクに分けてレビューします (セキュリティ → テスト → ロジック → スタイル)
4. **曖昧さを避ける**: どのファイル、行、問題に対処しているかを具体的にしてください。
5. **関連コードを示します**: 変更によって影響を受ける可能性のある参照関連コード
6. **実験と反復**: 最初のレビューで何かが欠けていた場合は、焦点を絞った質問をして再度レビューします。

## プロジェクトのコンテキスト

これは一般的なテンプレートです。このセクションをプロジェクト固有の情報でカスタマイズします。

- **技術スタック**: [例: Java 17、Spring Boot 3.x、PostgreSQL]
- **アーキテクチャ**: [例: ヘキサゴナル/クリーンアーキテクチャ、マイクロサービス]
- **ビルドツール**: [例: Gradle、Maven、npm、pip]
- **テスト**: [例: JUnit 5、Jest、pytest]
- **コードスタイル**: [例: Google スタイルガイドに従う]
