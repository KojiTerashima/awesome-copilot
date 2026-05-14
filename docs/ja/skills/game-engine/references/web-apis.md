# ゲーム開発用の Web API

ブラウザベースのゲームの構築に最も関連する Web プラットフォーム API をカバーする包括的なリファレンス。各セクションでは、API とは何か、ゲームにとって API が重要な理由、その主要なインターフェイスとメソッドについて説明し、該当する場合は簡単なコード例を示します。

---

## asm.js

### それは何ですか

asm.js は、ネイティブに近いパフォーマンスを実現するように設計された、厳格で高度に最適化可能な JavaScript のサブセットです。これは、JavaScript を整数、浮動小数点数、算術演算、単純な関数、ヒープ アクセスなどの狭い構成要素セットに制限し、オブジェクト、文字列、クロージャ、およびヒープ割り当てを必要とするものを禁止します。その結果、どのエンジンでも実行できる完全に有効な JavaScript が得られますが、サポートするエンジンは事前に積極的にコンパイルできます。

### ゲームにとってそれが重要な理由

- **ネイティブに近い速度**: Emscripten でコンパイルされた C/C++ ゲーム エンジンは、ブラウザ全体でネイティブに近いパフォーマンスで実行されます。
- **予測可能なパフォーマンス**: 制限された機能セットにより、非常に安定したフレーム レートが得られます。
- **C/C++ 移植性**: 既存のネイティブ ゲーム エンジンは、Emscripten を使用して asm.js にコンパイルし、Web 上にデプロイできます。
- **プラグインは必要ありません**: 最新のすべてのブラウザで標準の JavaScript として実行されます。

### 重要な概念

|コンセプト |説明 |
|----------|---------------|
|許可される構成体 | `while`、`if`、数値 (厳密な int/float)、トップレベルの名前付き関数、算術演算、関数呼び出し、ヒープ アクセス |
|許可されていない構成体 |オブジェクト、文字列、クロージャ、動的型強制、ヒープ割り当て構造 |
|コンパイラ ツールチェーン | Emscripten は C/C++ を asm.js にコンパイルします。
|エンジン認識 |ブラウザは `"use asm"` ディレクティブを検出し、事前コンパイルを適用します。

### 非推奨の通知

asm.js は非推奨になりました。 **WebAssembly (Wasm)** は最新の後継バージョンであり、より優れたパフォーマンス、より幅広いツール、より幅広い業界サポートを提供します。新しいプロジェクトは代わりに WebAssembly をターゲットにする必要があります。

### コード例```javascript
// asm.js module pattern (simplified)
function MyModule(stdlib, foreign, heap) {
  "use asm";

  var sqrt = stdlib.Math.sqrt;
  var HEAP32 = new stdlib.Int32Array(heap);

  function distance(x1, y1, x2, y2) {
    x1 = +x1; y1 = +y1; x2 = +x2; y2 = +y2;
    var dx = 0.0, dy = 0.0;
    dx = +(x2 - x1);
    dy = +(y2 - y1);
    return +sqrt(dx * dx + dy * dy);
  }

  return { distance: distance };
}
```---

## キャンバス API

### それは何ですか

Canvas API は、JavaScript および HTML `<canvas>` 要素を介して 2D グラフィックを描画する手段を提供します。これは、ブラウザベースのゲームの主要なレンダリング サーフェスの 1 つであり、ゲーム グラフィックス、アニメーション、画像操作、およびリアルタイム ビデオ処理をサポートします。

### ゲームにとってそれが重要な理由

- **2D レンダリング サーフェス**: ブラウザ ゲームでスプライト、タイルマップ、パーティクル、および HUD 要素を描画する標準的な方法。
- **ピクセル レベルの制御**: カスタム エフェクト、コリジョン マップ、プロシージャル生成のために、`ImageData` を介してピクセル データに直接アクセスします。
- **高性能**: 最新のブラウザではハードウェア アクセラレーションが行われ、60 fps のゲーム ループに適しています。
- **幅広いエコシステム**: Phaser、Konva.js、EaselJS、p5.js などのライブラリは、ゲーム開発用の Canvas 上に構築されます。

### 主要なインターフェース

|インターフェース |目的 |
|----------|----------|
| `HTMLCanvasElement` | `<canvas>` HTML 要素 |
| `CanvasRenderingContext2D` |メインの 2D 描画インターフェイス |
| `ImageData` |直接操作のための生のピクセルデータ |
| `ImageBitmap` |効率的な描画のためのビットマップ画像データ |
| `Path2D` |再利用可能なパス オブジェクト |
| `OffscreenCanvas` |オフスクリーン レンダリング、Web Workers で使用可能 |
| `CanvasPattern` |画像パターンの繰り返し |
| `CanvasGradient` |カラーグラデーション |
| `TextMetrics` |テキスト測定データ |

### 主要なメソッド (CanvasRenderingContext2D)

- `fillRect()`、`strokeRect()`、`clearRect()` -- 四角形の演算
- `drawImage()` -- 画像、スプライト、またはその他のキャンバスを描画します
- `beginPath()`、`arc()`、`lineTo()`、`fill()`、`stroke()` -- パスの描画
- `getImageData()`、`putImageData()` -- ピクセル操作
- `save()`、`restore()` -- 状態管理
- `translate()`、`rotate()`、`scale()`、`transform()` -- 変換

### コード例```html
<canvas id="game" width="800" height="600"></canvas>
```

```javascript
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// Clear the frame
ctx.clearRect(0, 0, canvas.width, canvas.height);

// Draw a filled rectangle (e.g., a platform)
ctx.fillStyle = "green";
ctx.fillRect(100, 400, 200, 20);

// Draw a sprite
const sprite = new Image();
sprite.src = "player.png";
sprite.onload = () => {
  ctx.drawImage(sprite, playerX, playerY, 32, 32);
};

// Game loop
function gameLoop(timestamp) {
  update(timestamp);
  render(ctx);
  requestAnimationFrame(gameLoop);
}
requestAnimationFrame(gameLoop);
```---

## CSS (カスケード スタイル シート)

### それは何ですか

CSS は、Web ドキュメントのプレゼンテーションを記述するために使用される言語です。ゲーム開発のコンテキストでは、CSS は、ゲーム キャンバスの上または横にある UI オーバーレイ、HUD 要素、メニュー、トランジション、アニメーション、視覚効果のスタイルを処理します。

### ゲームにとってそれが重要な理由

- **UI と HUD のスタイル**: ゲーム キャンバスに触れることなく、ヘルス バー、スコア表示、インベントリ パネル、ダイアログ ボックス、メニューのスタイルを設定します。
- **CSS アニメーションとトランジション**: 最小限の JavaScript を使用した、UI 要素 (フェードイン、スライドアウト、パルス効果) のハードウェア アクセラレーションによるアニメーション。
- **CSS 変換**: 視覚効果と UI の配置のために DOM 要素を移動、回転、拡大縮小、および傾斜させます。
- **フレックスボックスとグリッド**: 複雑なゲーム UI (設定パネル、リーダーボード、ロビー画面) をレスポンシブにレイアウトします。
- **カスタム プロパティ (CSS 変数)**: 実行時に変数値を変更することで、テーマ ゲーム UI を動的に変更します。
- **ポインターとカーソルの制御**: カーソルをカスタマイズまたは非表示にし、オーバーレイ要素上のポインター イベントを制御します。
- **メディア クエリ**: 画面サイズやデバイスの種類に応じてゲーム UI を適応させます。

### ゲームの主要なプロパティ

|特性・特長 |使用例 |
|---------------------|----------|
| `transform` | UI 要素の回転、拡大縮小、移動 |
| `transition` |スムーズなプロパティ変更 (ヘルスバーの幅など) |
| `animation` / `@keyframes` |ループまたはトリガーされた UI アニメーション |
| `opacity` |オーバーレイとモーダルのフェード効果 |
| `pointer-events` |クリックがオーバーレイレイヤーを通過してキャンバスに到達するようにします。
| `cursor` |カスタム カーソルを設定するか、カーソルを非表示にします (`cursor: none`) |
| `z-index` |ゲーム キャンバスの上のレイヤー UI |
| `position: fixed / absolute` | HUD 要素をビューポートに固定する |
| `display: flex / grid` |メニューとパネルのレスポンシブ レイアウト |
| `filter` | DOM 要素に対するぼかし、明るさ、コントラストの効果 |
| `mix-blend-mode` |オーバーレイ効果をキャンバスとブレンドする |
| `will-change` |アニメーションプロパティを最適化するようにブラウザにヒントを与える |

### コード例```css
/* Game HUD overlay */
.hud {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  padding: 10px;
  pointer-events: none;       /* clicks pass through to canvas */
  z-index: 10;
  font-family: "Press Start 2P", monospace;
  color: white;
  text-shadow: 2px 2px 0 black;
}

