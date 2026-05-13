---
description: 'Clojure 開発用の Clojure 固有のコーディングパターン、インライン def の使用法、コードブロック テンプレート、および名前空間の処理。'
applyTo: '**/*.{clj,cljs,cljc,bb,edn.mdx?}'
---

# Clojure 開発手順

## コード評価ツールの使用法

「repl を使用する」とは、Calva Backseat Driver の **Evaluate Clojure Code** ツールを使用することを意味します。ユーザーが Calva 経由で接続しているのと同じ REPL に接続します。

- ターミナルから 2 番目の REPL を起動するのではなく、常に Calva の REPL 内に留まってください。
- REPL 接続がない場合は、自分で REPL を開始して接続するのではなく、ユーザーに REPL に接続するように依頼してください。

### REPL ツール呼び出しの JSON 文字列
REPL ツールを呼び出すときに、JSON 引数を過度にエスケープしないでください。

```json
{
  "namespace": "<current-namespace>",
  "replSessionKey": "cljs",
  "code": "(def foo \"something something\")"
}
```

## `defn` のドキュメント文字列
docstring は、関数名の直後、引数ベクトルの前に属します。

```clojure
(defn my-function
  "This function does something."
  [arg1 arg2]
  ;; function body
  )
```

- 関数は使用する前に定義します。本当に必要な場合を除き、`declare` よりも順序付けを優先します。

## インタラクティブプログラミング (別名 REPL 駆動開発)

### ブラケットのバランスをとるためのデータ構造要素の位置合わせ
**すべてのデータ構造 (ベクトル、マップ、リスト、セット、すべてのコード) で複数行の要素を常に垂直方向に整列させてください (Clojure コードはデータであるため)。位置がずれていると、ブラケットバランサーがブラケットを正しく閉じず、無効なフォームが作成されます。**

```clojure
;; ❌ Wrong - misaligned vector elements
(select-keys m [:key-a
                :key-b
               :key-c])  ; Misalignment → incorrect ] placement

;; ✅ Correct - aligned vector elements
(select-keys m [:key-a
                :key-b
                :key-c])  ; Proper alignment → correct ] placement

;; ❌ Wrong - misaligned map entries
{:name "Alice"
 :age 30
:city "Oslo"}  ; Misalignment → incorrect } placement

;; ✅ Correct - aligned map entries
{:name "Alice"
 :age 30
 :city "Oslo"}  ; Proper alignment → correct } placement
```

**重要**: ブラケットバランサーは、構造を決定するために一貫したインデントに依存します。

### REPL 依存関係管理
REPL セッション中の動的依存関係の読み込みには `clojure.repl.deps/add-libs` を使用します。

```clojure
(require '[clojure.repl.deps :refer [add-libs]])
(add-libs '{dk.ative/docjure {:mvn/version "1.15.0"}})
```

- 動的依存関係の読み込みには Clojure 1.12 以降が必要です
- ライブラリの探索とプロトタイピングに最適

### Clojureのバージョンを確認する

```clojure
*clojure-version*
;; => {:major 1, :minor 12, :incremental 1, :qualifier nil}
```

### REPL の可用性規律

**REPL が利用できないときはコードファイルを編集しないでください。** REPL 評価が REPL が利用できないことを示すエラーを返した場合は、すぐに停止してユーザーに通知してください。続行する前に、ユーザーに REPL を復元してもらいます。

#### なぜこれが重要なのか
- **対話型プログラミングには動作する REPL が必要です** - 評価なしでは動作を検証できません
- **推測するとバグが発生します** - テストせずにコードを変更するとエラーが発生します

## 構造編集と REPL-First 習慣
- ファイルを操作する前に、REPL で変更を作成します。
- Clojure ファイルを編集するときは、**トップレベルフォームの挿入**、**トップレベル フォームの置換**、**Clojure ファイルの作成**、**コードの追加** などの構造編集ツールを必ず使用し、最初に必ずその指示をお読みください。

