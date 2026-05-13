# ゲームの操作メカニズム

このリファレンスでは、モバイルのタッチ操作、デスクトップのキーボードとマウス、ゲームパッドコントローラー、さらに非定型の入力方式を含む、Webベースのゲームで利用できる主要な操作メカニズムを解説します。

## モバイルのタッチ操作

モバイル端末向けのWebゲームでは、タッチ操作は不可欠です。モバイルファーストのアプローチにより、HTML5ゲームで最も広く使われているプラットフォームで、ゲームのアクセス性を確保できます。

### 主なイベントとAPI

ブラウザで利用できる基本的なタッチイベントは次のとおりです。

| Event | Description |
|-------|-------------|
| `touchstart` | ユーザーが画面に指を置いたときに発火 |
| `touchmove` | 画面に触れたまま指を動かしたときに発火 |
| `touchend` | ユーザーが画面から指を離したときに発火 |
| `touchcancel` | タッチがキャンセルまたは中断されたときに発火（例: 指が画面外に移動） |

**タッチイベントリスナーの登録:**

```javascript
const canvas = document.querySelector("canvas");
canvas.addEventListener("touchstart", handleStart);
canvas.addEventListener("touchmove", handleMove);
canvas.addEventListener("touchend", handleEnd);
canvas.addEventListener("touchcancel", handleCancel);
```

**タッチイベントのプロパティ:**

- `e.touches[0]` -- 最初のタッチポイントにアクセス（マルチタッチ時はゼロ始まりのインデックス）。
- `e.touches[0].pageX` / `e.touches[0].pageY` -- ページ基準のタッチ座標。
- canvas要素基準の位置を得るには、必ずcanvasのオフセットを差し引いてください。

### コード例

**Pure JavaScript のタッチハンドラー:**

```javascript
document.addEventListener("touchstart", touchHandler);
document.addEventListener("touchmove", touchHandler);

function touchHandler(e) {
  if (e.touches) {
    playerX = e.touches[0].pageX - canvas.offsetLeft - playerWidth / 2;
    playerY = e.touches[0].pageY - canvas.offsetTop - playerHeight / 2;
    e.preventDefault();
  }
}
```

**Phaser フレームワークのポインターシステム:**

Phaserは、各指を表す「ポインター」を通じてタッチ入力を管理します。

```javascript
// Access pointers
this.game.input.activePointer;       // Most recently active pointer
this.game.input.pointer1;            // First pointer
this.game.input.pointer2;            // Second pointer

// Add more pointers (up to 10 total)
this.game.input.addPointer();

// Global input events
this.game.input.onDown.add(itemTouched, this);
this.game.input.onUp.add(itemReleased, this);
this.game.input.onTap.add(itemTapped, this);
this.game.input.onHold.add(itemHeld, this);
```

**宇宙船移動用のドラッグ可能スプライト:**

```javascript
const player = this.game.add.sprite(30, 30, "ship");
player.inputEnabled = true;
player.input.enableDrag();
player.events.onDragStart.add(onDragStart, this);
player.events.onDragStop.add(onDragStop, this);

function onDragStart(sprite, pointer) {
  console.log(`Dragging at: ${pointer.x}, ${pointer.y}`);
}
```

**射撃用の不可視タッチ領域（画面右半分）:**

```javascript
this.buttonShoot = this.add.button(
  this.world.width * 0.5, 0,
  "button-alpha",    // transparent image
  null,
  this
);
this.buttonShoot.onInputDown.add(this.goShootPressed, this);
this.buttonShoot.onInputUp.add(this.goShootReleased, this);
```

**仮想ゲームパッドプラグイン:**

```javascript
this.gamepad = this.game.plugins.add(Phaser.Plugin.VirtualGamepad);
this.joystick = this.gamepad.addJoystick(100, 420, 1.2, "gamepad");
this.button = this.gamepad.addButton(400, 420, 1.0, "gamepad");
```

### ベストプラクティス

- 不要なスクロールやブラウザ既定動作を避けるため、タッチイベントでは常に `preventDefault()` を呼び出す。
- ゲーム画面を覆わないように、可視ボタンより不可視のボタン領域を使う。
- 画面上ボタンより直感的な、ドラッグのような自然なタッチジェスチャーを活用する。
- 位置計算時には、canvasのオフセットを差し引き、オブジェクトサイズも考慮する。
- タッチ可能領域は、快適に操作できる十分な大きさにする。
- マルチタッチ対応を計画する。Phaserは最大10本の同時ポインターに対応。
- デスクトップとモバイルの自動互換性のために、Phaserのようなフレームワークを使う。
- 高度なタッチ操作UIには、仮想ゲームパッド／ジョイスティックのプラグインを検討する。