/* Health bar with smooth transitions */
.health-bar {
  width: 200px;
  height: 20px;
  background: #333;
  border: 2px solid white;
}
.health-bar-fill {
  height: 100%;
  background: limegreen;
  transition: width 0.3s ease;
  will-change: width;
}

/* Pulsing damage indicator */
@keyframes damage-flash {
  0%, 100% { opacity: 0; }
  50% { opacity: 0.4; }
}
.damage-overlay {
  position: fixed;
  inset: 0;
  background: red;
  animation: damage-flash 0.3s ease;
  pointer-events: none;
}
```---

## フルスクリーン API

### それは何ですか

フルスクリーン API は、特定の要素 (およびその子孫) をフルスクリーン モードで表示し、ブラウザーのすべてのクロム要素と UI 要素を削除するメソッドを提供します。プログラムで全画面表示を開始および終了できるようにし、現在の全画面状態を報告します。

### ゲームにとってそれが重要な理由

- **没入型エクスペリエンス**: フルスクリーンによりブラウザーの邪魔がすべて取り除かれ、コンソールのようなゲーム体験が提供されます。
- **最大画面領域**: ディスプレイ全体がゲーム ビューポートに使用できます。
- **ゲームは主な使用例です**: MDN ドキュメントには、ターゲット アプリケーションとしてオンライン ゲームが明示的にリストされています。

### 主要なインターフェイスとメソッド

| API |説明 |
|-----|---------------|
| `Element.requestFullscreen()` |全画面モードに入ります。 `Promise` を返します。 |
| `Document.exitFullscreen()` |全画面モードを終了します。 `Promise` を返します。 |
| `Document.fullscreenElement` |現在全画面表示になっている要素、または `null`。 |
| `Document.fullscreenEnabled` |全画面表示が利用可能かどうかを示すブール値。 |
| `fullscreenchange` イベント |全画面状態が変化したときに発生します。 |
| `fullscreenerror` イベント |全画面表示への移行または全画面表示からの終了が失敗した場合に発生します。 |

### コード例```javascript
const gameContainer = document.getElementById("game-container");

// Enter fullscreen on button click
document.getElementById("fullscreenBtn").addEventListener("click", () => {
  if (document.fullscreenEnabled) {
    gameContainer.requestFullscreen().catch(err => {
      console.error("Fullscreen request failed:", err);
    });
  }
});

// Toggle fullscreen with a key press
document.addEventListener("keydown", (e) => {
  if (e.key === "F11") {
    e.preventDefault();
    if (!document.fullscreenElement) {
      gameContainer.requestFullscreen();
    } else {
      document.exitFullscreen();
    }
  }
});

// Respond to fullscreen changes (resize canvas, adjust UI)
document.addEventListener("fullscreenchange", () => {
  if (document.fullscreenElement) {
    resizeCanvasToFullscreen();
  } else {
    resizeCanvasToWindowed();
  }
});
```### 注記

- 全画面表示は、ユーザーのジェスチャ (クリック、キーの押下) に応答した場合にのみ要求できます。
- ユーザーはいつでも Esc キーまたは F11 を使用して終了できます。
- iframe に埋め込まれたゲームの場合、`allowfullscreen` 属性が必要です。
- UI で機能を提供する前に、`Document.fullscreenEnabled` を確認してください。

---

## ゲームパッド API

### それは何ですか

ゲームパッド API は、ゲームパッドおよびゲーム コントローラーからの入力を検出および読み取るための標準化されたインターフェイスを提供します。ボタンの押下、アナログ スティックの位置、コントローラーの接続イベントを公開し、ブラウザ ゲームでコンソール スタイルのコントロールを可能にします。

### ゲームにとってそれが重要な理由

- **コンソール品質の入力**: ブラウザ ゲームで Xbox、PlayStation、および汎用コントローラーをサポートします。
- **複数のコントローラー**: ローカル マルチプレイヤー用に複数のゲームパッドを同時に検出して処理します。
- **アナログ入力**: アナログ スティック軸と感圧トリガーを読み取り、微妙な制御を実現します。
- **触覚フィードバック**: `GamepadHapticActuator` による振動の実験的サポート。

### 主要なインターフェース

|インターフェース |説明 |
|----------|---------------|
| `Gamepad` |ボタン、軸、メタデータを含む接続されたコントローラーを表します。
| `GamepadButton` |単一のボタンを表します -- `pressed` (ブール値) および `value` (圧力 0..1) |
| `GamepadEvent` | `gamepadconnected` および `gamepaddisconnected` イベントのイベント オブジェクト |
| `GamepadHapticActuator` |触覚フィードバック用のハードウェア インターフェイス (実験的) |

### 主要なメソッドとイベント

| API |説明 |
|-----|---------------|
| `navigator.getGamepads()` |接続されているすべてのコントローラの `Gamepad` オブジェクトの配列を返します。
| `gamepadconnected` イベント |コントローラーが接続されているときに `window` で発生します。
| `gamepaddisconnected` イベント |コントローラーが切断されたときに `window` で発生します。

### コード例```javascript
// Detect controller connections
window.addEventListener("gamepadconnected", (e) => {
  console.log(`Gamepad connected: ${e.gamepad.id}`);
});

window.addEventListener("gamepaddisconnected", (e) => {
  console.log(`Gamepad disconnected: ${e.gamepad.id}`);
});

