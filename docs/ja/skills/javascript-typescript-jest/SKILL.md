---
name: javascript-typescript-jest
description: 'Jestを使ったJavaScript/TypeScriptのテスト作成におけるベストプラクティス。モック戦略、テスト構造、一般的なパターンを含む。'
---

### テスト構造
- テストファイルは `.test.ts` または `.test.js` のサフィックスを付ける
- テストファイルはテスト対象のコードの隣か専用の `__tests__` ディレクトリに配置する
- 期待される動作を説明するわかりやすいテスト名を使う
- 関連するテストを整理するためにネストした describe ブロックを使う
- 以下のパターンに従う：`describe('Component/Function/Class', () => { it('should do something', () => {}) })`

### 効果的なモック
- 外部依存（API、データベースなど）をモックしてテストを分離する
- モジュールレベルのモックには `jest.mock()` を使う
- 特定の関数のモックには `jest.spyOn()` を使う
- モックの振る舞いは `mockImplementation()` や `mockReturnValue()` で定義する
- 各テスト後に `afterEach` で `jest.resetAllMocks()` を呼び、モックをリセットする

### 非同期コードのテスト
- テストでは常に Promise を返すか async/await 構文を使う
- Promiseには `resolves` / `rejects` マッチャーを使う
- 遅いテストには `jest.setTimeout()` で適切なタイムアウトを設定する

### スナップショットテスト
- UIコンポーネントや頻繁に変わらない複雑なオブジェクトにスナップショットテストを使う
- スナップショットは小さく焦点を絞って保つ
- コミット前にスナップショットの変更を慎重に確認する

### Reactコンポーネントのテスト
- コンポーネントのテストには Enzyme より React Testing Library を使う
- ユーザーの振る舞いとコンポーネントのアクセシビリティをテストする
- アクセシビリティのロール、ラベル、テキスト内容で要素をクエリする
- より現実的なユーザー操作には `fireEvent` より `userEvent` を使う

## よく使うJestマッチャー
- 基本：`expect(value).toBe(expected)`, `expect(value).toEqual(expected)`
- 真偽値：`expect(value).toBeTruthy()`, `expect(value).toBeFalsy()`
- 数値：`expect(value).toBeGreaterThan(3)`, `expect(value).toBeLessThanOrEqual(3)`
- 文字列：`expect(value).toMatch(/pattern/)`, `expect(value).toContain('substring')`
- 配列：`expect(array).toContain(item)`, `expect(array).toHaveLength(3)`
- オブジェクト：`expect(object).toHaveProperty('key', value)`
- 例外：`expect(fn).toThrow()`, `expect(fn).toThrow(Error)`
- モック関数：`expect(mockFn).toHaveBeenCalled()`, `expect(mockFn).toHaveBeenCalledWith(arg1, arg2)`
