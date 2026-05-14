# パドルゲームテンプレート (2D ブレイクアウト)

純粋な JavaScript と HTML5 Canvas API を使用して 2D ブレークアウト ゲームを構築するための完全なステップバイステップ ガイド。このテンプレートは、キャンバスのセットアップからライフ システムと洗練されたゲーム ループの実装に至るまで、開発のあらゆる段階を順を追って説明します。

**構築するもの:** プレイヤーがパドルを操作してボールを跳ね返し、レンガのフィールドを破壊する古典的なブレイクアウト/パドル ゲーム。スコア追跡、勝敗条件、キーボードとマウスのコントロール、ライフ システムが備わっています。

**前提条件:** 基本から中級の JavaScript の知識と HTML に精通していること。

**出典:** [MDN 2D ブレイクアウト ゲーム チュートリアル](https://developer.mozilla.org/en-US/docs/Games/Tutorials/2D_Breakout_game_pure_JavaScript) に基づいています。

---

## ステップ 1: キャンバスを作成し、その上に描画します

最初のステップは、`<canvas>` 要素を使用して HTML ドキュメントを設定し、2D レンダリング コンテキストを使用して基本的な形状を描画する方法を学習することです。

### HTML 構造

Canvas 要素が埋め込まれたベース HTML ファイルを作成します。```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>Gamedev Canvas Workshop</title>
    <style>
      * {
        padding: 0;
        margin: 0;
      }
      canvas {
        background: #eeeeee;
        display: block;
        margin: 0 auto;
      }
    </style>
  </head>
  <body>
    <canvas id="myCanvas" width="480" height="320"></canvas>

    <script>
      // JavaScript code goes here
    </script>
  </body>
</html>
```### キャンバス参照と 2D コンテキストの取得

Canvas 要素は描画面を提供します。 2D レンダリング コンテキストを通じてアクセスします。```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
```- `canvas` は、HTML `<canvas>` 要素への参照です。
- `ctx` は、すべての描画メソッドを提供する 2D レンダリング コンテキスト オブジェクトです。

### 塗りつぶされた長方形の描画

`rect()` を使用して四角形を定義し、 `fill()` を使用してそれをレンダリングします。```javascript
ctx.beginPath();
ctx.rect(20, 40, 50, 50);
ctx.fillStyle = "red";
ctx.fill();
ctx.closePath();
```- 最初の 2 つのパラメータ (`20, 40`) は、左上隅の座標を設定します。
- 2 番目の 2 つのパラメータ (`50, 50`) は、幅と高さを設定します。
- `fillStyle` は塗りつぶしの色を設定します。
- `fill()` は、形状を塗りつぶしとしてレンダリングします。

### 円を描く

`arc()` を使用して円を定義します。```javascript
ctx.beginPath();
ctx.arc(240, 160, 20, 0, Math.PI * 2, false);
ctx.fillStyle = "green";
ctx.fill();
ctx.closePath();
```- `240, 160` -- 中心の x、y 座標。
- `20` -- 半径。
- `0` -- 開始角度 (ラジアン)。
- `Math.PI * 2` -- 終了角度 (全円)。
- `false` -- 時計回りに描画します。

### 線付き長方形の描画 (輪郭のみ)

アウトラインには `fill()` の代わりに `stroke()` を使用し、アウトラインの色には `strokeStyle` を使用します。```javascript
ctx.beginPath();
ctx.rect(160, 10, 100, 40);
ctx.strokeStyle = "rgb(0 0 255 / 50%)";
ctx.stroke();
ctx.closePath();
```- 50% のアルファ透明度を持つ RGB カラーを使用します。
- `stroke()` は輪郭のみを描画し、塗りつぶしは描画しません。

### 主要なメソッドのリファレンス

|方法 |目的 |
|--------|--------|
| `beginPath()` |新しい描画パスを開始する |
| `closePath()` |現在のパスを閉じる |
| `rect(x, y, width, height)` |長方形を定義する |
| `arc(x, y, radius, startAngle, endAngle, counterclockwise)` |円または円弧を定義する |
| `fillStyle` |塗りつぶしの色を設定する |
| `fill()` |塗りつぶし色 | で形状を塗りつぶします。
| `strokeStyle` |ストローク（輪郭）の色を設定 |
| `stroke()` |図形の輪郭を描く |

### ステップ 1 の完全なコード```html
<canvas id="myCanvas" width="480" height="320"></canvas>

<style>
  * { padding: 0; margin: 0; }
  canvas { background: #eeeeee; display: block; margin: 0 auto; }
</style>

<script>
  const canvas = document.getElementById("myCanvas");
  const ctx = canvas.getContext("2d");

  // Filled red square
  ctx.beginPath();
  ctx.rect(20, 40, 50, 50);
  ctx.fillStyle = "red";
  ctx.fill();
  ctx.closePath();

  // Filled green circle
  ctx.beginPath();
  ctx.arc(240, 160, 20, 0, Math.PI * 2, false);
  ctx.fillStyle = "green";
  ctx.fill();
  ctx.closePath();

  // Stroked blue rectangle (semi-transparent)
  ctx.beginPath();
  ctx.rect(160, 10, 100, 40);
  ctx.strokeStyle = "rgb(0 0 255 / 50%)";
  ctx.stroke();
  ctx.closePath();
</script>
```---

## ステップ 2: ボールを移動する

次に、各フレームでキャンバスを再描画し、速度変数を使用してボールの位置を更新するゲーム ループを作成して、ボールをアニメーション化します。

### 描画ループの作成

`setInterval` を使用して、繰り返し実行する `draw()` 関数を定義します。```javascript
function draw() {
  // drawing code
}
setInterval(draw, 10);
````setInterval(draw, 10)` calls the `draw` function every 10 milliseconds, creating approximately 100 frames per second.

### ボールを描く

Inside the `draw()` function, draw a ball (circle) at a fixed position:```javascript
ctx.beginPath();
ctx.arc(50, 50, 10, 0, Math.PI * 2);
ctx.fillStyle = "#0095DD";
ctx.fill();
ctx.closePath();
```### 位置変数の追加

ハードコードされた位置の代わりに変数を使用して、フレームごとに更新できるようにします。これらを `draw()` 関数の上に配置します。```javascript
let x = canvas.width / 2;
let y = canvas.height - 30;
```これにより、ボールはキャンバスの下部近くの水平方向の中央から開始されます。

### 速度変数の追加

水平方向 (`dx`) と垂直方向 (`dy`) の移動の速度と方向を定義します。```javascript
let dx = 2;
let dy = -2;
```- `dx = 2` は、フレームごとにボールを 2 ピクセル右に移動します。
- `dy = -2` はボールをフレームごとに 2 ピクセル上に移動します (負の y はキャンバス上で上になります)。

### 各フレームの位置を更新する

`draw()` 関数の最後に位置の更新を追加します。```javascript
x += dx;
y += dy;
```### キャンバスをクリアする

クリアしないとボールに跡が残る。各フレームの先頭に `clearRect()` を追加します。```javascript
ctx.clearRect(0, 0, canvas.width, canvas.height);
```### 別個のdrawBall() 関数へのリファクタリング

コードをクリーンで保守しやすいようにするには、ボール描画ロジックを分離します。```javascript
function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, 10, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}
```### ステップ 2 の完全なコード```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");

let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 2;
let dy = -2;

function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, 10, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBall();
  x += dx;
  y += dy;
}