## デスクトップ（マウスとキーボード）

デスクトップのキーボードとマウス操作は、Webゲームに高精度な入力を提供し、デスクトップブラウザにおける標準の操作方式です。

### 主なイベントとAPI

**キーボードイベント:**

```javascript
document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);
```

- `event.code` は `"ArrowLeft"`、`"ArrowRight"`、`"ArrowUp"`、`"ArrowDown"` などの読みやすいキー識別子を返します。
- 連続的なフレーム更新には `requestAnimationFrame()` を使います。

**Phaser のキーボードAPI:**

```javascript
this.cursors = this.input.keyboard.createCursorKeys();  // Arrow key objects
this.keyLeft = this.input.keyboard.addKey(Phaser.KeyCode.A);  // Custom key binding
// Check key state with .isDown property
// Listen for press events with .onDown.add()
```

**Phaser のマウスAPI:**

```javascript
this.game.input.mousePointer;                    // Mouse position and state
this.game.input.mousePointer.isDown;             // Is any mouse button pressed
this.game.input.mousePointer.x;                  // Mouse X coordinate
this.game.input.mousePointer.y;                  // Mouse Y coordinate
this.game.input.mousePointer.leftButton.isDown;  // Left mouse button
this.game.input.mousePointer.rightButton.isDown; // Right mouse button
this.game.input.activePointer;                   // Platform-independent (mouse + touch)
```

### コード例

**Pure JavaScript のキーボード状態追跡:**

```javascript
let rightPressed = false;
let leftPressed = false;
let upPressed = false;
let downPressed = false;

function keyDownHandler(event) {
  if (event.code === "ArrowRight") rightPressed = true;
  else if (event.code === "ArrowLeft") leftPressed = true;
  if (event.code === "ArrowDown") downPressed = true;
  else if (event.code === "ArrowUp") upPressed = true;
}

function keyUpHandler(event) {
  if (event.code === "ArrowRight") rightPressed = false;
  else if (event.code === "ArrowLeft") leftPressed = false;
  if (event.code === "ArrowDown") downPressed = false;
  else if (event.code === "ArrowUp") upPressed = false;
}
```

**入力処理付きゲームループ:**

```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  if (rightPressed) playerX += 5;
  else if (leftPressed) playerX -= 5;
  if (downPressed) playerY += 5;
  else if (upPressed) playerY -= 5;

  ctx.drawImage(img, playerX, playerY);
  requestAnimationFrame(draw);
}
```

**Phaser でのデュアル操作対応（矢印キー + WASD）:**

```javascript
this.cursors = this.input.keyboard.createCursorKeys();
this.keyLeft = this.input.keyboard.addKey(Phaser.KeyCode.A);
this.keyRight = this.input.keyboard.addKey(Phaser.KeyCode.D);
this.keyUp = this.input.keyboard.addKey(Phaser.KeyCode.W);
this.keyDown = this.input.keyboard.addKey(Phaser.KeyCode.S);

// In update:
if (this.cursors.left.isDown || this.keyLeft.isDown) {
  // move left
} else if (this.cursors.right.isDown || this.keyRight.isDown) {
  // move right
}
if (this.cursors.up.isDown || this.keyUp.isDown) {
  // move up
} else if (this.cursors.down.isDown || this.keyDown.isDown) {
  // move down
}
```

**複数の発射ボタン:**

```javascript
this.keyFire1 = this.input.keyboard.addKey(Phaser.KeyCode.X);
this.keyFire2 = this.input.keyboard.addKey(Phaser.KeyCode.SPACEBAR);

if (this.keyFire1.isDown || this.keyFire2.isDown) {
  // fire the weapon
}
```

**デバイス別の操作説明:**

```javascript
if (this.game.device.desktop) {
  moveText = "Arrow keys or WASD to move";
  shootText = "X or Space to shoot";
} else {
  moveText = "Tap and hold to move";
  shootText = "Tap to shoot";
}
```

### ベストプラクティス