// Poll gamepad state each frame
function pollGamepads() {
  const gamepads = navigator.getGamepads();
  for (const gp of gamepads) {
    if (!gp) continue;

    // Read analog sticks (axes)
    const leftStickX = gp.axes[0]; // -1 (left) to 1 (right)
    const leftStickY = gp.axes[1]; // -1 (up) to 1 (down)

    // Read buttons
    if (gp.buttons[0].pressed) {
      // A button / Cross -- jump
      player.jump();
    }
    if (gp.buttons[7].value > 0.1) {
      // Right trigger -- accelerate (analog pressure)
      player.accelerate(gp.buttons[7].value);
    }
  }
  requestAnimationFrame(pollGamepads);
}
requestAnimationFrame(pollGamepads);
```---

## IndexedDB API

### それは何ですか

IndexedDB は、ブラウザに組み込まれた低レベルの非同期トランザクションのクライアント側データベースです。キー インデックス付きオブジェクト ストアを使用して大量の構造化データ (ファイルや BLOB を含む) を格納し、高パフォーマンス クエリのインデックスをサポートします。

### ゲームにとってそれが重要な理由

- **ゲーム状態の保存**: プレイヤーの進行状況、インベントリ、キャラクター統計、レベル完了をセッション間で保持します。
- **アセットをローカルにキャッシュ**: テクスチャ、オーディオ ファイル、レベル データ、およびその他のアセットを保存して、ネットワーク リクエストを削減し、オフライン プレイを可能にします。
- **大容量ストレージ**: `localStorage` (最大 5 MB) よりもはるかに多くのデータを処理します。
- **ノンブロッキング**: 非同期操作により、保存/ロード操作中にゲーム ループがスムーズに実行されます。
- **トランザクション**: アトミックな読み取り/書き込み操作により、保存中のデータ破損を防ぎます。

### 主要なインターフェース

|インターフェース |目的 |ゲームの使用例 |
|----------|-----------|----------|
| `indexedDB.open()` |データベースを開くか作成する |起動時にゲームデータベースを初期化する |
| `IDBDatabase` |データベース接続 |接続の有効期間を管理する |
| `IDBTransaction` |読み取り/書き込みのスコープとアクセス制御 |アトミックセーブゲーム操作 |
| `IDBObjectStore` |プライマリ データ コンテナ |プレーヤーのプロフィール、レベルデータ、設定を保存 |
| `IDBIndex` |二次検索キー |タイプ、希少性、またはその他のプロパティによるアイテムのクエリ |
| `IDBCursor` |レコードを反復処理する |ゲームデータのバッチ操作 |
| `IDBKeyRange` |クエリのキー範囲を定義する |範囲内のスコア、最近の保存スロットを取得する |
| `IDBRequest` |非同期操作ハンドル |すべてのデータベース操作のコールバックを管理する |

### コード例```javascript
// Open (or create) the game database
const request = indexedDB.open("MyGameDB", 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  // Create an object store for save data
  const saveStore = db.createObjectStore("saves", { keyPath: "slotId" });
  saveStore.createIndex("timestamp", "timestamp");
};

request.onsuccess = (event) => {
  const db = event.target.result;

  // Save game state
  function saveGame(slot, gameState) {
    const tx = db.transaction("saves", "readwrite");
    const store = tx.objectStore("saves");
    store.put({
      slotId: slot,
      timestamp: Date.now(),
      playerHealth: gameState.health,
      playerPosition: gameState.position,
      inventory: gameState.inventory,
    });
  }

  // Load game state
  function loadGame(slot) {
    return new Promise((resolve, reject) => {
      const tx = db.transaction("saves", "readonly");
      const store = tx.objectStore("saves");
      const req = store.get(slot);
      req.onsuccess = () => resolve(req.result);
      req.onerror = () => reject(req.error);
    });
  }
};
```---

## JavaScript

### それは何ですか

JavaScript は、第一級の関数を備えた軽量で動的に型付けされたプロトタイプベースのプログラミング言語です。これは Web のスクリプト言語であり、すべてのブラウザベースのゲーム ロジックの基礎となる言語であり、命令型、関数型、オブジェクト指向のパラダイムをサポートします。

### ゲームにとってそれが重要な理由

- **実行環境**: JavaScript は、ブラウザ ゲームのロジックを実行する言語です。
- **イベント駆動型アーキテクチャ**: ネイティブ イベント処理は、入力、タイマー、および非同期リソースの読み込みをサポートします。
- **ファーストクラス関数**: コールバックとクロージャにより、ゲーム ループ、イベント ハンドラー、ビヘイビア ツリー、ステート マシンなどのパターンが可能になります。
- **動的オブジェクト**: ランタイム オブジェクトの作成と変更は、エンティティ コンポーネント システムとデータ駆動型設計をサポートします。
- **最新のクラス構文**: ES6+ クラスは、ゲーム エンティティにクリーンな継承階層を提供します。
- **非同期/待機**: アセットのロード、サーバー通信、シーン遷移のためのクリーンな非同期制御フロー。
- **ガベージ コレクション**: 自動メモリ管理 (ただし、スムーズなフレーム レートには GC 一時停止を意識することが重要です)。

### ゲームの主な言語機能

|特集 |ゲームアプリケーション |
|-------|-------|
|クラスと継承 |エンティティ階層 (GameObject、Player、Enemy) |
|クロージャ |コールバックとイベント ハンドラーのカプセル化された状態 |
| `requestAnimationFrame` |コア ゲーム ループ ドライバー |
| Promise / async-await |アセットのロード、サーバー呼び出し、シーン遷移 |
|破壊と拡散 |クリーンな構成と状態の受け渡し |
| `Map` と `Set` |エンティティ ルックアップ テーブル、一意の ID 追跡、衝突セット |
|テンプレートリテラル |デバッグ出力、動的テキストレンダリング |
|モジュール (インポート/エクスポート) |ゲームコードをシステムとコンポーネントに整理する |

### コード例```javascript
// ES6+ game entity pattern
class GameObject {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.active = true;
  }
  update(dt) { /* override in subclasses */ }
  render(ctx) { /* override in subclasses */ }
}

class Player extends GameObject {
  constructor(x, y) {
    super(x, y);
    this.health = 100;
    this.speed = 200;
  }
  update(dt) {
    if (input.left) this.x -= this.speed * dt;
    if (input.right) this.x += this.speed * dt;
  }
  render(ctx) {
    ctx.fillStyle = "blue";
    ctx.fillRect(this.x, this.y, 32, 32);
  }
}

// Game loop using requestAnimationFrame
let lastTime = 0;
function gameLoop(timestamp) {
  const dt = (timestamp - lastTime) / 1000;
  lastTime = timestamp;

  for (const entity of entities) {
    entity.update(dt);
  }
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  for (const entity of entities) {
    entity.render(ctx);
  }
  requestAnimationFrame(gameLoop);
}
requestAnimationFrame(gameLoop);
```---

## ポインターロック API

### それは何ですか

Pointer Lock API (以前の Mouse Lock API) は、絶対的なカーソル位置ではなく、生のマウスの動きのデルタへのアクセスを提供します。マウス イベントを単一の要素にロックし、カーソル移動の境界を削除し、カーソルを非表示にします。これは、一人称カメラ コントロールや同様の仕組みに不可欠です。

### ゲームにとってそれが重要な理由

- **一人称カメラ制御**: 画面端の制限なくマウスを物理的に動かすことでカメラを移動します。
- **カーソルが邪魔にならない**: カーソルが非表示になり、没入感が生まれます。
- **永続的ロック**: 一度実行されると、マウス ボタンの状態に関係なく、移動データが継続的に流れます。
- **Raw 入力オプション**: `unadjustedMovement` フラグは、対戦ゲームで一貫した狙いを定めるために OS レベルのマウス アクセラレーションを無効にします。
- **マウス ボタンを解放**: 動きをデルタのみで処理することで、クリックをゲーム アクション (射撃、対話) にマッピングできます。

### 主要なインターフェイスとメソッド

| API |説明 |
|-----|---------------|
| `element.requestPointerLock(options?)` |要素へのポインターをロックします。 `Promise` を返します。 |
| `document.exitPointerLock()` |ポインタのロックを解除します。 |
| `document.pointerLockElement` |現在ロックを保持している要素、または `null`。 |
| `MouseEvent.movementX` |最後の `mousemove` イベント以降の水平方向のデルタ。 |
| `MouseEvent.movementY` |最後の `mousemove` イベント以降の垂直デルタ。 |
| `pointerlockchange` イベント |ロック状態が変化したときに発生します。 |
| `pointerlockerror` イベント |ロックまたはロック解除が失敗した場合に発生します。 |

### コード例```javascript
const canvas = document.getElementById("game");