setInterval(draw, 10);
```**重要な概念:**
- **アニメーション ループ**: `setInterval(draw, 10)` はシーンを継続的に再描画します。
- **位置変数**: `x` および `y` はボールの現在位置を追跡します。
- **速度変数**: `dx` および `dy` はフレームごとの動きを決定します。
- **キャンバスのクリア**: `clearRect()` は、新しいフレームを描画する前に前のフレームを削除します。

---

## ステップ 3: 壁から跳ね返る

衝突検出を追加して、ボールが消えるのではなくキャンバスの端で跳ね返るようにします。

### ボール半径の定義

衝突計算で再利用するために、ボールの半径を名前付き定数に抽出します。```javascript
const ballRadius = 10;
```この変数を使用するには `drawBall()` を更新します。```javascript
function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}
```### 基本的な壁衝突 (半径調整なし)

最も単純なアプローチは、次のボールの位置がキャンバスの境界を越えるかどうかをチェックします。```javascript
// Left and right walls
if (x + dx > canvas.width || x + dx < 0) {
  dx = -dx;
}

// Top and bottom walls
if (y + dy > canvas.height || y + dy < 0) {
  dy = -dy;
}
````dx` または `dy` を反転（-1 を掛ける）すると、ボールの方向が変わります。

### 衝突の改善 (ボールの半径を考慮)

基本バージョンでは、ボールはバウンドする前に壁に半分沈みます。これを修正するには、ボールの半径を考慮します。```javascript
// Left and right walls
if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
  dx = -dx;
}

// Top and bottom walls
if (y + dy > canvas.height - ballRadius || y + dy < ballRadius) {
  dy = -dy;
}
```### 衝突検出条件

|壁 |状態 |アクション |
|------|-----------|----------|
| **左** | `x + dx < ballRadius` | `dx = -dx` |
| **右** | `x + dx > canvas.width - ballRadius` | `dx = -dx` |
| **トップ** | `y + dy < ballRadius` | `dy = -dy` |
| **下** | `y + dy > canvas.height - ballRadius` | `dy = -dy` |

### ステップ 3 の完全なコード```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
const ballRadius = 10;

