# Web API と DOM リファレンス

最新のブラウザで使用できるドキュメント オブジェクト モデル (DOM) と Web API の包括的なリファレンス。

## ドキュメント オブジェクト モデル (DOM)

### DOM とは何ですか?
DOM は、HTML および XML ドキュメントのプログラミング インターフェイスです。これは、JavaScript で操作できるオブジェクトのツリーとしてページ構造を表します。

**DOM ツリー構造**:```
Document
└── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── header
        ├── main
        └── footer
```### DOM ノードの種類

|ノードタイプ |説明 |例 |
|----------|---------------|----------|
|要素 | HTML要素 | `<div>`、`<p>` |
|テキスト |テキストの内容 |要素内のテキスト |
|コメント | HTML コメント | `<!-- comment -->` |
|ドキュメント |ルートドキュメント | `document` |
|ドキュメントフラグメント |軽量ドキュメント |バッチ操作の場合 |

### 要素の選択```javascript
// By ID
const element = document.getElementById('myId');

// By class name (returns HTMLCollection)
const elements = document.getElementsByClassName('myClass');

// By tag name (returns HTMLCollection)
const divs = document.getElementsByTagName('div');

// Query selector (first match)
const first = document.querySelector('.myClass');
const advanced = document.querySelector('div.container > p:first-child');

// Query selector all (returns NodeList)
const all = document.querySelectorAll('.myClass');

// Special selectors
document.body; // Body element
document.head; // Head element
document.documentElement; // <html> element
```### DOM の横断```javascript
const element = document.querySelector('#myElement');

// Parent
element.parentElement;
element.parentNode;

// Children
element.children; // HTMLCollection of child elements
element.childNodes; // NodeList of all child nodes
element.firstElementChild;
element.lastElementChild;

// Siblings
element.nextElementSibling;
element.previousElementSibling;

// Closest ancestor matching selector
element.closest('.container');

// Check if element contains another
parent.contains(child); // true/false
```### 要素の作成と変更```javascript
// Create element
const div = document.createElement('div');
const text = document.createTextNode('Hello');
const fragment = document.createDocumentFragment();

// Set content
div.textContent = 'Plain text'; // Safe (escaped)
div.innerHTML = '<strong>HTML</strong>'; // Can be unsafe with user input

// Set attributes
div.setAttribute('id', 'myDiv');
div.setAttribute('class', 'container');
div.id = 'myDiv'; // Direct property
div.className = 'container';
div.classList.add('active');
div.classList.remove('inactive');
div.classList.toggle('visible');
div.classList.contains('active'); // true/false

// Set styles
div.style.color = 'red';
div.style.backgroundColor = 'blue';
div.style.cssText = 'color: red; background: blue;';

// Data attributes
div.dataset.userId = '123'; // Sets data-user-id="123"
div.getAttribute('data-user-id'); // "123"

// Insert into DOM
parent.appendChild(div); // Add as last child
parent.insertBefore(div, referenceNode); // Insert before reference
parent.prepend(div); // Add as first child (modern)
parent.append(div); // Add as last child (modern)
element.after(div); // Insert after element
element.before(div); // Insert before element
element.replaceWith(newElement); // Replace element

// Remove from DOM
element.remove(); // Modern way
parent.removeChild(element); // Old way

// Clone element
const clone = element.cloneNode(true); // true = deep clone (with children)
```### 要素のプロパティ```javascript
// Dimensions and position
element.offsetWidth; // Width including border
element.offsetHeight; // Height including border
element.clientWidth; // Width excluding border
element.clientHeight; // Height excluding border
element.scrollWidth; // Total scrollable width
element.scrollHeight; // Total scrollable height
element.offsetTop; // Top position relative to offsetParent
element.offsetLeft; // Left position relative to offsetParent

// Bounding box
const rect = element.getBoundingClientRect();
// Returns: { x, y, width, height, top, right, bottom, left }

