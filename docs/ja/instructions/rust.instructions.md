---
description: 'Rust プログラミング言語のコーディング規約とベスト プラクティス'
applyTo: '**/*.rs'
---

# Rust のコーディング規約とベスト プラクティス

Rust code を書くときは、慣用的な Rust の実践とコミュニティ標準に従ってください。

これらの instruction は、[The Rust Book](https://doc.rust-lang.org/book/)、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)、[RFC 430 naming conventions](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md)、および [users.rust-lang.org](https://users.rust-lang.org) を中心とするより広い Rust community に基づいています。

## 一般 instruction

- 常に可読性、安全性、保守性を最優先する。
- 強い型付けを使い、メモリ安全性のため Rust の ownership system を活用する。
- 複雑な関数は、より小さく扱いやすい関数へ分割する。
- algorithm 関連の code では、採用したアプローチの説明を含める。
- 特定の設計判断を行った理由に関するコメントを含め、保守しやすい実践で code を書く。
- `Result<T, E>` を使ってエラーを丁寧に処理し、意味のある error message を提供する。
- 外部 dependency については、documentation 内で使用目的と理由を説明する。
- [RFC 430](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md) に従った一貫した命名規則を使う。
- borrow checker の規則に従う、慣用的で安全かつ効率的な Rust code を書く。
- warning なしで code が compile されることを確認する。

## 従うべきパターン

- logic のカプセル化には module (`mod`) と public interface (`pub`) を使う。
- エラーは `?`、`match`、`if let` を使って適切に処理する。
- serialization には `serde`、custom error には `thiserror` または `anyhow` を使う。
- service や外部 dependency を抽象化するため trait を実装する。
- async code は `async/await` と `tokio` または `async-std` を使って構成する。
- 型安全性のため、flag や state より enum を優先する。
- 複雑な object 作成には builder を使う。
- test 容易性と再利用性のため、binary code と library code (`main.rs` と `lib.rs`) を分ける。
- data parallelism や CPU-bound task には `rayon` を使う。
- index ベースの loop より iterator を使う。一般にその方が速く安全です。
- ownership が不要な関数 parameter には `String` ではなく `&str` を使う。
- 不要な allocation を避けるため、borrowing と zero-copy operation を優先する。

### Ownership、Borrowing、Lifetimes

- ownership の移動が必要でない限り、clone より borrowing (`&T`) を優先する。
- 借用したデータを変更する必要がある場合は `&mut T` を使う。
- compiler が lifetime を推論できない場合は明示的に annotate する。
- 単一 thread の参照カウントには `Rc<T>`、thread-safe な参照カウントには `Arc<T>` を使う。
- 単一 thread の内部可変性には `RefCell<T>`、複数 thread 文脈では `Mutex<T>` または `RwLock<T>` を使う。

## 避けるべきパターン

- 絶対に必要な場合を除き `unwrap()` や `expect()` を使わず、適切なエラー処理を優先する。
- library code で panic を起こさず、代わりに `Result` を返す。
- global な mutable state に依存せず、dependency injection や thread-safe container を使う。
- 深いネストの logic は避け、関数や combinator によって再構成する。
- warning を無視せず、CI では error として扱う。
- 必要かつ十分な文書化がない限り `unsafe` を使わない。
- ownership の移動が必要でない限り `clone()` を多用せず、cloning より borrowing を優先する。
- 早すぎる `collect()` を避け、実際に collection が必要になるまで iterator を lazy に保つ。
- 不要な allocation は避け、borrowing と zero-copy operation を優先する。

## コード スタイルとフォーマット

- Rust Style Guide に従い、自動整形には `rustfmt` を使う。
- 可能な限り 1 行 100 文字未満に保つ。
- 関数や struct の documentation は、対象項目の直前に `///` を使って書く。
- よくあるミスの検出と best practice の強制には `cargo clippy` を使う。

## エラー処理

- 回復可能な error には `Result<T, E>` を使い、回復不能な error にだけ `panic!` を使う。
- error 伝播には `unwrap()` や `expect()` より `?` operator を優先する。
- `thiserror` を使うか `std::error::Error` を実装して custom error type を作る。
- 存在するかもしれないし存在しないかもしれない値には `Option<T>` を使う。
- 意味のある error message と context を提供する。
- error type は意味があり、標準 trait を実装した well-behaved なものにする。
- 関数引数を検証し、不正入力には適切な error を返す。

## API 設計ガイドライン

### 共通 Trait の実装
適切な場面では、一般的な trait を積極的に実装します。
- `Copy`、`Clone`、`Eq`、`PartialEq`、`Ord`、`PartialOrd`、`Hash`、`Debug`、`Display`、`Default`
- 標準の変換 trait を使う: `From`、`AsRef`、`AsMut`
- collection は `FromIterator` と `Extend` を実装するべきです
- 注: `Send` と `Sync` は安全な場合 compiler が自動実装します。`unsafe` code を使う場合を除き、手動実装は避けます

### 型安全性と予測可能性
- static な区別を提供するため newtype を使う
- 引数は型を通じて意味を伝えるべきであり、汎用的な `bool` parameter より具体的な型を優先する
- 本当に optional な値に対してのみ `Option<T>` を適切に使う
- 明確な receiver がある関数は method にする
- `Deref` と `DerefMut` は smart pointer だけが実装するべきです

### 将来への備え
- 下流での実装を防ぐため sealed trait を使う
- struct は private field を持つようにする
- 関数は引数を検証するべきです
- すべての public type は `Debug` を実装しなければなりません

## テストとドキュメント

- `#[cfg(test)]` module と `#[test]` annotation を使って包括的な unit test を書く。
- test module は対象 code の近くに `mod tests { ... }` として置く。
- integration test は `tests/` directory に説明的な filename で書く。
- 各関数、struct、enum、複雑な logic には明確で簡潔なコメントを書く。
- 関数には説明的な名前を付け、包括的な documentation を含める。
- [API Guidelines](https://rust-lang.github.io/api-guidelines/) に従い、すべての public API を rustdoc (`///` comment) で文書化する。
- 実装詳細を public documentation から隠すには `#[doc(hidden)]` を使う。
- error 条件、panic シナリオ、安全性の考慮事項を文書化する。
- 例では `unwrap()` や廃止された `try!` macro ではなく `?` operator を使う。

## プロジェクト構成

- `Cargo.toml` では semantic versioning を使う。
- `description`、`license`、`repository`、`keywords`、`categories` といった包括的 metadata を含める。
- optional な機能には feature flag を使う。
- code は `mod.rs` または名前付き file を使って module に整理する。
- `main.rs` や `lib.rs` は最小限に保ち、logic は module に移す。

## 品質チェックリスト

Rust code を公開または review する前に、次を確認してください。

### コア要件
- [ ] **Naming**: RFC 430 命名規則に従っている
- [ ] **Traits**: 適切な箇所で `Debug`、`Clone`、`PartialEq` を実装している
- [ ] **Error Handling**: `Result<T, E>` を使い、意味のある error type を提供している
- [ ] **Documentation**: すべての public item に、例を含む rustdoc comment がある
- [ ] **Testing**: edge case を含む包括的な test coverage がある

### 安全性と品質
- [ ] **Safety**: 不要な `unsafe` code がなく、適切なエラー処理がある
- [ ] **Performance**: iterator を効率的に使い、allocation が最小限である
- [ ] **API Design**: 関数が予測可能で柔軟かつ型安全である
- [ ] **Future Proofing**: struct の private field、必要に応じた sealed trait がある
- [ ] **Tooling**: code が `cargo fmt`、`cargo clippy`、`cargo test` を通過する
