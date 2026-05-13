---
description: 'Go の慣用的な実践とコミュニティ標準に従って Go コードを書くための instruction'
applyTo: '**/*.go,**/go.mod,**/go.sum'
---

# Go 開発 instruction

[Effective Go](https://go.dev/doc/effective_go)、[Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)、[Google's Go Style Guide](https://google.github.io/styleguide/go/) に基づき、Go の慣用的な実践とコミュニティ標準に従って Go コードを書いてください。

## 一般 instruction

- シンプルで明確、かつ慣用的な Go コードを書く
- 巧妙さよりも明快さと単純さを優先する
- 最小驚きの原則に従う
- happy path を左寄せに保つ (インデントを最小化する)
- ネストを減らすため早期 return を使う
- if-else 連鎖より早期 return を優先し、`if condition { return }` パターンを使って else ブロックを避ける
- zero value を有用にする
- 明確で説明的な名前による自己文書化コードを書く
- export される型、関数、メソッド、package を文書化する
- 依存管理には Go module を使う
- 車輪の再発明ではなく Go 標準ライブラリを活用する (例: 文字列連結には `strings.Builder`、パス構築には `filepath.Join`)
- 同等機能がある場合は custom 実装より標準ライブラリ解決策を優先する
- コメントは既定で英語で書き、翻訳はユーザー要求時のみ行う
- コードとコメントで emoji を使わない

## 命名規則

### Packages

- 小文字の単語 1 語の package 名を使う
- underscore、hyphen、mixedCaps を避ける
- package が含むものではなく、提供するものを表す名前を選ぶ
- `util`、`common`、`base` のような汎用名を避ける
- package 名は複数形ではなく単数形にする

#### Package Declaration Rules (重要):
- **`package` 宣言を絶対に重複させない** - 各 Go ファイルには `package` 行がちょうど 1 つ必要
- 既存の `.go` ファイルを編集するとき:
  - 既存の `package` 宣言を **保持** し、もう 1 つ追加しない
  - ファイル内容全体を置き換える必要がある場合も、既存の package 名で始める
- 新しい `.go` ファイルを作成するとき:
  - **コードを書く前に**、同じディレクトリ内の他の `.go` ファイルがどの package 名を使っているか確認する
  - そのディレクトリ内の既存ファイルと **同じ** package 名を使う
  - 新しいディレクトリなら、ディレクトリ名を package 名に使う
  - ファイルの先頭に **ちょうど 1 行だけ** `package <name>` を書く
- ファイル作成や置換ツールを使うとき:
  - `package` 宣言を追加する前に、対象ファイルにすでに宣言がないか **必ず確認** する
  - 内容を置き換える場合、新しい内容には `package` 宣言を 1 つだけ含める
  - 複数の `package` 行や重複宣言を含むファイルを **絶対に** 作らない

### Variables and Functions

- underscore ではなく mixedCaps または MixedCaps (camelCase) を使う
- 短くても説明的な名前にする
- 単一文字変数は、ごく短いスコープ (ループ インデックスなど) に限る
- export される名前は大文字で始める
- export されない名前は小文字で始める
- どもりを避ける (例: `http.HTTPServer` ではなく `http.Server`)

### Interfaces

- 可能であれば interface 名には -er 接尾辞を付ける (例: `Reader`, `Writer`, `Formatter`)
- 単一メソッド interface は、そのメソッド名由来で命名する (例: `Read` → `Reader`)
- interface は小さく責務を絞る

### Constants

- export される定数は MixedCaps を使う
- export されない定数は mixedCaps を使う
- 関連する定数は `const` block でまとめる
- より高い型安全性のため typed constant を検討する

## コードスタイルとフォーマット

### Formatting

- コード整形には常に `gofmt` を使う
- import 管理には `goimports` を使う
- 行長は常識的な範囲に保つ (厳密な上限はないが、可読性を考慮する)
- 論理グループを区切るため空行を追加する

### Comments

- 自己文書化コードを目指し、コメントより明確な変数名、関数名、構造を優先する
- コメントは、複雑なロジック、業務ルール、自明でない振る舞いを説明する必要がある場合にのみ書く
- コメントは既定で英語の完全な文で書く
- 他言語への翻訳は明示的なユーザー要求時のみ行う
- 文は対象となるものの名前で始める
- package comment は "Package [name]" で始める
- 多くの場合は line comment (`//`) を使う
- block comment (`/* */`) は主に package ドキュメントなど、限定的に使う
- "何を" より "なぜ" を説明する。ただし "何を" の説明が複雑な場合は除く
- コメントやコードで emoji を使わない

### Error Handling

- 関数呼び出し直後に error を確認する
- 正当な理由がない限り `_` で error を無視しない (理由を書く)
- `%w` を使った `fmt.Errorf` で context を付けて error を wrap する
- 特定エラーを判定する必要がある場合は custom error type を作る
- error return は最後の return value に置く
- error 変数名は `err` にする
- error message は小文字で始め、句点を付けない

## アーキテクチャとプロジェクト構造

### Package Organization

- 標準的な Go project layout に従う
- `main` package は `cmd/` ディレクトリに置く
- 再利用可能な package は `pkg/` または `internal/` に置く
- 外部 project から import させたくない package には `internal/` を使う
- 関連機能を package ごとにまとめる
- 循環依存を避ける

### Dependency Management

- Go module (`go.mod` と `go.sum`) を使う
- 依存は最小限に保つ
- セキュリティ修正のため依存を定期的に更新する
- 未使用依存を整理するには `go mod tidy` を使う
- 依存の vendor 化は必要な場合だけ行う

## 型安全性と言語機能

### Type Definitions

- 意味付けと型安全性のために型を定義する
- JSON、XML、database mapping には struct tag を使う
- 明示的な型変換を優先する
- type assertion は慎重に使い、第 2 戻り値を確認する
- unconstrained type より generics を優先し、真に unconstrained type が必要な場合は `interface{}` ではなく予約済み alias `any` を使う (Go 1.18+)

### Pointers vs Values

- 大きな struct、または receiver を変更する必要がある場合は pointer receiver を使う
- 小さな struct や不変性を保ちたい場合は value receiver を使う
- 引数を変更する必要がある場合や大きな struct には pointer parameter を使う
- 小さな struct や変更を防ぎたい場合は value parameter を使う
- 1 つの型の method set 内では一貫性を保つ
- pointer receiver と value receiver を選ぶときは zero value を考慮する

### Interfaces and Composition

- interface を受け取り、具体型を返す
- interface は小さく保つ (理想は 1〜3 メソッド)
- 合成には embedding を使う
- interface は実装場所ではなく使用場所の近くで定義する
- 必要がない限り interface を export しない

## Concurrency

### Goroutines

- library 内で goroutine を作るのは慎重に行い、可能なら呼び出し側に concurrency 制御を委ねる
- どうしても library 内で goroutine を作る必要がある場合は、明確なドキュメントと cleanup 手段を提供する
- goroutine がどう終了するかを常に把握する
- goroutine の待機には `sync.WaitGroup` または channel を使う
- cleanup を保証して goroutine leak を防ぐ

### Channels

- goroutine 間通信には channel を使う
- メモリ共有で通信するのではなく、通信でメモリ共有する
- channel は receiver 側ではなく sender 側で close する
- 容量が分かっている場合は buffered channel を使う
- non-blocking operation には `select` を使う

### Synchronization

- 共有状態の保護には `sync.Mutex` を使う
- クリティカル セクションは小さく保つ
- reader が多い場合は `sync.RWMutex` を使う
- channel と mutex は用途で使い分ける: 通信には channel、状態保護には mutex
- 一度きりの初期化には `sync.Once` を使う
- Go version ごとの WaitGroup 利用:
	- `go.mod` の `go >= 1.25` なら、新しい `WaitGroup.Go` メソッドを使う ([documentation](https://pkg.go.dev/sync#WaitGroup)):
		```go
		var wg sync.WaitGroup
		wg.Go(task1)
		wg.Go(task2)
		wg.Wait()
		```
	- `go < 1.25` なら、従来の `Add`/`Done` パターンを使う

## Error Handling Patterns

### Creating Errors

- 単純な固定 error には `errors.New` を使う
- 動的な error には `fmt.Errorf` を使う
- ドメイン固有 error には custom error type を作る
- sentinel error には export された error 変数を使う
- error 判定には `errors.Is` と `errors.As` を使う

### Error Propagation

- stack を上がる際は context を付与する
- error を log して返すことはしない (どちらか一方)
- error は適切なレベルで処理する
- より良いデバッグのため structured error を検討する

## API Design

### HTTP Handlers

- 単純な handler には `http.HandlerFunc` を使う
- 状態が必要な handler には `http.Handler` を実装する
- 横断的関心事には middleware を使う
- 適切な status code と header を設定する
- error は丁寧に処理し、適切な error response を返す
- Go version ごとの router 使用:
	- `go >= 1.22` なら、pattern-based routing と method matching を備えた強化版 `net/http` `ServeMux` を優先する
	- `go < 1.22` なら、従来の `ServeMux` を使って method/path を手動処理する (正当な理由があれば third-party router を使ってよい)

### JSON APIs

- JSON marshaling を制御するには struct tag を使う
- 入力データを検証する
- 任意フィールドには pointer を使う
- 遅延パースには `json.RawMessage` の利用を検討する
- JSON error を適切に処理する

### HTTP Clients

- client struct には設定と依存関係だけを持たせる (例: base URL、`*http.Client`、auth、既定 header)。request ごとの状態は保持しない
- client struct の中に `*http.Request` を保存またはキャッシュしてはならず、call をまたいで request 固有状態を保持してはならない。各 method 呼び出しごとに新しい request を組み立てる
- method は `context.Context` と入力パラメーターを受け取り、ローカル (または call ごとの短命 builder/helper) で `*http.Request` を組み立てた後に `c.httpClient.Do(req)` を呼ぶ
- request 作成ロジックを再利用する必要がある場合は、unexported helper function または call ごとの builder type に切り出す。長寿命 client 上に `http.Request` (URL param、body、header) を field として保持してはならない
- 基盤となる `*http.Client` は適切に設定し (timeout、transport)、concurrent use に安全であることを保証する。最初の利用後に `Transport` を変更しない
- 送信する request instance に対して必ず header を設定し、response body は `defer resp.Body.Close()` で閉じつつ error を適切に扱う

## パフォーマンス最適化

### メモリ管理

- ホットパスでの allocation を最小化する
- 可能ならオブジェクトを再利用する (`sync.Pool` を検討)
- 小さな struct には value receiver を使う
- サイズが分かっている slice は事前確保する
- 不要な文字列変換を避ける

### I/O: Reader と Buffer

- 多くの `io.Reader` stream は一度しか消費できません。読み取りは状態を進めます。特別な対処なしに再読できると仮定してはいけません
- データを複数回読む必要がある場合は、一度 buffer してから必要に応じて reader を作り直します:
	- `io.ReadAll` (または制限付き read) で `[]byte` を取得し、再利用ごとに `bytes.NewReader(buf)` または `bytes.NewBuffer(buf)` で新しい reader を作る
	- 文字列には `strings.NewReader(s)` を使う。`*bytes.Reader` なら `Seek(0, io.SeekStart)` で巻き戻せる
- HTTP request では、消費済みの `req.Body` を再利用してはいけません。代わりに:
	- 元 payload を `[]byte` として保持し、送信前ごとに `req.Body = io.NopCloser(bytes.NewReader(buf))` を設定する
	- redirect/retry で transport が body を再生成できるよう、`req.GetBody = func() (io.ReadCloser, error) { return io.NopCloser(bytes.NewReader(buf)), nil }` を設定することを優先する
- 読みながら stream を複製するには `io.TeeReader` (buffer へ複写しつつ通す) や `io.MultiWriter` (複数 sink へ書く) を使う
- buffer 済み reader を再利用するには `(*bufio.Reader).Reset(r)` で新しい基底 reader を割り当てる。読み取り元が seek をサポートしない限り「巻き戻せる」と期待してはいけない
- 大きな payload では無制限 buffer を避け、streaming、`io.LimitReader`、または一時ファイル保存でメモリ使用量を制御する

- payload 全体を buffer せずに stream するには `io.Pipe` を使う:
	- 別 goroutine で `*io.PipeWriter` へ書き、reader 側が消費する
	- writer は必ず close し、失敗時は `CloseWithError(err)` を使う
	- `io.Pipe` は streaming 用であり、巻き戻しや reader の再利用用ではない

- **警告:** `io.Pipe` を使うとき (特に multipart writer と組み合わせる場合)、すべての書き込みは厳密な順序で直列に行わなければなりません。同時実行や順不同の書き込みはしないでください。multipart boundary と chunk の順序を保持する必要があります。順不同または並列の書き込みは stream を破損し、error を引き起こします。

- `io.Pipe` で multipart/form-data を stream する:
	- `pr, pw := io.Pipe()`; `mw := multipart.NewWriter(pw)`; `pr` を HTTP request body に使う
	- `Content-Type` は `mw.FormDataContentType()` に設定する
	- goroutine 内で、すべての part を正しい順序で `mw` に書く。error 時は `pw.CloseWithError(err)`、成功時は `mw.Close()` のあと `pw.Close()`
	- 長寿命 client に request/in-flight form の状態を保存しない。call ごとに組み立てる
	- stream された body は巻き戻せない。retry/redirect 用には小さな payload を buffer するか `GetBody` を提供する

### Profiling

- 組み込み profiling tool (`pprof`) を使う
- 重要なコード パスを benchmark する
- 最適化前に profile を取る
- まずアルゴリズム改善に集中する
- benchmark には `testing.B` の利用を検討する

## Testing

### Test Organization

- test は同じ package に置く (white-box testing)
- black-box testing には `_test` package suffix を使う
- test file 名は `_test.go` suffix にする
- test file は対象コードの隣に置く

### Writing Tests

- 複数ケースには table-driven test を使う
- test 名は `Test_functionName_scenario` 形式で分かりやすくする
- 整理しやすくするため `t.Run` で subtest を使う
- 成功ケースと error ケースの両方をテストする
- `testify` などの library は価値がある場合に限って使い、単純な test を不必要に複雑化しない

### Test Helpers

- helper 関数には `t.Helper()` を付ける
- 複雑な setup には test fixture を作る
- test と benchmark の両方で使う関数には `testing.TB` interface を使う
- リソース cleanup には `t.Cleanup()` を使う

## セキュリティ ベストプラクティス

### Input Validation

- 外部入力はすべて検証する
- 無効状態を防ぐため強い型付けを使う
- SQL query に使う前にデータを sanitize する
- ユーザー入力から来る file path は慎重に扱う
- 文脈ごとに適切な検証と escape を行う (HTML、SQL、shell)

### Cryptography

- 標準ライブラリの crypto package を使う
- 暗号を自作しない
- 乱数生成には crypto/rand を使う
- パスワード保存には bcrypt、scrypt、または argon2 を使う (必要に応じて golang.org/x/crypto を検討)
- ネットワーク通信には TLS を使う

## Documentation

### Code Documentation

- 明確な命名と構造を通じた自己文書化コードを優先する
- export されるすべての symbol を、明確で簡潔な説明で文書化する
- documentation は symbol 名で始める
- documentation は既定で英語で書く
- 必要に応じて例を documentation に含める
- documentation はコードの近くに置く
- コード変更時は documentation も更新する
- documentation や comment で emoji を使わない

### README と Documentation Files

- 明確な setup 手順を含める
- 依存関係と要件を文書化する
- 使用例を提供する
- 設定オプションを文書化する
- トラブルシューティング セクションを含める

## Tool と開発ワークフロー

### Essential Tools

- `go fmt`: コード整形
- `go vet`: 疑わしい構文を検出
- `golangci-lint`: 追加 lint (`golint` は非推奨)
- `go test`: test 実行
- `go mod`: 依存管理
- `go generate`: コード生成

### Development Practices

- commit 前に test を実行する
- formatting と lint には pre-commit hook を使う
- commit は焦点を絞り atomic に保つ
- 意味のある commit message を書く
- commit 前に diff を見直す

## 避けるべき一般的な落とし穴

- error を確認しないこと
- race condition を無視すること
- goroutine leak を作ること
- cleanup に defer を使わないこと
- map を並行変更すること
- nil interface と nil pointer の違いを理解しないこと
- リソース (file、connection) の close を忘れること
- 不必要に global variable を使うこと
- unconstrained type (`any` など) の使いすぎ。具体型または制約付き generic type parameter を優先し、必要な場合も `interface{}` ではなく `any` を使う
- 型の zero value を考慮しないこと
- **`package` 宣言を重複作成すること** - これは compile error になるため、package 宣言追加前に既存ファイルを必ず確認する