- 複数の入力方式をサポートする: 移動は矢印キーとWASDの両方、攻撃は複数ボタン（例: X と Space）を提供する。
- マウスとタッチの両方をシームレスに扱うため、`mousePointer` ではなく `activePointer` を使う。
- デバイスタイプを検出し、プレイヤーに適切な操作説明を表示する。
- 滑らかなアニメーションのために `requestAnimationFrame()` を使い、個別キー押下に反応するよりゲームループでキー状態を確認する。
- キーボードショートカットで非ゲーム画面をスキップ可能にする（例: Enterで開始、任意キーでイントロスキップ）。
- ブラウザ差異やエッジケースを自動処理できるため、クロスブラウザ互換性にはPhaser等のフレームワークを使う。

## デスクトップ（ゲームパッド）

Gamepad APIにより、Webゲームはゲームパッド／コントローラー入力を検出して応答でき、ブラウザ上でコンソールライクな体験を実現できます。

### 主なイベントとAPI

**基本イベント:**

```javascript
window.addEventListener("gamepadconnected", gamepadHandler);
window.addEventListener("gamepaddisconnected", gamepadHandler);
```

**Gamepadオブジェクトのプロパティ:**

- `controller.id` -- デバイス識別文字列。
- `controller.buttons[]` -- ボタンオブジェクト配列。各要素は `.pressed` の真偽値プロパティを持つ。
- `controller.axes[]` -- -1〜1 のアナログスティック値配列。

**標準ボタン／軸マッピング（Xbox 360 レイアウト）:**

| Input | Index | Type |
|-------|-------|------|
| A Button | 0 | Button |
| B Button | 1 | Button |
| X Button | 2 | Button |
| Y Button | 3 | Button |
| D-Pad Up | 12 | Button |
| D-Pad Down | 13 | Button |
| D-Pad Left | 14 | Button |
| D-Pad Right | 15 | Button |
| Left Stick X | axes[0] | Axis |
| Left Stick Y | axes[1] | Axis |
| Right Stick X | axes[2] | Axis |
| Right Stick Y | axes[3] | Axis |

### コード例

**Pure JavaScript の接続ハンドラー:**

```javascript
let controller = {};
let buttonsPressed = [];

function gamepadHandler(e) {
  controller = e.gamepad;
  console.log(`Gamepad: ${controller.id}`);
}

window.addEventListener("gamepadconnected", gamepadHandler);
```

**各フレームでボタン状態をポーリング:**

```javascript
function gamepadUpdateHandler() {
  buttonsPressed = [];
  if (controller.buttons) {
    for (const [i, button] of controller.buttons.entries()) {
      if (button.pressed) {
        buttonsPressed.push(i);
      }
    }
  }
}

function gamepadButtonPressedHandler(button) {
  return buttonsPressed.includes(button);
}
```

**ゲームループへの統合:**

```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  gamepadUpdateHandler();

  if (gamepadButtonPressedHandler(12)) playerY -= 5;  // D-Pad Up
  else if (gamepadButtonPressedHandler(13)) playerY += 5;  // D-Pad Down
  if (gamepadButtonPressedHandler(14)) playerX -= 5;  // D-Pad Left
  else if (gamepadButtonPressedHandler(15)) playerX += 5;  // D-Pad Right
  if (gamepadButtonPressedHandler(0)) alert("BOOM!");  // A Button

  ctx.drawImage(img, playerX, playerY);
  requestAnimationFrame(draw);
}
```

**ホールドと押下を判定できる再利用可能な GamepadAPI ライブラリ:**

