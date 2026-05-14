# 非同期テスト パターン - 酵素 → RTL 移行

Enzyme の非同期テストを React 18 互換パターンを使用して React Testing Library に書き換えるためのリファレンス。

## 核心的な問題

Enzyme の非同期テストでは通常、次のいずれかのアプローチが使用されます。

- `wrapper.update()` 状態変更後
- `setTimeout` / `Promise.resolve()` マイクロタスクをフラッシュします
- `setImmediate` 非同期キューをフラッシュします
- `wrapper.update()` が続く直接インスタンス メソッド呼び出し

これらはどれも RTL では機能しません。 RTL では、代わりに `waitFor`、`findBy*`、および `act` が提供されます。

---

## パターン 1 - 状態変更後のwrapper.update()

Enzyme では、非同期状態の変更後に強制的に再レンダリングを行うために `wrapper.update()` が必要でした。```jsx
// 酵素:
it('データをロード', async () => {
  const ラッパー = mount(<UserList />);
  Promise.resolve() を待ちます。 // マイクロタスクをフラッシュします
  ラッパー.update();        // Enzyme を強制的に DOM と同期させます
  Expect(wrapper.find('li')).toHaveLength(3);
});
「」

```jsx
// RTL - waitFor は再レンダリングを自動的に処理します。
import { render, screen, waitFor } from '@testing-library/react';

it('データをロード', async () => {
  render(<UserList />);
  await waitFor(() => {
    Expect(screen.getAllByRole('listitem')).toHaveLength(3);
  });
});
「」---

## パターン 2 - ユーザー操作によってトリガーされる非同期アクション```jsx
// 酵素:
it('ボタンクリック時にユーザーを取得', async () => {
  const ラッパー = mount(<UserCard />);
  Wrapper.find('button').simulate('click');
  await 新しい Promise(resolve => setTimeout(resolve, 0));
  ラッパー.update();
  Expect(wrapper.find('.user-name').text()).toBe('John Doe');
});
「」

```jsx
// RTL:
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
userEvent を '@testing-library/user-event' からインポートします。

it('ボタンクリック時にユーザーを取得', async () => {
  render(<UserCard />);
  await userEvent.setup().click(screen.getByRole('button', { name: /load/i }));
  // findBy* は最大 1000 ミリ秒まで自動待機します (構成可能)
  Expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
「」---

## パターン 3 - 状態アサーションの読み込み中```jsx
// 酵素 - 読み込み状態を同期的にアサートし、フラッシュ後の最終状態をアサートします。
it('読み込みと結果を表示', async () => {
  const ラッパー = mount(<SearchResults query="react" />);
  Expect(wrapper.find('.spinner').exists()).toBe(true);
  await 新しい Promise(resolve => setTimeout(resolve, 100));
  ラッパー.update();
  Expect(wrapper.find('.spinner').exists()).toBe(false);
  Expect(wrapper.find('.result')).toHaveLength(5);
});
「」

```jsx
// RTL:
it('読み込みと結果を表示', async () => {
  render(<SearchResults query="react" />);
  // 状態を読み込み中 - 表示されることを確認します
  Expect(screen.getByRole('progressbar')).toBeInTheDocument();
  // または、読み込みがテキストの場合:
  Expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // 結果が表示されるまで待ちます (読み込みが消え、結果が表示されます)
  await waitFor(() => {
    Expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
  });
  Expect(screen.getAllByRole('listitem')).toHaveLength(5);
});
「」---

## パターン 4 - Apollo MockedProvider 非同期テスト```jsx
// Apollo を使用した酵素 - 複数のティックでフラッシュするために使用されます:
it('クエリからユーザーをレンダリング', async () => {
  const ラッパー = マウント(
    <MockedProvider モック={モック} addTypename={false}>
      <ユーザープロファイル id="1" />
    </モックプロバイダー>
  );
  await 新しい Promise(resolve => setTimeout(resolve, 0)); // Apollo キューをフラッシュします
  ラッパー.update();
  Expect(wrapper.find('.username').text()).toBe('Alice');
});
「」

