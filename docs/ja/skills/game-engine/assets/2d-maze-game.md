# 2D迷路ゲームテンプレート

プレイヤーが障害物だらけの迷路でボールを導き、目標の穴を目指す、モバイル最適化された2D迷路ゲームです。モバイルでは傾き操作のために **Device Orientation API** を使用し、デスクトップではキーボードの矢印キーに対応しています。**Phaser** フレームワーク（v2.x + Arcade Physics）で構築されており、複数レベルの進行、衝突判定、音声フィードバック、バイブレーション（触覚）フィードバック、タイマーシステムを備えています。

**参考ソース:** [MDN - HTML5 Gamedev Phaser Device Orientation](https://developer.mozilla.org/en-US/docs/Games/Tutorials/HTML5_Gamedev_Phaser_Device_Orientation)
**ライブデモ:** [Cyber Orb](https://orb.enclavegames.com/)
**ソースコード:** [GitHub - EnclaveGames/Cyber-Orb](https://github.com/EnclaveGames/Cyber-Orb)

---

## ゲームコンセプト

プレイヤーはモバイル端末を傾ける、または矢印キーを押すことでボール（"orb"）を操作します。ボールは水平・垂直の壁セグメントで構成された迷路内を転がります。各レベルの目的は、壁を避けながら画面上部の穴までボールを運ぶことです。壁との衝突時にはバウンド、効果音、（任意で）バイブレーションが発生します。タイマーは各レベルおよびゲーム全体の所要時間を計測します。

---

## プロジェクト構成

```
project/
  index.html
  src/
    phaser-arcade-physics.2.2.2.min.js
    Boot.js
    Preloader.js
    MainMenu.js
    Howto.js
    Game.js
  img/
    ball.png
    hole.png
    element-horizontal.png
    element-vertical.png
    button-start.png
    loading-bg.png
    loading-bar.png
  audio/
    bounce.ogg
    bounce.mp3
    bounce.m4a
```

---

## Phaserのセットアップと初期化

### HTMLエントリーポイント

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Cyber Orb</title>
  <style>
    body { margin: 0; background: #333; }
  </style>
  <script src="src/phaser-arcade-physics.2.2.2.min.js"></script>
  <script src="src/Boot.js"></script>
  <script src="src/Preloader.js"></script>
  <script src="src/MainMenu.js"></script>
  <script src="src/Howto.js"></script>
  <script src="src/Game.js"></script>
</head>
<body>
  <script>
    (() => {
      const game = new Phaser.Game(320, 480, Phaser.CANVAS, "game");
      game.state.add("Boot", Ball.Boot);
      game.state.add("Preloader", Ball.Preloader);
      game.state.add("MainMenu", Ball.MainMenu);
      game.state.add("Howto", Ball.Howto);
      game.state.add("Game", Ball.Game);
      game.state.start("Boot");
    })();
  </script>
</body>
</html>
```

- キャンバスサイズ: `320 x 480`
- レンダラー: `Phaser.CANVAS`（代替: `Phaser.WEBGL`, `Phaser.AUTO`）

---

## ゲームステートアーキテクチャ

このゲームは次の線形ステートフローに従います:

```
Boot --> Preloader --> MainMenu --> Howto --> Game
```

### Boot State

ローディング画面用の最小限アセットを読み込み、スケーリングを設定します。

```javascript
const Ball = {
  _WIDTH: 320,
  _HEIGHT: 480,
};

Ball.Boot = function (game) {};
Ball.Boot.prototype = {
  preload() {
    this.load.image("preloaderBg", "img/loading-bg.png");
    this.load.image("preloaderBar", "img/loading-bar.png");
  },
  create() {
    this.game.scale.scaleMode = Phaser.ScaleManager.SHOW_ALL;
    this.game.scale.pageAlignHorizontally = true;
    this.game.scale.pageAlignVertically = true;
    this.game.state.start("Preloader");
  },
};
```

### Preloader State

すべてのゲームアセットを読み込みながら、視覚的なローディングバーを表示します。音声はブラウザ互換性のため複数形式で読み込みます。

```javascript
Ball.Preloader = function (game) {};
Ball.Preloader.prototype = {
  preload() {
    this.preloadBg = this.add.sprite(
      (Ball._WIDTH - 297) * 0.5,
      (Ball._HEIGHT - 145) * 0.5,
      "preloaderBg"
    );
    this.preloadBar = this.add.sprite(
      (Ball._WIDTH - 158) * 0.5,
      (Ball._HEIGHT - 50) * 0.5,
      "preloaderBar"
    );
    this.load.setPreloadSprite(this.preloadBar);

    this.load.image("ball", "img/ball.png");
    this.load.image("hole", "img/hole.png");
    this.load.image("element-w", "img/element-horizontal.png");
    this.load.image("element-h", "img/element-vertical.png");
    this.load.spritesheet("button-start", "img/button-start.png", 146, 51);
    this.load.audio("audio-bounce", [
      "audio/bounce.ogg",
      "audio/bounce.mp3",
      "audio/bounce.m4a",
    ]);
  },
  create() {
    this.game.state.start("MainMenu");
  },
};
```

### MainMenu State

スタートボタン付きのタイトル画面を表示します。

```javascript
Ball.MainMenu = function (game) {};
Ball.MainMenu.prototype = {
  create() {
    this.add.sprite(0, 0, "screen-mainmenu");
    this.gameTitle = this.add.sprite(Ball._WIDTH * 0.5, 40, "title");
    this.gameTitle.anchor.set(0.5, 0);

    this.startButton = this.add.button(
      Ball._WIDTH * 0.5, 200, "button-start",
      this.startGame, this,
      2, 0, 1  // hover, out, down frames
    );
    this.startButton.anchor.set(0.5, 0);
    this.startButton.input.useHandCursor = true;
  },
  startGame() {
    this.game.state.start("Howto");
  },
};
```

### Howto State

ゲーム開始前に表示される、ワンクリックの説明画面です。

```javascript
Ball.Howto = function (game) {};
Ball.Howto.prototype = {
  create() {
    this.buttonContinue = this.add.button(
      0, 0, "screen-howtoplay",
      this.startGame, this
    );
  },
  startGame() {
    this.game.state.start("Game");
  },
};
```

---

## Device Orientation API の利用

Device Orientation API は、デバイスの物理的な傾きに関するリアルタイムデータを提供します。使用する軸は2つです。

| プロパティ | 軸 | 範囲 | 効果 |
|----------|------|-------|--------|
| `event.gamma` | 左右の傾き | -90〜90度 | ボールの水平方向速度 |
| `event.beta` | 前後の傾き | -180〜180度 | ボールの垂直方向速度 |

### リスナーの登録

```javascript
// In the Game state's create() method
window.addEventListener("deviceorientation", this.handleOrientation);
```

### Orientationイベントの処理

```javascript
handleOrientation(e) {
  const x = e.gamma; // left-right tilt
  const y = e.beta;  // front-back tilt
  Ball._player.body.velocity.x += x;
  Ball._player.body.velocity.y += y;
}
```

### 傾き挙動

- デバイスを左に傾ける: gammaが負、ボールは左へ転がる
- デバイスを右に傾ける: gammaが正、ボールは右へ転がる
- デバイスを前に傾ける: betaが正、ボールは下へ転がる
- デバイスを後ろに傾ける: betaが負、ボールは上へ転がる

傾き角度は速度の増分に直接マッピングされます。つまり、傾きが大きいほど、各フレームでボールに加わる力も大きくなります。

---

## コアゲームメカニクス

### ゲームステート構造

```javascript
Ball.Game = function (game) {};
Ball.Game.prototype = {
  create() {},
  initLevels() {},
  showLevel(level) {},
  updateCounter() {},
  managePause() {},
  manageAudio() {},
  update() {},
  wallCollision() {},
  handleOrientation(e) {},
  finishLevel() {},
};
```

### ボール生成と物理設定

```javascript
// In create()
this.ball = this.add.sprite(this.ballStartPos.x, this.ballStartPos.y, "ball");
this.ball.anchor.set(0.5);
this.physics.enable(this.ball, Phaser.Physics.ARCADE);
this.ball.body.setSize(18, 18);
this.ball.body.bounce.set(0.3, 0.3);
```

- 中心 `(0.5, 0.5)` にアンカーを設定し、中点を軸に回転
- 物理ボディ: 18x18ピクセル
- 反発係数: 0.3（壁衝突後に速度の30%を維持）

### キーボード操作（デスクトップ向けフォールバック）

```javascript
// In create()
this.keys = this.game.input.keyboard.createCursorKeys();

// In update()
if (this.keys.left.isDown) {
  this.ball.body.velocity.x -= this.movementForce;
} else if (this.keys.right.isDown) {
  this.ball.body.velocity.x += this.movementForce;
}
if (this.keys.up.isDown) {
  this.ball.body.velocity.y -= this.movementForce;
} else if (this.keys.down.isDown) {
  this.ball.body.velocity.y += this.movementForce;
}
```

### 穴（ゴール）の設定

```javascript
this.hole = this.add.sprite(Ball._WIDTH * 0.5, 90, "hole");
this.physics.enable(this.hole, Phaser.Physics.ARCADE);
this.hole.anchor.set(0.5);
this.hole.body.setSize(2, 2);
```

穴は正確な重なり判定のために、2x2の小さな衝突ボディを持ちます。

---

## レベルシステム

### レベルデータ形式

各レベルは、位置とタイプを持つ壁セグメントオブジェクトの配列です:

```javascript
this.levelData = [
  [{ x: 96, y: 224, t: "w" }],                           // Level 1
  [
    { x: 72, y: 320, t: "w" },
    { x: 200, y: 320, t: "h" },
    { x: 72, y: 150, t: "w" },
  ],                                                       // Level 2
  // ... more levels
];
```

- `x, y`: ピクセル単位の位置
- `t`: タイプ -- `"w"` は水平壁、`"h"` は垂直壁

### レベル構築

```javascript
initLevels() {
  for (let i = 0; i < this.maxLevels; i++) {
    const newLevel = this.add.group();
    newLevel.enableBody = true;
    newLevel.physicsBodyType = Phaser.Physics.ARCADE;

    for (const item of this.levelData[i]) {
      newLevel.create(item.x, item.y, `element-${item.t}`);
    }

    newLevel.setAll("body.immovable", true);
    newLevel.visible = false;
    this.levels.push(newLevel);
  }
}
```

### レベル表示

```javascript
showLevel(level) {
  const lvl = level || this.level;
  if (this.levels[lvl - 2]) {
    this.levels[lvl - 2].visible = false;
  }
  this.levels[lvl - 1].visible = true;
}
```

---

## 衝突判定

### 壁衝突（バウンド）

```javascript
// In update()
this.physics.arcade.collide(
  this.ball, this.borderGroup,
  this.wallCollision, null, this
);
this.physics.arcade.collide(
  this.ball, this.levels[this.level - 1],
  this.wallCollision, null, this
);
```

`collide` はボールを壁でバウンドさせ、コールバックを発火します。

### 穴との重なり（すり抜け判定）

```javascript
this.physics.arcade.overlap(
  this.ball, this.hole,
  this.finishLevel, null, this
);
```

`overlap` は物理的な衝突反応なしで交差を検出します。

### 壁衝突コールバック

```javascript
wallCollision() {
  if (this.audioStatus) {
    this.bounceSound.play();
  }
  if ("vibrate" in window.navigator) {
    window.navigator.vibrate(100);
  }
}
```

---

## オーディオシステム

```javascript
// In create()
this.bounceSound = this.game.add.audio("audio-bounce");

// Toggle
manageAudio() {
  this.audioStatus = !this.audioStatus;
}
```

---

## Vibration API

```javascript
if ("vibrate" in window.navigator) {
  window.navigator.vibrate(100); // 100ms vibration pulse
}
```

呼び出す前に機能検出を行います。対応モバイル端末で触覚フィードバックを提供します。

---

## タイマーシステム

```javascript
// In create()
this.timer = 0;
this.totalTimer = 0;
this.timerText = this.game.add.text(15, 15, "Time: 0", this.fontBig);
this.totalTimeText = this.game.add.text(120, 30, "Total time: 0", this.fontSmall);
this.time.events.loop(Phaser.Timer.SECOND, this.updateCounter, this);

// Counter callback
updateCounter() {
  this.timer++;
  this.timerText.setText(`Time: ${this.timer}`);
  this.totalTimeText.setText(`Total time: ${this.totalTimer + this.timer}`);
}
```

---

## レベルクリア処理

```javascript
finishLevel() {
  if (this.level >= this.maxLevels) {
    this.totalTimer += this.timer;
    alert(`Congratulations, game completed!\nTotal time: ${this.totalTimer}s`);
    this.game.state.start("MainMenu");
  } else {
    alert(`Level ${this.level} completed!`);
    this.totalTimer += this.timer;
    this.timer = 0;
    this.level++;
    this.timerText.setText(`Time: ${this.timer}`);
    this.totalTimeText.setText(`Total time: ${this.totalTimer}`);
    this.levelText.setText(`Level: ${this.level} / ${this.maxLevels}`);
    this.ball.body.x = this.ballStartPos.x;
    this.ball.body.y = this.ballStartPos.y;
    this.ball.body.velocity.x = 0;
    this.ball.body.velocity.y = 0;
    this.showLevel();
  }
}
```

---

## 完全な更新ループ

```javascript
update() {
  // Keyboard input
  if (this.keys.left.isDown) {
    this.ball.body.velocity.x -= this.movementForce;
  } else if (this.keys.right.isDown) {
    this.ball.body.velocity.x += this.movementForce;
  }
  if (this.keys.up.isDown) {
    this.ball.body.velocity.y -= this.movementForce;
  } else if (this.keys.down.isDown) {
    this.ball.body.velocity.y += this.movementForce;
  }

  // Wall collisions
  this.physics.arcade.collide(
    this.ball, this.borderGroup, this.wallCollision, null, this
  );
  this.physics.arcade.collide(
    this.ball, this.levels[this.level - 1], this.wallCollision, null, this
  );

  // Hole overlap
  this.physics.arcade.overlap(
    this.ball, this.hole, this.finishLevel, null, this
  );
}
```

---

## Phaser API クイックリファレンス

| 関数 | 用途 |
|----------|---------|
| `this.add.sprite(x, y, key)` | ゲームオブジェクトを作成 |
| `this.add.group()` | オブジェクト用コンテナを作成 |
| `this.add.button(x, y, key, cb, ctx, over, out, down)` | インタラクティブなボタンを作成 |
| `this.add.text(x, y, text, style)` | テキスト表示を作成 |
| `this.physics.enable(obj, system)` | オブジェクトの物理演算を有効化 |
| `this.physics.arcade.collide(a, b, cb)` | バウンドありの衝突を検出 |
| `this.physics.arcade.overlap(a, b, cb)` | バウンドなしの重なりを検出 |
| `this.load.image(key, path)` | 画像アセットを読み込み |
| `this.load.spritesheet(key, path, w, h)` | スプライトアニメーションシートを読み込み |
| `this.load.audio(key, paths[])` | フォーマットフォールバック付きで音声を読み込み |
| `this.game.add.audio(key)` | 音声オブジェクトを生成 |
| `this.time.events.loop(interval, cb, ctx)` | 繰り返しタイマーを作成 |