```javascript
const GamepadAPI = {
  active: false,
  controller: {},

  connect(event) {
    GamepadAPI.controller = event.gamepad;
    GamepadAPI.active = true;
  },

  disconnect(event) {
    delete GamepadAPI.controller;
    GamepadAPI.active = false;
  },

  update() {
    GamepadAPI.buttons.cache = [...GamepadAPI.buttons.status];
    GamepadAPI.buttons.status = [];

    const c = GamepadAPI.controller || {};
    const pressed = [];

    if (c.buttons) {
      for (let b = 0; b < c.buttons.length; b++) {
        if (c.buttons[b].pressed) {
          pressed.push(GamepadAPI.buttons.layout[b]);
        }
      }
    }

    const axes = [];
    if (c.axes) {
      for (const ax of c.axes) {
        axes.push(ax.toFixed(2));
      }
    }

    GamepadAPI.axes.status = axes;
    GamepadAPI.buttons.status = pressed;
    return pressed;
  },

  buttons: {
    layout: ["A", "B", "X", "Y", "LB", "RB", "LT", "RT",
             "Back", "Start", "LS", "RS",
             "DPad-Up", "DPad-Down", "DPad-Left", "DPad-Right"],
    cache: [],
    status: [],
    pressed(button, hold) {
      let newPress = false;
      if (GamepadAPI.buttons.status.includes(button)) {
        newPress = true;
      }
      if (!hold && GamepadAPI.buttons.cache.includes(button)) {
        newPress = false;
      }
      return newPress;
    }
  },

  axes: {
    status: []
  }
};

window.addEventListener("gamepadconnected", GamepadAPI.connect);
window.addEventListener("gamepaddisconnected", GamepadAPI.disconnect);
```

**デッドゾーンしきい値を使ったアナログスティック移動:**

```javascript
if (GamepadAPI.axes.status) {
  if (GamepadAPI.axes.status[0] > 0.5) playerX += 5;       // Right
  else if (GamepadAPI.axes.status[0] < -0.5) playerX -= 5; // Left
  if (GamepadAPI.axes.status[1] > 0.5) playerY += 5;       // Down
  else if (GamepadAPI.axes.status[1] < -0.5) playerY -= 5; // Up
}
```

**コンテキストに応じた操作表示:**

```javascript
if (this.game.device.desktop) {
  if (GamepadAPI.active) {
    moveText = "DPad or left Stick to move";
    shootText = "A to shoot, Y for controls";
  } else {
    moveText = "Arrow keys or WASD to move";
    shootText = "X or Space to shoot";
  }
} else {
  moveText = "Tap and hold to move";
  shootText = "Tap to shoot";
}
```

### ベストプラクティス

- ゲームパッド入力を処理する前に、必ず `GamepadAPI.active` を確認する。
- 前フレームのボタン状態をキャッシュし、「ホールド」（連続）と「押下」（新規1回）を区別する。
- 意図しないドリフト入力を避けるため、アナログ値にはデッドゾーンしきい値（例: 0.5）を適用する。
- デバイスごとにボタン配置が異なる可能性があるため、ボタンマッピングシステムを作る。
- `requestAnimationFrame` 内で更新関数を呼び、毎フレームゲームパッド状態をポーリングする。
- ゲームパッド接続時は、画面上インジケーターと適切な操作説明を表示する。
- ブラウザ対応率は世界で約63%のため、必ずキーボード／マウスのフォールバック操作を提供する。

## その他の操作メカニズム

非定型の操作メカニズムは、ユニークなゲーム体験を提供し、従来の入力デバイスを超えた新しいハードウェアを活用できます。

### TVリモコン

**説明:** スマートTVのリモコンは標準キーボードイベントを発行するため、Webゲームはほぼ無改造でTV画面上で動作できます。

**主なイベントとAPI:**

- リモコンの方向ボタンは標準の矢印キーコードに対応。
- カスタムボタンはメーカー固有のキーコードを持つ。

**コード例:**

```javascript
// Standard arrow key controls work automatically with TV remotes
this.cursors = this.input.keyboard.createCursorKeys();
if (this.cursors.right.isDown) {
  // move player right
}

// Discover manufacturer-specific remote key codes
window.addEventListener("keydown", (event) => {
  console.log(event.keyCode);
});

// Handle custom remote buttons (codes vary by manufacturer)
window.addEventListener("keydown", (event) => {
  switch (event.keyCode) {
    case 8:   // Pause (Panasonic example)
      break;
    case 588: // Custom action
      break;
  }
});
```

**ベストプラクティス:**

- 開発中はコンソールにキーコードを出力し、リモコンのボタンマッピングを把握する。
- リモコンはキーボードイベントを発行するため、既存のキーボード操作実装を再利用する。
- キーコード対応は、メーカーのドキュメントやチートシートを参照する。

### Leap Motion（手のジェスチャー認識）

**説明:** Leap Motionセンサーを使い、物理接触なしで手の位置・回転・握り強度を検出し、ジェスチャーベースの操作を実現します。

**主なイベントとAPI:**

- `Leap.loop()` -- フレームベースの手追跡コールバック。
- `hand.roll()` -- 水平回転（ラジアン）。
- `hand.pitch()` -- 垂直回転（ラジアン）。
- `hand.grabStrength` -- 握り強度（0: 手を開いた状態〜1: 握りこぶし）。