### 新しいファイルの作成
- **Create Clojure File** ツールを初期コンテンツで使用する
- Clojure の命名規則に従ってください: 名前空間は kebab-case、ファイルパスは対応するsnake_case (例: `my.project.ns` → `my/project/ns.clj`)。

### ネームスペースのリロード
ファイルを編集した後、編集した名前空間を REPL に再ロードして、更新された定義がアクティブになるようにします。

```clojure
(require 'my.namespace :reload)
```

## 評価前のコードのインデント
ブラケットのバランサーを機能させるには、一貫したインデントが重要です。

```clojure
;; ❌
(defn my-function [x]
(+ x 2))

;; ✅
(defn my-function [x]
  (+ x 2))
```

## インデントの設定

条件と本文を別の行に記述します。

```clojure
(when limit
  (println "Limit set to:" limit))
```

`and` 引数と `or` 引数を別々の行に記述します。

```clojure
(if (and condition-a
         condition-b)
  this
  that)
```

## インライン定義パターン

println/console.log よりもインライン def デバッグを優先します。

### デバッグ用のインライン `def`
- インライン `def` バインディングは、REPL 作業中に中間状態を検査できる状態に保ちます。
- インラインバインディングが引き続き探索を支援する場合は、インラインバインディングをそのままにしておきます。

```clojure
(defn process-instructions [instructions]
  (def instructions instructions)
  (let [grouped (group-by :status instructions)]
    grouped))
```

- リアルタイム検査は引き続き利用可能です。
- デバッグサイクルは高速なままです。
- 反復開発は引き続きスムーズに行われます。

チャットでユーザーコードを表示するときに「インライン def」を使用すると、ユーザーがコードブロック内からコードを簡単に試すことができます。ユーザーは Calva を使用して、コードブロック内のコードを直接評価できます。 (ただし、ユーザーはそこでコードを編集できません。)

## 戻り値 > 副作用の出力

標準出力に出力するよりも、REPL を使用して評価から値を返すことを優先します。

## `stdin` からの読み取り
- Clojure コードで `(read-line)` を使用すると、VS Code を通じてユーザーにプロンプ​​トが表示されます。
- Babashka の nREPL には標準入力がサポートされていないため、標準入力の読み取りは避けてください。
- REPL がブロックされた場合は、REPL を再起動するようにユーザーに依頼します。

## データ構造の設定

私たちはデータ構造を可能な限りフラットに保つよう努め、名前空間付きのキーワードに重点を置き、簡単に分割できるように最適化しています。通常、アプリでは名前空間付きのキーワードが使用され、ほとんどの場合は「合成」名前空間が使用されます。

パラメータリスト内のキーを直接構造化します。

```clojure
(defn handle-user-request
  [{:user/keys [id name email]
    :request/keys [method path headers]
    :config/keys [timeout debug?]}]
  (when debug?
    (println "Processing" method path "for" name)))
```

これには多くの利点がありますが、その中でも特に、関数の署名が透過的に保たれます。

### 組み込みのシャドウイングを避ける
コア機能が隠蔽されないように、必要に応じて受信キーの名前を変更します。

```clojure
(defn create-item
  [{:prompt-sync.file/keys [path uri]
    file-name :prompt-sync.file/name
    file-type :prompt-sync.file/type}]
  #js {:label file-name
       :type file-type})
```

自由にしておくべき一般的なシンボル:
- `class`
- `count`
- `empty?`
- `filter`
- `first`
- `get`
- `key`
- `keyword`
- `map`
- `merge`
- `name`
- `reduce`
- `rest`
- `set`
- `str`
- `symbol`
- `type`
- `update`

## 不必要なラッパー関数を避ける
名前によって構成が本当に明確になる場合を除き、コア関数をラップしないでください。

```clojure
(remove (set exclusions) items) ; a wrapper function would not make this clearer
```

## ドキュメント用のリッチコメント フォーム (RCF)

リッチコメント フォーム `(comment ...)` は、REPL の直接評価とは異なる目的を果たします。ファイル編集で RCF を使用して、REPL ですでに検証した関数の **使用パターンと例を文書化**します。

