---
name: csharp-async
description: 'C# async programming の best practice を取得します'
---

# C# Async Programming Best Practices

あなたの目標は、C# の asynchronous programming に関する best practice に従えるよう支援することです。

## Naming Conventions

- すべての async method で `Async` suffix を使う
- 該当する場合は synchronous counterpart と対応する method 名にする（例: `GetData()` に対する `GetDataAsync()`）

## Return Types

- method が値を返す場合は `Task<T>` を返す
- method が値を返さない場合は `Task` を返す
- allocation を減らすため、高 performance scenario では `ValueTask<T>` を検討する
- event handler を除き、async method で `void` を返すのは避ける

## Exception Handling

- await 式の周囲では try/catch block を使う
- async method で exception を握りつぶさない
- library code では deadlock 防止のため、適切な場面で `ConfigureAwait(false)` を使う
- async Task returning method では throw する代わりに `Task.FromException()` で exception を伝播する

## Performance

- 複数 task の並列実行には `Task.WhenAll()` を使う
- timeout 実装や最初に完了した task を採用する用途には `Task.WhenAny()` を使う
- 単に task 結果をそのまま返すだけなら、不必要な async/await を避ける
- 長時間実行される処理では cancellation token を検討する

## Common Pitfalls

- async code で `.Wait()`、`.Result`、`.GetAwaiter().GetResult()` を使わない
- blocking code と async code を混在させない
- async void method を作らない（event handler を除く）
- `Task` を返す method は常に await する

## Implementation Patterns

- 長時間処理には async command pattern を実装する
- sequence を非同期処理するには async stream (`IAsyncEnumerable<T>`) を使う
- public API には task-based asynchronous pattern (TAP) を検討する

私の C# code を review するときは、これらの問題を特定し、これらの best practice に沿った改善案を提案してください。
