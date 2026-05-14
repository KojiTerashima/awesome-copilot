# JavaScript とプログラミングのリファレンス

JavaScript、ECMAScript、プログラミングの概念、最新の JS パターンに関する包括的なリファレンス。

## コアコンセプト

### JavaScript
ECMAScript 仕様に準拠した高レベルのインタープリタ型プログラミング言語。 HTML や CSS と並ぶ Web 開発の主要言語。

**主な特徴**:
- 動的に型付けされる
- プロトタイプベースの継承
- 第一級の関数
- イベント駆動型
- 非同期実行

### ECMAScript
JavaScript が実装する標準化された仕様。

**メジャー バージョン**:
- **ES5** (2009): 厳密モード、JSON サポート
- **ES6/ES2015**: クラス、アロー関数、プロミス、モジュール
- **ES2016+**: 非同期/待機、オプションのチェーン、ヌル合体

## データ型

### プリミティブ型```javascript
// String
let name = "John";
let greeting = 'Hello';
let template = `Hello, ${name}!`; // Template literal

// Number
let integer = 42;
let float = 3.14;
let negative = -10;
let scientific = 1e6; // 1000000

// BigInt (for very large integers)
let big = 9007199254740991n;

// Boolean
let isTrue = true;
let isFalse = false;

// Undefined (declared but not assigned)
let undefined_var;
console.log(undefined_var); // undefined

// Null (intentional absence of value)
let empty = null;

// Symbol (unique identifier)
let sym = Symbol('description');
```### 型チェック```javascript
typeof "hello"; // "string"
typeof 42; // "number"
typeof true; // "boolean"
typeof undefined; // "undefined"
typeof null; // "object" (historical bug)
typeof Symbol(); // "symbol"
typeof {}; // "object"
typeof []; // "object"
typeof function() {}; // "function"

// Better array check
Array.isArray([]); // true

// Null check
value === null; // true if null
```### 型強制と変換```javascript
// Implicit coercion
"5" + 2; // "52" (string concatenation)
"5" - 2; // 3 (numeric subtraction)
"5" * "2"; // 10 (numeric multiplication)
!!"value"; // true (boolean conversion)

// Explicit conversion
String(123); // "123"
Number("123"); // 123
Number("abc"); // NaN
Boolean(0); // false
Boolean(1); // true
parseInt("123px"); // 123
parseFloat("3.14"); // 3.14
```### 真実の価値観と虚偽の価値観

**偽の値** (偽と評価される):
- `false`
- `0`、`-0`
- `""` (空の文字列)
- @@コード4@@
- `undefined`
- `NaN`

**その他の内容はすべて真実です**。以下を含みます。
- `"0"` (文字列)
- `"false"` (文字列)
- `[]` (空の配列)
- `{}` (空のオブジェクト)
- `function() {}` (空の関数)

## 変数と定数```javascript
// var (function-scoped, hoisted - avoid in modern code)
var oldStyle = "avoid this";

// let (block-scoped, can be reassigned)
let count = 0;
count = 1; // ✓ works

// const (block-scoped, cannot be reassigned)
const MAX = 100;
MAX = 200; // ✗ TypeError

// const with objects/arrays (content can change)
const person = { name: "John" };
person.name = "Jane"; // ✓ works (mutating object)
person = {}; // ✗ TypeError (reassigning variable)
```## 関数

### 関数の宣言```javascript
function greet(name) {
  return `Hello, ${name}!`;
}
```### 関数式```javascript
const greet = function(name) {
  return `Hello, ${name}!`;
};
```### アロー関数```javascript
// Basic syntax
const add = (a, b) => a + b;

// With block body
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// Single parameter (parentheses optional)
const square = x => x * x;

// No parameters
const getRandom = () => Math.random();

// Implicit return of object (wrap in parentheses)
const makePerson = (name, age) => ({ name, age });
```### 第一級関数

関数は次のような値です。
- 変数への代入
- 引数として渡されます
- 他の関数から返される```javascript
// Assign to variable
const fn = function() { return 42; };