// Request pointer lock on click (user gesture required)
canvas.addEventListener("click", async () => {
  if (!document.pointerLockElement) {
    await canvas.requestPointerLock({
      unadjustedMovement: true, // raw input, no OS acceleration
    });
  }
});

// Respond to lock state changes
document.addEventListener("pointerlockchange", () => {
  if (document.pointerLockElement === canvas) {
    document.addEventListener("mousemove", handleMouseMove);
  } else {
    document.removeEventListener("mousemove", handleMouseMove);
  }
});

// Use movement deltas for camera rotation
const sensitivity = 0.002;
function handleMouseMove(e) {
  camera.yaw   += e.movementX * sensitivity;
  camera.pitch  += e.movementY * sensitivity;
  camera.pitch   = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, camera.pitch));
}
```### 注記

- ポインタのロックは、ユーザーのジェスチャ (クリック、キーの押下) に応答した場合にのみ要求できます。
- ユーザーは、Esc キーを使用していつでも終了できます。
- サンドボックス化された iframe には `allow-pointer-lock` 属性が必要です。

---

## SVG (スケーラブル ベクター グラフィックス)

### それは何ですか

SVG は、2 次元ベクトル グラフィックスを記述するための XML ベースのマークアップ言語です。ラスター形式 (PNG、JPEG) とは異なり、SVG 画像は品質を損なうことなく任意の解像度に拡大縮小できます。 SVG は CSS、DOM、JavaScript と統合されており、要素をスクリプト化可能でインタラクティブにします。

### ゲームにとってそれが重要な理由

- **解像度に依存しない**: 単一の SVG アセットは、どのような画面サイズやピクセル密度でも鮮明に表示されます。応答性の高いゲーム UI に最適です。
- **軽量**: テキストベースで圧縮可能であるため、UI アートとアイコンのダウンロード サイズが削減されます。
- **DOM 経由でスクリプト可能**: SVG 要素は、JavaScript を使用してリアルタイムで作成、変更、アニメーション化できます。
- **CSS スタイル**: SVG シェイプは、塗りつぶし、ストローク、不透明度、変換、フィルター、アニメーションの CSS ルールを受け入れます。
- **組み込みアニメーション**: 宣言モーション用の SMIL アニメーション要素 (`<animate>`、`<animateTransform>`、`<animateMotion>`)。
- **フィルターとエフェクト**: SVG フィルター プリミティブによるガウスぼかし、ドロップ シャドウ、カラー マトリックス、およびブレンド モード。

### 重要な要素

|要素 |ゲームの使用例 |
|-------|---------------|
| `<rect>` |ヘルスバー、UI パネル、プラットフォーム |
| `<circle>`、`<ellipse>` |ターゲット、粒子、インジケーター |
| `<path>` |複雑なベクター アート、カスタム形状 | 写真
| `<polygon>`、`<polyline>` |グリッド オーバーレイ、ワイヤーフレーム要素 |
| `<g>` |集団変換のためのグループ要素 |
| `<defs>`、`<use>`、`<symbol>` |再利用可能なスプライト定義 |
| `<text>`、`<tspan>` |スコア表示、ラベル、ダイアログ |
| `<filter>` |ぼかし、影、色の効果 |
| `<clipPath>`、`<mask>` |ビューポートのクリッピング、エフェクトを明らかにする |
| `<linearGradient>`、`<radialGradient>` |シェーディングと深度効果 |
| `<animate>`、`<animateTransform>` |宣言型 UI アニメーション |

### コード例```html
<!-- A simple SVG health bar -->
<svg width="220" height="30" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="healthGrad" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="limegreen" />
      <stop offset="100%" stop-color="green" />
    </linearGradient>
  </defs>
  <!-- Background -->
  <rect x="1" y="1" width="218" height="28" rx="5" fill="#333" stroke="#fff" stroke-width="1" />
  <!-- Health fill (width controlled via JS) -->
  <rect id="health-fill" x="3" y="3" width="160" height="24" rx="4" fill="url(#healthGrad)">
    <animate attributeName="width" from="214" to="60" dur="3s" fill="freeze" />
  </rect>
</svg>
```

```javascript
// Update health bar programmatically
function setHealth(percent) {
  const maxWidth = 214;
  document.getElementById("health-fill")
    .setAttribute("width", maxWidth * (percent / 100));
}
```---

## 型付き配列

### それらは何ですか

型付き配列は、生のバイナリ データ バッファー (`ArrayBuffer`) に対する配列のようなビューです。通常の JavaScript 配列とは異なり、各型付き配列には要素の型とサイズが固定されているため、予測可能なメモリ レイアウトと効率的なデータ アクセスが実現します。単一の `TypedArray` コンストラクターはありません。代わりに、`Float32Array`、`Uint8Array`、`Uint16Array` などの特定のコンストラクターが使用されます。

### それらがゲームにとって重要な理由

- **WebGL 頂点バッファおよびインデックス バッファ**: WebGL メソッドは、位置、法線、テクスチャ座標、色、インデックスの型付き配列を直接受け入れます。
- **Web オーディオ バッファ**: オーディオ サンプル データは `Float32Array` として保存および操作されます。
- **バイナリ アセットの読み込み**: バイナリ ファイル形式 (モデル、テクスチャ、レベル データ) を直接解析します。
- **メモリ効率が高い**: ボックス化オーバーヘッドのない固定サイズの要素。
- **WebAssembly 相互運用機能**: `SharedArrayBuffer` および型付き配列ビューを介して JavaScript モジュールと Wasm モジュール間でメモリを共有します。
- **ネットワークシリアル化**: マルチプレイヤー送信用にゲーム状態を効率的にパックします。

### キーの種類

|タイプ |バイト |範囲 |ゲームの使用例 |
|------|-------|------|------|
| `Float32Array` | 4 | ~3.4e38 |頂点の位置、法線、UV、物理値 |
| `Float64Array` | 8 | ~1.8e308 |高精度計算・シミュレーション |
| `Uint8Array` | 1 | 0 ～ 255 |テクスチャ/ピクセル データ、カラー チャネル |
| `Uint8ClampedArray` | 1 | 0 -- 255 (クランプ) | `ImageData` ピクセル操作 |
| `Uint16Array` | 2 | 0 ～ 65535 |インデックスバッファ (小さなメッシュ) |
| `Uint32Array` | 4 | 0 -- ~43 億 |インデックス バッファー (大きなメッシュ)、ID |
| `Int16Array` | 2 | -32768 -- 32767 |オーディオ サンプル、量子化法線 |
| `Int32Array` | 4 | ～-21 億 -- ～21 億 |整数ゲームデータ |

### 主要なプロパティとメソッド```javascript
const verts = new Float32Array([0, 0, 0,  1, 0, 0,  0, 1, 0]);

verts.buffer;             // The underlying ArrayBuffer
verts.byteLength;         // Total size in bytes
verts.byteOffset;         // Byte offset into the buffer
verts.length;             // Number of elements
verts.BYTES_PER_ELEMENT;  // 4 for Float32Array

// Write data
verts.set([1, 2, 3], 0);            // Copy values at offset
verts.copyWithin(6, 0, 3);          // Duplicate first vertex to third slot

// Read sub-views (no copy)
const firstTriangle = verts.subarray(0, 9);

// Functional methods
const scaled = verts.map(v => v * 2);
const max = verts.reduce((a, v) => Math.max(a, v), -Infinity);
```### コード例```javascript
// Build a quad for WebGL rendering
const positions = new Float32Array([
  -0.5, -0.5, 0,   // bottom-left
   0.5, -0.5, 0,   // bottom-right
   0.5,  0.5, 0,   // top-right
  -0.5,  0.5, 0,   // top-left
]);

