---
name: unit-test-vue-pinia
category: testing
description: 'Write and review unit tests for Vue 3 + TypeScript + Vitest + Pinia codebases. Use when creating or updating tests for components, composables, and stores; mocking Pinia with createTestingPinia; applying Vue Test Utils patterns; and enforcing black-box assertions over implementation details.'
---
# 単体テスト-vue-pinia

このスキルを使用して、Vue コンポーネント、コンポーザブル、Ponia ストアの単体テストを作成またはレビューします。テストは小さく、決定論的で、動作を第一に考えてください。

## ワークフロー

1. 最初に動作の境界を特定します (コンポーネント UI の動作、コンポーザブルの動作、またはストアの動作)。
2. その動作を証明できる最も範囲の狭いテスト スタイルを選択します。
3. シナリオをカバーできる最も強力でないオプションを使用して Pinia をセットアップします。
4. 小道具、フォームの更新、ボタンのクリック、発行された子イベント、ストア API などのパブリック入力を通じてテストを実行します。
5. インスタンス レベルのアサーションを考慮する前に、観察可能な出力と副作用をアサートします。
6. 動作指向の明確な名前を付けてテストを返すかレビューし、残りのカバレッジ ギャップに注意します。

## コアルール

- テストごとに 1 つの動作をテストします。
- 観察可能な入出力動作を最初にアサートします (レンダリングされたテキスト、発行されたイベント、コールバック呼び出し、ストア状態の変更)。
- 実装に結合されたアサーションを避けます。
- `wrapper.vm` にアクセスするのは、適切な DOM、prop、emit、またはストアレベルのアサーションがない例外的な場合に限られます。
- `beforeEach()` での明示的なセットアップを優先し、テストごとにモックをリセットします。
- `references/pinia-patterns.md` にチェックインされた参照資料を、標準の Pinia テスト設定の信頼できるローカル ソースとして使用します。

## Pinia テストのアプローチ

最初に `references/pinia-patterns.md` を使用し、チェックインされたサンプルでケースがカバーされない場合は、Ponia のテスト クックブックに戻ります。

### コンポーネントテストのデフォルトパターン

マウント中に `createTestingPinia` をグローバル プラグインとして使用します。
一貫性とアクションスパイアサーションを容易にするために、`createSpy: vi.fn` をデフォルトとして推奨します。```ts
const wrapper = mount(ComponentUnderTest, {
	global: {
		plugins: [
			createTestingPinia({
				createSpy: vi.fn,
			}),
		],
	},
});
```デフォルトでは、アクションはスタブ化され監視されます。
テストでアクションが呼び出された (または呼び出されなかった) かどうかだけを確認する必要がある場合は、`stubActions: true` (デフォルト) を使用します。

### 最小限の Pinia セットアップを受け入れました

以下も有効であり、間違っているとフラグを立てるべきではありません。

- `createTestingPinia({})` テストで Pinia アクションのスパイ動作がアサートされない場合。
- `createSpy` を使用しない `createTestingPinia({ initialState: ... })` または `createTestingPinia({ stubActions: ... })`。テストで必要なのは状態シードまたはアクション スタブ動作のみで、生成されたスパイを検査しない場合です。
- `setActivePinia(createTestingPinia(...))` は、依存ストアのモック/シードが必要な場合のストア/コンポーザブルに重点を置いたテスト (コンポーネントのマウントなし) で使用します。

アクション スパイ アサーションがテスト意図の一部である場合は、`createSpy: vi.fn` を使用します。

### 必要な場合にのみ実際のアクションを実行する

`stubActions: false` は、テストでアクションの実際の動作と副作用を検証する必要がある場合にのみ使用します。単純な「呼び出された」アサーションの場合は、デフォルトでオンにしないでください。```ts
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
```### `initialState` を使用したシード ストアの状態```ts
const wrapper = mount(ComponentUnderTest, {
	global: {
		plugins: [
			createTestingPinia({
				createSpy: vi.fn,
				initialState: {
					counter: { n: 20 },
					user: { name: "Leia Organa" },
				},
			}),
		],
	},
});
```### `createTestingPinia` を通じて Pinia プラグインを追加します```ts
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
```### エッジケースのゲッターオーバーライドパターン```ts
const pinia = createTestingPinia({ createSpy: vi.fn });
const store = useCounterStore(pinia);

