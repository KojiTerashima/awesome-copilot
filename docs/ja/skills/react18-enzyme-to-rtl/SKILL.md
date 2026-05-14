---
name: react18-enzyme-to-rtl
description: 'Provides exact Enzyme → React Testing Library migration patterns for React 18 upgrades. Use this skill whenever Enzyme tests need to be rewritten - shallow, mount, wrapper.find(), wrapper.simulate(), wrapper.prop(), wrapper.state(), wrapper.instance(), Enzyme configure/Adapter calls, or any test file that imports from enzyme. This skill covers the full API mapping and the philosophy shift from implementation testing to behavior testing. Always read this skill before rewriting Enzyme tests - do not translate Enzyme APIs 1:1, that produces brittle RTL tests.'
---
# React 18 酵素 → RTL 移行

Enzyme には React 18 アダプターも React 18 サポート パスもありません。すべての酵素テストは、React Testing Library を使用して書き直す必要があります。

## 哲学の転換 (最初にお読みください)

酵素テストの実施。 RTL は動作をテストします。```jsx
// 酵素: コンポーネントが正しい内部状態を持っているかどうかをテストします
Expect(wrapper.state('count')).toBe(3);
Expect(wrapper.instance().handleClick).toBeDefined();
Expect(wrapper.find('Button').prop('disabled')).toBe(true);

// RTL: ユーザーが実際に何を見て何ができるかをテストします
Expect(screen.getByText('Count: 3')).toBeInTheDocument();
Expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled();
「」これは 1 対 1 の翻訳ではありません。内部状態またはインスタンス メソッドを検証する酵素テストには、RTL に相当するものがありません。これは、RTL が内部を意図的に公開していないためです。 **代わりに目に見える結果をアサートするようにテストを書き直してください。**

## API マップ

各酵素 API の完全な前/後コードについては、以下をお読みください。
- **`references/enzyme-api-map.md`** - 完全なマッピング: 浅い、マウント、検索、シミュレート、プロパティ、状態、インスタンス、構成
- **`references/async-patterns.md`** - waitFor、findBy、act()、Apollo MockedProvider、ロード状態、エラー状態

## コア書き換えテンプレート```jsx
// すべての酵素テストは次の形状に書き換えられます。
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
userEvent を '@testing-library/user-event' からインポートします。
MyComponent を './MyComponent' からインポートします。

description('MyComponent', () => {
  it('そのことを実行します', async () => {
    // 1. レンダリング (シャロー/マウントを置き換えます)
    render(<MyComponent prop="value" />);

    // 2. クエリ (wrapper.find() を置き換えます)
    const button = screen.getByRole('button', { name: /submit/i });

    // 3. インタラクト (simulator() を置き換えます)
    userEvent.setup().click(button); を待ちます。

    // 4. 可視出力でアサートします (wrapper.state() /wrapper.prop() を置き換えます)
    Expect(screen.getByText('Submitted!')).toBeInTheDocument();
  });
});
「」## RTL クエリの優先順位 (この順序で使用)

1. `getByRole` - アクセス可能な役割 (ボタン、テキストボックス、見出し、チェックボックスなど) と一致します。
2. `getByLabelText` - ラベルにリンクされたフォームフィールド
3. `getByPlaceholderText` - プレースホルダーの入力
4. `getByText` - 表示されるテキスト コンテンツ
5. `getByDisplayValue` - input/select/textarea の現在値
6. `getByAltText` - 画像の代替テキスト
7. `getByTitle` - タイトル属性
8. `getByTestId` - `data-testid` 属性 (最後の手段)

`getByTestId` よりも `getByRole` を優先します。アクセシビリティもテストします。

## プロバイダーによるラッピング```jsx
// コンテキスト付きの酵素:
const ラッパー = マウント(
  <ApolloProvider client={client}>
    <ThemeProvider テーマ={テーマ}>
      <MyComponent />
    </テーマプロバイダ>
  </Apolloプロバイダ>
);

// RTL に相当するもの (プロジェクトの CustomRender を使用するか、インラインでラップします):
import { render } から '@testing-library/react';
レンダリング(
  <MockedProvider モック={モック} addTypename={false}>
    <ThemeProvider テーマ={テーマ}>
      <MyComponent />
    </テーマプロバイダ>
  </モックプロバイダー>
);
// または、プロバイダーをラップする場合はプロジェクトの CustomRender ヘルパーを使用します
「」
