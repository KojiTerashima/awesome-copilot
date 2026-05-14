# 2D プラットフォーム ゲーム テンプレート

Phaser (v2.x / Phaser CE) と Arcade Physics を使用して 2D プラットフォーマー ゲームを構築するための完全なステップバイステップ ガイド。このテンプレートは、プロジェクトのセットアップ、JSON レベルのデータからのプラットフォームの作成、物理ベースの動きとジャンプを備えたヒーローの追加、収集可能なコイン、歩く敵、死と踏みつけのメカニズム、スコアボード、スプライト アニメーション、ドア/キー システムによる勝利条件、マルチレベルの進行など、開発のあらゆる段階を順を追って説明します。

**構築するもの:** 古典的な横スクロール プラットフォーマーで、ヒーローがプラットフォームを移動し、コインを集め、クモの敵を避けたり踏みつけたり、ドアのロックを解除する鍵を見つけたり、スコア追跡、アニメーション、物理学を使用して複数のレベルを進んでいきます。

**前提条件:** 基本から中級の JavaScript の知識、HTML、開発用のローカル Web サーバー (ブラウザ同期、ライブサーバー、Python の SimpleHTTPServer など) に精通していること。

**出典:** [Mozilla HTML5 ゲーム ワークショップ - プラットフォーマー](https://mozdevs.github.io/html5-games-workshop/en/guides/platformer/start-here/) に基づいています。プロジェクト スターター ファイルはワークショップ リポジトリで入手できます。

---

## ここから始めてください

このチュートリアルでは、**Phaser** フレームワークを使用して 2D プラットフォーマーを構築します。 Phaser はレンダリング、物理学、入力、オーディオ、アセットの読み込みを処理するため、ゲーム ロジックに集中できます。

### あなたが構築するもの

完成したゲームの特徴は次のとおりです。

- プレイヤーがキーボードで操作するヒーローキャラクター
- 主人公が歩いたりジャンプしたりできるプラットフォーム
- スコアを増やす収集可能なコイン
- 歩くクモの敵は接触するとヒーローを殺します（ただし上から踏みつけられる可能性があります）
- キーとドアのシステム: 主人公はドアのロックを解除し、レベルを完了するためにキーを拾う必要があります。
- JSON データ ファイルからロードされた複数のレベル
- 集めたコインを示すスコアボード
- ヒーローのスプライトアニメーション (アイドル、ランニング、ジャンプ、落下)

### プロジェクトの構造```
project/
  index.html
  js/
    phaser.min.js        (Phaser 2.6.2 or Phaser CE)
    main.js              (all game code goes here)
  audio/
    sfx/
      jump.wav
      coin.wav
      stomp.wav
      key.wav
      door.wav
  images/
    background.png
    ground.png
    grass:8x1.png        (platform tile images in various sizes)
    grass:6x1.png
    grass:4x1.png
    grass:2x1.png
    grass:1x1.png
    hero.png             (hero spritesheet: 36x42 per frame)
    hero_stopped.png     (single frame for initial steps)
    coin_animated.png    (coin spritesheet)
    spider.png           (spider spritesheet)
    invisible_wall.png   (invisible boundary for enemy AI)
    key.png              (key spritesheet)
    door.png             (door spritesheet)
    key_icon.png         (HUD icon for key)
    font:numbers.png     (bitmap font for score)
  data/
    level00.json
    level01.json
```### レベルデータフォーマット

各レベルは JSON ファイルで定義されます。 JSON 構造は、すべてのエンティティの位置を記述します。```json
{
    "hero": { "x": 21, "y": 525 },
    "door": { "x": 169, "y": 546 },
    "key": { "x": 750, "y": 524 },
    "platforms": [
        { "image": "ground", "x": 0, "y": 546 },
        { "image": "grass:8x1", "x": 208, "y": 420 },
        { "image": "grass:4x1", "x": 420, "y": 336 },
        { "image": "grass:2x1", "x": 680, "y": 252 }
    ],
    "coins": [
        { "x": 147, "y": 525 },
        { "x": 189, "y": 525 },
        { "x": 399, "y": 399 },
        { "x": 441, "y": 336 }
    ],
    "spiders": [
        { "x": 121, "y": 399 }
    ],
    "decoration": {
        "grass": [
            { "x": 84, "y": 504, "frame": 0 },
            { "x": 420, "y": 504, "frame": 1 }
        ]
    }
}
```各エンティティ タイプ (ヒーロー、ドア、キー、プラットフォーム、コイン、スパイダー) には `x` および `y` 座標があります。プラットフォームは、そのプラットフォーム タイルに使用する `image` アセットも指定します。

---

## フェイザーを初期化する

最初のステップは、HTML ファイルを設定し、Phaser ゲーム インスタンスを作成することです。

### HTML エントリ ポイント

Phaser とゲーム スクリプトをロードする `index.html` ファイルを作成します。```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Platformer Game</title>
    <style>
        html, body {
            margin: 0;
            padding: 0;
            background: #000;
        }
    </style>
    <script src="js/phaser.min.js"></script>
    <script src="js/main.js"></script>
</head>
<body>
    <div id="game"></div>
</body>
</html>
```- `<div id="game">` は、Phaser がゲーム キャンバスを挿入するコンテナーです。
- Phaser が最初にロードされ、次にゲーム スクリプトがロードされます。

### ゲームインスタンスの作成

`js/main.js` で、Phaser ゲーム オブジェクトを作成し、ゲームの状態を登録します。```javascript
// Create a Phaser game instance
// Parameters: width, height, renderer, DOM element ID
window.onload = function () {
    let game = new Phaser.Game(960, 600, Phaser.AUTO, 'game');

    // Add and start the play state
    game.state.add('play', PlayState);
    game.state.start('play');
};
```- `960, 600` は、ゲーム キャンバスの寸法をピクセル単位で設定します。
- `Phaser.AUTO` は、Phaser が WebGL レンダリングと Canvas レンダリングのどちらかを自動的に選択できるようにします。
- `'game'` は、キャンバスを含む DOM 要素の ID です。

### PlayState オブジェクト

ライフサイクル メソッドを使用してゲームの状態をオブジェクトとして定義します。```javascript
PlayState = {};

PlayState.init = function () {
    // Called first when the state starts
};

PlayState.preload = function () {
    // Load all assets here
};

PlayState.create = function () {
    // Create game entities and set up the world
};

PlayState.update = function () {
    // Called every frame (~60 times per second)
    // Handle game logic, input, collisions here
};
```- `init` -- 最初に実行されます。設定とパラメータの受信に使用されます。
- `preload` -- ゲームの開始前にすべてのアセット (画像、オーディオ、JSON) をロードするために使用されます。
- `create` -- アセットがロードされた後に 1 回呼び出されます。スプライト、グループ、ゲーム オブジェクトの作成に使用されます。
- `update` -- フレームごとに ~60fps で呼び出されます。入力処理、物理チェック、ゲーム ロジックに使用されます。

この時点で、ページ上に空の黒いキャンバスが表示されるはずです。

---

## ゲームループ

Phaser はゲーム ループ アーキテクチャを使用します。フレームごとに、Phaser は `update()` を呼び出します。ここで入力を処理し、スプライトを移動し、衝突をチェックします。ループが開始する前に、`preload()` はアセットをロードし、`create()` はゲームの初期状態を設定します。

### 背景のロードと表示

まず背景画像をロードして表示し、ゲーム ループが機能していることを確認します。```javascript
PlayState.preload = function () {
    this.game.load.image('background', 'images/background.png');
};

PlayState.create = function () {
    // Add the background image at position (0, 0)
    this.game.add.image(0, 0, 'background');
};
```- `this.game.load.image(key, path)` は画像をロードし、後で参照できるようにキーを割り当てます。
- `this.game.add.image(x, y, key)` は、指定された位置に静止画像を作成します。

ゲーム キャンバスに背景画像がレンダリングされているのが表示されます。

### フレームサイクルを理解する```
preload() -> [assets loaded] -> create() -> update() -> update() -> update() -> ...
````update()` への各呼び出しは 1 つのフレームを表します。ゲームは 1 秒あたり 60 フレームをターゲットとしています。すべての移動、入力読み取り、衝突検出は `update()` 内で行われます。

---

## プラットフォームの作成

プラットフォームは、主人公が歩いたりジャンプしたりする表面です。これらはレベル JSON データからロードされ、グループに配置された物理対応スプライトとして作成されます。

### プラットフォーム資産のロード

レベルの JSON データとすべてのプラットフォーム タイル イメージを `preload` にロードします。```javascript
PlayState.preload = function () {
    this.game.load.image('background', 'images/background.png');

    // Load level data
    this.game.load.json('level:1', 'data/level01.json');

    // Load platform images
    this.game.load.image('ground', 'images/ground.png');
    this.game.load.image('grass:8x1', 'images/grass_8x1.png');
    this.game.load.image('grass:6x1', 'images/grass_6x1.png');
    this.game.load.image('grass:4x1', 'images/grass_4x1.png');
    this.game.load.image('grass:2x1', 'images/grass_2x1.png');
    this.game.load.image('grass:1x1', 'images/grass_1x1.png');
};
```### レベルデータからプラットフォームを生成する

レベルをロードし、各プラットフォームを物理グループ内のスプライトとして生成するメソッドを作成します。```javascript
PlayState.create = function () {
    // Add the background
    this.game.add.image(0, 0, 'background');

    // Load level data and spawn entities
    this._loadLevel(this.game.cache.getJSON('level:1'));
};

PlayState._loadLevel = function (data) {
    // Create a group for platforms
    this.platforms = this.game.add.group();

    // Spawn each platform from the level data
    data.platforms.forEach(this._spawnPlatform, this);
};

PlayState._spawnPlatform = function (platform) {
    // Add a sprite at the platform's position using the specified image
    let sprite = this.platforms.create(platform.x, platform.y, platform.image);

    // Enable physics on this platform
    this.game.physics.enable(sprite);

    // Make platform immovable so it doesn't get pushed by the hero
    sprite.body.allowGravity = false;
    sprite.body.immovable = true;
};
```- `this.game.add.group()` は、バッチ操作と衝突検出を可能にする関連するスプライトのコンテナーである Phaser グループを作成します。
- `this.platforms.create(x, y, key)` はグループ内にスプライトを作成します。
- `sprite.body.immovable = true` は、プラットフォームが他の物理ボディによってプッシュされるのを防ぎます。
- `sprite.body.allowGravity = false` は、重力によるプラットフォームの落下を防ぎます。

地面と草のプラットフォームのタイルが画面上にレンダリングされているのが表示されます。

---

## 主人公のスプライト

次に、プレイヤーが操作するヒーローキャラクターを追加します。

### ヒーロー画像のロード

ヒーロー画像を `preload` に追加します。最初は単一の静的画像を使用します。後でアニメーション用にスプライトシートに切り替えます。```javascript
// In PlayState.preload:
this.game.load.image('hero', 'images/hero_stopped.png');
```### ヒーローのスポーン

ヒーローを `_loadLevel` に追加し、スポーン メソッドを作成します。```javascript
PlayState._loadLevel = function (data) {
    this.platforms = this.game.add.group();
    data.platforms.forEach(this._spawnPlatform, this);

    // Spawn the hero at the position defined in level data
    this._spawnCharacters({ hero: data.hero });
};

PlayState._spawnCharacters = function (data) {
    // Create the hero sprite
    this.hero = this.game.add.sprite(data.hero.x, data.hero.y, 'hero');

    // Set the anchor to the bottom-center for easier positioning
    this.hero.anchor.set(0.5, 1);
};
```- `anchor.set(0.5, 1)` は、スプライトの原点を水平方向の中央と垂直方向の下部に設定します。これにより、`y` の位置が左上隅ではなくヒーローの足元を指すため、ヒーローをプラットフォームの上に配置することが容易になります。

---

## キーボードコントロール

キーボード入力をキャプチャして、プレイヤーがヒーローを左右に動かしたり、ジャンプしたりできるようにします。

### 入力キーの設定

`init` で、キーボード コントロールを構成します。```javascript
PlayState.init = function () {
    // Force integer rendering for pixel-art crispness
    this.game.renderer.renderSession.roundPixels = true;

    // Capture arrow keys
    this.keys = this.game.input.keyboard.addKeys({
        left: Phaser.KeyCode.LEFT,
        right: Phaser.KeyCode.RIGHT,
        up: Phaser.KeyCode.UP
    });
};
```- `addKeys()` は、指定されたキーをキャプチャし、キー状態参照を持つオブジェクトを返します。
- `Phaser.KeyCode.LEFT`、`RIGHT`、`UP`は矢印キーに対応します。
- `renderSession.roundPixels = true` は、サブピクセル レンダリングによってピクセル アートのスプライトがぼやけて見えるのを防ぎます。

### アップデートでの入力の読み取り

`update` でキーの状態を処理します。今のところは、方向を記録するだけです。次のステップでは、物理ベースの動きを追加します。```javascript
PlayState.update = function () {
    this._handleInput();
};

PlayState._handleInput = function () {
    if (this.keys.left.isDown) {
        // Move hero left
    } else if (this.keys.right.isDown) {
        // Move hero right
    } else {
        // Stop (no key held)
    }
};
```- `this.keys.left.isDown` は、左矢印キーを押している間、`true` を返します。
- `else` 句は、左も右も押されていない場合を処理します (主人公は停止する必要があります)。

---

## 物理演算を使用してスプライトを移動する

アーケード フィジックスを有効にすると、ヒーローが速度を上げて移動し、衝突を通じてプラットフォームと対話できるようになります。

### 物理エンジンの有効化

`init` でアーケード フィジックスを有効にする:```javascript
PlayState.init = function () {
    this.game.renderer.renderSession.roundPixels = true;

    this.keys = this.game.input.keyboard.addKeys({
        left: Phaser.KeyCode.LEFT,
        right: Phaser.KeyCode.RIGHT,
        up: Phaser.KeyCode.UP
    });

    // Enable Arcade Physics
    this.game.physics.startSystem(Phaser.Physics.ARCADE);
};
```### ヒーローに物理ボディを追加する

`_spawnCharacters` でヒーロー スプライトの物理演算を有効にします。```javascript
PlayState._spawnCharacters = function (data) {
    this.hero = this.game.add.sprite(data.hero.x, data.hero.y, 'hero');
    this.hero.anchor.set(0.5, 1);

    // Enable physics body on the hero
    this.game.physics.enable(this.hero);
};
```### 速度に合わせて移動する

ここで `_handleInput` を更新して、キーの押下に基づいてヒーローの速度を設定します。```javascript
const SPEED = 200; // pixels per second

PlayState._handleInput = function () {
    if (this.keys.left.isDown) {
        this.hero.body.velocity.x = -SPEED;
    } else if (this.keys.right.isDown) {
        this.hero.body.velocity.x = SPEED;
    } else {
        this.hero.body.velocity.x = 0;
    }
};
```- `body.velocity.x` は、水平速度をピクセル/秒で設定します。
- 負の値を指定すると、スプライトが左に移動します。ポジティブにすると右に移動します。
- キーが押されていないときにベロシティを `0` に設定すると、ヒーローはすぐに停止します。

ヒーローは左右に移動できるようになりましたが、重力や衝突の処理がまだないため、プラットフォームを通って落ちたり、画面の外に落ちたりします。

---

## 重力

重力を追加すると、ヒーローが下に落ちてプラットフォームに衝突します。

### グローバル重力の設定

`init` で物理世界全体の重力を有効にします。```javascript
PlayState.init = function () {
    this.game.renderer.renderSession.roundPixels = true;

    this.keys = this.game.input.keyboard.addKeys({
        left: Phaser.KeyCode.LEFT,
        right: Phaser.KeyCode.RIGHT,
        up: Phaser.KeyCode.UP
    });

    this.game.physics.startSystem(Phaser.Physics.ARCADE);

    // Set global gravity
    this.game.physics.arcade.gravity.y = 1200;
};
```- `gravity.y = 1200` は、すべての物理演算が有効なスプライトに 1200 ピクセル/秒の 2 乗の下向き加速を適用します (`allowGravity = false` でオプトアウトしない限り)。

### ヒーローとプラットフォーム間の衝突検出

`update` に衝突検出を追加して、ヒーローが落ちずにプラットフォームに着地するようにします。```javascript
PlayState.update = function () {
    this._handleCollisions();
    this._handleInput();
};

PlayState._handleCollisions = function () {
    // Make the hero collide with the platform group
    this.game.physics.arcade.collide(this.hero, this.platforms);
};
```- `arcade.collide(spriteA, groupB)` は、ヒーローとプラットフォーム グループ内のすべてのスプライトの間の物理衝突をチェックします。ヒーローがプラットフォームに着地すると、物理エンジンがヒーローの通過を阻止し、重なりを解決します。
- 入力を処理するときに衝突データ (ヒーローが地面に触れているかどうかなど) が最新になるように、`_handleInput()` の前に `_handleCollisions()` を呼び出すことが重要です。

主人公は重力により落下し、プラットフォームに着地します。ホーム上は左右に歩くことができます。

---

## ジャンプ

上矢印キーを押したときにヒーローがジャンプできるようにします。ただし、プラットフォームに立っている場合に限ります (空中ジャンプはできません)。

### ジャンプメカニズムの実装

ジャンプ定数を追加し、`_handleInput` を更新します。```javascript
const SPEED = 200;
const JUMP_SPEED = 600;

PlayState._handleInput = function () {
    if (this.keys.left.isDown) {
        this.hero.body.velocity.x = -SPEED;
    } else if (this.keys.right.isDown) {
        this.hero.body.velocity.x = SPEED;
    } else {
        this.hero.body.velocity.x = 0;
    }

    // Handle jumping
    if (this.keys.up.isDown) {
        this._jump();
    }
};

PlayState._jump = function () {
    let canJump = this.hero.body.touching.down;

    if (canJump) {
        this.hero.body.velocity.y = -JUMP_SPEED;
    }

    return canJump;
};
```- ヒーローの物理ボディがその下面で別のボディに触れている場合、`this.hero.body.touching.down` は `true` になります。つまり、ヒーローが何かの上に立っていることを意味します。
- `velocity.y` を負の値に設定すると、ヒーローが上向きに起動します (Y 軸は画面座標で下を指します)。
- `canJump` チェックは、ヒーローがすでに空中にいるときにジャンプすることを防ぎ、シングル ジャンプ動作を強制します。
- このメソッドは、ジャンプが実行されたかどうかを返します。これは、後で効果音を再生するときに役立ちます。

### ジャンプ効果音の追加

ジャンプ サウンドをロードし、ジャンプが成功したときに再生します。```javascript
// In PlayState.preload:
this.game.load.audio('sfx:jump', 'audio/sfx/jump.wav');

// In PlayState.create:
this.sfx = {
    jump: this.game.add.audio('sfx:jump')
};

// In PlayState._jump, after setting velocity:
PlayState._jump = function () {
    let canJump = this.hero.body.touching.down;

    if (canJump) {
        this.hero.body.velocity.y = -JUMP_SPEED;
        this.sfx.jump.play();
    }

    return canJump;
};
```---

## 選択可能なコイン

プレイヤーがスコアを増やすために拾える収集可能なコインを追加します。

### コイン資産のロード

コインのスプライトシートとコインの効果音を `preload` にロードします。```javascript
// In PlayState.preload:
this.game.load.spritesheet('coin', 'images/coin_animated.png', 22, 22);
this.game.load.audio('sfx:coin', 'audio/sfx/coin.wav');
```- `load.spritesheet(key, path, frameWidth, frameHeight)` は、スプライトシートをロードし、アニメーション用に 22x22 ピクセルの個々のフレームにスライスします。

### レベルデータからコインを生成する

`_loadLevel` を更新してコイン グループを作成し、各コインを生成します。```javascript
PlayState._loadLevel = function (data) {
    this.platforms = this.game.add.group();
    this.coins = this.game.add.group();

    data.platforms.forEach(this._spawnPlatform, this);
    data.coins.forEach(this._spawnCoin, this);

    this._spawnCharacters({ hero: data.hero });
};

PlayState._spawnCoin = function (coin) {
    let sprite = this.coins.create(coin.x, coin.y, 'coin');
    sprite.anchor.set(0.5, 0.5);

    // Add a tween animation to make the coin bob up and down
    this.game.physics.enable(sprite);
    sprite.body.allowGravity = false;

    // Coin bobbing animation with a tween
    sprite.animations.add('rotate', [0, 1, 2, 1], 6, true); // 6fps, looping
    sprite.animations.play('rotate');
};
```- 衝突検出を容易にするために、各コインは `coins` グループ内に作成されます。
- `allowGravity = false`はコインの落下を防ぎます。
- `animations.add` は、スプライトシートのフレーム 0、1、2、1 を 6fps で使用し、連続的にループするフレーム アニメーションを作成します。

### コインを集める

コインの音を sfx オブジェクトに追加し、ヒーローとコインの間の重なりを検出します。```javascript
// In PlayState.create, add to the sfx object:
this.sfx = {
    jump: this.game.add.audio('sfx:jump'),
    coin: this.game.add.audio('sfx:coin')
};

// In PlayState._handleCollisions:
PlayState._handleCollisions = function () {
    this.game.physics.arcade.collide(this.hero, this.platforms);

    // Detect overlap between hero and coins (no physical collision, just overlap)
    this.game.physics.arcade.overlap(
        this.hero, this.coins, this._onHeroVsCoin, null, this
    );
};

PlayState._onHeroVsCoin = function (hero, coin) {
    this.sfx.coin.play();
    coin.kill();  // Remove the coin from the game
    this.coinPickupCount++;
};
```- `arcade.overlap()` は、物理的に衝突を解決せずに 2 つのスプライト/グループが重なっているかどうかをチェックします。重複を検出するとコールバック関数(`_onHeroVsCoin`)を呼び出します。
- `coin.kill()` は、ゲーム世界からコイン スプライトを削除します。
- `this.coinPickupCount` は、収集されたコインの数を追跡します (`_loadLevel` で初期化します)。

### Initializing the Coin Counter```javascript
PlayState._loadLevel = function (data) {
    this.platforms = this.game.add.group();
    this.coins = this.game.add.group();

    data.platforms.forEach(this._spawnPlatform, this);
    data.coins.forEach(this._spawnCoin, this);

    this._spawnCharacters({ hero: data.hero });

    // Initialize coin counter
    this.coinPickupCount = 0;
};
```---

## 歩く敵

プラットフォームを行ったり来たりするクモの敵を追加します。主人公は上から踏みつけることはできますが、横から触れると死んでしまいます。

### 敵のアセットをロードする```javascript
// In PlayState.preload:
this.game.load.spritesheet('spider', 'images/spider.png', 42, 32);
this.game.load.image('invisible-wall', 'images/invisible_wall.png');
this.game.load.audio('sfx:stomp', 'audio/sfx/stomp.wav');
```- スパイダーのスプリットシートには、這うアニメーション用のフレームがあります。
- クモが立ち去るのを防ぐために、目に見えない壁がプラットフォームの端に配置されます。壁は視覚的にレンダリングされませんが、物理ボディを持ちます。

### 敵の出現

`_loadLevel` を更新し、スパイダーのスポーン メソッドを追加します。```javascript
PlayState._loadLevel = function (data) {
    this.platforms = this.game.add.group();
    this.coins = this.game.add.group();
    this.spiders = this.game.add.group();
    this.enemyWalls = this.game.add.group();

    data.platforms.forEach(this._spawnPlatform, this);
    data.coins.forEach(this._spawnCoin, this);
    data.spiders.forEach(this._spawnSpider, this);

    this._spawnCharacters({ hero: data.hero });

    // Make enemy walls invisible
    this.enemyWalls.visible = false;

    this.coinPickupCount = 0;
};
```### プラットフォーム上に見えない壁を作成する

`_spawnPlatform` を変更して、各プラットフォームの両端に目に見えない壁を追加します。```javascript
PlayState._spawnPlatform = function (platform) {
    let sprite = this.platforms.create(platform.x, platform.y, platform.image);
    this.game.physics.enable(sprite);
    sprite.body.allowGravity = false;
    sprite.body.immovable = true;

    // Spawn invisible walls at the left and right edges of this platform
    this._spawnEnemyWall(platform.x, platform.y, 'left');
    this._spawnEnemyWall(platform.x + sprite.width, platform.y, 'right');
};

PlayState._spawnEnemyWall = function (x, y, side) {
    let sprite = this.enemyWalls.create(x, y, 'invisible-wall');

    // Anchor to the bottom of the wall and adjust position based on side
    sprite.anchor.set(side === 'left' ? 1 : 0, 1);

    this.game.physics.enable(sprite);
    sprite.body.immovable = true;
    sprite.body.allowGravity = false;
};
```- 各プラットフォームには、各端に 1 つずつ、2 つの目に見えない壁があります。
- 壁は、クモが端から歩き出すのを防ぐ障壁として機能します。
- 壁がプラットフォームの正しい側に揃うようにアンカーが設定されています。

### スパイダーのスポーンとアニメーション化```javascript
PlayState._spawnSpider = function (spider) {
    let sprite = this.spiders.create(spider.x, spider.y, 'spider');
    sprite.anchor.set(0.5, 1);

    // Add the crawl animation
    sprite.animations.add('crawl', [0, 1, 2], 8, true);
    sprite.animations.add('die', [0, 4, 0, 4, 0, 4, 3, 3, 3, 3, 3, 3], 12);
    sprite.animations.play('crawl');

    // Enable physics
    this.game.physics.enable(sprite);

    // Set initial movement speed
    sprite.body.velocity.x = Spider.SPEED;
};

// Spider speed constant
const Spider = { SPEED: 100 };
```- スパイダーには、`crawl` (ループ) と `die` (死亡時に 1 回再生) の 2 つのアニメーションがあります。
- `velocity.x = 100` は、スパイダーを毎秒 100 ピクセルで右に移動させます。

### クモを壁に跳ね返らせる

衝突処理を追加して、スパイダーが目に見えない壁やプラットフォームの端にぶつかったときに方向を逆転させるようにします。```javascript
// In PlayState._handleCollisions:
PlayState._handleCollisions = function () {
    this.game.physics.arcade.collide(this.hero, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.enemyWalls);

    this.game.physics.arcade.overlap(
        this.hero, this.coins, this._onHeroVsCoin, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.spiders, this._onHeroVsEnemy, null, this
    );
};
```壁に衝突したときにスパイダーの方向を反転させるには、フレームごとに速度を確認して反転させます。```javascript
// In PlayState.update, after collision handling, update spider directions:
PlayState.update = function () {
    this._handleCollisions();
    this._handleInput();

    // Update spider facing direction based on velocity
    this.spiders.forEach(function (spider) {
        if (spider.body.touching.right || spider.body.blocked.right) {
            spider.body.velocity.x = -Spider.SPEED; // Turn left
        } else if (spider.body.touching.left || spider.body.blocked.left) {
            spider.body.velocity.x = Spider.SPEED; // Turn right
        }
    }, this);
};
```- クモは右側の壁に触れると反転して左に移動し、その逆も同様です。
- `body.touching` は、衝突解決後に Phaser によって設定されます。

---

## 死

敵に触れたときのヒーローの死と、敵を倒すためのストンプメカニズムを実装します。

### ヒーロー vs 敵: ストンプ・オア・ダイ

ヒーローがクモと重なったときに、ヒーローが落ちている (踏みつけている) かどうかを確認します。```javascript
PlayState._onHeroVsEnemy = function (hero, enemy) {
    if (hero.body.velocity.y > 0) {
        // Hero is falling -> stomp the enemy
        enemy.body.velocity.x = 0; // Stop enemy movement
        enemy.body.enable = false; // Disable enemy physics

        // Play die animation then remove the enemy
        enemy.animations.play('die');
        enemy.events.onAnimationComplete.addOnce(function () {
            enemy.kill();
        });

        // Bounce the hero up after stomping
        hero.body.velocity.y = -JUMP_SPEED / 2;

        this.sfx.stomp.play();
    } else {
        // Hero touched enemy from side or below -> die
        this._killHero();
    }
};

PlayState._killHero = function () {
    this.hero.kill();
    // Restart the level after a short delay
    this.game.time.events.add(500, function () {
        this.game.state.restart(true, false, { level: this.level });
    }, this);
};
```- `hero.body.velocity.y > 0` の場合、主人公は下方向に移動 (落下) しており、ストンプを示します。
- ストンプ時: 敵は停止し、死亡アニメーションが再生され、排除されます。主人公は飛び起きます。
- 主人公が倒れていない場合、主人公は死亡します。 `this.hero.kill()` は、ゲームからヒーローを削除します。
- 500 ミリ秒後、状態全体が再起動され、レベルが効果的にリロードされます。

### ストンプサウンドを追加する```javascript
// In PlayState.create, add to sfx:
this.sfx = {
    jump: this.game.add.audio('sfx:jump'),
    coin: this.game.add.audio('sfx:coin'),
    stomp: this.game.add.audio('sfx:stomp')
};
```### ヒーローの死亡アニメーションを追加する

死亡時にヒーローが点滅して画面から落ちるようにします。```javascript
PlayState._killHero = function () {
    this.hero.alive = false;

    // Play a "dying" visual: the hero jumps up and falls off screen
    this.hero.body.velocity.y = -JUMP_SPEED / 2;
    this.hero.body.velocity.x = 0;
    this.hero.body.allowGravity = true;

    // Disable collisions so the hero falls through platforms
    this.hero.body.collideWorldBounds = false;

    // Restart after a delay
    this.game.time.events.add(1000, function () {
        this.game.state.restart(true, false, { level: this.level });
    }, this);
};
```### デッド時の入力の保護

死後に入力がヒーローを制御できないようにします。```javascript
PlayState._handleInput = function () {
    if (!this.hero.alive) { return; }

    if (this.keys.left.isDown) {
        this.hero.body.velocity.x = -SPEED;
    } else if (this.keys.right.isDown) {
        this.hero.body.velocity.x = SPEED;
    } else {
        this.hero.body.velocity.x = 0;
    }

    if (this.keys.up.isDown) {
        this._jump();
    }
};
```- `_killHero` の `this.hero.alive` が `false` に設定されているため、死亡後は入力が無視され、主人公は自然に画面から落ちます。

---

## スコアボード

集めたコインの枚数をビットマップフォントで画面上に表示します。

### ビットマップフォントのロード```javascript
// In PlayState.preload:
this.game.load.image('font:numbers', 'images/numbers.png');
this.game.load.image('icon:coin', 'images/coin_icon.png');
```### HUD の作成

コインのアイコンとカウントを表示する固定 HUD (ヘッドアップ ディスプレイ) を作成します。```javascript
PlayState._createHud = function () {
    let coinIcon = this.game.make.image(0, 0, 'icon:coin');

    // Create a dynamic text label for the coin count
    this.hud = this.game.add.group();

    // Use a retroFont or a regular text object for the score
    let scoreStyle = {
        font: '30px monospace',
        fill: '#fff'
    };
    this.coinFont = this.game.add.text(
        coinIcon.width + 7, 0, 'x0', scoreStyle
    );

    this.hud.add(coinIcon);
    this.hud.add(this.coinFont);

    this.hud.position.set(10, 10);
    this.hud.fixedToCamera = true;
};
```あるいは、Phaser の `RetroFont` を使用してピクセル アート番号をレンダリングすることもできます。```javascript
PlayState._createHud = function () {
    // Bitmap-based number rendering using RetroFont
    this.coinFont = this.game.add.retroFont(
        'font:numbers', 20, 26,
        '0123456789X ', 6
    );

    let coinIcon = this.game.make.image(0, 0, 'icon:coin');

    let coinScoreImg = this.game.make.image(
        coinIcon.x + coinIcon.width + 7, 0, this.coinFont
    );

    this.hud = this.game.add.group();
    this.hud.add(coinIcon);
    this.hud.add(coinScoreImg);
    this.hud.position.set(10, 10);
    this.hud.fixedToCamera = true;
};
```- `retroFont` は、文字グリフを含むスプライトシートからビットマップ フォントを作成します。
- パラメータ: 画像キー、文字幅、文字高さ、文字セット文字列、行ごとの文字数。

### create での createHud の呼び出し```javascript
PlayState.create = function () {
    this.game.add.image(0, 0, 'background');

    this._loadLevel(this.game.cache.getJSON('level:1'));

    // Create the HUD
    this._createHud();
};
```Error 504 (Server Error)!!1504.That’s an error.There was an error. Please try again later.That’s all we know.```javascript
PlayState._onHeroVsCoin = function (hero, coin) {
    this.sfx.coin.play();
    coin.kill();
    this.coinPickupCount++;

    // Update the HUD
    this.coinFont.text = 'x' + this.coinPickupCount;
};
```---

## 主人公のアニメーション

静的なヒーロー画像をスプライトシートに置き換え、アイドル (停止)、走行、ジャンプ、落下などのさまざまな状態のアニメーションを追加します。

### ヒーロー スプライトシートのロード

単一の画像ロードを `preload` のスプライトシートに置き換えます。```javascript
// Replace: this.game.load.image('hero', 'images/hero_stopped.png');
// With:
this.game.load.spritesheet('hero', 'images/hero.png', 36, 42);
```- ヒーローのスプライトシートは、フレームごとに幅 36 ピクセル、高さ 42 ピクセルです。
- フレームには、アイドル、ウォーク サイクル、ジャンプ、および落下のポーズが含まれます。

### アニメーションの定義

`_spawnCharacters` で、ヒーロー スプライトの作成後にアニメーション定義を追加します。```javascript
PlayState._spawnCharacters = function (data) {
    this.hero = this.game.add.sprite(data.hero.x, data.hero.y, 'hero');
    this.hero.anchor.set(0.5, 1);
    this.game.physics.enable(this.hero);

    // Define animations
    this.hero.animations.add('stop', [0]);               // Single frame: idle
    this.hero.animations.add('run', [1, 2], 8, true);    // 2 frames at 8fps, looping
    this.hero.animations.add('jump', [3]);                // Single frame: jumping up
    this.hero.animations.add('fall', [4]);                // Single frame: falling down
};
```- `animations.add(name, frames, fps, loop)` は、指定された名前でアニメーションを登録します。
- `stop`、`jump`、`fall` などの単一フレーム アニメーションは、静的なポーズを効果的に設定します。
- `run` アニメーションは 8fps でフレーム 1 とフレーム 2 を交互に繰り返します。

### 正しいアニメーションの再生

ヒーローの現在の状態に基づいて適切なアニメーションを決定して再生するメソッドを追加します。```javascript
PlayState._getAnimationName = function () {
    let name = 'stop'; // Default: standing still

    if (!this.hero.alive) {
        name = 'stop'; // Use idle frame when dead
    } else if (this.hero.body.velocity.y < 0) {
        name = 'jump'; // Moving upward
    } else if (this.hero.body.velocity.y > 0 && !this.hero.body.touching.down) {
        name = 'fall'; // Moving downward and not on ground
    } else if (this.hero.body.velocity.x !== 0 && this.hero.body.touching.down) {
        name = 'run';  // Moving horizontally on the ground
    }

    return name;
};
```### 方向に基づいてスプライトを反転する

ヒーローの向いている方向を更新し、`update` でアニメーションを再生します。```javascript
PlayState.update = function () {
    this._handleCollisions();
    this._handleInput();

    // Flip sprite based on movement direction
    if (this.hero.body.velocity.x < 0) {
        this.hero.scale.x = -1; // Face left
    } else if (this.hero.body.velocity.x > 0) {
        this.hero.scale.x = 1;  // Face right
    }

    // Play the appropriate animation
    this.hero.animations.play(this._getAnimationName());

    // Update spider directions
    this.spiders.forEach(function (spider) {
        if (spider.body.touching.right || spider.body.blocked.right) {
            spider.body.velocity.x = -Spider.SPEED;
        } else if (spider.body.touching.left || spider.body.blocked.left) {
            spider.body.velocity.x = Spider.SPEED;
        }
    }, this);
};
```- `this.hero.scale.x = -1` は、スプライトを水平方向に反転して左向きにします。 `1` にすると右向きになります。アンカーが `(0.5, 1)` にあるため、反転は自然に見えます。
- `animations.play()` は、名前が変更された場合にのみアニメーションを再開するため、フレームごとに呼び出すのが安全で効率的です。

---

## 勝利条件

ドアと鍵の仕組みを追加します。レベルを完了するには、主人公は鍵を収集し、ドアに到達する必要があります。

### ドアとキーアセットのロード```javascript
// In PlayState.preload:
this.game.load.spritesheet('door', 'images/door.png', 42, 66);
this.game.load.spritesheet('key', 'images/key.png', 20, 22);  // Key bobbing animation
this.game.load.image('icon:key', 'images/key_icon.png');

this.game.load.audio('sfx:key', 'audio/sfx/key.wav');
this.game.load.audio('sfx:door', 'audio/sfx/door.wav');
```### ドアと鍵の生成

`_loadLevel` と `_spawnCharacters` を更新します。```javascript
PlayState._loadLevel = function (data) {
    this.platforms = this.game.add.group();
    this.coins = this.game.add.group();
    this.spiders = this.game.add.group();
    this.enemyWalls = this.game.add.group();
    this.bgDecoration = this.game.add.group();

    // Must spawn decorations first (background layer)
    // Spawn door before hero so it renders behind the hero
    data.platforms.forEach(this._spawnPlatform, this);
    data.coins.forEach(this._spawnCoin, this);
    data.spiders.forEach(this._spawnSpider, this);

    this._spawnDoor(data.door.x, data.door.y);
    this._spawnKey(data.key.x, data.key.y);
    this._spawnCharacters({ hero: data.hero });

    this.enemyWalls.visible = false;

    this.coinPickupCount = 0;
    this.hasKey = false;
};

PlayState._spawnDoor = function (x, y) {
    this.door = this.bgDecoration.create(x, y, 'door');
    this.door.anchor.setTo(0.5, 1);

    this.game.physics.enable(this.door);
    this.door.body.allowGravity = false;
};

PlayState._spawnKey = function (x, y) {
    this.key = this.bgDecoration.create(x, y, 'key');
    this.key.anchor.set(0.5, 0.5);

    this.game.physics.enable(this.key);
    this.key.body.allowGravity = false;

    // Add a bobbing up-and-down tween to the key
    this.key.y -= 3;
    this.game.add.tween(this.key)
        .to({ y: this.key.y + 6 }, 800, Phaser.Easing.Sinusoidal.InOut)
        .yoyo(true)
        .loop()
        .start();
};
```- ドアは背景装飾グループに配置されているため、主人公の後ろにレンダリングされます。
- キーには正弦波の上下トゥイーンがあり、800 ミリ秒にわたって 6 ピクセル上下に移動し、永久にループします。

### 鍵を集めてドアを開ける

鍵とドアのサウンドエフェクトを sfx オブジェクトに追加します。```javascript
// In PlayState.create sfx:
this.sfx = {
    jump: this.game.add.audio('sfx:jump'),
    coin: this.game.add.audio('sfx:coin'),
    stomp: this.game.add.audio('sfx:stomp'),
    key: this.game.add.audio('sfx:key'),
    door: this.game.add.audio('sfx:door')
};
````_handleCollisions` に鍵とドアの重複検出を追加します。```javascript
PlayState._handleCollisions = function () {
    this.game.physics.arcade.collide(this.hero, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.enemyWalls);

    this.game.physics.arcade.overlap(
        this.hero, this.coins, this._onHeroVsCoin, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.spiders, this._onHeroVsEnemy, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.key, this._onHeroVsKey, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.door, this._onHeroVsDoor,
        // Only trigger if the hero has the key
        function (hero, door) {
            return this.hasKey && hero.body.touching.down;
        }, this
    );
};
```- ドアのオーバーラップには **プロセス コールバック** (4 番目の引数) があり、`this.hasKey` が true でヒーローが何かの上に立っている場合にのみオーバーラップ コールバックをトリガーします。これにより、主人公が転落したり、鍵を持たずにドアに入るのを防ぎます。

### 鍵とドアのコールバック```javascript
PlayState._onHeroVsKey = function (hero, key) {
    this.sfx.key.play();
    key.kill();
    this.hasKey = true;
};

PlayState._onHeroVsDoor = function (hero, door) {
    this.sfx.door.play();

    // Freeze the hero and play the door opening animation
    hero.body.velocity.x = 0;
    hero.body.velocity.y = 0;
    hero.body.enable = false;

    // Play door open animation (transition from closed to open frame)
    door.frame = 1; // Switch to "open" frame

    // Advance to the next level after a short delay
    this.game.time.events.add(500, this._goToNextLevel, this);
};

PlayState._goToNextLevel = function () {
    this.camera.fade('#000');
    this.camera.onFadeComplete.addOnce(function () {
        this.game.state.restart(true, false, {
            level: this.level + 1
        });
    }, this);
};
```- 主人公がキーに触れると、キーが削除され、`hasKey` が `true` に設定されます。
- 主人公が (鍵を持って) ドアに到達すると、主人公はフリーズし、ドアが開き、遅れてゲームが次のレベルに移行します。
- `camera.fade()` は、洗練されたレベル スイッチのフェードから黒へのトランジションを作成します。

### HUD に鍵アイコンを表示する

`_createHud` を更新して、ヒーローがキーを収集したかどうかを表示します。```javascript
PlayState._createHud = function () {
    this.keyIcon = this.game.make.image(0, 19, 'icon:key');
    this.keyIcon.anchor.set(0, 0.5);

    // ... existing coin HUD code ...

    this.hud.add(this.keyIcon);
    this.hud.add(coinIcon);
    this.hud.add(coinScoreImg);
    this.hud.position.set(10, 10);
    this.hud.fixedToCamera = true;
};
````update` の各フレームでキー アイコンの外観を更新します。```javascript
// In PlayState.update, add:
this.keyIcon.frame = this.hasKey ? 1 : 0;
```- フレーム 0 にはグレー表示された鍵アイコンが表示されます。フレーム 1 は収集された鍵のアイコンを示しています。

---

## レベルの切り替え

レベル インデックスに基づいて異なる JSON ファイルをロードすることで、複数のレベルをサポートします。

### init を介してレベル番号を渡す

レベル パラメータを受け入れるように `init` を変更します。```javascript
PlayState.init = function (data) {
    this.game.renderer.renderSession.roundPixels = true;

    this.keys = this.game.input.keyboard.addKeys({
        left: Phaser.KeyCode.LEFT,
        right: Phaser.KeyCode.RIGHT,
        up: Phaser.KeyCode.UP
    });

    this.game.physics.startSystem(Phaser.Physics.ARCADE);
    this.game.physics.arcade.gravity.y = 1200;

    // Store the current level number (default to 0)
    this.level = (data.level || 0) % LEVEL_COUNT;
};

const LEVEL_COUNT = 2; // Total number of levels
```- `data` は、`game.state.start()` または `game.state.restart()` から渡されたオブジェクトです。
- モジュロ演算 (`% LEVEL_COUNT`) は、最後のレベルの後にレベル 0 にラップアラウンドし、レベルの無限ループを作成します。

### レベルデータを動的にロードする

`preload` を更新して、`this.level` に基づいて正しいレベルをロードします。```javascript
PlayState.preload = function () {
    this.game.load.image('background', 'images/background.png');

    // Load the current level's JSON data
    this.game.load.json('level:0', 'data/level00.json');
    this.game.load.json('level:1', 'data/level01.json');

    // ... load all other assets ...
};
```正しいレベル データを使用するように `create` を更新します。```javascript
PlayState.create = function () {
    this.sfx = {
        jump: this.game.add.audio('sfx:jump'),
        coin: this.game.add.audio('sfx:coin'),
        stomp: this.game.add.audio('sfx:stomp'),
        key: this.game.add.audio('sfx:key'),
        door: this.game.add.audio('sfx:door')
    };

    this.game.add.image(0, 0, 'background');

    // Load level data based on current level number
    this._loadLevel(this.game.cache.getJSON('level:' + this.level));

    this._createHud();
};
```### ゲームをレベル 0 から開始する

初期状態を更新してレベル 0 を通過します。```javascript
window.onload = function () {
    let game = new Phaser.Game(960, 600, Phaser.AUTO, 'game');
    game.state.add('play', PlayState);
    game.state.start('play', true, false, { level: 0 });
};
```- 3 番目と 4 番目の `start` 引数は、ワールド/キャッシュのクリアを制御します。 `true, false` は、再起動の間にキャッシュを保持します (そのため、アセットを再ロードする必要はありません) が、ワールドをクリアします。
- `{ level: 0 }` は `data` パラメータとして `init` に渡されます。

### レベル移行の流れ

完全なレベル フローは次のとおりです。

1. ヒーローがキーを収集 -> `hasKey = true`
2. ヒーローがドアに到達 -> `_onHeroVsDoor` が起動
3. カメラが黒にフェードアウト -> `_goToNextLevel` が起動
4. `{ level: this.level + 1 }` でステートが再開されます。
5. `init` は新しいレベル番号を受け取ります
6. 正しいレベルの JSON がロードされ、ゲームが続行されます。

---

## 前進する

おめでとうございます -- 完全な 2D プラットフォーマーが構築されました。ゲームをさらに拡張するためのアイデアは次のとおりです。

### 提案された改善点

- **モバイル/タッチ コントロール:** タッチ対応デバイスの場合は、`game.input.onDown` を使用してオンスクリーン ボタンまたはスワイプ ジェスチャを追加します。
- **さらなるレベル:** 新しいプラットフォーム レイアウト、コインの配置、敵の構成を含む追加の JSON レベル ファイルを作成します。
- **メニュー画面:** `PlayState`を入力する前に、タイトル画面とスタートボタンを含む`MenuState`を追加します。
- **ゲームオーバー画面:** すぐに再開する代わりに、スコアを含む「ゲームオーバー」画面を表示します。
- **ライフシステム:** 即座に再起動する代わりに、ヒーローに複数のライフを与えます。
- **パワーアップ:** 速度ブースト、ダブルジャンプ、無敵などのアイテムを追加します。
- **移動プラットフォーム:** トゥイーンを使用してパスに沿って移動するプラットフォームを作成します。
- **さまざまな敵のタイプ:** 飛行する敵、発射物を発射する敵、またはさまざまな動きパターンを持つ敵を追加します。
- **視差スクロール:** 奥行きを持たせるために異なる速度でスクロールする複数の背景レイヤーを追加します。
- **カメラのスクロール:** 画面より広いレベルの場合、`game.camera.follow(this.hero)` を使用してヒーローと一緒にスクロールします。
- **サウンドと音楽:** バックグラウンドミュージックと追加のサウンドエフェクトを追加して、より洗練されたエクスペリエンスを実現します。
- **パーティクル エフェクト:** フェイザーのパーティクル エミッターを使用して、コイン収集の輝き、敵の死亡エフェクト、または着陸時の粉塵を演出します。

### 完全なゲーム ソース リファレンス

以下は、参考のためにすべての手順を組み合わせた完全な `main.js` ファイルです。これは、すべての機能を備えたゲームの最終状態を表します。```javascript
// =============================================================================
// Constants
// =============================================================================

const SPEED = 200;
const JUMP_SPEED = 600;
const LEVEL_COUNT = 2;
const Spider = { SPEED: 100 };

// =============================================================================
// Game State: PlayState
// =============================================================================

PlayState = {};

// -----------------------------------------------------------------------------
// init
// -----------------------------------------------------------------------------

PlayState.init = function (data) {
    this.game.renderer.renderSession.roundPixels = true;

    this.keys = this.game.input.keyboard.addKeys({
        left: Phaser.KeyCode.LEFT,
        right: Phaser.KeyCode.RIGHT,
        up: Phaser.KeyCode.UP
    });

    this.game.physics.startSystem(Phaser.Physics.ARCADE);
    this.game.physics.arcade.gravity.y = 1200;

    this.level = (data.level || 0) % LEVEL_COUNT;
};

// -----------------------------------------------------------------------------
// preload
// -----------------------------------------------------------------------------

PlayState.preload = function () {
    // Background
    this.game.load.image('background', 'images/background.png');

    // Level data
    this.game.load.json('level:0', 'data/level00.json');
    this.game.load.json('level:1', 'data/level01.json');

    // Platform tiles
    this.game.load.image('ground', 'images/ground.png');
    this.game.load.image('grass:8x1', 'images/grass_8x1.png');
    this.game.load.image('grass:6x1', 'images/grass_6x1.png');
    this.game.load.image('grass:4x1', 'images/grass_4x1.png');
    this.game.load.image('grass:2x1', 'images/grass_2x1.png');
    this.game.load.image('grass:1x1', 'images/grass_1x1.png');

    // Characters
    this.game.load.spritesheet('hero', 'images/hero.png', 36, 42);
    this.game.load.spritesheet('spider', 'images/spider.png', 42, 32);
    this.game.load.image('invisible-wall', 'images/invisible_wall.png');

    // Collectibles
    this.game.load.spritesheet('coin', 'images/coin_animated.png', 22, 22);
    this.game.load.spritesheet('key', 'images/key.png', 20, 22);
    this.game.load.spritesheet('door', 'images/door.png', 42, 66);

    // HUD
    this.game.load.image('icon:coin', 'images/coin_icon.png');
    this.game.load.image('icon:key', 'images/key_icon.png');
    this.game.load.image('font:numbers', 'images/numbers.png');

    // Audio
    this.game.load.audio('sfx:jump', 'audio/sfx/jump.wav');
    this.game.load.audio('sfx:coin', 'audio/sfx/coin.wav');
    this.game.load.audio('sfx:stomp', 'audio/sfx/stomp.wav');
    this.game.load.audio('sfx:key', 'audio/sfx/key.wav');
    this.game.load.audio('sfx:door', 'audio/sfx/door.wav');
};

// -----------------------------------------------------------------------------
// create
// -----------------------------------------------------------------------------

PlayState.create = function () {
    // Sound effects
    this.sfx = {
        jump: this.game.add.audio('sfx:jump'),
        coin: this.game.add.audio('sfx:coin'),
        stomp: this.game.add.audio('sfx:stomp'),
        key: this.game.add.audio('sfx:key'),
        door: this.game.add.audio('sfx:door')
    };

    // Background
    this.game.add.image(0, 0, 'background');

    // Load level
    this._loadLevel(this.game.cache.getJSON('level:' + this.level));

    // HUD
    this._createHud();
};

// -----------------------------------------------------------------------------
// update
// -----------------------------------------------------------------------------

PlayState.update = function () {
    this._handleCollisions();
    this._handleInput();

    // Update hero sprite direction and animation
    if (this.hero.body.velocity.x < 0) {
        this.hero.scale.x = -1;
    } else if (this.hero.body.velocity.x > 0) {
        this.hero.scale.x = 1;
    }
    this.hero.animations.play(this._getAnimationName());

    // Update spider directions when hitting walls
    this.spiders.forEach(function (spider) {
        if (spider.body.touching.right || spider.body.blocked.right) {
            spider.body.velocity.x = -Spider.SPEED;
        } else if (spider.body.touching.left || spider.body.blocked.left) {
            spider.body.velocity.x = Spider.SPEED;
        }
    }, this);

    // Update key icon in HUD
    this.keyIcon.frame = this.hasKey ? 1 : 0;
};

// -----------------------------------------------------------------------------
// Level Loading
// -----------------------------------------------------------------------------

PlayState._loadLevel = function (data) {
    // Create groups (order matters for rendering layers)
    this.bgDecoration = this.game.add.group();
    this.platforms = this.game.add.group();
    this.coins = this.game.add.group();
    this.spiders = this.game.add.group();
    this.enemyWalls = this.game.add.group();

    // Spawn entities from level data
    data.platforms.forEach(this._spawnPlatform, this);
    data.coins.forEach(this._spawnCoin, this);
    data.spiders.forEach(this._spawnSpider, this);

    this._spawnDoor(data.door.x, data.door.y);
    this._spawnKey(data.key.x, data.key.y);
    this._spawnCharacters({ hero: data.hero });

    // Hide invisible walls
    this.enemyWalls.visible = false;

    // Initialize game state
    this.coinPickupCount = 0;
    this.hasKey = false;
};

// -----------------------------------------------------------------------------
// Spawn Methods
// -----------------------------------------------------------------------------

PlayState._spawnPlatform = function (platform) {
    let sprite = this.platforms.create(platform.x, platform.y, platform.image);
    this.game.physics.enable(sprite);
    sprite.body.allowGravity = false;
    sprite.body.immovable = true;

    // Add invisible walls at both edges for enemy AI
    this._spawnEnemyWall(platform.x, platform.y, 'left');
    this._spawnEnemyWall(platform.x + sprite.width, platform.y, 'right');
};

PlayState._spawnEnemyWall = function (x, y, side) {
    let sprite = this.enemyWalls.create(x, y, 'invisible-wall');
    sprite.anchor.set(side === 'left' ? 1 : 0, 1);
    this.game.physics.enable(sprite);
    sprite.body.immovable = true;
    sprite.body.allowGravity = false;
};

PlayState._spawnCharacters = function (data) {
    this.hero = this.game.add.sprite(data.hero.x, data.hero.y, 'hero');
    this.hero.anchor.set(0.5, 1);
    this.game.physics.enable(this.hero);
    this.hero.body.collideWorldBounds = true;

    // Hero animations
    this.hero.animations.add('stop', [0]);
    this.hero.animations.add('run', [1, 2], 8, true);
    this.hero.animations.add('jump', [3]);
    this.hero.animations.add('fall', [4]);
};

PlayState._spawnCoin = function (coin) {
    let sprite = this.coins.create(coin.x, coin.y, 'coin');
    sprite.anchor.set(0.5, 0.5);
    this.game.physics.enable(sprite);
    sprite.body.allowGravity = false;

    sprite.animations.add('rotate', [0, 1, 2, 1], 6, true);
    sprite.animations.play('rotate');
};

PlayState._spawnSpider = function (spider) {
    let sprite = this.spiders.create(spider.x, spider.y, 'spider');
    sprite.anchor.set(0.5, 1);
    this.game.physics.enable(sprite);

    sprite.animations.add('crawl', [0, 1, 2], 8, true);
    sprite.animations.add('die', [0, 4, 0, 4, 0, 4, 3, 3, 3, 3, 3, 3], 12);
    sprite.animations.play('crawl');

    sprite.body.velocity.x = Spider.SPEED;
};

PlayState._spawnDoor = function (x, y) {
    this.door = this.bgDecoration.create(x, y, 'door');
    this.door.anchor.setTo(0.5, 1);
    this.game.physics.enable(this.door);
    this.door.body.allowGravity = false;
};

PlayState._spawnKey = function (x, y) {
    this.key = this.bgDecoration.create(x, y, 'key');
    this.key.anchor.set(0.5, 0.5);
    this.game.physics.enable(this.key);
    this.key.body.allowGravity = false;

    // Bobbing tween
    this.key.y -= 3;
    this.game.add.tween(this.key)
        .to({ y: this.key.y + 6 }, 800, Phaser.Easing.Sinusoidal.InOut)
        .yoyo(true)
        .loop()
        .start();
};

// -----------------------------------------------------------------------------
// Input
// -----------------------------------------------------------------------------

PlayState._handleInput = function () {
    if (!this.hero.alive) { return; }

    if (this.keys.left.isDown) {
        this.hero.body.velocity.x = -SPEED;
    } else if (this.keys.right.isDown) {
        this.hero.body.velocity.x = SPEED;
    } else {
        this.hero.body.velocity.x = 0;
    }

    if (this.keys.up.isDown) {
        this._jump();
    }
};

PlayState._jump = function () {
    let canJump = this.hero.body.touching.down;
    if (canJump) {
        this.hero.body.velocity.y = -JUMP_SPEED;
        this.sfx.jump.play();
    }
    return canJump;
};

// -----------------------------------------------------------------------------
// Collisions
// -----------------------------------------------------------------------------

PlayState._handleCollisions = function () {
    // Physical collisions
    this.game.physics.arcade.collide(this.hero, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.platforms);
    this.game.physics.arcade.collide(this.spiders, this.enemyWalls);

    // Overlap detection (no physical push)
    this.game.physics.arcade.overlap(
        this.hero, this.coins, this._onHeroVsCoin, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.spiders, this._onHeroVsEnemy, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.key, this._onHeroVsKey, null, this
    );
    this.game.physics.arcade.overlap(
        this.hero, this.door, this._onHeroVsDoor,
        function (hero, door) {
            return this.hasKey && hero.body.touching.down;
        }, this
    );
};

// -----------------------------------------------------------------------------
// Collision Callbacks
// -----------------------------------------------------------------------------

PlayState._onHeroVsCoin = function (hero, coin) {
    this.sfx.coin.play();
    coin.kill();
    this.coinPickupCount++;
    this.coinFont.text = 'x' + this.coinPickupCount;
};

PlayState._onHeroVsEnemy = function (hero, enemy) {
    if (hero.body.velocity.y > 0) {
        // Stomp: hero is falling onto the enemy
        enemy.body.velocity.x = 0;
        enemy.body.enable = false;
        enemy.animations.play('die');
        enemy.events.onAnimationComplete.addOnce(function () {
            enemy.kill();
        });
        hero.body.velocity.y = -JUMP_SPEED / 2;
        this.sfx.stomp.play();
    } else {
        // Hero dies
        this._killHero();
    }
};

PlayState._onHeroVsKey = function (hero, key) {
    this.sfx.key.play();
    key.kill();
    this.hasKey = true;
};

PlayState._onHeroVsDoor = function (hero, door) {
    this.sfx.door.play();
    hero.body.velocity.x = 0;
    hero.body.velocity.y = 0;
    hero.body.enable = false;

    door.frame = 1; // Open door

    this.game.time.events.add(500, this._goToNextLevel, this);
};

// -----------------------------------------------------------------------------
// Death and Level Transitions
// -----------------------------------------------------------------------------

PlayState._killHero = function () {
    this.hero.alive = false;
    this.hero.body.velocity.y = -JUMP_SPEED / 2;
    this.hero.body.velocity.x = 0;
    this.hero.body.allowGravity = true;
    this.hero.body.collideWorldBounds = false;

    this.game.time.events.add(1000, function () {
        this.game.state.restart(true, false, { level: this.level });
    }, this);
};

PlayState._goToNextLevel = function () {
    this.camera.fade('#000');
    this.camera.onFadeComplete.addOnce(function () {
        this.game.state.restart(true, false, {
            level: this.level + 1
        });
    }, this);
};

// -----------------------------------------------------------------------------
// Animations
// -----------------------------------------------------------------------------

PlayState._getAnimationName = function () {
    let name = 'stop';

    if (!this.hero.alive) {
        name = 'stop';
    } else if (this.hero.body.velocity.y < 0) {
        name = 'jump';
    } else if (this.hero.body.velocity.y > 0 && !this.hero.body.touching.down) {
        name = 'fall';
    } else if (this.hero.body.velocity.x !== 0 && this.hero.body.touching.down) {
        name = 'run';
    }

    return name;
};

// -----------------------------------------------------------------------------
// HUD
// -----------------------------------------------------------------------------

PlayState._createHud = function () {
    this.keyIcon = this.game.make.image(0, 19, 'icon:key');
    this.keyIcon.anchor.set(0, 0.5);

    let coinIcon = this.game.make.image(
        this.keyIcon.width + 7, 0, 'icon:coin'
    );

    let scoreStyle = { font: '24px monospace', fill: '#fff' };
    this.coinFont = this.game.add.text(
        coinIcon.x + coinIcon.width + 7, 0, 'x0', scoreStyle
    );

    this.hud = this.game.add.group();
    this.hud.add(this.keyIcon);
    this.hud.add(coinIcon);
    this.hud.add(this.coinFont);
    this.hud.position.set(10, 10);
    this.hud.fixedToCamera = true;
};

// =============================================================================
// Entry Point
// =============================================================================

window.onload = function () {
    let game = new Phaser.Game(960, 600, Phaser.AUTO, 'game');
    game.state.add('play', PlayState);
    game.state.start('play', true, false, { level: 0 });
};
```### 重要な概念のまとめ

|コンセプト |フェイザー API |目的 |
|----------|-----------|----------|
|ゲームインスタンス | `new Phaser.Game(w, h, renderer, container)` |ゲームのキャンバスとエンジンを作成します |
|ゲームの状態 | `game.state.add()` / `game.state.start()` |コードを init/preload/create/update ライフサイクルに整理します |
|画像をロード中 | `game.load.image(key, path)` |静的画像アセットをロードします |
|スプライトシートのロード | `game.load.spritesheet(key, path, fw, fh)` |アニメーション化されたスプライトシートをロードします |
| JSON の読み込み中 | `game.load.json(key, path)` | JSON データをロードします (レベル定義) |
|オーディオをロードしています | `game.load.audio(key, path)` |効果音をロードします |
|スプライトグループ | `game.add.group()` |関連するスプライトのコンテナ。バッチ衝突検出を有効にする |
|物理体 | `game.physics.enable(sprite)` | Arcade Physics ボディをスプライトに追加します。
|重力 | `game.physics.arcade.gravity.y` |世界的な下降加速 |
|衝突 | `arcade.collide(a, b)` |物理的な衝突の解決 (スプライトが互いに押し合う) |
|オーバーラップ | `arcade.overlap(a, b, callback)` |物理的に押さずに検出（ピックアップ用） |
|速度 | `sprite.body.velocity.x/y` |移動速度 (ピクセル/秒) |
|不動 | `sprite.body.immovable = true` |スプライトが衝突によって押し出されるのを防ぎます |
|アニメーション | `sprite.animations.add(name, frames, fps, loop)` |フレームアニメーションを定義します |
|トゥイーン | `game.add.tween(target).to(props, duration, easing)` |スムーズなプロパティ アニメーション |
|キーボード入力 | `game.input.keyboard.addKeys({...})` |特定のキーボード キーをキャプチャします。
|カメラ | `this.camera.fade()` |画面遷移効果 |
|アンカー | `sprite.anchor.set(x, y)` |位置決めと回転の原点を設定します |
|スプライト反転 | `sprite.scale.x = -1` |スプライトを水平方向にミラーリングします |