```jsx
// Apollo を使用した RTL:
import { render, screen, waitFor } from '@testing-library/react';
import { MockedProvider } から '@apollo/client/testing';

it('クエリからユーザーをレンダリング', async () => {
  レンダリング(
    <MockedProvider モック={モック} addTypename={false}>
      <ユーザープロファイル id="1" />
    </モックプロバイダー>
  );

  // Apollo がクエリを解決するまで待ちます
  Expect(await screen.findByText('Alice')).toBeInTheDocument();
  // または:
  await waitFor(() => {
    Expect(screen.getByText('Alice')).toBeInTheDocument();
  });
});
「」**RTL での Apollo ロード状態:**```jsx
it('データの読み込みを示します', async () => {
  レンダリング(
    <MockedProvider モック={モック} addTypename={false}>
      <ユーザープロファイル id="1" />
    </モックプロバイダー>
  );
  // Apollo の読み込み状態 - レンダリング直後に確認します
  Expect(screen.getByText(/loading/i)).toBeInTheDocument();
  // その後データを待ちます
  Expect(await screen.findByText('Alice')).toBeInTheDocument();
});
「」---

## パターン 5 - 非同期操作によるエラー状態```jsx
// 酵素:
it('失敗したフェッチでエラーが表示される', async () => {
  server.use(rest.get('/api/user', (req, res, ctx) => res(ctx.status(500))));
  const ラッパー = mount(<UserCard />);
  Wrapper.find('button').simulate('click');
  await 新しい Promise(resolve => setTimeout(resolve, 0));
  ラッパー.update();
  Expect(wrapper.find('.error-message').text()).toContain('何か問題が発生しました');
});
「」

```jsx
// RTL:
it('失敗したフェッチでエラーが表示される', async () => {
  // (フェッチに MSW または jest.mock を想定)
  render(<UserCard />);
  await userEvent.setup().click(screen.getByRole('button', { name: /load/i }));
  Expect(await screen.findByText(/何か問題が発生した/i)).toBeInTheDocument();
});
「」---

## パターン 6 - 手動非同期制御の act()

非同期タイミングを明示的に制御する必要がある場合 (RTL ではまれですが、クラス コンポーネントのテストで必要になる場合があります):```jsx
// きめ細かい非同期制御のための act() を使用した RTL:
import {act} から 'react';

it('順次状態更新を処理します', async () => {
  render(<MultiStepForm />);

  await act(async () => {
    fireEvent.click(screen.getByRole('button', { name: /next/i }));
    Promise.resolve() を待ちます。 // マイクロタスクキューをフラッシュします
  });

  Expect(screen.getByText('ステップ 2')).toBeInTheDocument();
});
「」---

## RTL 非同期クエリ ガイド

|方法 |行動 | | の場合に使用します。
|---|---|---|
| `getBy*` |同期 - 見つからない場合はスローします。要素は常にすぐに存在します。
| `queryBy*` |同期 - 見つからない場合は null を返します。チェック要素が存在しません |
| `findBy*` |非同期 - 最大 1000 ミリ秒待機し、見つからない場合は拒否します。要素は非同期的に表示されます |
| `getAllBy*` |同期 - 0 が見つかった場合にスローします。複数の要素が常に存在します。
| `queryAllBy*` |同期 - 何も見つからない場合は [] を返します。カウントまたは存在しないことを確認する |
| `findAllBy*` |非同期 - 要素が表示されるのを待ちます。複数の要素が非同期的に表示されます。
| `waitFor(fn)` |エラーまたはタイムアウトがなくなるまで fn を再試行します。ポーリングが必要なカスタム アサーション |
| `waitForElementToBeRemoved(el)` |要素が消えるまで待つ |ロード状態、削除 |

**デフォルトのタイムアウト:** 1000ms。 `jest.config.js` でグローバルに構成します。```js
// 遅い CI 環境のタイムアウトを増やします
// jest.config.js
module.exports = {
  testEnvironmentOptions: {
    asyncUtilTimeout: 3000、
  }、
};
「」---

## よくある移行の間違い```jsx
// 誤り - 非同期クエリと同期アサーションが混在しています:
const el = await screen.findByText('Result');
// el はここですでに解決されています - findBy は Promise ではなく要素を返します
Expect(el を待つ).toBeInTheDocument(); // 不要な秒待ち

// 正解:
const el = await screen.findByText('Result');
Expect(el).toBeInTheDocument();
// または単純に:
Expect(await screen.findByText('Result')).toBeInTheDocument();
「」

```jsx
// 間違っています - 非同期的に表示される要素に getBy* を使用します。
fireEvent.click(ボタン);
Expect(screen.getByText('Loaded!')).toBeInTheDocument(); // データをロードする前にスローします

// 正解:
fireEvent.click(ボタン);
Expect(await screen.findByText('Loaded!')).toBeInTheDocument(); // 待ちます
「」