// Scroll position
element.scrollTop; // Vertical scroll position
element.scrollLeft; // Horizontal scroll position
element.scrollTo(0, 100); // Scroll to position
element.scrollIntoView(); // Scroll element into view

// Check visibility
element.checkVisibility(); // Modern API
```## イベント処理

### イベントリスナーの追加```javascript
// addEventListener (modern, recommended)
element.addEventListener('click', handleClick);
element.addEventListener('click', handleClick, { once: true }); // Remove after first trigger

function handleClick(event) {
  console.log('Clicked!', event);
}

// Event options
element.addEventListener('scroll', handleScroll, {
  passive: true, // Won't call preventDefault()
  capture: false, // Bubble phase (default)
  once: true // Remove after one call
});

// Remove event listener
element.removeEventListener('click', handleClick);
```### 一般的なイベント

|カテゴリー |イベント |
|----------|----------|
|マウス | `click`、`dblclick`、`mousedown`、`mouseup`、`mousemove`、`mouseenter`、`mouseleave`、`contextmenu` |
|キーボード | `keydown`、`keyup`、`keypress` (非推奨) |
|フォーム | `submit`、`change`、`input`、`focus`、`blur`、`invalid` |
|ウィンドウ | `load`、`DOMContentLoaded`、`resize`、`scroll`、`beforeunload`、`unload` |
|タッチ | `touchstart`、`touchmove`、`touchend`、`touchcancel` |
|ドラッグ | `drag`、`dragstart`、`dragend`、`dragover`、`drop` |
|メディア | `play`、`pause`、`ended`、`timeupdate`、`loadeddata` |
|アニメーション | `animationstart`、`animationend`、`animationiteration` |
|移行 | `transitionstart`、`transitionend` |

### イベントオブジェクト```javascript
element.addEventListener('click', (event) => {
  // Target elements
  event.target; // Element that triggered event
  event.currentTarget; // Element with listener attached
  
  // Mouse position
  event.clientX; // X relative to viewport
  event.clientY; // Y relative to viewport
  event.pageX; // X relative to document
  event.pageY; // Y relative to document
  
  // Keyboard
  event.key; // 'a', 'Enter', 'ArrowUp'
  event.code; // 'KeyA', 'Enter', 'ArrowUp'
  event.ctrlKey; // true if Ctrl pressed
  event.shiftKey; // true if Shift pressed
  event.altKey; // true if Alt pressed
  event.metaKey; // true if Meta/Cmd pressed
  
  // Control event flow
  event.preventDefault(); // Prevent default action
  event.stopPropagation(); // Stop bubbling
  event.stopImmediatePropagation(); // Stop other listeners
});
```### イベントの委任

個々の子ではなく親でイベントを処理します。```javascript
// Instead of adding listener to each button
document.querySelector('.container').addEventListener('click', (event) => {
  if (event.target.matches('button')) {
    console.log('Button clicked:', event.target);
  }
});
```## Web ストレージ API

### ローカルストレージ

永続ストレージ (有効期限なし):```javascript
// Set item
localStorage.setItem('key', 'value');
localStorage.setItem('user', JSON.stringify({ name: 'John' }));

// Get item
const value = localStorage.getItem('key');
const user = JSON.parse(localStorage.getItem('user'));

// Remove item
localStorage.removeItem('key');

// Clear all
localStorage.clear();

// Get key by index
localStorage.key(0);

// Number of items
localStorage.length;

// Iterate all items
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(key, value);
}
```### セッションストレージ

タブを閉じるとストレージがクリアされます:```javascript
// Same API as localStorage
sessionStorage.setItem('key', 'value');
sessionStorage.getItem('key');
sessionStorage.removeItem('key');
sessionStorage.clear();
```**ストレージ制限**: オリジンあたり最大 5 ～ 10MB

## API を取得する

HTTP リクエスト用の最新の API:```javascript
// Basic GET request
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Async/await
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    
    // Check if successful
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);
  }
}