let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 2;
let dy = -2;

function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBall();

  // Collision detection - left and right walls
  if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
    dx = -dx;
  }

  // Collision detection - top and bottom walls
  if (y + dy > canvas.height - ballRadius || y + dy < ballRadius) {
    dy = -dy;
  }

  x += dx;
  y += dy;
}

setInterval(draw, 10);
```---

## ステップ 4: パドルとキーボードのコントロール

次に、プレーヤーが制御するパドルを画面の下部に追加し、キーボード入力 (左/右矢印キー) を接続します。

### パドル変数の定義```javascript
const paddleHeight = 10;
const paddleWidth = 75;
let paddleX = (canvas.width - paddleWidth) / 2;
```- `paddleHeight` および `paddleWidth` はパドルの寸法を定義します。
- `paddleX` は、水平方向の中央にパドルを開始します。プレイヤーが動かすと変化するため、`let` です。

### パドルを描く

`drawPaddle()` 関数を作成します。パドルはキャンバスの一番下にあります。```javascript
function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}
```Error 500 (Server Error)!!1500.That’s an error.There was an error. Please try again later.That’s all we know.```javascript
let rightPressed = false;
let leftPressed = false;
```### キー押下のイベント リスナー

`keydown` (キーが押された) および `keyup` (キーが放された) のハンドラーを登録します。```javascript
document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);
```### キーハンドラー関数

どのキーが押されたか、または放されたかに基づいてブール値フラグを設定します。```javascript
function keyDownHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = true;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = true;
  }
}

function keyUpHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = false;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = false;
  }
}
````"ArrowRight"` (最新のブラウザ) と `"Right"` (従来の IE/Edge) の両方の互換性がチェックされます。

### パドル移動ロジック (境界チェックあり)

これを `draw()` 関数内に追加して、キャンバスの境界内に保ちながら、キーの状態に基づいてパドルを移動します。```javascript
if (rightPressed) {
  paddleX = Math.min(paddleX + 7, canvas.width - paddleWidth);
} else if (leftPressed) {
  paddleX = Math.max(paddleX - 7, 0);
}
```- パドルはフレームごとに 7 ピクセルを移動します。
- `Math.min` は、パドルが右端を越えるのを防ぎます。
- `Math.max` は、左端を越えることを防ぎます。

### ステップ 4 の完全なコード```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
const ballRadius = 10;

let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 2;
let dy = -2;

const paddleHeight = 10;
const paddleWidth = 75;
let paddleX = (canvas.width - paddleWidth) / 2;

let rightPressed = false;
let leftPressed = false;

document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);

function keyDownHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = true;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = true;
  }
}

function keyUpHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = false;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = false;
  }
}

function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBall();
  drawPaddle();

  if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
    dx = -dx;
  }
  if (y + dy > canvas.height - ballRadius || y + dy < ballRadius) {
    dy = -dy;
  }

  if (rightPressed) {
    paddleX = Math.min(paddleX + 7, canvas.width - paddleWidth);
  } else if (leftPressed) {
    paddleX = Math.max(paddleX - 7, 0);
  }

  x += dx;
  y += dy;
}

setInterval(draw, 10);
```---

## ステップ 5: ゲームオーバー

底壁のバウンドを実際のゲーム ロジックに置き換えます。ボールはパドルから跳ね返るはずですが、外れた場合はゲーム オーバーです。

### 間隔リファレンスの保存

ゲームオーバー時にゲームループを停止するには、インターバル ID を保存します。```javascript
let interval = 0;
```次に、`setInterval` の戻り値を割り当てます。```javascript
interval = setInterval(draw, 10);
```### ゲームオーバーとパドル衝突の実装

底壁衝突チェックを交換します。下端で跳ね返るのではなく、ボールがパドルに当たるか外れるかを確認します。```javascript
if (y + dy < ballRadius) {
  // Ball hits top wall -- bounce
  dy = -dy;
} else if (y + dy > canvas.height - ballRadius) {
  // Ball reaches bottom edge
  if (x > paddleX && x < paddleX + paddleWidth) {
    // Ball hits paddle -- bounce
    dy = -dy;
  } else {
    // Ball missed the paddle -- game over
    alert("GAME OVER");
    document.location.reload();
    clearInterval(interval);
  }
}
```**パドル衝突の仕組み:**
- `x > paddleX` -- ボールはパドルの左端を越えました。
- `x < paddleX + paddleWidth` -- ボールはパドルの右端の前にあります。
- 両方が真の場合、ボールはパドルの上にあるため、バウンドします。
- ボールがパドルに当たらずに底に到達した場合、ゲームは終了します。

### ステップ 5 の完全なコード```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
const ballRadius = 10;

