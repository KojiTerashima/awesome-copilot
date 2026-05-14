---
title: React 19 Actions Pattern Reference
---
# React 19 アクション パターン リファレンス

React 19 では、読み込み状態、エラー処理、オプティミスティック更新が組み込まれた非同期操作 (フォーム送信など) を処理するためのパターンである **アクション** が導入されています。これにより、`useReducer + state` パターンがより単純な API に置き換えられます。

## アクションとは何ですか?

**アクション**は、次のような非同期関数です。

- フォームの送信時またはボタンのクリック時に自動的に呼び出すことができます
- 自動読み込み/保留状態で実行
- 完了すると UI が自動的に更新されます
- サーバーコンポーネントと連携して直接サーバーを変更します。

---

## useActionState()

`useActionState` は、クライアント側のアクション フックです。フォーム処理のために `useReducer + useEffect` を置き換えます。

### React 18 パターン```jsx
// React 18  form with useReducer + state:
function Form() {
  const [state, dispatch] = useReducer(
    (state, action) => {
      switch (action.type) {
        case 'loading':
          return { ...state, loading: true, error: null };
        case 'success':
          return { ...state, loading: false, data: action.data };
        case 'error':
          return { ...state, loading: false, error: action.error };
      }
    },
    { loading: false, data: null, error: null }
  );
  
  async function handleSubmit(e) {
    e.preventDefault();
    dispatch({ type: 'loading' });
    try {
      const result = await submitForm(new FormData(e.target));
      dispatch({ type: 'success', data: result });
    } catch (err) {
      dispatch({ type: 'error', error: err.message });
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="email" />
      {state.loading && <Spinner />}
      {state.error && <Error msg={state.error} />}
      {state.data && <Success data={state.data} />}
      <button disabled={state.loading}>Submit</button>
    </form>
  );
}
```### React 19 useActionState() パターン```jsx
// React 19  same form with useActionState:
import { useActionState } from 'react';

async function submitFormAction(prevState, formData) {
  // prevState = previous return value from this function
  // formData = FormData from <form action={submitFormAction}>
  
  try {
    const result = await submitForm(formData);
    return { data: result, error: null };
  } catch (err) {
    return { data: null, error: err.message };
  }
}

function Form() {
  const [state, formAction, isPending] = useActionState(
    submitFormAction,
    { data: null, error: null } // initial state
  );
  
  return (
    <form action={formAction}>
      <input name="email" />
      {isPending && <Spinner />}
      {state.error && <Error msg={state.error} />}
      {state.data && <Success data={state.data} />}
      <button disabled={isPending}>Submit</button>
    </form>
  );
}
```**相違点:**

- `useReducer` + ロジックの代わりに 1 つのフック
- `formAction` は `onSubmit` を置き換え、フォームは自動的に FormData を収集します
- `isPending` はブール値であり、ディスパッチ呼び出しはありません
- アクション関数は`(prevState, formData)`を受け取ります

---

## useFormStatus()

`useFormStatus` は、最も近いフォームから保留状態を読み取る **子コンポーネント フック**です。これは、プロップの穴あけなしで、組み込みの `isPending` 信号のように機能します。```jsx
// React 18  must pass isPending as prop:
function SubmitButton({ isPending }) {
  return <button disabled={isPending}>Submit</button>;
}

function Form({ isPending, formAction }) {
  return (
    <form action={formAction}>
      <input />
      <SubmitButton isPending={isPending} />
    </form>
  );
}

// React 19  useFormStatus reads it automatically:
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>Submit</button>;
}