// POST request with JSON
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'John', age: 30 })
})
  .then(response => response.json())
  .then(data => console.log(data));

// With various options
fetch(url, {
  method: 'GET', // GET, POST, PUT, DELETE, etc.
  headers: {
    'Authorization': 'Bearer token',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data), // For POST/PUT
  mode: 'cors', // cors, no-cors, same-origin
  credentials: 'include', // include, same-origin, omit
  cache: 'no-cache', // default, no-cache, reload, force-cache
  redirect: 'follow', // follow, error, manual
  referrerPolicy: 'no-referrer' // no-referrer, origin, etc.
});

// Response methods
const text = await response.text(); // Plain text
const json = await response.json(); // JSON
const blob = await response.blob(); // Binary data
const arrayBuffer = await response.arrayBuffer(); // ArrayBuffer
const formData = await response.formData(); // FormData
```## その他の重要な Web API

### コンソール API```javascript
console.log('Message'); // Log message
console.error('Error'); // Error message (red)
console.warn('Warning'); // Warning message (yellow)
console.info('Info'); // Info message
console.table([{ a: 1 }, { a: 2 }]); // Table format
console.group('Group'); // Start group
console.groupEnd(); // End group
console.time('timer'); // Start timer
console.timeEnd('timer'); // End timer and log duration
console.clear(); // Clear console
console.assert(condition, 'Error message'); // Assert condition
```### タイマー```javascript
// Execute once after delay
const timeoutId = setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// Cancel timeout
clearTimeout(timeoutId);

// Execute repeatedly
const intervalId = setInterval(() => {
  console.log('Executed every second');
}, 1000);

// Cancel interval
clearInterval(intervalId);

// RequestAnimationFrame (for animations)
function animate() {
  // Animation code
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```### URL API```javascript
const url = new URL('https://example.com:8080/path?query=value#hash');

url.protocol; // 'https:'
url.hostname; // 'example.com'
url.port; // '8080'
url.pathname; // '/path'
url.search; // '?query=value'
url.hash; // '#hash'
url.href; // Full URL

// URL parameters
url.searchParams.get('query'); // 'value'
url.searchParams.set('newParam', 'newValue');
url.searchParams.append('query', 'another');
url.searchParams.delete('query');
url.searchParams.has('query'); // true/false

// Convert to string
url.toString(); // Full URL
```### フォームデータ API```javascript
// Create FormData from form
const form = document.querySelector('form');
const formData = new FormData(form);

// Create FormData manually
const data = new FormData();
data.append('username', 'john');
data.append('file', fileInput.files[0]);

// Get values
data.get('username'); // 'john'
data.getAll('files'); // Array of all 'files' values

// Iterate
for (const [key, value] of data.entries()) {
  console.log(key, value);
}

// Send with fetch
fetch('/api/upload', {
  method: 'POST',
  body: formData // Don't set Content-Type header
});
```### 交差点オブザーバー API

要素がビューポートに入ったときを検出します。```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element is visible');
      entry.target.classList.add('visible');
    }
  });
}, {
  threshold: 0.5, // 50% visible
  rootMargin: '0px'
});

observer.observe(element);
observer.unobserve(element);
observer.disconnect(); // Stop observing all
```### ミューテーションオブザーバー API

DOM の変更を監視します。```javascript
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('DOM changed:', mutation.type);
  });
});

observer.observe(element, {
  attributes: true, // Watch attribute changes
  childList: true, // Watch child elements
  subtree: true, // Watch all descendants
  characterData: true // Watch text content
});

observer.disconnect(); // Stop observing
```### 地理位置情報 API```javascript
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log(position.coords.latitude);
    console.log(position.coords.longitude);
  },
  (error) => {
    console.error('Error getting location:', error);
  },
  {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0
  }
);