// Pass as argument
function execute(callback) {
  return callback();
}
execute(() => console.log("Hello"));

// Return from function
function createMultiplier(factor) {
  return function(x) {
    return x * factor;
  };
}
const double = createMultiplier(2);
double(5); // 10
```### 閉鎖

語彙範囲を記憶する関数:```javascript
function createCounter() {
  let count = 0; // Private variable
  
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.decrement(); // 1
counter.getCount(); // 1
```### コールバック関数

後で実行される引数として渡される関数:```javascript
// Array methods use callbacks
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(num => console.log(num));

const doubled = numbers.map(num => num * 2);

const evens = numbers.filter(num => num % 2 === 0);

const sum = numbers.reduce((acc, num) => acc + num, 0);
```### IIFE (即時に呼び出される関数式)```javascript
(function() {
  // Code here runs immediately
  console.log("IIFE executed");
})();

// With parameters
(function(name) {
  console.log(`Hello, ${name}`);
})("World");

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE");
})();
```## オブジェクト

### オブジェクトの作成```javascript
// Object literal
const person = {
  name: "John",
  age: 30,
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

// Constructor function
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const john = new Person("John", 30);

// Object.create
const proto = { greet() { return "Hello"; } };
const obj = Object.create(proto);
```### プロパティへのアクセス```javascript
const obj = { name: "John", age: 30 };

// Dot notation
obj.name; // "John"

// Bracket notation
obj["age"]; // 30
const key = "name";
obj[key]; // "John"

// Optional chaining (ES2020)
obj.address?.city; // undefined (no error if address doesn't exist)
obj.getName?.(); // undefined (no error if getName doesn't exist)
```### オブジェクトメソッド```javascript
const person = { name: "John", age: 30, city: "NYC" };

// Get keys
Object.keys(person); // ["name", "age", "city"]

// Get values
Object.values(person); // ["John", 30, "NYC"]

// Get entries
Object.entries(person); // [["name", "John"], ["age", 30], ["city", "NYC"]]

// Assign (merge objects)
const extended = Object.assign({}, person, { country: "USA" });

// Spread operator (modern alternative)
const merged = { ...person, country: "USA" };

// Freeze (make immutable)
Object.freeze(person);
person.age = 31; // Silently fails (throws in strict mode)

// Seal (prevent adding/removing properties)
Object.seal(person);
```### 構造の分割```javascript
// Object destructuring
const person = { name: "John", age: 30, city: "NYC" };
const { name, age } = person;

// With different variable names
const { name: personName, age: personAge } = person;

// With defaults
const { name, country = "USA" } = person;

// Nested destructuring
const user = { profile: { email: "john@example.com" } };
const { profile: { email } } = user;

// Array destructuring
const numbers = [1, 2, 3, 4, 5];
const [first, second, ...rest] = numbers;
// first = 1, second = 2, rest = [3, 4, 5]

// Skip elements
const [a, , c] = numbers;
// a = 1, c = 3
```## 配列```javascript
// Create arrays
const arr = [1, 2, 3];
const empty = [];
const mixed = [1, "two", { three: 3 }, [4]];

// Access elements
arr[0]; // 1
arr[arr.length - 1]; // Last element
arr.at(-1); // 3 (ES2022 - negative indexing)

// Modify arrays
arr.push(4); // Add to end
arr.pop(); // Remove from end
arr.unshift(0); // Add to beginning
arr.shift(); // Remove from beginning
arr.splice(1, 2, 'a', 'b'); // Remove 2 elements at index 1, insert 'a', 'b'

// Iteration
arr.forEach(item => console.log(item));
for (let item of arr) { console.log(item); }
for (let i = 0; i < arr.length; i++) { console.log(arr[i]); }

// Transformation
const doubled = arr.map(x => x * 2);
const evens = arr.filter(x => x % 2 === 0);
const sum = arr.reduce((acc, x) => acc + x, 0);

// Search
arr.includes(2); // true
arr.indexOf(2); // Index or -1
arr.find(x => x > 2); // First matching element
arr.findIndex(x => x > 2); // Index of first match

// Test
arr.some(x => x > 5); // true if any match
arr.every(x => x > 0); // true if all match

// Sort and reverse
arr.sort((a, b) => a - b); // Ascending
arr.reverse(); // Reverse in place

// Combine
const combined = arr.concat([4, 5]);
const spread = [...arr, 4, 5];

// Slice (copy portion)
const portion = arr.slice(1, 3); // Index 1 to 3 (exclusive)

// Flat (flatten nested arrays)
[[1, 2], [3, 4]].flat(); // [1, 2, 3, 4]
```## 制御フロー

### 条件文```javascript
// if/else
if (condition) {
  // code
} else if (otherCondition) {
  // code
} else {
  // code
}

// Ternary operator
const result = condition ? valueIfTrue : valueIfFalse;

// Switch statement
switch (value) {
  case 1:
    // code
    break;
  case 2:
  case 3:
    // code for 2 or 3
    break;
  default:
    // default code
}

// Nullish coalescing (ES2020)
const value = null ?? "default"; // "default"
const value = 0 ?? "default"; // 0 (0 is not nullish)

// Logical OR for defaults (pre-ES2020)
const value = falsy || "default";

// Optional chaining
const city = user?.address?.city;
```### ループ```javascript
// for loop
for (let i = 0; i < 10; i++) {
  console.log(i);
}

// while loop
let i = 0;
while (i < 10) {
  console.log(i);
  i++;
}

// do-while loop
do {
  console.log(i);
  i++;
} while (i < 10);

// for...of (iterate values)
for (const item of array) {
  console.log(item);
}

// for...in (iterate keys - avoid for arrays)
for (const key in object) {
  console.log(key, object[key]);
}

// break and continue
for (let i = 0; i < 10; i++) {
  if (i === 5) break; // Exit loop
  if (i === 3) continue; // Skip iteration
  console.log(i);
}
```## 非同期 JavaScript

### コールバック```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback("Data received");
  }, 1000);
}