const indices = new Uint16Array([
  0, 1, 2,  // first triangle
  0, 2, 3,  // second triangle
]);

// Upload to WebGL
const posBuf = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, posBuf);
gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);

const idxBuf = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, idxBuf);
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, indices, gl.STATIC_DRAW);

// Generate a 440 Hz sine wave for Web Audio
const sampleRate = 44100;
const audioBuffer = new Float32Array(sampleRate); // 1 second
for (let i = 0; i < sampleRate; i++) {
  audioBuffer[i] = Math.sin(2 * Math.PI * 440 * i / sampleRate);
}
```---

## ウェブオーディオ API

### それは何ですか

Web Audio API は、Web 上のオーディオを制御するための高レベルのシステムです。これは、オーディオ ソースがエフェクト ノードを介して宛先 (スピーカー) に接続されるモジュラー ルーティング グラフを使用します。高精度のタイミング、低遅延、およびソース リスナー モデルに基づく組み込み 3D 空間オーディオを提供します。

### ゲームにとってそれが重要な理由

- **低遅延再生**: サウンドエフェクトは最小限の遅延でゲーム イベントに応答します。
- **3D 空間オーディオ**: 方向性および距離ベースのオーディオの場合、プレーヤー/リスナーを基準にして 3D 空間にサウンドを配置します。
- **モジュラー エフェクト パイプライン**: ゲイン、リバーブ、フィルター、圧縮、ディストーション ノードをチェーンしてダイナミックなサウンドスケープを実現します。
- **正確なスケジューリング**: リズム ゲーム、シーケンス音楽、時間指定イベントのサンプル精度に合わせてサウンドをスケジュールします。
- **リアルタイム分析**: `AnalyserNode` は、オーディオに反応するビジュアルの周波数および波形データを提供します。
- **プロシージャル オーディオ**: `OscillatorNode` は、合成された効果音と UI トーンの波形を生成します。

### 主要なインターフェース

|インターフェース |目的 |
|----------|----------|
| `AudioContext` |メインのオーディオ処理グラフ。最初に作成する必要があります |
| `AudioBufferSourceNode` | `AudioBuffer` からプリロードされたオーディオ (SFX、音楽) を再生します。
| `OscillatorNode` |波形 (正弦波、方形波、三角波、ノコギリ波) を生成します。
| `GainNode` |ボリューム/振幅をコントロール |
| `BiquadFilterNode` |ローパス、ハイパス、バンドパス フィルター |
| `ConvolverNode` |インパルス応答を使用したコンボリューションリバーブ |
| `DelayNode` |ディレイラインエフェクト（エコー、コーラス） |
| `DynamicsCompressorNode` |多くの音をミックスする際のクリッピングを防止 |
| `PannerNode` | 3D 空間に音源を配置 |
| `AudioListener` |プレイヤーの耳を 3D 空間で表現 |
| `StereoPannerNode` |シンプルな左/右パン |
| `AnalyserNode` |リアルタイムの周波数および時間領域解析 |
| `AudioWorkletNode` |メインスレッドからのカスタムオーディオ処理 |

### 一般的なルーティング パターン

|使用例 |ルーティンググラフ |
|----------|--------------|
|バックグラウンドミュージック | `BufferSource` -> `GainNode` -> `Destination` |
|ポジショナルSFX | `BufferSource` -> `PannerNode` -> `GainNode` -> `Destination` |
|リバーブ環境 | `BufferSource` -> `ConvolverNode` -> `GainNode` -> `Destination` |
| UI フィードバック トーン | `OscillatorNode` -> `GainNode` -> `Destination` |
|マスターミックス |複数のソース -> 個別の `GainNode` -> `DynamicsCompressorNode` -> `Destination` |

### コード例```javascript
const audioCtx = new AudioContext();

// Load and play a sound effect
async function playSFX(url) {
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  const audioBuffer = await audioCtx.decodeAudioData(arrayBuffer);

  const source = audioCtx.createBufferSource();
  source.buffer = audioBuffer;

  // Add gain control
  const gainNode = audioCtx.createGain();
  gainNode.gain.value = 0.8;

  // Connect: source -> gain -> speakers
  source.connect(gainNode);
  gainNode.connect(audioCtx.destination);

  source.start(0);
}

// 3D positional audio
function playPositionalSound(buffer, x, y, z) {
  const source = audioCtx.createBufferSource();
  source.buffer = buffer;

  const panner = audioCtx.createPanner();
  panner.panningModel = "HRTF";
  panner.distanceModel = "inverse";
  panner.refDistance = 1;
  panner.maxDistance = 100;
  panner.positionX.value = x;
  panner.positionY.value = y;
  panner.positionZ.value = z;

  source.connect(panner);
  panner.connect(audioCtx.destination);
  source.start(0);
}

// Update listener position each frame (matches camera/player)
function updateListener(playerPos, playerForward, playerUp) {
  const listener = audioCtx.listener;
  listener.positionX.value = playerPos.x;
  listener.positionY.value = playerPos.y;
  listener.positionZ.value = playerPos.z;
  listener.forwardX.value = playerForward.x;
  listener.forwardY.value = playerForward.y;
  listener.forwardZ.value = playerForward.z;
  listener.upX.value = playerUp.x;
  listener.upY.value = playerUp.y;
  listener.upZ.value = playerUp.z;
}
```---

## WebGL API

### それは何ですか

WebGL (Web Graphics Library) は、ブラウザ内でハードウェア アクセラレーションによる 2D および 3D グラフィックスをレンダリングするための JavaScript API です。 OpenGL ES 2.0 (WebGL 1) および OpenGL ES 3.0 (WebGL 2) に準拠したプロファイルを実装し、HTML `<canvas>` 要素を通じて動作し、レンダリングにデバイス GPU を使用します。

### ゲームにとってそれが重要な理由

- **GPU アクセラレーションによるレンダリング**: デバイスの GPU を使用した高フレーム レートでのリアルタイム 3D グラフィックス。
- **シェーダー プログラミング**: GLSL の頂点シェーダーとフラグメント シェーダーにより、カスタムの視覚効果、照明、影、後処理が可能になります。
- **3D および 2D**: フル 3D ゲームと高性能 2D レンダリングの両方に適しています。
- **プラグインなし**: すべての最新ブラウザでネイティブに実行されます。
- **豊富なエコシステム**:three.js、Babylon.js、PlayCanvas、Pixi.js などのライブラリにより、WebGL ゲーム開発が簡素化されます。

### 主要なインターフェース

|インターフェース |目的 |
|----------|----------|
| `WebGLRenderingContext` | WebGL 1 レンダリング コンテキスト (OpenGL ES 2.0) |
| `WebGL2RenderingContext` | WebGL 2 レンダリング コンテキスト (OpenGL ES 3.0) |
| `WebGLProgram` |リンクバーテックス + フラグメントシェーダープログラム |
| `WebGLShader` |個々の頂点またはフラグメント シェーダ |
| `WebGLBuffer` | GPU メモリ バッファー (頂点、インデックス) |
| `WebGLTexture` |サーフェスのテクスチャ データ |
| `WebGLFramebuffer` |オフスクリーン レンダー ターゲット (シャドウ マップ、後処理) |
| `WebGLRenderbuffer` |非テクスチャ レンダー バッファー (深度、ステンシル) |
| `WebGLVertexArrayObject` |キャッシュされた頂点属性の設定 (WebGL 2) |
| `WebGLUniformLocation` |シェーダユニフォーム変数への参照 |
| `WebGLSampler` |テクスチャ サンプリング パラメータ (WebGL 2) |
| `WebGLTransformFeedback` | GPU 間のデータ ストリーミング (WebGL 2) |

### ゲームにとって重要な WebGL 2 機能

- **3D テクスチャ**: ボリューム レンダリング、ルックアップ テーブル。
- **インスタンス レンダリング**: `drawArraysInstanced()` / `drawElementsInstanced()` 数千の同一オブジェクトを効率的に描画します。
- **複数のレンダー ターゲット**: `drawBuffers()` 遅延レンダリング パイプライン用。
- **均一バッファ オブジェクト**: 描画呼び出し全体でシェーダ データを効率的に共有します。
- **変換フィードバック**: GPU 駆動のパーティクル システムおよびシミュレーションの頂点シェーダー出力をキャプチャします。
- **頂点配列オブジェクト**: 頂点状態をキャッシュして、描画ごとのセットアップのオーバーヘッドを削減します。

### コンテキスト管理イベント

|イベント |説明 |
|------|-----------|
| `webglcontextlost` | GPU コンテキストが失われました (デバイスの切断、リソース制限)。ゲームはこれを適切に処理する必要があります。 |
| `webglcontextrestored` | GPU コンテキストが回復されました。ゲームは GPU リソースをリロードする必要があります。 |
| `webglcontextcreationerror` |コンテキストの初期化に失敗しました。 |

### コード例```javascript
const canvas = document.getElementById("game");
const gl = canvas.getContext("webgl2");