let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 2;
let dy = -2;

const paddleHeight = 10;
const paddleWidth = 75;
let paddleX = (canvas.width - paddleWidth) / 2;

let rightPressed = false;
let leftPressed = false;
let interval = 0;

document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);

function keyDownHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = true;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = true;
  }
}

function keyUpHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = false;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = false;
  }
}

function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBall();
  drawPaddle();

  // Left and right wall collision
  if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
    dx = -dx;
  }

  // Top wall collision
  if (y + dy < ballRadius) {
    dy = -dy;
  } else if (y + dy > canvas.height - ballRadius) {
    // Bottom edge: paddle collision or game over
    if (x > paddleX && x < paddleX + paddleWidth) {
      dy = -dy;
    } else {
      alert("GAME OVER");
      document.location.reload();
      clearInterval(interval);
    }
  }

  // Paddle movement
  if (rightPressed) {
    paddleX = Math.min(paddleX + 7, canvas.width - paddleWidth);
  } else if (leftPressed) {
    paddleX = Math.max(paddleX - 7, 0);
  }

  x += dx;
  y += dy;
}

interval = setInterval(draw, 10);
```---

## ステップ 6: レンガフィールドを構築する

次に、ボールが破壊するレンガのグリッドを作成します。レンガは 2D 配列に保存され、行と列で描画されます。

### ブリック構成変数

ブリック フィールドのレイアウトを制御する定数を定義します。```javascript
const brickRowCount = 3;
const brickColumnCount = 5;
const brickWidth = 75;
const brickHeight = 20;
const brickPadding = 10;
const brickOffsetTop = 30;
const brickOffsetLeft = 30;
```- `brickRowCount` / `brickColumnCount` -- レンガの行数と列数。
- `brickWidth` / `brickHeight` -- 個々のレンガの寸法。
- `brickPadding` -- レンガ間のスペース。
- `brickOffsetTop` / `brickOffsetLeft` -- キャンバスの上端と左端から最初のレンガまでの距離。

### Bricks 2D 配列の作成

ネストされたループを使用して 2D 配列を作成します。各ブリックは、その `x` および `y` の位置 (最初は `0`、描画中に計算されます) を保存します。```javascript
const bricks = [];
for (let c = 0; c < brickColumnCount; c++) {
  bricks[c] = [];
  for (let r = 0; r < brickRowCount; r++) {
    bricks[c][r] = { x: 0, y: 0 };
  }
}
```###drawBricks() 関数

すべてのレンガをループし、その位置を計算して保存し、描画します。```javascript
function drawBricks() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
      const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
      bricks[c][r].x = brickX;
      bricks[c][r].y = brickY;
      ctx.beginPath();
      ctx.rect(brickX, brickY, brickWidth, brickHeight);
      ctx.fillStyle = "#0095DD";
      ctx.fill();
      ctx.closePath();
    }
  }
}
```**位置計算式:**
- `brickX = column * (brickWidth + brickPadding) + brickOffsetLeft`
- `brickY = row * (brickHeight + brickPadding) + brickOffsetTop`

これにより、一貫したパディングとマージンを持つ等間隔のグリッドが作成されます。

### ゲームループでのdrawBricks()の呼び出し

キャンバスをクリアした後、`draw()` 関数の先頭に呼び出しを追加します。```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();
  // ... rest of draw function
}
```### ステップ 6 の完全なコード```javascript
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
const ballRadius = 10;

let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 2;
let dy = -2;

const paddleHeight = 10;
const paddleWidth = 75;
let paddleX = (canvas.width - paddleWidth) / 2;

let rightPressed = false;
let leftPressed = false;
let interval = 0;

const brickRowCount = 3;
const brickColumnCount = 5;
const brickWidth = 75;
const brickHeight = 20;
const brickPadding = 10;
const brickOffsetTop = 30;
const brickOffsetLeft = 30;

const bricks = [];
for (let c = 0; c < brickColumnCount; c++) {
  bricks[c] = [];
  for (let r = 0; r < brickRowCount; r++) {
    bricks[c][r] = { x: 0, y: 0 };
  }
}

document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);

function keyDownHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = true;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = true;
  }
}

function keyUpHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight") {
    rightPressed = false;
  } else if (e.key === "Left" || e.key === "ArrowLeft") {
    leftPressed = false;
  }
}

function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = "#0095DD";
  ctx.fill();
  ctx.closePath();
}

