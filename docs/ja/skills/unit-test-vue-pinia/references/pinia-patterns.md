# Pinia テスト スニペット (クックブックに合わせた)

`@pinia/testing` を使用してテストを作成する場合は、これらのパターンを直接使用します。

## `createTestingPinia` を使用したコンポーネントのマウント```ts
import { mount } from "@vue/test-utils";
import { createTestingPinia } from "@pinia/testing";
import { vi } from "vitest";

const wrapper = mount(ComponentUnderTest, {
  global: {
    plugins: [
      createTestingPinia({
        createSpy: vi.fn,
      }),
    ],
  },
});
```## 実際のアクションを実行する

これは、アクション内の動作を実行する必要がある場合にのみ使用します。
テストで通話/非通話の予測のみをチェックする場合は、デフォルトのスタブ (`stubActions: true`) を維持します。```ts
const wrapper = mount(ComponentUnderTest, {
  global: {
    plugins: [
      createTestingPinia({
        createSpy: vi.fn,
        stubActions: false,
      }),
    ],
  },
});
```## シード開始状態```ts
const wrapper = mount(ComponentUnderTest, {
  global: {
    plugins: [
      createTestingPinia({
        createSpy: vi.fn,
        initialState: {
          counter: { n: 10 },
          profile: { name: "Sherlock Holmes" },
        },
      }),
    ],
  },
});
```## テストでストアを使用し、アクション呼び出しをアサートします```ts
const pinia = createTestingPinia({ createSpy: vi.fn });
const store = useCounterStore(pinia);

store.increment();
expect(store.increment).toHaveBeenCalledTimes(1);
```## テスト対象のプラグインを追加```ts
const wrapper = mount(ComponentUnderTest, {
  global: {
    plugins: [
      createTestingPinia({
        createSpy: vi.fn,
        plugins: [myPiniaPlugin],
      }),
    ],
  },
});
```## エッジ テストのゲッターをオーバーライドしてリセットする```ts
const pinia = createTestingPinia({ createSpy: vi.fn });
const store = useCounterStore(pinia);

store.double = 42;
expect(store.double).toBe(42);

// @ts-expect-error test-only reset
store.double = undefined;
```