// Vertex shader
const vsSource = `#version 300 es
  in vec4 aPosition;
  uniform mat4 uModelViewProjection;
  void main() {
    gl_Position = uModelViewProjection * aPosition;
  }
`;

// Fragment shader
const fsSource = `#version 300 es
  precision mediump float;
  out vec4 fragColor;
  void main() {
    fragColor = vec4(1.0, 0.5, 0.2, 1.0);
  }
`;

function compileShader(gl, source, type) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    console.error(gl.getShaderInfoLog(shader));
    gl.deleteShader(shader);
    return null;
  }
  return shader;
}

const vs = compileShader(gl, vsSource, gl.VERTEX_SHADER);
const fs = compileShader(gl, fsSource, gl.FRAGMENT_SHADER);

const program = gl.createProgram();
gl.attachShader(program, vs);
gl.attachShader(program, fs);
gl.linkProgram(program);
gl.useProgram(program);

// Upload vertex data
const positions = new Float32Array([0, 0.5, 0, -0.5, -0.5, 0, 0.5, -0.5, 0]);
const buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);

const aPos = gl.getAttribLocation(program, "aPosition");
gl.enableVertexAttribArray(aPos);
gl.vertexAttribPointer(aPos, 3, gl.FLOAT, false, 0, 0);

// Render
gl.clearColor(0, 0, 0, 1);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 3);
```### 推奨ライブラリ

|図書館 |説明 |
|----------|---------------|
| 3.js |フル機能の 3D エンジン |
|バビロン.js |物理学、オーディオ、ネットワークを備えた完全なゲーム エンジン |
|プレイキャンバス |クラウドベースのゲームエンジン |
|ピクシー.js |軽量 2D レンダラー |
|グルマトリックス |行列とベクトルの数学ライブラリ |

---

## WebRTC API

### それは何ですか

WebRTC (Web Real-Time Communication) は、プラグインや中継サーバーを必要とせずに、オーディオ、ビデオ、および任意のデータ交換のためのブラウザ間のピアツーピア通信を可能にします (ただし、シグナリング サーバーと STUN/TURN は接続セットアップと NAT トラバーサルに使用されます)。

### ゲームにとってそれが重要な理由

- **ピアツーピア マルチプレイヤー**: プレイヤー間の直接接続を確立し、遅延を削減し、小規模ゲームの専用ゲーム サーバーを排除します。
- **低遅延データ チャネル**: `RTCDataChannel` は、最小限のオーバーヘッドでバイナリのゲーム ステート更新を送信し、信頼できる配信モードと信頼できない配信モードの両方をサポートします。
- **ボイスチャット**: 内蔵のオーディオ/ビデオストリーミングにより、ゲーム内での音声コミュニケーションが可能になります。
- **サーバー コストの削減**: 直接ピア接続により、帯域幅と処理が集中サーバーからオフロードされます。

### 主要なインターフェース

|インターフェース |目的 |
|----------|----------|
| `RTCPeerConnection` |メディア ストリームやデータ チャネルなど、2 つのピア間の接続を管理します。
| `RTCDataChannel` |任意のデータ (ゲーム状態、コマンド、チャット) の双方向チャネル |
| `RTCSessionDescription` | SDP によるセッション ネゴシエーション (オファー/アンサー モデル) |
| `RTCIceCandidate` | NAT/ファイアウォールトラバーサルの接続候補 |
| `RTCRtpSender` / `RTCRtpReceiver` |オーディオ/ビデオのエンコードと送信を管理する |
| `RTCStatsReport` |最適化のための接続統計 (遅延、パケット損失、帯域幅) |

### 主要なイベント

|イベント |説明 |
|------|-----------|
| `datachannel` |リモート ピアがデータ チャネルをオープンしました |
| `connectionstatechange` |ピア接続状態が変更されました |
| `icecandidate` |新しい ICE 候補が利用可能 |
| `track` |受信メディア トラック (オーディオ/ビデオ) |

### 接続ライフサイクル

1. 各ピアに `RTCPeerConnection` を作成します。
2. シグナリング サーバー (通常は WebSocket) を介して SDP オファー/アンサーを交換します。
3. NAT トラバーサル用の ICE 候補を交換します。
4. ピアは直接接続します。
5. ゲーム データの `RTCDataChannel` を開くか、音声のメディア トラックを追加します。
6. `RTCStatsReport` を使用してパフォーマンスを監視します。
7. セッションが終了したら、チャネルと接続を閉じます。

### コード例```javascript
// Peer A: Create connection and data channel
const peerA = new RTCPeerConnection({
  iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
});

const gameChannel = peerA.createDataChannel("game", {
  ordered: false,       // Allow out-of-order delivery (lower latency)
  maxRetransmits: 0,    // Unreliable mode (like UDP)
});

gameChannel.onopen = () => {
  // Send game state updates
  gameChannel.send(JSON.stringify({ type: "move", x: 10, y: 20 }));
};

gameChannel.onmessage = (event) => {
  const data = JSON.parse(event.data);
  applyRemoteGameState(data);
};

// Peer B: Receive the data channel
const peerB = new RTCPeerConnection({
  iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
});

peerB.ondatachannel = (event) => {
  const channel = event.channel;
  channel.onmessage = (e) => {
    const data = JSON.parse(e.data);
    applyRemoteGameState(data);
  };
};