fetchData(data => console.log(data));
```### 約束```javascript
// Create promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve("Success!");
    } else {
      reject("Error!");
    }
  }, 1000);
});

// Use promise
promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log("Done"));

// Promise utilities
Promise.all([promise1, promise2]); // Wait for all
Promise.race([promise1, promise2]); // First to complete
Promise.allSettled([promise1, promise2]); // Wait for all (ES2020)
Promise.any([promise1, promise2]); // First to succeed (ES2021)
```### 非同期/待機```javascript
// Async function
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Use async function
fetchData().then(data => console.log(data));

// Top-level await (ES2022, in modules)
const data = await fetchData();
```## クラス```javascript
class Person {
  // Constructor
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Instance method
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // Getter
  get info() {
    return `${this.name}, ${this.age}`;
  }
  
  // Setter
  set birthYear(year) {
    this.age = new Date().getFullYear() - year;
  }
  
  // Static method
  static species() {
    return "Homo sapiens";
  }
}

// Inheritance
class Employee extends Person {
  constructor(name, age, jobTitle) {
    super(name, age); // Call parent constructor
    this.jobTitle = jobTitle;
  }
  
  // Override method
  greet() {
    return `${super.greet()}, I'm a ${this.jobTitle}`;
  }
}

// Usage
const john = new Person("John", 30);
john.greet(); // "Hello, I'm John"
Person.species(); // "Homo sapiens"

const jane = new Employee("Jane", 25, "Developer");
jane.greet(); // "Hello, I'm Jane, I'm a Developer"
```## モジュール

### ES6 モジュール (ESM)```javascript
// Export (math.js)
export const PI = 3.14159;
export function add(a, b) {
  return a + b;
}
export default class Calculator {
  // ...
}