function drawBricks() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
      const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
      bricks[c][r].x = brickX;
      bricks[c][r].y = brickY;
      ctx.beginPath();
      ctx.rect(brickX, brickY, brickWidth, brickHeight);
      ctx.fillStyle = "#0095DD";
      ctx.fill();
      ctx.closePath();
    }
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();

  if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
    dx = -dx;
  }
  if (y + dy < ballRadius) {
    dy = -dy;
  } else if (y + dy > canvas.height - ballRadius) {
    if (x > paddleX && x < paddleX + paddleWidth) {
      dy = -dy;
    } else {
      alert("GAME OVER");
      document.location.reload();
      clearInterval(interval);
    }
  }

  if (rightPressed) {
    paddleX = Math.min(paddleX + 7, canvas.width - paddleWidth);
  } else if (leftPressed) {
    paddleX = Math.max(paddleX - 7, 0);
  }

  x += dx;
  y += dy;
}

interval = setInterval(draw, 10);
```---

## ステップ 7: 衝突検出

画面上のレンガでは、ボールがレンガに当たったときを検出して、レンガを消す必要があります。各ブリックは `status` プロパティを取得します。`1` は表示を意味し、`0` は破棄を意味します。

### Status プロパティをブリックに追加する

`status` フラグを含めるようにブリックの初期化を更新します。```javascript
const bricks = [];
for (let c = 0; c < brickColumnCount; c++) {
  bricks[c] = [];
  for (let r = 0; r < brickRowCount; r++) {
    bricks[c][r] = { x: 0, y: 0, status: 1 };
  }
}
```###collisionDetection() 関数

すべてのレンガをループし、ボールの中心がレンガの境界ボックス内にあるかどうかを確認します。```javascript
function collisionDetection() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      const b = bricks[c][r];
      if (b.status === 1) {
        if (
          x > b.x &&
          x < b.x + brickWidth &&
          y > b.y &&
          y < b.y + brickHeight
        ) {
          dy = -dy;
          b.status = 0;
        }
      }
    }
  }
}
```**衝突条件 (4 つすべてが同時に満たされる必要があります):**
- `x > b.x` -- ボールの中心はレンガの左端の右側にあります。
- `x < b.x + brickWidth` -- ボールの中心はレンガの右端の左側にあります。
- `y > b.y` -- ボールの中心がレンガの上端の下にあります。
- `y < b.y + brickHeight` -- ボールの中心がレンガの下端の上にあります。

衝突が検出された場合:
- `dy = -dy` はボールの垂直方向を反転します (バウンス)。
- `b.status = 0` はレンガを破壊済みとしてマークします。

### ステータスを尊重するためのdrawBricks()の更新

まだアクティブなレンガのみを描画します (`status === 1`):```javascript
function drawBricks() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      if (bricks[c][r].status === 1) {
        const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
        const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
        bricks[c][r].x = brickX;
        bricks[c][r].y = brickY;
        ctx.beginPath();
        ctx.rect(brickX, brickY, brickWidth, brickHeight);
        ctx.fillStyle = "#0095DD";
        ctx.fill();
        ctx.closePath();
      }
    }
  }
}
```### ゲームループでのcollisionDetection()の呼び出し

すべての要素を描画した後、`draw()` 関数に呼び出しを追加します。```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();
  collisionDetection();
  // ... rest of draw function
}
```---

## ステップ 8: スコアを追跡して勝利する

レンガが破壊されるたびに増加するスコア カウンターと、すべてのレンガがなくなったときにトリガーされる勝利条件を追加します。

### スコアの初期化```javascript
let score = 0;
```###drawScore() 関数

テキストレンダリングを使用して現在のスコアをキャンバスに表示します。```javascript
function drawScore() {
  ctx.font = "16px Arial";
  ctx.fillStyle = "#0095DD";
  ctx.fillText(`Score: ${score}`, 8, 20);
}
```- `ctx.font` は、フォント サイズとファミリー (CSS など) を設定します。
- `ctx.fillText(text, x, y)` は、指定された座標でテキストをレンダリングします。
- 位置 `(8, 20)` は、スコアを左上隅に配置します。

### スコアの増加

`collisionDetection()` 関数で、レンガがヒットしたときにスコアを増加させます。```javascript
dy = -dy;
b.status = 0;
score++;
```### 勝利条件の追加

スコアを増やした後、プレイヤーがすべてのレンガを破壊したかどうかを確認します。```javascript
score++;
if (score === brickRowCount * brickColumnCount) {
  alert("YOU WIN, CONGRATULATIONS!");
  document.location.reload();
  clearInterval(interval);
}
```レンガの合計数は `brickRowCount * brickColumnCount` です。スコアがその数値に達すると、すべてのレンガが破壊されます。

### スコアと勝利を伴ってcollisionDetection()を完了する```javascript
function collisionDetection() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      const b = bricks[c][r];
      if (b.status === 1) {
        if (
          x > b.x &&
          x < b.x + brickWidth &&
          y > b.y &&
          y < b.y + brickHeight
        ) {
          dy = -dy;
          b.status = 0;
          score++;
          if (score === brickRowCount * brickColumnCount) {
            alert("YOU WIN, CONGRATULATIONS!");
            document.location.reload();
            clearInterval(interval);
          }
        }
      }
    }
  }
}
```### ゲームループでのdrawScore()の呼び出し

`draw()` 関数に呼び出しを追加します。```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();
  drawScore();
  collisionDetection();
  // ... rest of draw function
}
```### キャンバス テキスト メソッドのリファレンス

|メソッド/プロパティ |目的 |
|-----------------|-----------|
| `ctx.font` |フォント サイズとファミリーを設定する |
| `ctx.fillStyle` |テキストの色を設定する |
| `ctx.fillText(text, x, y)` |座標 | に塗りつぶしテキストを描画します。

---

## ステップ 9: マウス コントロール

キーボード コントロールに加えて、マウスのサポートも追加され、プレイヤーはマウスを動かしてパドルを動かすことができます。

### Mousemove イベント リスナーの追加

既存のキーボード リスナーと一緒にハンドラーを登録します。```javascript
document.addEventListener("mousemove", mouseMoveHandler);
```### MouseMoveHandler 関数

キャンバスに対するマウスの水平位置を計算し、パドルの位置を更新します。```javascript
function mouseMoveHandler(e) {
  const relativeX = e.clientX - canvas.offsetLeft;
  if (relativeX > 0 && relativeX < canvas.width) {
    paddleX = relativeX - paddleWidth / 2;
  }
}
```**仕組み:**
- `e.clientX` -- ブラウザのビューポート内のマウスの水平位置。
- `canvas.offsetLeft` -- キャンバスの左端からビューポートの左端までの距離。
- `relativeX` -- キャンバス (ビューポートではない) を基準としたマウスの位置。
- 境界チェック (`relativeX > 0 && relativeX < canvas.width`) により、マウスがキャンバス上にある場合にのみパドルが移動することが保証されます。
- `paddleX = relativeX - paddleWidth / 2` は、パドルの幅の半分を引いて、パドルをマウス カーソルの下の中央に配置します。

### イベント リスナーのセットアップを完了する (キーボード + マウス)```javascript
document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);
document.addEventListener("mousemove", mouseMoveHandler);
```両方の制御方法が同時に機能します。プレーヤーは矢印キーまたはマウスを使用したり、いつでもそれらを切り替えることができます。

---

## ステップ 10: 仕上げ

最後のステップでは、ライフ システムを追加し (プレイヤーに複数のチャンスが与えられるように)、ゲーム ループを `setInterval` から `requestAnimationFrame` にアップグレードして、レンダリングをよりスムーズにします。

### Lives 変数の追加```javascript
let lives = 3;
```###drawLives() 関数

右上隅に残りのライフを表示します。```javascript
function drawLives() {
  ctx.font = "16px Arial";
  ctx.fillStyle = "#0095DD";
  ctx.fillText(`Lives: ${lives}`, canvas.width - 65, 20);
}
```### ライブシステムの実装

即時ゲームオーバーのロジックをライフベースのシステムに置き換えます。ボールがパドルを外したとき:```javascript
if (y + dy < ballRadius) {
  dy = -dy;
} else if (y + dy > canvas.height - ballRadius) {
  if (x > paddleX && x < paddleX + paddleWidth) {
    dy = -dy;
  } else {
    lives--;
    if (!lives) {
      alert("GAME OVER");
      document.location.reload();
    } else {
      // Reset ball and paddle positions
      x = canvas.width / 2;
      y = canvas.height - 30;
      dx = 2;
      dy = -2;
      paddleX = (canvas.width - paddleWidth) / 2;
    }
  }
}
```**人命が失われるとどうなるか:**
- `lives--` は、ライフ カウンタをデクリメントします。
- `lives` が `0` に達すると、アラートが表示されてゲームが終了し、ページがリロードされます。
- それ以外の場合、ボールは中央下にリセットされ、速度はリセットされ、パドルは中央にリセットされます。

### requestAnimationFrame へのアップグレード

よりスムーズでブラウザーに最適化されたゲーム ループを実現するには、`setInterval` を `requestAnimationFrame` に置き換えます。

**古いアプローチ (削除):**```javascript
interval = setInterval(draw, 10);
```**新しいアプローチ:**
`draw()` 関数の最後に `requestAnimationFrame(draw)` を追加します。```javascript
function draw() {
  // ... all drawing and logic ...
  requestAnimationFrame(draw);
}