// Signaling (offer/answer exchange via your signaling server)
async function connect() {
  const offer = await peerA.createOffer();
  await peerA.setLocalDescription(offer);
  // Send offer to Peer B via signaling server...

  // Peer B receives offer, sets remote description, creates answer
  await peerB.setRemoteDescription(offer);
  const answer = await peerB.createAnswer();
  await peerB.setLocalDescription(answer);
  // Send answer back to Peer A via signaling server...

  await peerA.setRemoteDescription(answer);
}
```---

## WebSocket API

### それは何ですか

WebSocket API を使用すると、単一の TCP 接続を介したブラウザとサーバー間の永続的な全二重通信が可能になります。 HTTP リクエストとレスポンスとは異なり、WebSocket 接続は開いたままになるため、サーバーはいつでもクライアントにデータをプッシュできます。

### ゲームにとってそれが重要な理由

- **リアルタイム マルチプレイヤー**: クライアントとサーバー間でプレイヤーの位置、ゲーム イベント、世界の状態を最小限の遅延でストリーミングします。
- **サーバー プッシュ アップデート**: サーバーは、ポーリングせずに、接続されているすべてのプレイヤーにゲームの状態の変更を即座にブロードキャストできます。
- **低オーバーヘッド**: すべてのメッセージで HTTP ヘッダーが繰り返されません。永続的な接続上でデータをフレーム化しただけです。
- **バイナリ データ サポート**: ゲーム ステートを効率的にシリアル化するために `ArrayBuffer` および `Blob` データを送信します。
- **Web Worker 互換**: バックグラウンド スレッドで WebSocket 通信を実行し、ゲーム ループがブロックされないようにします。

### 主要なインターフェイス: WebSocket

|メンバー |説明 |
|----------|---------------|
| `new WebSocket(url, protocols?)` |サーバーへの接続を開きます |
| `send(data)` |送信データ (文字列、ArrayBuffer、Blob) |
| `close(code?, reason?)` |接続を正常に閉じます |
| `readyState` |現在の状態: 接続中 (0)、オープン (1)、クローズ中 (2)、クローズド (3) |
| `bufferedAmount` |キューに入れられているがまだ送信されていないバイト数 (フロー制御用) |
| `binaryType` |バイナリ データの場合は `"arraybuffer"` または `"blob"` に設定します。

### イベント

|イベント |説明 |
|------|-----------|
| `open` |接続が確立され準備完了 |
| `message` |サーバーから受信したデータ (`event.data` 経由でアクセス) |
| `close` |接続が閉じられました (`CloseEvent` によるアクセス コード/理由) |
| `error` |エラーが発生しました |

### コード例```javascript
// Connect to the game server
const socket = new WebSocket("wss://game.example.com/ws");
socket.binaryType = "arraybuffer";

socket.addEventListener("open", () => {
  // Authenticate and join a game room
  socket.send(JSON.stringify({
    type: "join",
    room: "room-42",
    playerId: "player-1"
  }));
});

socket.addEventListener("message", (event) => {
  if (typeof event.data === "string") {
    const msg = JSON.parse(event.data);
    switch (msg.type) {
      case "state":
        updateWorldState(msg.state);
        break;
      case "playerJoined":
        addRemotePlayer(msg.player);
        break;
      case "playerLeft":
        removeRemotePlayer(msg.playerId);
        break;
    }
  } else {
    // Binary data -- e.g., compressed game state
    const view = new DataView(event.data);
    processRawGameState(view);
  }
});

socket.addEventListener("close", (event) => {
  console.log(`Disconnected: ${event.code} ${event.reason}`);
  showReconnectPrompt();
});

// Send player input to the server each tick
function sendInput(input) {
  if (socket.readyState === WebSocket.OPEN) {
    socket.send(JSON.stringify({
      type: "input",
      keys: input.keys,
      mouseX: input.mouseX,
      mouseY: input.mouseY,
      timestamp: performance.now(),
    }));
  }
}
```### 注記

- ブラウザのバックフォワード キャッシュのブロックを避けるために、プレイヤーが移動するときに WebSocket 接続を閉じます。
- 信頼性の低い (UDP のような) 配信を必要とするゲームの場合は、WebRTC データ チャネルまたは新しい WebTransport API を検討してください。
- 一般的なサーバー ライブラリ: Socket.IO、ws (Node.js)、Gorilla WebSocket (Go)、SignalR (.NET)。

---

## WebVR API (非推奨)

### それは何ですか

WebVR API は、ブラウザから仮想現実デバイス (Oculus Rift や HTC Vive などのヘッドマウント ディスプレイ) にアクセスするためのインターフェイスを提供します。没入型 VR 体験のための表示プロパティ、ヘッドトラッキングポーズデータ、ステレオレンダリング機能を公開します。

### ゲームにとってそれが重要な理由

- **没入型 VR ゲーム**: リアルタイムのヘッド トラッキングによって立体的な 3D シーンをレンダリングします。
- **ルームスケールのエクスペリエンス**: `VRStageParameters` は、物理的なプレイエリアの寸法を示しています。
- **コントローラーの統合**: VR コントローラーはゲームパッド API を通じてアクセスでき、各コントローラーを `gamepad.displayId` 経由で `VRDisplay` にリンクします。

### 非推奨の通知

WebVR API は **非推奨であり、非標準**です。これは Web 標準として承認されることはなく、**WebXR Device API** に取って代わられました。この API は VR と AR の両方をサポートし、より広範なブラウザーをサポートし、標準化の方向に進んでいます。新しい VR ゲーム開発はすべて WebXR をターゲットにする必要があります。

### 主要なインターフェース

|インターフェース |目的 |
|----------|----------|
| `VRDisplay` | VR ヘッドセットを表します。コアメソッド: `requestPresent()`、`requestAnimationFrame()`、`getFrameData()`、`submitFrame()`。 |
| `VRFrameData` |現在のフレームのポーズ、表示行列、および投影行列。 |
| `VRPose` |特定のタイムスタンプにおける位置、方向、速度、加速度。 |
| `VREyeParameters` |目ごとの視野とレンダリング オフセット。 |
| `VRStageParameters` |ルームスケールのプレイエリアの寸法と変形。 |
| `VRDisplayCapabilities` |デバイス機能フラグ (位置追跡機能がある、外部ディスプレイがあるなど)。 |
| `Navigator.getVRDisplays()` |接続された `VRDisplay` オブジェクトの配列に解決される Promise を返します。 |

### 主要なイベント

|イベント |説明 |
|------|-----------|
| `vrdisplayconnect` | VR ヘッドセットが接続されました |
| `vrdisplaydisconnect` | VR ヘッドセットが切断されました |
| `vrdisplaypresentchange` |ヘッドセットがプレゼンテーション モードに入ったか、プレゼンテーション モードを終了しました。
| `vrdisplayactivate` |ヘッドセットを提示する準備ができました |

### コード例```javascript
// Check for WebVR support
if (navigator.getVRDisplays) {
  navigator.getVRDisplays().then(displays => {
    if (displays.length === 0) return;
    const vrDisplay = displays[0];

    // Start presenting to the headset
    vrDisplay.requestPresent([{ source: canvas }]).then(() => {
      const frameData = new VRFrameData();

      function renderLoop() {
        vrDisplay.requestAnimationFrame(renderLoop);
        vrDisplay.getFrameData(frameData);

        // Render left eye
        gl.viewport(0, 0, canvas.width / 2, canvas.height);
        renderScene(frameData.leftProjectionMatrix, frameData.leftViewMatrix);

        // Render right eye
        gl.viewport(canvas.width / 2, 0, canvas.width / 2, canvas.height);
        renderScene(frameData.rightProjectionMatrix, frameData.rightViewMatrix);

        vrDisplay.submitFrame();
      }
      renderLoop();
    });
  });
}
```### WebXR への移行

新しいプロジェクトの場合は、代わりに **WebXR Device API** を使用してください。 WebXR をサポートするフレームワークには次のものがあります。

- **A-Frame** -- 宣言型エンティティ コンポーネント VR フレームワーク
- **Babylon.js** -- WebXR をサポートするフル機能の 3D/ゲーム エンジン
- **three.js** -- WebXR 統合を備えた軽量 3D ライブラリ
- **WebXR Polyfill** -- 古いブラウザ用の下位互換性レイヤー

---

## Web ワーカー API

### それは何ですか

Web Workers API を使用すると、メイン スレッドとは別のバックグラウンド スレッドで JavaScript を実行できます。ワーカーは独自のグローバル スコープ (`DedicatedWorkerGlobalScope` または `SharedWorkerGlobalScope`) で動作し、DOM に直接アクセスできず、メッセージ パッシング (`postMessage` / `onmessage`) 経由でメイン スレッドと通信します。

### ゲームにとってそれが重要な理由

- **負荷の高い計算をオフロード**: 物理シミュレーション、経路探索、AI、手続き型生成、および衝突検出をバックグラウンド スレッドに移動し、メイン スレッドが 60 fps に留まるようにします。
- **並列アセット処理**: レンダリングをブロックすることなく、画像のデコード、データの解凍、またはレベル ファイルの解析を行います。
- **OffscreenCanvas**: ワーカー内からキャンバスにレンダリングし、並列レンダリング パイプラインを有効にします。
- **ノンブロッキング ネットワーク**: ゲーム ループをスムーズに保つために、ワーカーで `fetch()` または XHR 呼び出しを実行します。

### ワーカーのタイプ

|タイプ |説明 |ゲームの使用例 |
|------|---------------|------|
|専任ワーカー (`Worker`) |シングルオーナーのバックグラウンド スレッド | 1 つのゲーム インスタンスの物理学、AI、経路探索 |
|シェアワーカー (`SharedWorker`) |複数のウィンドウ/タブ間で共有 |マルチタブまたはマルチ iframe ゲームのシナリオ |
|サービスワーカー |オフラインをサポートするネットワーク プロキシ |アセットのキャッシュ、オフライン プレイ |

### 主要なインターフェース

| API |説明 |
|-----|---------------|
| `new Worker(scriptURL)` |スクリプト ファイルから専用のワーカーを作成する |
| `worker.postMessage(data)` |ワーカーにデータを送信 |
| `worker.onmessage` |ワーカーからデータを受信します (`event.data` 経由) |
| `worker.terminate()` |直ちに作業員を停止させてください。
|社内ワーカー: `self.postMessage(data)` |データをメインスレッドに送り返す |
|社内ワーカー: `self.onmessage` |メインスレッドからデータを受信する |

### 制限事項

- ワーカーからの DOM アクセスはありません。
- `window` オブジェクトはありません。世界的な範囲が限られています。
- データはデフォルトでコピーされます (構造化クローン)。ゼロコピー転送には `Transferable` オブジェクト (ArrayBuffer、OffscreenCanvas) を使用します。
- ワーカー スクリプトは同一生成元である必要があります。

### コード例

**メインスレッド (game.js):**```javascript
// Create a physics worker
const physicsWorker = new Worker("physics-worker.js");