// Import
import Calculator, { PI, add } from './math.js';
import * as math from './math.js';
import { add as sum } from './math.js'; // Rename
```### CommonJS (Node.js)```javascript
// Export (math.js)
module.exports = {
  add(a, b) {
    return a + b;
  }
};

// Import
const math = require('./math');
```## エラー処理```javascript
// Try/catch
try {
  // Code that might throw
  throw new Error("Something went wrong");
} catch (error) {
  console.error(error.message);
} finally {
  // Always runs
  console.log("Cleanup");
}

// Custom errors
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

throw new ValidationError("Invalid input");
```## ベストプラクティス

### やるべきこと
- ✅ デフォルトでは `const` を使用し、必要に応じて `let` を使用します
- ✅ 厳密モードを使用する (`'use strict';`)
- ✅ コールバックにアロー関数を使用する
- ✅ 文字列補間にテンプレート リテラルを使用する
- ✅ よりクリーンなコードのために分割を使用する
- ✅ 非同期コードには async/await を使用します
- ✅ エラーを適切に処理する
- ✅ わかりやすい変数名を使用する
- ✅ 機能を小さく、集中的に保つ
- ✅ 最新の ES6+ 機能を使用する

### やってはいけないこと
- ❌ `var` を使用します (`let` または `const` を使用します)
- ❌ グローバルスコープを汚染する
- ❌ `==` を使用します (厳密な等価性を得るには `===` を使用します)
- ❌ 関数パラメータを変更する
- ❌ `eval()` または `with()` を使用します
- ❌ エラーをサイレントに無視します
- ❌ I/O 操作に同期コードを使用する
- ❌ 深くネストされたコールバックを作成する (コールバック地獄)

## 用語集の用語

**対象となる重要な用語**:
- アルゴリズム
- 引数
- 配列
- 非同期
- バインディング
- BigInt
- ビットごとのフラグ
- ブロック (スクリプト)
- ブール値
- コールバック関数
- キャメルケース
- クラス
- 閉鎖
- コードポイント
- コード単位
- コンパイル
- コンパイル時間
- 条件付き
- 定数
- コンストラクター
- 制御フロー
- ディープコピー
- デシリアライズ
- ECMAScript
- カプセル化
- 例外
- エキスパンド
- 最高級の機能
- 機能
- 吊り上げ
- IIFE
- 識別子
- 不変
- 継承
- インスタンス
- JavaScript
- JSON
- JSON型表現
- ジャストインタイムコンパイル (JIT)
- ケバブケース
- キーワード
- リテラル
- ローカルスコープ
- ローカル変数
- ループ
- 方法
- ミックスイン
- モジュール性
- 変更可能
- 名前空間
- NaN
- ネイティブ
- ヌル
- ヌル値
- 番号
- オブジェクト
- オブジェクト参照
- OOP
- オペランド
- オペレーター
- パラメータ
- 解析する
- ポリモーフィズム
- 原始的
- 約束
- プロパティ (JavaScript)
- プロトタイプ
- プロトタイプベースのプログラミング
- 疑似コード
- 再帰
- 正規表現
- 範囲
- 連載
- シリアル化可能なオブジェクト
- 浅いコピー
- シグネチャ（機能）
- ずさんなモード
- ヘビケース
- 静的メソッド
- 静的型付け
- 声明
- ストリクトモード
- 文字列
- ストリンファイアー
- シンボル
- 同期
- 構文
- 構文エラー
- タイプ
- 型の強制
- 型変換
- 真実
- 偽りの
- 未定義
- 値
- 変数

## 追加のリソース

- [MDN JavaScript リファレンス](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [ECMAScript仕様](https://tc39.es/ecma262/)
- [JavaScript.info](https://javascript.info/)
- [You Don't Know JS (書籍シリーズ)](https://github.com/getify/You-Dont-Know-JS)