// Watch position (continuous updates)
const watchId = navigator.geolocation.watchPosition(callback);
navigator.geolocation.clearWatch(watchId);
```### ウェブワーカー

バックグラウンド スレッドで JavaScript を実行します。```javascript
// Main thread
const worker = new Worker('worker.js');

worker.postMessage({ data: 'Hello' });

worker.onmessage = (event) => {
  console.log('From worker:', event.data);
};

worker.onerror = (error) => {
  console.error('Worker error:', error);
};

worker.terminate(); // Stop worker

// worker.js
self.onmessage = (event) => {
  console.log('From main:', event.data);
  self.postMessage({ result: 'Done' });
};
```### キャンバス API

グラフィックを描画する:```javascript
const canvas = document.querySelector('canvas');
const ctx = canvas.getContext('2d');

// Draw rectangle
ctx.fillStyle = 'blue';
ctx.fillRect(10, 10, 100, 50);

// Draw circle
ctx.beginPath();
ctx.arc(100, 100, 50, 0, Math.PI * 2);
ctx.fillStyle = 'red';
ctx.fill();

// Draw text
ctx.font = '20px Arial';
ctx.fillText('Hello', 10, 50);

// Draw image
const img = new Image();
img.onload = () => {
  ctx.drawImage(img, 0, 0);
};
img.src = 'image.jpg';
```### インデックス付きDB

大量の構造化データ用のクライアント側データベース:```javascript
// Open database
const request = indexedDB.open('MyDatabase', 1);

request.onerror = () => console.error('Database error');

request.onsuccess = (event) => {
  const db = event.target.result;
  // Use database
};

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const objectStore = db.createObjectStore('users', { keyPath: 'id' });
  objectStore.createIndex('name', 'name', { unique: false });
};

// Add data
const transaction = db.transaction(['users'], 'readwrite');
const objectStore = transaction.objectStore('users');
objectStore.add({ id: 1, name: 'John' });

// Get data
const request = objectStore.get(1);
request.onsuccess = () => console.log(request.result);
```## ベストプラクティス

### やるべきこと
- ✅ インライン イベント ハンドラーでは `addEventListener` を使用します
- ✅ 不要になったイベントリスナーを削除します
- ✅ 動的コンテンツにイベント委任を使用する
- ✅ DOM クエリを変数にキャッシュします
- ✅ プレーンテキストには `textContent` を使用します (`innerHTML` より安全です)
- ✅ バッチ DOM 操作には DocumentFragment を使用します
- ✅ デバウンス/スロットルスクロールおよびサイズ変更ハンドラー
- ✅ アニメーションには `requestAnimationFrame` を使用します
- ✅ ユーザー入力を検証し、サニタイズします

### やってはいけないこと
- ❌ 信頼できないデータには `innerHTML` を使用します (XSS リスク)
- ❌ ループ内で DOM を繰り返しクエリする
- ❌ タイトなループで DOM を変更する (バッチ操作)
- ❌ `document.write()` を使用します (非推奨)
- ❌ 同期 XMLHttpRequest を使用する
- ❌ 機密データを localStorage に保存する
- ❌ 非同期コードのエラー処理を無視する
- ❌ 重い計算を行うメインスレッドをブロックする

## 用語集の用語

**対象となる重要な用語**:
- API
- アプリケーションコンテキスト
- ビーコン
- 点滅
- 点滅要素
- ブラウザ
- コンテキストの閲覧
- バッファー
- キャンバス
- DOM (ドキュメント オブジェクト モデル)
- ドキュメント環境
- イベント
- エキスパンド
- グローバルオブジェクト
- グローバルな範囲
- 吊り上げ
- インデックス付きDB
- 補間
- ノード (DOM)
- シャドウツリー
- ウィンドウプロキシ
- ラッパー

## 追加のリソース

- [MDN DOM リファレンス](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [MDN Web API](https://developer.mozilla.org/en-US/docs/Web/API)
- [JavaScript.info DOM](https://javascript.info/document)