// Send world state to the worker each frame
function updatePhysics(entities) {
  // Transfer the buffer for zero-copy performance
  const buffer = serializeEntities(entities);
  physicsWorker.postMessage({ type: "step", buffer }, [buffer]);
}

// Receive results from the worker
physicsWorker.onmessage = (event) => {
  const { type, buffer } = event.data;
  if (type === "result") {
    applyPhysicsResults(buffer);
  }
};
```**ワーカー スレッド (physics-worker.js):**```javascript
self.onmessage = (event) => {
  const { type, buffer } = event.data;
  if (type === "step") {
    const positions = new Float32Array(buffer);

    // Run physics simulation
    for (let i = 0; i < positions.length; i += 3) {
      positions[i + 1] -= 9.8 * (1 / 60); // gravity on Y axis
    }

    // Send results back, transferring the buffer
    self.postMessage({ type: "result", buffer: positions.buffer }, [positions.buffer]);
  }
};
```---

## XMLHttpRequest

### それは何ですか

`XMLHttpRequest` (XHR) は、ページをリロードせずにサーバーに HTTP リクエストを送信するための組み込みブラウザ API です。その名前に反して、JSON、バイナリ (ArrayBuffer、Blob)、プレーン テキスト、XML、HTML など、あらゆるデータ型を取得できます。新しいコードの Fetch API に大部分が置き換えられましたが、依然として広く使用されており、完全にサポートされています。

### ゲームにとってそれが重要な理由

- **アセットの読み込み**: ゲーム ループをブロックすることなく、ゲーム アセット (画像、オーディオ、JSON レベル データ、バイナリ モデル ファイル) を非同期的に取得します。
- **バイナリ データ サポート**: WebGL または Web Audio の型付き配列にバイナリ アセットを直接ロードするには、`responseType` を `"arraybuffer"` または `"blob"` に設定します。
- **進行状況の追跡**: `progress` イベントはダウンロードの進行状況を報告し、読み込みバーを有効にします。
- **サーバー通信**: スコアの送信、プレーヤーの認証、リーダーボードの取得、ゲームの状態とバックエンド サービスの同期を行います。
- **Web Worker との互換性**: XHR は Web Workers 内でバックグラウンドでのアセットの読み込みに使用できます。

### 主要なメソッド

|方法 |説明 |
|----------|---------------|
| `open(method, url, async?)` |リクエストの初期化 (GET、POST など) |
| `send(body?)` |リクエストを送信します。 `body` には、文字列、FormData、ArrayBuffer、Blob を指定できます。
| `setRequestHeader(name, value)` | HTTP ヘッダーを設定します (`open` の後、`send` の前に呼び出します)。
| `abort()` |進行中のリクエストをキャンセルする |
| `getResponseHeader(name)` |特定の応答ヘッダー値を取得する |

### 主要なプロパティ

|プロパティ |説明 |
|----------|---------------|
| `response` | `responseType` | で指定されたタイプの応答本文
| `responseType` |予期される応答形式: `""`、`"text"`、`"json"`、`"arraybuffer"`、`"blob"`、`"document"` |
| `status` | HTTP ステータス コード (200、404 など) |
| `readyState` |リクエストのライフサイクル状態 (0 = UNSENT ～ 4 = DONE) |
| `timeout` |リクエストが自動中止されるまでのミリ秒数 |
| `withCredentials` |クロスオリジンリクエストに Cookie を含めるかどうか |

### イベント

|イベント |説明 |
|------|-----------|
| `load` |リクエストは正常に完了しました |
| `error` |リクエストが失敗しました |
| `progress` |ダウンロード中の定期的な進行状況の更新 |
| `abort` |リクエストは中止されました |
| `readystatechange` | `readyState` が変更されました |

### コード例```javascript
// Load a JSON level file
function loadLevel(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.responseType = "json";

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(xhr.response);
      } else {
        reject(new Error(`Failed to load level: ${xhr.status}`));
      }
    };
    xhr.onerror = () => reject(new Error("Network error"));
    xhr.send();
  });
}

// Load a binary asset with progress tracking
function loadBinaryAsset(url, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.responseType = "arraybuffer";

    xhr.onprogress = (event) => {
      if (event.lengthComputable && onProgress) {
        onProgress(event.loaded / event.total);
      }
    };

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(xhr.response); // ArrayBuffer
      } else {
        reject(new Error(`Failed to load asset: ${xhr.status}`));
      }
    };
    xhr.onerror = () => reject(new Error("Network error"));
    xhr.send();
  });
}

// Usage
loadLevel("levels/level1.json").then(data => initLevel(data));
loadBinaryAsset("models/tank.bin", pct => updateLoadingBar(pct))
  .then(buf => parseModel(new Float32Array(buf)));
```### Fetch API に関する注意事項

新しいプロジェクトの場合は、通常、XHR より **Fetch API** (`fetch()`) が優先されます。これは、よりクリーンな Promise ベースのインターフェイスを提供し、`ReadableStream` によるストリーミングをサポートし、async/await と適切に統合します。ただし、アップロード時に進行状況イベントが必要な場合や、従来のコードベースとの幅広い互換性が必要な場合には、XHR が引き続き関連します。