# シンプル2Dプラットフォーマーエンジン テンプレート

*Dead Cells* のリード開発者である **Sebastien Benard**（deepnight）による、グリッドベースの2Dプラットフォーマーエンジンチュートリアルです。このテンプレートは、高性能なプラットフォーマーに必要な基本アーキテクチャを網羅しています。具体的には、整数グリッドセルとサブピクセル精度を組み合わせる二重座標システム、速度と摩擦の仕組み、重力、そして堅牢な衝突判定・応答システムです。アプローチ自体は言語非依存ですが、例では Haxe を使用しています。

**参考元:**
- [Part 1 - Basics](https://deepnight.net/tutorial/a-simple-platformer-engine-part-1-basics/)
- [Part 2 - Collisions](https://deepnight.net/tutorial/a-simple-platformer-engine-part-2-collisions/)

**著者:** [Sebastien Benard / deepnight](https://deepnight.net)

---

## エンジンアーキテクチャ概要

このエンジンは、各セルが固定ピクセルサイズ（例: 16x16）を持つグリッドベースのワールドを中心に構築されています。エンティティはこのグリッド内で **二重座標システム** を使って存在します。つまり、粗い位置を表す整数セル座標と、各セル内でのサブピクセル精度を表す浮動小数比率です。この設計により、グリッドに対するピクセル単位の正確な衝突判定を維持しつつ、滑らかで流れるような移動を実現できます。

### コア原則

1. **グリッドが真実:** ワールドは2Dセルグリッド。衝突データはグリッドに存在する。
2. **エンティティはセルをまたぐ:** エンティティの位置は、占有セル（`cx`, `cy`）と、そのセル内の進行度（`xr`, `yr`）で定義される。
3. **速度はグリッド比率単位:** 移動差分（`dx`, `dy`）は生のピクセルではなく、1ステップあたりのセルの分数を表す。
4. **衝突はグリッド参照:** スプライト境界とジオメトリを照合する代わりに、エンジンはエンティティがこれから入るグリッドセルをチェックする。

---

## Part 1: Basics

### グリッド

レベルは2D配列で、各セルは空かソリッドかのどちらかです。セルサイズ（ピクセル）は定数で定義します:

```haxe
static inline var GRID = 16;
```

衝突データはシンプルな2Dブーリアンまたは整数マップとして保持します:

```haxe
// Check if a grid cell is solid
function hasCollision(cx:Int, cy:Int):Bool {
  // Look up cell value in the level data
  return level.getCollision(cx, cy) != 0;
}
```

### エンティティ配置: 二重座標

すべてのエンティティは4つの値で位置を管理します:

| 変数 | 型 | 説明 |
|----------|------|-------------|
| `cx` | Int | セルX座標（エンティティがいる列） |
| `cy` | Int | セルY座標（エンティティがいる行） |
| `xr` | Float | セル内X比率、範囲 0.0〜1.0 |
| `yr` | Float | セル内Y比率、範囲 0.0〜1.0 |

`cx=5, cy=3, xr=0.5, yr=1.0` のエンティティは、セル (5,3) の水平方向中央にあり、下端に接しています。

### ピクセル座標への変換

エンティティを描画するには、グリッド座標をピクセル位置に変換します:

```haxe
// Pixel position for rendering
var pixelX : Float = (cx + xr) * GRID;
var pixelY : Float = (cy + yr) * GRID;
```

これにより、衝突システムは離散的なグリッドセルで動作していても、描画時には滑らかなサブピクセル精度の位置が得られます。

### 速度と移動

速度は **固定ステップあたりのセル比率単位** で表現します（フレームあたりピクセルではありません）:

```haxe
var dx : Float = 0; // Horizontal velocity (cells per step)
var dy : Float = 0; // Vertical velocity (cells per step)
```

固定ステップ更新ごとに、速度を比率に加算します:

```haxe
// Apply horizontal movement
xr += dx;

// Apply vertical movement
yr += dy;
```

### セルオーバーフロー

比率が 0..1 の範囲を超えたら、エンティティは隣接セルに移動したことになります:

```haxe
// X overflow
while (xr > 1) { xr--; cx++; }
while (xr < 0) { xr++; cx--; }

// Y overflow
while (yr > 1) { yr--; cy++; }
while (yr < 0) { yr++; cy--; }
```

### 摩擦

摩擦は各ステップで乗算され、速度をゼロへ減衰させます:

```haxe
var frictX : Float = 0.82; // Horizontal friction (0 = instant stop, 1 = no friction)
var frictY : Float = 0.82; // Vertical friction

// Applied each step after movement
dx *= frictX;
dy *= frictY;

// Clamp very small values to zero
if (Math.abs(dx) < 0.0005) dx = 0;
if (Math.abs(dy) < 0.0005) dy = 0;
```

代表的な摩擦値:
- `0.82` -- 標準的な地面摩擦（応答がよく、すぐ止まる）
- `0.94` -- 氷や滑りやすい地面（減速が遅い）
- `0.96` -- 空中摩擦（水平方向の減速が非常に遅い）

### 重力

重力は各ステップで `dy` に加算される定数です:

```haxe
static inline var GRAVITY = 0.05; // In cell-ratio units per step^2

// In fixedUpdate:
dy += GRAVITY;
```

`dy` は蓄積され、さらに摩擦が適用されるため、エンティティは自然な終端速度に達します。

### 描画 / スプライト同期

物理ステップ後、計算済みのピクセル位置にスプライトを配置します:

```haxe
// In postUpdate, after physics is done:
sprite.x = (cx + xr) * GRID;
sprite.y = (cy + yr) * GRID;
```

プラットフォーマーのキャラクターでは、アンカーポイントは通常スプライトの下中央です。`yr = 1.0` が現在セルの下端を表すため、スプライトの足が床に揃います。

### 基本エンティティテンプレート

```haxe
class Entity {
  // Grid coordinates
  var cx : Int = 0;
  var cy : Int = 0;
  var xr : Float = 0.5;
  var yr : Float = 1.0;

  // Velocity
  var dx : Float = 0;
  var dy : Float = 0;

  // Friction
  var frictX : Float = 0.82;
  var frictY : Float = 0.82;

  // Gravity
  static inline var GRAVITY = 0.05;

  // Grid size
  static inline var GRID = 16;

  // Pixel position (computed)
  public var attachX(get, never) : Float;
  inline function get_attachX() return (cx + xr) * GRID;

  public var attachY(get, never) : Float;
  inline function get_attachY() return (cy + yr) * GRID;

  public function fixedUpdate() {
    // Gravity
    dy += GRAVITY;

    // Apply velocity
    xr += dx;
    yr += dy;

    // Apply friction
    dx *= frictX;
    dy *= frictY;

    // Clamp small values
    if (Math.abs(dx) < 0.0005) dx = 0;
    if (Math.abs(dy) < 0.0005) dy = 0;

    // Cell overflow
    while (xr > 1) { xr--; cx++; }
    while (xr < 0) { xr++; cx--; }
    while (yr > 1) { yr--; cy++; }
    while (yr < 0) { yr++; cy--; }
  }

  public function postUpdate() {
    sprite.x = attachX;
    sprite.y = attachY;
  }
}
```

---

## Part 2: Collisions

### 衝突の考え方

バウンディングボックス同士の衝突判定（斜面、一方通行プラットフォーム、エッジケースで複雑化しやすい）を使う代わりに、このエンジンではグリッドセルを直接チェックします。エンティティ位置はすでにグリッド基準で表現されているため、衝突判定は単純な整数参照の連続になります。

### コアアイデア

エンティティが隣接セルへ移動することを許可する前に、そのセルがソリッドかを確認します。ソリッドなら、その軸の比率をクランプし、速度をゼロにします。

### 軸分離

衝突は **軸ごと** に処理します。まずX、次にY（またはその逆）です。これによりロジックが単純になり、角でのすり抜けなどの問題を避けられます。

### X軸衝突

`dx` を `xr` に適用した後、セルオーバーフロー処理前に衝突チェックを行います:

```haxe
// Apply X movement
xr += dx;

// Check collision to the RIGHT
if (dx > 0 && hasCollision(cx + 1, cy) && xr >= 0.7) {
  xr = 0.7;   // Clamp: stop before entering the solid cell
  dx = 0;     // Kill horizontal velocity
}

// Check collision to the LEFT
if (dx < 0 && hasCollision(cx - 1, cy) && xr <= 0.3) {
  xr = 0.3;   // Clamp: stop before entering the solid cell
  dx = 0;     // Kill horizontal velocity
}

// Cell overflow (after collision check)
while (xr > 1) { xr--; cx++; }
while (xr < 0) { xr++; cx--; }
```

**なぜ 0.7 と 0.3 なのか？** これらのしきい値は、セル内におけるエンティティの当たり判定半径を表します。`xr = 0.5` が中心で、半幅が 0.3 セルなら、右側は `xr = 0.7`、左側は `xr = 0.3` で衝突します。値はエンティティ幅に合わせて調整してください。

### Y軸衝突

同様に、`dy` を `yr` に適用した後に:

```haxe
// Apply Y movement
yr += dy;

// Check collision BELOW (floor)
if (dy > 0 && hasCollision(cx, cy + 1) && yr >= 1.0) {
  yr = 1.0;   // Clamp: land on top of the solid cell
  dy = 0;     // Kill vertical velocity
}

// Check collision ABOVE (ceiling)
if (dy < 0 && hasCollision(cx, cy - 1) && yr <= 0.3) {
  yr = 0.3;   // Clamp: stop before entering ceiling cell
  dy = 0;     // Kill vertical velocity
}

// Cell overflow
while (yr > 1) { yr--; cy++; }
while (yr < 0) { yr++; cy--; }
```

床衝突では、`yr = 1.0` はエンティティが現在セルの下端（＝下のセルの上端）に正確に乗っていることを意味します。これが自然な「地面に立っている」位置です。

### 接地判定

エンティティが地面に立っているか（ジャンプ処理、アニメーションなど）を判定するには:

```haxe
function isOnGround() : Bool {
  return hasCollision(cx, cy + 1) && yr >= 0.98;
}
```

`1.0` ではなく `0.98` を使うのは、浮動小数点のわずかな誤差を許容するためです。

### 衝突対応込みの完全なエンティティ

```haxe
class Entity {
  var cx : Int = 0;
  var cy : Int = 0;
  var xr : Float = 0.5;
  var yr : Float = 1.0;
  var dx : Float = 0;
  var dy : Float = 0;
  var frictX : Float = 0.82;
  var frictY : Float = 0.82;

  static inline var GRID = 16;
  static inline var GRAVITY = 0.05;

  // Collision radius (half-width in cell-ratio units)
  var collRadius : Float = 0.3;

  function hasCollision(testCx:Int, testCy:Int):Bool {
    return level.isCollision(testCx, testCy);
  }

  function isOnGround():Bool {
    return hasCollision(cx, cy + 1) && yr >= 0.98;
  }

  public function fixedUpdate() {
    // --- Gravity ---
    dy += GRAVITY;

    // --- X Axis ---
    xr += dx;

    // Right collision
    if (dx > 0 && hasCollision(cx + 1, cy) && xr >= 1.0 - collRadius) {
      xr = 1.0 - collRadius;
      dx = 0;
    }

    // Left collision
    if (dx < 0 && hasCollision(cx - 1, cy) && xr <= collRadius) {
      xr = collRadius;
      dx = 0;
    }

    // X cell overflow
    while (xr > 1) { xr--; cx++; }
    while (xr < 0) { xr++; cx--; }

    // --- Y Axis ---
    yr += dy;

    // Floor collision
    if (dy > 0 && hasCollision(cx, cy + 1) && yr >= 1.0) {
      yr = 1.0;
      dy = 0;
    }

    // Ceiling collision
    if (dy < 0 && hasCollision(cx, cy - 1) && yr <= collRadius) {
      yr = collRadius;
      dy = 0;
    }

    // Y cell overflow
    while (yr > 1) { yr--; cy++; }
    while (yr < 0) { yr++; cy--; }

    // --- Friction ---
    dx *= frictX;
    dy *= frictY;

    if (Math.abs(dx) < 0.0005) dx = 0;
    if (Math.abs(dy) < 0.0005) dy = 0;
  }

  public function postUpdate() {
    sprite.x = (cx + xr) * GRID;
    sprite.y = (cy + yr) * GRID;
  }
}
```

---

## 衝突のエッジケースと解決策

### 斜め移動 / 角の食い込み

衝突は軸ごとに順番にチェックするため、角へ斜め移動したエンティティは自然にどちらか一方の軸で先に解決されます。これにより角で引っかかりにくく、複雑な斜め衝突ロジックが不要になります。

### 高速時のすり抜け（トンネリング）

`dx` または `dy` が大きすぎて1ステップでセルを丸ごと飛び越えると、壁をすり抜ける可能性があります。対策:

1. **速度に上限を設ける:** `dx` と `dy` を最大 0.5（1ステップあたり半セル）にクランプ
2. **ステップを分割する:** しきい値を超える速度なら、より小さい増分で衝突チェックを実行
3. **グリッドをレイマーチする:** 移動経路上のすべてのセルをチェック

```haxe
// Simple velocity cap
if (dx > 0.5) dx = 0.5;
if (dx < -0.5) dx = -0.5;
if (dy > 0.5) dy = 0.5;
if (dy < -0.5) dy = -0.5;
```

### 一方通行プラットフォーム

下からはすり抜けられるが、上からは着地できるプラットフォーム:

```haxe
// In Y collision, check for one-way platform
if (dy > 0 && isOneWayPlatform(cx, cy + 1) && yr >= 1.0 && prevYr < 1.0) {
  yr = 1.0;
  dy = 0;
}
```

ポイント: エンティティが下向きに移動している（`dy > 0`）かつ、直前フレームではプラットフォームより上にいた（`prevYr < 1.0`）場合にのみ衝突させること。

### 斜面

基本的な斜面対応では、二値の衝突判定ではなく、セル内のエンティティX位置に応じた斜面高さを取得します:

```haxe
// Pseudocode for slope collision
var slopeHeight = getSlopeHeight(cx, cy + 1, xr);
if (yr >= slopeHeight) {
  yr = slopeHeight;
  dy = 0;
}
```

---

## ジャンプ

ジャンプは単純に負の `dy` インパルスを与えるだけです:

```haxe
function jump() {
  if (isOnGround()) {
    dy = -0.5; // Jump impulse (in cell-ratio units)
  }
}
```

重力が上方向移動を自然に減速させ、放物線軌道を作ります。可変ジャンプ（長押しするほど高く跳ぶ）にするには:

```haxe
// On jump button release, reduce upward velocity
function onJumpRelease() {
  if (dy < 0) {
    dy *= 0.5; // Cut remaining upward velocity
  }
}
```

---

## 座標系図

```
  Cell (cx, cy)           Next Cell (cx+1, cy)
  +-------------------+   +-------------------+
  |                   |   |                   |
  |  xr=0.0    xr=1.0 --> |  xr=0.0           |
  |                   |   |                   |
  |         *         |   |                   |
  |     (xr=0.5,      |   |                   |
  |      yr=0.5)      |   |                   |
  |                   |   |                   |
  +-------------------+   +-------------------+
  yr=0.0      yr=1.0 = top of cell below

  Pixel position = (cx + xr) * GRID, (cy + yr) * GRID
```

---

## 更新順序の要約

```
fixedUpdate():
  1. Apply gravity          dy += GRAVITY
  2. Apply X velocity       xr += dx
  3. Check X collisions     Clamp xr, zero dx if colliding
  4. Handle X cell overflow cx/xr normalization
  5. Apply Y velocity       yr += dy
  6. Check Y collisions     Clamp yr, zero dy if colliding
  7. Handle Y cell overflow cy/yr normalization
  8. Apply friction         dx *= frictX, dy *= frictY
  9. Zero out tiny values   Threshold check

postUpdate():
  1. Sync sprite position   sprite.x/y = pixel coords
  2. Update animation       Based on state/velocity
  3. Camera follow          Track entity
```

---

## 設計上の利点

| 機能 | 利点 |
|---------|---------|
| グリッドベース衝突 | チェックごとに O(1) 参照、広域フェーズ不要 |
| 二重座標 | 整数衝突判定を維持しつつサブピクセルで滑らかに描画 |
| 軸ごとの衝突 | ロジックが単純で、角の処理も自然 |
| 比率ベース速度 | 解像度非依存の移動 |
| 摩擦乗数 | 地面タイプごとに操作感を調整可能 |
| セルオーバーフロー while ループ | 複数セル移動も安全に処理可能 |