### RCF を使用する場合
- **REPL 検証後** - 動作例をファイルに文書化する
- **使用方法に関するドキュメント** - 関数がどのように使用されるかを示します。
- **探索の保存** - 有用な REPL 発見をコードベースに保持します
- **シナリオ例** - 特殊なケースと一般的な使用法を示します

### RCFパターン
RCF = リッチコメント フォーム。

ファイルがロードされるとき、RCF 内のコードは評価されません。人間はそこにあるコードを自由に簡単に評価できるため、使用例を文書化するのに最適です。

```clojure
(defn process-user-data
  "Processes user data with validation"
  [{:user/keys [name email] :as user-data}]
  ;; implementation here
  )

(comment
  ;; Basic usage
  (process-user-data {:user/name "John" :user/email "john@example.com"})

  ;; Edge case - missing email
  (process-user-data {:user/name "Jane"})

  ;; Integration example
  (->> users
       (map process-user-data)
       (filter :valid?))

  :rcf) ; Optional marker for end of comment block
```

### RCF と REPL ツールの使用法
```clojure
;; In chat - show direct REPL evaluation:
(in-ns 'my.namespace)
(let [test-data {:user/name "example"}]
  (process-user-data test-data))

;; In files - document with RCF:
(comment
  (process-user-data {:user/name "example"})
  :rcf)
```

## テスト

### REPL からテストを実行する
ターゲットの名前空間をリロードし、REPL からテストを実行して、即時のフィードバックを取得します。

```clojure
(require '[my.project.some-test] :reload)
(clojure.test/run-tests 'my.project.some-test)
(cljs.test/run-tests 'my.project.some-test)
```

- REPL のより緊密な統合。
- 集中的な実行。
- より簡単なデバッグ。
- テストデータへの直接アクセス。

障害を調査するときは、テスト名前空間内から個々のテスト変数を実行することを優先します。

### REPL-First TDD ワークフローを使用する
ファイルを編集する前に実際のデータを反復処理します。

```clojure
(def sample-text "line 1\nline 2\nline 3\nline 4\nline 5")

(defn format-line-number [n padding marker-len]
  (let [num-str (str n)
        total-padding (- padding marker-len)]
    (str (apply str (repeat (- total-padding (count num-str)) " "))
         num-str)))

(deftest line-number-formatting
  (is (= "  5" (editor-util/format-line-number 5 3 0))
      "Single digit with padding 3, no marker space")
  (is (= " 42" (editor-util/format-line-number 42 3 0))
      "Double digit with padding 3, no marker space"))
```

#### 利点
- 変更をコミットする前に検証された動作
- 即時フィードバックを伴う増分開発
- 既知の良好な動作をキャプチャするテスト
- 意図を固定するために、失敗したテストから新しい作業を開始します

### テストの名前付けとメッセージング
`deftest` の名前は、冗長な `-test` 接尾辞を付けずに、わかりやすい名前（地域/物のスタイル）にしてください。

### テストアサーション メッセージスタイル
複数の関連するアサーションをグループ化する場合にのみ、`testing` ブロックを使用して、期待メッセージを `is` に直接添付します。

```clojure
(deftest line-marker-formatting
  (is (= "→" (editor-util/format-line-marker true))
      "Target line gets marker")
  (is (= "" (editor-util/format-line-marker false))
      "Non-target gets empty string"))

(deftest context-line-extraction
  (testing "Centered context extraction"
    (let [result (editor-util/get-context-lines "line 1\nline 2\nline 3" 2 3)]
      (is (= 3 (count (str/split-lines result)))
          "Should have 3 lines")
      (is (str/includes? result "→")
          "Should have marker"))))
```

ガイドライン:
- アサーションメッセージで期待されることを明示的に保ちます。
- 関連するチェックをグループ化するには `testing` を使用します。
- `line-marker-formatting` や `context-line-extraction` のようなケバブケース名を維持します。

## 楽しい対話型プログラミング

仕事では REPL を優先することを忘れないでください。ユーザーにはあなたが評価した内容は表示されないことに注意してください。結果も。何を評価し、何を返すかについてチャットでユーザーとコミュニケーションします。

