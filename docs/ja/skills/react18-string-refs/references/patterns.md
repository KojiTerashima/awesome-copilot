# 文字列参照 - すべての移行パターン

## DOM 要素の単一参照 {#single-ref}

最も一般的なケースは、1 つの DOM ノードに対する 1 つの参照です。```jsx
// Before:
class SearchBox extends React.Component {
  handleSearch() {
    const value = this.refs.searchInput.value;
    this.props.onSearch(value);
  }

  focusInput() {
    this.refs.searchInput.focus();
  }

  render() {
    return (
      <div>
        <input ref="searchInput" type="text" placeholder="Search..." />
        <button onClick={() => this.handleSearch()}>Search</button>
      </div>
    );
  }
}
```

```jsx
// After:
class SearchBox extends React.Component {
  searchInputRef = React.createRef();

  handleSearch() {
    const value = this.searchInputRef.current.value;
    this.props.onSearch(value);
  }

  focusInput() {
    this.searchInputRef.current.focus();
  }

  render() {
    return (
      <div>
        <input ref={this.searchInputRef} type="text" placeholder="Search..." />
        <button onClick={() => this.handleSearch()}>Search</button>
      </div>
    );
  }
}
```---

## 1 つのコンポーネント内の複数の参照 {#multiple-refs}

各文字列 ref は、独自の名前付き `createRef()` フィールドになります。```jsx
// Before:
class LoginForm extends React.Component {
  handleSubmit(e) {
    e.preventDefault();
    const email = this.refs.emailField.value;
    const password = this.refs.passwordField.value;
    this.props.onSubmit({ email, password });
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input ref="emailField" type="email" />
        <input ref="passwordField" type="password" />
        <button type="submit">Log in</button>
      </form>
    );
  }
}
```

```jsx
// After:
class LoginForm extends React.Component {
  emailFieldRef = React.createRef();
  passwordFieldRef = React.createRef();

  handleSubmit(e) {
    e.preventDefault();
    const email = this.emailFieldRef.current.value;
    const password = this.passwordFieldRef.current.value;
    this.props.onSubmit({ email, password });
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input ref={this.emailFieldRef} type="email" />
        <input ref={this.passwordFieldRef} type="password" />
        <button type="submit">Log in</button>
      </form>
    );
  }
}
```---

## リスト内の参照 / 動的参照 {#list-refs}

マップ/ループ内の文字列参照 - 最も注意が必要なケースです。各項目には独自の参照が必要です。```jsx
// Before:
class TabPanel extends React.Component {
  focusTab(index) {
    this.refs[`tab_${index}`].focus();
  }

  render() {
    return (
      <div>
        {this.props.tabs.map((tab, i) => (
          <button key={tab.id} ref={`tab_${i}`}>
            {tab.label}
          </button>
        ))}
      </div>
    );
  }
}
```

```jsx
// After - use a Map to store refs dynamically:
class TabPanel extends React.Component {
  tabRefs = new Map();

  getOrCreateRef(id) {
    if (!this.tabRefs.has(id)) {
      this.tabRefs.set(id, React.createRef());
    }
    return this.tabRefs.get(id);
  }

  focusTab(index) {
    const tab = this.props.tabs[index];
    this.tabRefs.get(tab.id)?.current?.focus();
  }

  render() {
    return (
      <div>
        {this.props.tabs.map((tab) => (
          <button key={tab.id} ref={this.getOrCreateRef(tab.id)}>
            {tab.label}
          </button>
        ))}
      </div>
    );
  }
}
```**代替案 - リストのコールバック参照 (より単純):**```jsx
class TabPanel extends React.Component {
  tabRefs = {};

  focusTab(index) {
    this.tabRefs[index]?.focus();
  }

  render() {
    return (
      <div>
        {this.props.tabs.map((tab, i) => (
          <button
            key={tab.id}
            ref={el => { this.tabRefs[i] = el; }}  // callback ref stores DOM node directly
          >
            {tab.label}
          </button>
        ))}
      </div>
    );
  }
}
// Note: callback refs store the DOM node directly (not wrapped in .current)
// this.tabRefs[i] is the element, not this.tabRefs[i].current
```---

## コールバック参照 (createRef の代替) {#callback-refs}

コールバック参照は `createRef()` の代替です。これらはリスト (上記) や、ref のアタッチ/デタッチ時にコードを実行する必要がある場合に便利です。```jsx
// Callback ref syntax:
class MyComponent extends React.Component {
  // Callback ref - called with the element when it mounts, null when it unmounts
  setInputRef = (el) => {
    this.inputEl = el; // stores the DOM node directly (no .current needed)
  };

  focusInput() {
    this.inputEl?.focus(); // direct DOM node access
  }

  render() {
    return <input ref={this.setInputRef} />;
  }
}
```**コールバック参照と createRef を使用する場合:**

- `createRef()` - コンポーネント定義時に既知の固定数の参照用 (ほとんどの場合)
- コールバック参照 - 動的リストの場合、アタッチ/デタッチに反応する必要がある場合、または参照が変更される可能性がある場合

**重要:** インライン コールバック ref (レンダリングで定義) は、レンダリングごとに新しい関数を再作成します。これにより、各レンダリング サイクルで `null` を使用して ref が呼び出され、次に要素が呼び出されます。代わりに、バインドされたメソッドまたはクラス フィールドのアロー関数を使用します。```jsx
// AVOID - new function every render, causes ref flicker:
render() {
  return <input ref={(el) => { this.inputEl = el; }} />;  // inline - bad
}

// PREFER - stable reference:
setInputRef = (el) => { this.inputEl = el; };  // class field - good
render() {
  return <input ref={this.setInputRef} />;
}
```---

## 子コンポーネントに渡される参照 {#forwarded-refs}

文字列参照がカスタム コンポーネント (DOM 要素ではない) に渡された場合、移行には子も更新する必要があります。```jsx
// Before:
class Parent extends React.Component {
  handleClick() {
    this.refs.myInput.focus(); // Parent accesses child's DOM node
  }
  render() {
    return (
      <div>
        <MyInput ref="myInput" />
        <button onClick={() => this.handleClick()}>Focus</button>
      </div>
    );
  }
}

// MyInput.js (child - class component):
class MyInput extends React.Component {
  render() {
    return <input className="my-input" />;
  }
}
```

```jsx
// After:
class Parent extends React.Component {
  myInputRef = React.createRef();

  handleClick() {
    this.myInputRef.current.focus();
  }

  render() {
    return (
      <div>
        {/* React 18: forwardRef needed. React 19: ref is a direct prop */}
        <MyInput ref={this.myInputRef} />
        <button onClick={() => this.handleClick()}>Focus</button>
      </div>
    );
  }
}

// MyInput.js (React 18 - use forwardRef):
import { forwardRef } from 'react';
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input ref={ref} className="my-input" />;
});

// MyInput.js (React 19 - ref as direct prop, no forwardRef):
function MyInput({ ref, ...props }) {
  return <input ref={ref} className="my-input" />;
}
```

---