// Start the game by calling draw() once:
draw();
````requestAnimationFrame` を使用すると、ブラウザーは最適なフレーム レート (通常は 60fps) でレンダリングをスケジュールできます。これは、固定の 10 ミリ秒間隔よりも効率的です。

### ゲームループでのdrawLives()の呼び出し```javascript
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();
  drawScore();
  drawLives();
  collisionDetection();
  // ... rest of logic ...
  requestAnimationFrame(draw);
}
```---

## 最終的なゲーム コードを完了する

以下は、単一の自己完結型 HTML ファイル内のゲーム全体です。これは、10 のステップすべてを組み合わせた最終製品です。```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>2D Breakout Game</title>
    <style>
      * {
        padding: 0;
        margin: 0;
      }
      canvas {
        background: #eeeeee;
        display: block;
        margin: 0 auto;
      }
    </style>
  </head>
  <body>
    <canvas id="myCanvas" width="480" height="320"></canvas>

    <script>
      const canvas = document.getElementById("myCanvas");
      const ctx = canvas.getContext("2d");

      // --- Ball ---
      const ballRadius = 10;
      let x = canvas.width / 2;
      let y = canvas.height - 30;
      let dx = 2;
      let dy = -2;

      // --- Paddle ---
      const paddleHeight = 10;
      const paddleWidth = 75;
      let paddleX = (canvas.width - paddleWidth) / 2;

      // --- Controls ---
      let rightPressed = false;
      let leftPressed = false;

      // --- Bricks ---
      const brickRowCount = 3;
      const brickColumnCount = 5;
      const brickWidth = 75;
      const brickHeight = 20;
      const brickPadding = 10;
      const brickOffsetTop = 30;
      const brickOffsetLeft = 30;

      const bricks = [];
      for (let c = 0; c < brickColumnCount; c++) {
        bricks[c] = [];
        for (let r = 0; r < brickRowCount; r++) {
          bricks[c][r] = { x: 0, y: 0, status: 1 };
        }
      }

      // --- Score and Lives ---
      let score = 0;
      let lives = 3;

      // =====================
      // Event Listeners
      // =====================
      document.addEventListener("keydown", keyDownHandler);
      document.addEventListener("keyup", keyUpHandler);
      document.addEventListener("mousemove", mouseMoveHandler);

      function keyDownHandler(e) {
        if (e.key === "Right" || e.key === "ArrowRight") {
          rightPressed = true;
        } else if (e.key === "Left" || e.key === "ArrowLeft") {
          leftPressed = true;
        }
      }

      function keyUpHandler(e) {
        if (e.key === "Right" || e.key === "ArrowRight") {
          rightPressed = false;
        } else if (e.key === "Left" || e.key === "ArrowLeft") {
          leftPressed = false;
        }
      }

      function mouseMoveHandler(e) {
        const relativeX = e.clientX - canvas.offsetLeft;
        if (relativeX > 0 && relativeX < canvas.width) {
          paddleX = relativeX - paddleWidth / 2;
        }
      }

      // =====================
      // Collision Detection
      // =====================
      function collisionDetection() {
        for (let c = 0; c < brickColumnCount; c++) {
          for (let r = 0; r < brickRowCount; r++) {
            const b = bricks[c][r];
            if (b.status === 1) {
              if (
                x > b.x &&
                x < b.x + brickWidth &&
                y > b.y &&
                y < b.y + brickHeight
              ) {
                dy = -dy;
                b.status = 0;
                score++;
                if (score === brickRowCount * brickColumnCount) {
                  alert("YOU WIN, CONGRATULATIONS!");
                  document.location.reload();
                }
              }
            }
          }
        }
      }

      // =====================
      // Drawing Functions
      // =====================
      function drawBall() {
        ctx.beginPath();
        ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
        ctx.fillStyle = "#0095DD";
        ctx.fill();
        ctx.closePath();
      }

      function drawPaddle() {
        ctx.beginPath();
        ctx.rect(
          paddleX,
          canvas.height - paddleHeight,
          paddleWidth,
          paddleHeight
        );
        ctx.fillStyle = "#0095DD";
        ctx.fill();
        ctx.closePath();
      }

      function drawBricks() {
        for (let c = 0; c < brickColumnCount; c++) {
          for (let r = 0; r < brickRowCount; r++) {
            if (bricks[c][r].status === 1) {
              const brickX =
                c * (brickWidth + brickPadding) + brickOffsetLeft;
              const brickY =
                r * (brickHeight + brickPadding) + brickOffsetTop;
              bricks[c][r].x = brickX;
              bricks[c][r].y = brickY;
              ctx.beginPath();
              ctx.rect(brickX, brickY, brickWidth, brickHeight);
              ctx.fillStyle = "#0095DD";
              ctx.fill();
              ctx.closePath();
            }
          }
        }
      }

      function drawScore() {
        ctx.font = "16px Arial";
        ctx.fillStyle = "#0095DD";
        ctx.fillText(`Score: ${score}`, 8, 20);
      }

      function drawLives() {
        ctx.font = "16px Arial";
        ctx.fillStyle = "#0095DD";
        ctx.fillText(`Lives: ${lives}`, canvas.width - 65, 20);
      }

      // =====================
      // Main Game Loop
      // =====================
      function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawBricks();
        drawBall();
        drawPaddle();
        drawScore();
        drawLives();
        collisionDetection();

        // Left and right wall collision
        if (
          x + dx > canvas.width - ballRadius ||
          x + dx < ballRadius
        ) {
          dx = -dx;
        }

        // Top wall collision
        if (y + dy < ballRadius) {
          dy = -dy;
        } else if (y + dy > canvas.height - ballRadius) {
          // Bottom edge: paddle collision or lose a life
          if (x > paddleX && x < paddleX + paddleWidth) {
            dy = -dy;
          } else {
            lives--;
            if (!lives) {
              alert("GAME OVER");
              document.location.reload();
            } else {
              x = canvas.width / 2;
              y = canvas.height - 30;
              dx = 2;
              dy = -2;
              paddleX = (canvas.width - paddleWidth) / 2;
            }
          }
        }

        // Paddle movement (keyboard)
        if (rightPressed) {
          paddleX = Math.min(
            paddleX + 7,
            canvas.width - paddleWidth
          );
        } else if (leftPressed) {
          paddleX = Math.max(paddleX - 7, 0);
        }

        x += dx;
        y += dy;
        requestAnimationFrame(draw);
      }

      draw();
    </script>
  </body>