**コード例:**

```html
<script src="https://js.leapmotion.com/leap-0.6.4.min.js"></script>
```

```javascript
const toDegrees = 1 / (Math.PI / 180);
let horizontalDegree = 0;
let verticalDegree = 0;
const degreeThreshold = 30;
let grabStrength = 0;

Leap.loop({
  hand(hand) {
    horizontalDegree = Math.round(hand.roll() * toDegrees);
    verticalDegree = Math.round(hand.pitch() * toDegrees);
    grabStrength = hand.grabStrength;
  },
});

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  if (horizontalDegree > degreeThreshold) playerX -= 5;
  else if (horizontalDegree < -degreeThreshold) playerX += 5;

  if (verticalDegree > degreeThreshold) playerY += 5;
  else if (verticalDegree < -degreeThreshold) playerY -= 5;

  if (grabStrength === 1) fireWeapon();

  ctx.drawImage(img, playerX, playerY);
  requestAnimationFrame(draw);
}
```

**ベストプラクティス:**

- 小さな手ぶれやノイズを除去するため、角度しきい値（例: 30度）を使用する。
- 開発中に診断データを出力して感度を調整する。
- 複雑な多入力より、操縦や射撃のようなシンプルな操作に限定する。
- Leap Motionドライバーのインストールが必要。

### ドップラー効果（マイクベースのジェスチャー検出）

**説明:** デバイスのマイクで取得した音波の周波数シフトを解析し、手の移動方向と大きさを検出します。発した音が手で反射し、その周波数差から移動方向を推定します。

**主なイベントとAPI:**

- ドップラー効果検出ライブラリを使用。
- `bandwidth.left` と `bandwidth.right` が周波数解析値を提供。

**コード例:**

```javascript
doppler.init((bandwidth) => {
  const diff = bandwidth.left - bandwidth.right;
  // Positive diff = movement in one direction
  // Negative diff = movement in the other direction
});
```

**ベストプラクティス:**

- スクロールや上下移動など、1軸のシンプルな操作に向いている。
- Leap Motionやゲームパッド入力より精度は低い。
- 左右の周波数差比較により方向情報を得られる。

### Makey Makey（物理オブジェクトコントローラー）

**説明:** 導電性のある物体（バナナ、粘土、描いた回路、水など）を、キーボード／マウス入力をエミュレートする基板に接続し、ゲーム向けの創造的な物理インターフェースを実現します。

**主なイベントとAPI（カスタムハードウェアでの Cylon.js 利用時）:**

- Arduino / Raspberry Pi のカスタム構成用 `makey-button` ドライバー。
- ボタン有効化の `"push"` イベントリスナー。
- Makey Makey基板自体はUSB経由で動作し、カスタムコードなしで標準キーボードイベントを発行。

**コード例（Cylon.js を使ったカスタム構成）:**

```javascript
const Cylon = require("cylon");

Cylon.robot({
  connections: {
    arduino: { adaptor: "firmata", port: "/dev/ttyACM0" },
  },
  devices: {
    makey: { driver: "makey-button", pin: 2 },
  },
  work(my) {
    my.makey.on("push", () => {
      console.log("Button pushed!");
      // Trigger game action
    });
  },
}).start();
```

**ベストプラクティス:**

- Makey Makey基板はUSB接続で標準キーボードイベントを発行するため、既存キーボード操作はそのまま動作する。
- カスタム構成でGPIO接続する場合は、10 MOhm 抵抗を使用する。
- 展示やインスタレーションに特に適した、創造的なフィジカルゲーム体験を実現できる。

### 非定型操作に関する一般的な推奨事項

- できるだけ広いユーザー層に届けるため、複数の操作メカニズムを実装する。
- 多くの非定型コントローラーは標準入力をエミュレートまたは補完するため、キーボードとゲームパッドを基盤に設計する。
- 精度の低いハードウェアでは、しきい値を使ってノイズや誤入力を除去する。
- 開発中はコンソール出力や画面表示値で視覚的な診断を行う。
- ゲーム要件に合わせて操作の複雑さを調整する。すべてのメカニズムがすべてのゲームに適するわけではない。
- その上にゲームロジックを実装する前に、ハードウェア構成を十分にテストする。