function Form() {
  const [state, formAction] = useActionState(submitFormAction, {});
  
  return (
    <form action={formAction}>
      <input />
      <SubmitButton /> {/* No prop needed */}
    </form>
  );
}
```**重要なポイント:** `useFormStatus` は `<form action={...}>` 内でのみ機能し、通常の `<form onSubmit>` はトリガーしません。

---

## useOptimistic()

`useOptimistic` は、非同期操作の実行中に UI をすぐに更新します。操作が成功すると、確認されたデータが楽観的な値を置き換えます。失敗すると、UI が元に戻ります。

### React 18 パターン```jsx
// React 18  manual optimistic update:
function TodoList({ todos, onAddTodo }) {
  const [optimistic, setOptimistic] = useState(todos);
  
  async function handleAddTodo(text) {
    const newTodo = { id: Date.now(), text, completed: false };
    
    // Show optimistic update immediately
    setOptimistic([...optimistic, newTodo]);
    
    try {
      const result = await addTodo(text);
      // Update with confirmed result
      setOptimistic(prev => [
        ...prev.filter(t => t.id !== newTodo.id),
        result
      ]);
    } catch (err) {
      // Revert on error
      setOptimistic(optimistic);
    }
  }
  
  return (
    <ul>
      {optimistic.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```### React 19 useOptimistic() パターン```jsx
import { useOptimistic } from 'react';

async function addTodoAction(prevTodos, formData) {
  const text = formData.get('text');
  const result = await addTodo(text);
  return [...prevTodos, result];
}

function TodoList({ todos }) {
  const [optimistic, addOptimistic] = useOptimistic(
    todos,
    (state, newTodo) => [...state, newTodo]
  );
  
  const [, formAction] = useActionState(addTodoAction, todos);
  
  async function handleAddTodo(formData) {
    const text = formData.get('text');
    // Optimistic update:
    addOptimistic({ id: Date.now(), text, completed: false });
    // Then call the form action:
    formAction(formData);
  }
  
  return (
    <>
      <ul>
        {optimistic.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      <form action={handleAddTodo}>
        <input name="text" />
        <button>Add</button>
      </form>
    </>
  );
}
```**重要なポイント:**

- `useOptimistic(currentState, updateFunction)`
- `updateFunction` は `(state, optimisticInput)` を受け取り、新しい状態を返します
- `addOptimistic(input)` を呼び出してオプティミスティック更新をトリガーします
- サーバー アクションの戻り値は、完了時に楽観的な状態を置き換えます。

---

## 完全な例: すべてのフックを含む Todo リスト```jsx
import { useActionState, useFormStatus, useOptimistic } from 'react';

// Server action:
async function addTodoAction(prevTodos, formData) {
  const text = formData.get('text');
  if (!text) throw new Error('Text required');
  const newTodo = await api.post('/todos', { text });
  return [...prevTodos, newTodo];
}

// Submit button with useFormStatus:
function AddButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Adding...' : 'Add Todo'}</button>;
}

// Main component:
function TodoApp({ initialTodos }) {
  const [optimistic, addOptimistic] = useOptimistic(
    initialTodos,
    (state, newTodo) => [...state, newTodo]
  );
  
  const [todos, formAction] = useActionState(
    addTodoAction,
    initialTodos
  );
  
  async function handleAddTodo(formData) {
    const text = formData.get('text');
    // Optimistic: show it immediately
    addOptimistic({ id: Date.now(), text });
    // Then submit the form (which updates when server confirms)
    await formAction(formData);
  }
  
  return (
    <>
      <ul>
        {optimistic.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      <form action={handleAddTodo}>
        <input name="text" placeholder="Add a todo..." required />
        <AddButton />
      </form>
    </>
  );
}
```---

## 移行戦略

### フェーズ 1 変更は必要ありません

アクションはオプトインです。既存の `useReducer + onSubmit` パターンはすべて引き続き機能します。強制移住はありません。

### フェーズ 2 リファクタリング候補を特定する

React 19 の移行が安定したら、`useReducer + async` パターンのプロファイルを作成します。```bash
grep -rn "useReducer.*case.*'loading\|useReducer.*case.*'success" src/ --include="*.js" --include="*.jsx"
```リファクタリングする価値のあるパターン:

- 読み込み中/エラー状態のフォーム送信
- ユーザーイベントによってトリガーされる非同期操作
- 現在のコードは `dispatch({ type: '...' })` を使用しています
- 単純な状態形状 (`loading`、`error`、`data` を持つオブジェクト)

### フェーズ 3 useActionState へのリファクタリング```jsx
// Before:
function LoginForm() {
  const [state, dispatch] = useReducer(loginReducer, { loading: false, error: null, user: null });
  
  async function handleSubmit(e) {
    e.preventDefault();
    dispatch({ type: 'loading' });
    try {
      const user = await login(e.target);
      dispatch({ type: 'success', data: user });
    } catch (err) {
      dispatch({ type: 'error', error: err.message });
    }
  }
  
  return <form onSubmit={handleSubmit}>...</form>;
}

// After:
async function loginAction(prevState, formData) {
  try {
    const user = await login(formData);
    return { user, error: null };
  } catch (err) {
    return { user: null, error: err.message };
  }
}

function LoginForm() {
  const [state, formAction] = useActionState(loginAction, { user: null, error: null });
  
  return <form action={formAction}>...</form>;
}
```---

## 比較表

|特集 |反応18 |反応19 |
|---|---|---|
|フォーム処理 | `onSubmit` + useReducer | `action` + useActionState |
|ロード状態 |手動ディスパッチ |自動 `isPending` |
|子コンポーネントの保留状態 |プロペラ穴あけ | `useFormStatus` フック |
|楽観的なアップデート |マニュアルステートダンス | `useOptimistic` フック |
|エラー処理 |マニュアル発送中 |行動から戻る |
|複雑さ |定型文をもっと見る |定型文を減らす |