</html>
```---

## クイック リファレンス: すべてのゲーム変数

|変数 |タイプ |目的 |
|----------|------|----------|
| `canvas` |定数 | HTML キャンバス要素への参照 |
| `ctx` |定数 | 2D レンダリング コンテキスト |
| `ballRadius` |定数 |ボールの半径 (10) |
| `x`、`y` |させてください |現在のボールの位置 |
| `dx`、`dy` |させてください |ボール速度 (フレームあたりのピクセル) |
| `paddleHeight` |定数 |パドルの高さ (10) |
| `paddleWidth` |定数 |パドルの幅 (75) |
| `paddleX` |させてください |パドルの現在の水平位置 |
| `rightPressed` |させてください |右矢印キーが押されているかどうか |
| `leftPressed` |させてください |左矢印キーが押されているかどうか |
| `brickRowCount` |定数 |レンガの列の数 (3) |
| `brickColumnCount` |定数 |レンガ柱の数 (5) |
| `brickWidth` |定数 |各レンガの幅 (75) |
| `brickHeight` |定数 |各レンガの高さ (20) |
| `brickPadding` |定数 |レンガ間のスペース (10) |
| `brickOffsetTop` |定数 |キャンバスの上部から最初のレンガ列までの距離 (30) |
| `brickOffsetLeft` |定数 |キャンバスの左から最初のレンガ柱までの距離 (30) |
| `bricks` |定数 |すべてのレンガ オブジェクトを保持する 2D 配列 |
| `score` |させてください |現在のプレイヤーのスコア |
| `lives` |させてください |残機(3からスタート) |

## クイックリファレンス: すべての関数

|機能 |目的 |
|----------|----------|
| `keyDownHandler(e)` |キーを押すと `rightPressed` または `leftPressed` を `true` に設定します |
| `keyUpHandler(e)` |キーを放したときに `rightPressed` または `leftPressed` を `false` に設定します。
| `mouseMoveHandler(e)` |マウスの水平位置に合わせてパドルを移動します。
| `collisionDetection()` |すべてのアクティブなレンガに対してボールをチェックします。ヒットしたレンガを破壊し、スコアを増加させ、勝利を確認します |
| `drawBall()` |現在の `(x, y)` 位置でボールをレンダリングします。
| `drawPaddle()` |現在の `paddleX` 位置でパドルをレンダリングします。
| `drawBricks()` | `status === 1` を使用してすべてのレンガをレンダリングします。
| `drawScore()` |左上隅にスコアテキストをレンダリングします。
| `drawLives()` |右上隅にライフのテキストを表示します。
| `draw()` |メイン ゲーム ループ: キャンバスをクリアし、すべてを描画し、衝突を処理し、位置を更新します。