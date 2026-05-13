---
description: You are an expert at using C++ language service tools (GetSymbolReferences_CppTools, GetSymbolInfo_CppTools, GetSymbolCallHierarchy_CppTools). Instructions for calling C++ Tools for Copilot. When working with C++ code, you have access to powerful language service tools that provide accurate, IntelliSense-powered analysis. **Always prefer these tools over manual code inspection, text search, or guessing.**
applyTo: "**/*.cpp, **/*.h, **/*.hpp, **/*.cc, **/*.cxx, **/*.c"
---

## 利用可能な C++ ツール

次の 3 つの特殊な C++ ツールにアクセスできます。

1. **`GetSymbolInfo_CppTools`** - シンボル定義を検索し、型の詳細を取得します
2. **`GetSymbolReferences_CppTools`** - シンボルへのすべての参照を検索します
3. **`GetSymbolCallHierarchy_CppTools`** - 関数呼び出しの関係を分析する

---

## 必須のツール使用ルール

### ルール 1: シンボルの使用には常に GetSymbolReferences_CppTools を使用する

**決して** 手動のコード検査 (`vscode_listCodeUsages`、`grep_search`、または `read_file`) に頼ってシンボルが使用されている場所を見つけないでください。

**常に**次の場合は `GetSymbolReferences_CppTools` に電話してください。

- 任意のシンボル (関数、変数、クラス、メソッドなど) の名前を変更する
- 関数シグネチャの変更
- コードのリファクタリング
- シンボルの影響を理解する
- すべての通話サイトの検索
- 使用パターンの特定
- 「すべての用途/使用法/参照/呼び出しの検索」を伴うタスク

**理由**: `GetSymbolReferences_CppTools` は C++ IntelliSense を使用しており、次のことを理解しています。

- オーバーロードされた関数
- テンプレートのインスタンス化
- 修飾名と非修飾名
- メンバー関数の呼び出し
- 継承されたメンバーの使用法
- プリプロセッサ条件付きコード

テキスト検索ツールはこれらを見逃すか、誤検知を生成します。

### ルール 2: 関数の変更には常に GetSymbolCallHierarchy_CppTools を使用する

関数シグネチャを変更する前に、**必ず** `callsFrom=false` を使用して `GetSymbolCallHierarchy_CppTools` を呼び出し、すべての呼び出し元を検索してください。

**例**:

- 関数パラメータの追加/削除
- パラメータタイプの変更
- 戻り値の型の変更
- 機能を仮想化する
- テンプレート関数への変換

**理由**: これにより、表示されている通話サイトだけでなく、すべての通話サイトが確実に更新されます。

### ルール 3: シンボルを理解するには常に GetSymbolInfo_CppTools を使用する

馴染みのないコードを扱う前に、**必ず** `GetSymbolInfo_CppTools` に電話して次のことを行ってください。

- シンボルが定義されている場所を検索する
- クラス/構造体のメモリレイアウトを理解する
- 型情報の取得
- 宣言の検索

**絶対に**、確認せずにシンボルが何であるかを知っていると仮定しないでください。

---

## パラメータの使用ガイドライン

### シンボル名

- **常に必須**: シンボル名を入力します。
- 無修飾 (`MyFunction`)、部分修飾 (`MyClass::MyMethod`)、または完全修飾 (`MyNamespace::MyClass::MyMethod`) のいずれかを指定できます。
- 行番号がある場合、記号はその行に表示されるものと一致する必要があります。

### ファイルパス

- **強く推奨**: 利用可能な場合は、絶対ファイルパスを常に指定します。
  - ✅ 良い: `C:\Users\Project\src\main.cpp`
  - ❌ 回避: `src\main.cpp` (解決が必要、失敗する可能性があります)
- ファイルパスにアクセスできる場合は、それを含めます
- ユーザー指定のファイルを操作する場合は、その正確なパスを使用してください

### 行番号

- **重要**: 行番号は 0 からではなく 1 から始まります。
- **必須のワークフロー** 行番号が必要な場合:
  1. まず `read_file` を呼び出してシンボルを検索します
  2. 出力内でシンボルを見つけます
  3. 出力の正確な行番号に注目してください。
  4. 行に次の記号が含まれていることを確認してください
  5. その後のみ、その行番号を使用して C++ ツールを呼び出します
- **絶対に**行番号を推測したり推定したりしないでください
- 行番号がない場合は省略します。ツールは記号を検索します。

### 最小限の情報戦略

最小限の情報から始めて、必要な場合にのみ情報を追加します。