store.double = 999;
// @ts-expect-error test-only reset of overridden getter
store.double = undefined;
```### 純粋なストア単体テスト

コンポーネントのレンダリングを行わずにストアの状態遷移とアクションの動作を検証することが目的の場合は、`createPinia()` を使用した純粋なストア テストを推奨します。 `createTestingPinia()` は、スタブ化された依存ストア、シードされたテスト ダブル、またはアクション スパイが必要な場合にのみ使用します。```ts
beforeEach(() => {
	setActivePinia(createPinia());
});

it("increments", () => {
	const counter = useCounterStore();
	counter.increment();
	expect(counter.n).toBe(1);
});
```## Vue テストユーティリティのアプローチ

Vue Test Utils のガイダンスに従ってください: <https://test-utils.vuejs.org/guide/>

- 集中的な単体テストのためにデフォルトで浅くマウントします。
- 統合動作が対象となる場合にのみ、完全なコンポーネント ツリーをマウントします。
- 小道具、ユーザーのようなインタラクション、発行されたイベントを通じて動作を推進します。
- 親の内部に触れる代わりに、子スタブ イベントに対して `findComponent(...).vm.$emit(...)` を優先します。
- `nextTick` は、更新が非同期である場合にのみ使用します。
- 発行されたイベントとペイロードを `wrapper.emitted(...)` でアサートします。
- `wrapper.vm` にアクセスするのは、DOM アサーション、発行されたイベント アサーション、prop アサーション、またはストア レベル アサーションが動作を表現できない場合のみです。これを例外として扱い、アサーションの範囲を狭くしてください。

## 主要なテストのスニペット

ペイロードを発行してアサートします。```ts
await wrapper.find("button").trigger("click");
expect(wrapper.emitted("submit")?.[0]?.[0]).toBe("Mango Mission");
```入力を更新し、出力をアサートします。```ts
await wrapper.find("input").setValue("Agent Violet");
await wrapper.find("form").trigger("submit");
expect(wrapper.emitted("save")?.[0]?.[0]).toBe("Agent Violet");
```## テスト作成のワークフロー

1. テストする動作の境界を特定します。
2. 最小限のフィクスチャ データ (その動作に必要なフィールドのみ) を構築します。
3. Pinia と必要なテスト ダブルを構成します。
4. パブリック入力を通じて動作をトリガーします。
5. 公開された成果と副作用を主張します。
6. 実装ではなく動作を説明するためにテスト名をリファクタリングします。

## 制約と安全性

- プライベート/内部実装の詳細をテストしないでください。
- 動的 UI 動作のためにスナップショットを過度に使用しないでください。
- 1 つの動作だけが重要な場合は、大きなオブジェクトのすべてのフィールドをアサートしないでください。
- 偽のデータを決定論的に保ちます。ランダムな値は避けてください。
- 上記の許容される最小限のセットアップの 1 つである Pinia セットアップが間違っていると主張しないでください。
- テスト対象の動作に追加の表面積が必要な場合を除き、動作テストをより深い実装や実際の動作に向けて書き直さないでください。
- レビュー中に、欠落しているテスト カバレッジ、脆弱なセレクター、および実装に結合されたアサーションに明示的にフラグを立てます。

## 出力コントラクト

- `create` または `update` の場合は、完成したテスト コードと、選択した Pinia 戦略を説明する短いメモを返します。
- `review` の場合は、最初に具体的な結果を返し、次にカバレッジの欠落または脆弱性のリスクを返します。
- 最も安全な選択があいまいな場合は、選択したテスト設定を推進した仮定を述べてください。

## 参考文献

- @@コード3@@
- Pinia テスト クックブック: <https://pinia.vuejs.org/cookbook/testing.html>
- Vue テスト ユーティリティ ガイド: <https://test-utils.vuejs.org/guide/>