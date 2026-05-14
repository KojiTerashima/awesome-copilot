# 酵素 API マップ - 完全な前/後

## セットアップ/構成```jsx
// 酵素:
「酵素」から酵素をインポートします。
'enzyme-adapter-react-16' からアダプターをインポートします。
Enzyme.configure({ アダプター: 新しいアダプター() });

// RTL: これを完全に削除します - セットアップは必要ありません
// (jest.config.js setupFilesAfterFramework は @testing-library/jest-dom マッチャーを処理します)
「」---

## レンダリング```jsx
// 酵素 - 浅い (子はレンダリングされません):
import { 浅い } から '酵素';
const ラッパー = 浅い(<MyComponent prop="value" />);

// RTL - レンダリング (完全なレンダリング、子が含まれる):
import { render } から '@testing-library/react';
render(<MyComponent prop="value" />);
// ラッパー変数は必要ありません - 画面経由でクエリします
「」

```jsx
// 酵素 - マウント (DOM による完全なレンダリング):
import { マウント } から '酵素';
const ラッパー = mount(<MyComponent />);

// RTL - 同じ render() 呼び出しがこれを処理します
render(<MyComponent />);
「」---

## クエリを実行する```jsx
// 酵素 - コンポーネントの種類で検索します。
const button =wrapper.find('button');
const comp =wrapper.find(ChildComponent);
const items =wrapper.find('.list-item');

// RTL - アクセス可能な属性によるクエリ:
const button = screen.getByRole('button');
const button = screen.getByRole('button', { name: /submit/i });
constHeading = screen.getByRole('Heading', { name: /title/i });
const input = screen.getByLabelText('電子メール');
const items = screen.getAllByRole('listitem');
「」

```jsx
// 酵素 - テキストで検索:
Wrapper.find('.message').text() === 'こんにちは'

// RTL:
screen.getByText('こんにちは')
screen.getByText(/hello/i) // 大文字と小文字を区別しない正規表現
「」---

## ユーザーインタラクション```jsx
// 酵素:
Wrapper.find('button').simulate('click');
Wrapper.find('input').simulate('change', { target: { value: 'hello' } });
Wrapper.find('form').simulate('submit');

// RTL - fireEvent (同期、低レベル):
import { fireEvent } から '@testing-library/react';
fireEvent.click(screen.getByRole('button'));
fireEvent.change(screen.getByRole('textbox'), { target: { value: 'hello' } });
fireEvent.submit(screen.getByRole('form'));

// RTL - userEvent (推奨、実際のユーザーの動作をシミュレートします):
userEvent を '@testing-library/user-event' からインポートします。
const ユーザー = userEvent.setup();
await user.click(screen.getByRole('button'));
await user.type(screen.getByRole('textbox'), 'hello');
await user.selectOptions(screen.getByRole('combobox'), 'option1');
「」**ほとんどの操作には `userEvent` を使用してください** - 実際のユーザーと同じように、完全なイベント シーケンス (ポインター ダウン、マウス ダウン、フォーカス、クリックなど) が発生します。 `fireEvent` は、特定のイベント プロパティをテストする場合にのみ使用します。

---

## Props と State のアサーション```jsx
// 酵素 - prop アサーション:
Expect(wrapper.find('input').prop('disabled')).toBe(true);
Expect(wrapper.prop('className')).toContain('active');

// RTL - 表示される属性をアサートします。
Expect(screen.getByRole('textbox')).toBeDisabled();
Expect(screen.getByRole('button')).toHaveAttribute('type', 'submit');
Expect(screen.getByRole('listitem')).toHaveClass('active');
「」

```jsx
// 酵素 - 状態アサーション (RTL に相当するものはありません):
Expect(wrapper.state('count')).toBe(3);
Expect(wrapper.state('loading')).toBe(false);

// RTL - 状態がレンダリングする内容をアサートします。
Expect(screen.getByText('Count: 3')).toBeInTheDocument();
Expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
「」**重要な原則:** 状態値をテストしないでください - 状態が UI で生成するものをテストしてください。コンポーネントが `<span>Count: {this.state.count}</span>` をレンダリングする場合は、そのスパンをテストします。

---

## インスタンスメソッド```jsx
// 酵素 - 直接メソッド呼び出し (RTL に相当するものはありません):
「wrapper.instance().handleSubmit();」
「wrapper.instance().loadData();」

// RTL - UI を介してトリガーします。
await userEvent.setup().click(screen.getByRole('button', { name: /submit/i }));
// または、UI トリガーが存在しない場合は、内部メソッドを直接テストする必要があるかどうか再考してください。
// 通常、答えは「いいえ」です。代わりに、レンダリングされた結果をテストします。
「」---

## 存在チェック```jsx
// 酵素:
Expect(wrapper.find('.error')).toHaveLength(1);
Expect(wrapper.find('.error')).toHaveLength(0);
Expect(wrapper.exists('.error')).toBe(true);

// RTL:
Expect(screen.getByText('エラー メッセージ')).toBeInTheDocument();
Expect(screen.queryByText('エラー メッセージ')).not.toBeInTheDocument();
// queryBy は見つからない場合にスローする代わりに null を返します
// 見つからない場合は getBy をスローします - 肯定的なアサーションで使用します
// findBy は Promise を返します - 非同期要素に使用します
「」---

## 複数の要素```jsx
// 酵素:
Expect(wrapper.find('li')).toHaveLength(5);
Wrapper.find('li').forEach((item, i) => {
  Expect(item.text()).toBe(expectedItems[i]);
});

// RTL:
const items = screen.getAllByRole('listitem');
Expect(items).toHaveLength(5);
items.forEach((item, i) => {
  Expect(item).toHaveTextContent(expectedItems[i]);
});
「」---

## 前/後: コンポーネントのテストを完了する```jsx
// 酵素のバージョン:
import { 浅い } から '酵素';

description('ログインフォーム', () => {
  it('資格情報を使用して送信', () => {
    const モックサブミット = jest.fn();
    const ラッパー = 浅い(<LoginForm onSubmit={mockSubmit} />);

    Wrapper.find('input[name="email"]').simulate('change', {
      ターゲット: { 値: 'user@example.com' }
    });
    Wrapper.find('input[name="パスワード"]').simulate('change', {
      ターゲット: { 値: 'password123' }
    });
    Wrapper.find('button[type="submit"]').simulate('click');

    Expect(wrapper.state('loading')).toBe(true);
    Expect(mockSubmit).toHaveBeenCalledWith({
      電子メール: 'user@example.com'、
      パスワード: 'password123'
    });
  });
});
「」

```jsx
// RTL バージョン:
import { render, screen } from '@testing-library/react';
userEvent を '@testing-library/user-event' からインポートします。

description('ログインフォーム', () => {
  it('資格情報を使用して送信', async () => {
    const モックサブミット = jest.fn();
    const ユーザー = userEvent.setup();
    render(<LoginForm onSubmit={mockSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // 状態ではなく、表示される出力でアサートします
    Expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled(); // 読み込み状態
    Expect(mockSubmit).toHaveBeenCalledWith({
      電子メール: 'user@example.com'、
      パスワード: 'password123'
    });
  });
});
「」