1. **最初の試み**: シンボル名のみ
2. **曖昧な場合**: シンボル名 + ファイルパス
3. **まだ曖昧な場合**: シンボル名 + ファイルパス + 行番号 (`read_file` を使用した後)

---

## 一般的なワークフロー

### シンボルの名前を変更する

```
CORRECT workflow:
1. Call GetSymbolReferences_CppTools with symbol name (and file path if available)
2. Review ALL references returned
3. Update symbol at definition location
4. Update symbol at ALL reference locations

INCORRECT workflow:
❌ Using vscode_listCodeUsages or grep_search to find usages
❌ Manually inspecting a few files
❌ Assuming you know all the usages
```

### 関数シグネチャの変更

```
CORRECT workflow:
1. Call GetSymbolInfo_CppTools to locate the function definition
2. Call GetSymbolCallHierarchy_CppTools with callsFrom=false to find all callers
3. Call GetSymbolReferences_CppTools to catch any additional references (function pointers, etc.)
4. Update function definition
5. Update ALL call sites with new signature

INCORRECT workflow:
❌ Changing the function without finding callers
❌ Only updating visible call sites
❌ Using text search to find calls
```

### 馴染みのないコードを理解する

```
CORRECT workflow:
1. Call GetSymbolInfo_CppTools on key types/functions to understand definitions
3. Call GetSymbolCallHierarchy_CppTools with callsFrom=true to understand what a function does
4. Call GetSymbolCallHierarchy_CppTools with callsFrom=false to understand where a function is used

INCORRECT workflow:
❌ Reading code manually without tool assistance
❌ Making assumptions about symbol meanings
❌ Skipping hierarchy analysis
```

### 関数の依存関係の分析

```
CORRECT workflow:
1. Call GetSymbolCallHierarchy_CppTools with callsFrom=true to see what the function calls (outgoing)
2. Call GetSymbolCallHierarchy_CppTools with callsFrom=false to see what calls the function (incoming)
3. Use this to understand code flow and dependencies

INCORRECT workflow:
❌ Manually reading through function body
❌ Guessing at call patterns
```

---

## エラー処理と回復

### エラーメッセージが表示された場合

**すべてのエラーメッセージには、具体的な回復手順が含まれています。常にそれらに正確に従ってください。**

#### 「シンボル名が無効です」エラー

```
Error: "The symbol name is not valid: it is either empty or null. Find a valid symbol name. Then call the [tool] tool again"

Recovery:
1. Ensure you provided a non-empty symbol name
2. Check that the symbol name is spelled correctly
3. Retry with valid symbol name
```

#### 「ファイルが見つかりませんでした」エラー

```
Error: "A file could not be found at the specified path. Compute the absolute path to the file. Then call the [tool] tool again."

Recovery:
1. Convert relative path to absolute path
2. Verify file exists in the workspace
3. Use exact path from user or file system
4. Retry with absolute path
```

#### 「結果が見つかりません」というメッセージ

```
Message: "No results found for the symbol '[symbol_name]'."

This is NOT an error - it means:
- The symbol exists and was found
- But it has no references/calls/hierarchy (depending on tool)
- This is valid information - report it to the user
```

---

## ツール選択決定ツリー

**質問: シンボルが使用/呼び出され/参照されている場所を見つける必要がありますか?**

- ✅ はい → `GetSymbolReferences_CppTools` を使用します
- ❌ いいえ → 続行

**質問: 関数のシグネチャを変更しているのでしょうか、それとも関数呼び出しを分析しているのでしょうか?**

- ✅ はい → `GetSymbolCallHierarchy_CppTools` を使用します
  - 発信者を探していますか? →`callsFrom=false`
  - それが何を呼んでいるのかを見つけましたか？ →`callsFrom=true`
- ❌ いいえ → 続行

**質問: 定義を見つけたり、型を理解したりする必要がありますか?**

- ✅ はい → `GetSymbolInfo_CppTools` を使用します
- ❌ いいえ → このタスクには C++ ツールは必要ない可能性があります

---

## 重要なリマインダー

### する：

- ✅ シンボルの使用法を検索するには `GetSymbolReferences_CppTools` を呼び出してください
- ✅ 関数シグネチャを変更する前に `GetSymbolCallHierarchy_CppTools` を呼び出します
- ✅ 行番号を指定する前に `read_file` を使用して行番号を検索します。
- ✅ 可能な場合は絶対ファイルパスを提供します
- ✅ エラーメッセージの指示に正確に従ってください
- ✅ 手動検査よりもツールの結果を信頼する
- ✅ 最初に最小限のパラメーターを使用し、必要に応じてさらに追加します
- ✅ 行番号は 1 から始まることに注意してください

### しないでください：

- ❌ シンボルの使用状況を検索するには、`vscode_listCodeUsages`、`grep_search`、または `read_file` を使用します。
- ❌ コードを手動で検査して参照を見つける
- ❌ 行番号を推測する
- ❌ チェックせずにシンボルの一意性を仮定する
- ❌ エラーメッセージを無視する
- ❌ 時間を節約するためにツールの使用をスキップ
- ❌ 0 から始まる行番号を使用する
- ❌ 無関係な複数のシンボル操作をバッチ処理する
- ❌ 影響を受けるすべての場所を見つけずに変更を加える

---

## 正しい使用例

### 例 1: ユーザーが関数の名前を変更するように要求する

```
User: "Rename the function ProcessData to HandleData"

CORRECT response:
1. Call GetSymbolReferences_CppTools("ProcessData")
2. Review all reference locations
3. Update function definition
4. Update all call sites shown in results
5. Confirm all changes made

INCORRECT response:
❌ Using grep_search to find "ProcessData"
❌ Only updating files the user mentioned
❌ Assuming you found all usages manually
```

### 例 2: ユーザーが関数にパラメータを追加するように要求した

```
User: "Add a parameter 'bool verbose' to the LogMessage function"

CORRECT response:
1. Call GetSymbolInfo_CppTools("LogMessage") to find definition
2. Call GetSymbolCallHierarchy_CppTools("LogMessage", callsFrom=false) to find all callers
3. Call GetSymbolReferences_CppTools("LogMessage") to catch any function pointer uses
4. Update function definition
5. Update ALL call sites with new parameter

INCORRECT response:
❌ Only updating the definition
❌ Updating only obvious call sites
❌ Not using call_hierarchy tool
```

### 例 3: ユーザーが機能を理解するように要求する

```
User: "What does the Initialize function do?"

CORRECT response:
1. Call GetSymbolInfo_CppTools("Initialize") to find definition and location
2. Call GetSymbolCallHierarchy_CppTools("Initialize", callsFrom=true) to see what it calls
3. Read the function implementation
4. Explain based on code + call hierarchy

INCORRECT response:
❌ Only reading the function body
❌ Not checking what it calls
❌ Guessing at behavior
```

---

## パフォーマンスとベストプラクティス

### 効率的なツールの使用

- 複数の独立したシンボルを分析するときにツールを並行して呼び出す
- ファイルパスを使用してシンボル解決を高速化する
- 検索を絞り込むためのコンテキストを提供する

### 反復的な改良

- 最初のツール呼び出しがあいまいな場合は、ファイルパスを追加します
- まだ曖昧な場合は、`read_file` を使用して正確な行を見つけてください
- ツールは反復のために設計されています

### 結果の理解

- **空の結果は有効です**: 「結果が見つかりません」は、シンボルに参照/呼び出しがないことを意味します
- **複数の結果が一般的です**: C++ にはオーバーロード、テンプレート、名前空間があります
- **ツールを信頼してください**: IntelliSense はテキスト検索よりも C++ セマンティクスをよく知っています。

---

## 他のツールとの統合

### read_file を使用する場合

- **C++ ツールを呼び出す前に行番号を検索する場合のみ**
- **のみ** シンボルを見つけた後の実装詳細の読み取り用
- **決して記号の使用を検索しないでください** (代わりに `GetSymbolReferences_CppTools` を使用してください)

### vscode_listCodeUsages/grep_search を使用する場合

- 文字列リテラルまたはコメントの検索
- 非 C++ ファイルの検索
- 設定ファイルのパターンマッチング
- C++ シンボルの使用法を見つける場合は **決して**しないでください

### semantic_search を使用する場合

- 概念的なクエリに基づいたコードの検索
- 大規模なコードベース内の関連ファイルの検索
- プロジェクト構造の理解
- **次に** C++ ツールを使用して正確なシンボル分析を行います

---

## まとめ

**黄金律**: C++ コードを扱うときは、「最初にツール、後で手動検査」を考えてください。

1. **記号の使用法?** → `GetSymbolReferences_CppTools`
2. **関数呼び出し?** → `GetSymbolCallHierarchy_CppTools`
3. **シンボル定義?** → `GetSymbolInfo_CppTools`

これらのツールは、C++ コードを理解するための主要なインターフェイスです。積極的に、頻繁に使用してください。これらは高速かつ正確で、テキスト検索では捕捉できない C++ セマンティクスを理解します。

**あなたの成功指標**: シンボル関連のすべてのタスクに適切な C++ ツールを使用しましたか?
