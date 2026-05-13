# ゲーム開発アルゴリズム

ライン描画、レイキャスティング、衝突判定、物理シミュレーション、ベクトル数学など、
ゲーム開発で不可欠なアルゴリズムを網羅した包括的リファレンスです。

---

## Bresenham の線分アルゴリズム -- レイキャスティング、視線判定、経路探索

> Source: https://deepnight.net/tutorial/bresenham-magic-raycasting-line-of-sight-pathfinding/

### 概要

Bresenham の線分アルゴリズムは、グリッド上の2点間を結ぶ直線上にどのセルが
存在するかを効率的に求める手法です。もともとはラスターディスプレイでピクセルを
描画するために開発されましたが、現在ではゲーム開発におけるレイキャスティング、
視線（Line of Sight）判定、グリッドベース経路探索の基盤的ツールになっています。
このアルゴリズムは整数演算（加算、減算、ビットシフト）のみを使うため、非常に高速です。

### 数学的 / アルゴリズム的な概念

基本的な考え方は、主軸（距離が大きい軸）に沿って1セルずつ進みながら、
誤差項を蓄積して、真の直線が現在の副軸位置からどれだけずれているかを追跡することです。
誤差が閾値を超えたら、副軸の座標を1増やします。

主な性質:
- **整数演算のみ**: 浮動小数点の除算や乗算は不要。
- **誤差の逐次蓄積**: 小数の傾きは整数の誤差項で追跡するため、ドリフトを回避。
- **対称性**: ステップ符号を調整することで、線分方向に依存せず同じ処理で動作。

2つのグリッド点 `(x0, y0)` と `(x1, y1)` が与えられたとき:

```
dx = abs(x1 - x0)
dy = abs(y1 - y0)
```

誤差項は初期化され、各ステップで更新されます。ゼロをまたいだら副軸を進めます。

### 擬似コード

```
function bresenham(x0, y0, x1, y1):
    dx = abs(x1 - x0)
    dy = abs(y1 - y0)
    sx = sign(x1 - x0)   // -1 or +1
    sy = sign(y1 - y0)   // -1 or +1
    err = dx - dy

    while true:
        visit(x0, y0)          // process or record this cell

        if x0 == x1 AND y0 == y1:
            break

        e2 = 2 * err

        if e2 > -dy:
            err = err - dy
            x0  = x0 + sx

        if e2 < dx:
            err = err + dx
            y0  = y0 + sy
```

### Haxe 実装（出典より）

```haxe
public function hasLineOfSight(x0:Int, y0:Int, x1:Int, y1:Int):Bool {
    var dx = hxd.Math.iabs(x1 - x0);
    var dy = hxd.Math.iabs(y1 - y0);
    var sx = (x0 < x1) ? 1 : -1;
    var sy = (y0 < y1) ? 1 : -1;
    var err = dx - dy;

    while (true) {
        if (isBlocking(x0, y0))
            return false;

        if (x0 == x1 && y0 == y1)
            return true;

        var e2 = 2 * err;
        if (e2 > -dy) {
            err -= dy;
            x0 += sx;
        }
        if (e2 < dx) {
            err += dx;
            y0 += sy;
        }
    }
}
```

### ゲーム開発での実践的な用途

- **視線判定（LOS）**: エンティティからターゲットまで Bresenham 線をたどり、経路上に
  壁や障害物セルがあれば視線は遮られる。
- **グリッド上のレイキャスティング**: 発生源から複数方向へレイを飛ばし、
  可視マップや視野コーンを計算する。
- **グリッドベース経路探索の検証**: A* などで経路計算後、ウェイポイント間の直線ショートカットが
  遮られていないか Bresenham 判定で確認する。
- **弾道トレース**: タイルベースゲームで、弾丸や飛翔体がどのタイルを通過するかを判定する。
- **ライティングと影生成**: 光源からレイを追跡し、2D グリッド上の照明セルと影セルを計算する。

---

## 衝突判定と応答システム

> Source: https://medium.com/@erikkubiak/dev-log-1-custom-engine-writing-my-collision-system-2a97856f9a93

### 概要

衝突システムは、ゲームオブジェクト同士が重なったり交差したりした瞬間を検出し、
その重なりを解消して、オブジェクトが物理的に反応（反射、停止、滑り）するようにする役割を持ちます。
独自の衝突システムを構築するには、適切な境界形状の選定、重なり判定の実装、
そして解決戦略の設計が必要です。

### 数学的 / アルゴリズム的な概念

#### 境界形状

- **AABB (Axis-Aligned Bounding Box)**: 座標軸に平行な辺を持つ長方形。
  位置（中心または左上）と半幅で定義される。重なり判定は高速だが、
  回転形状や不規則形状には不正確。
- **Circle / Sphere colliders**: 中心と半径で定義。重なり判定は単純な距離比較。
- **OBB (Oriented Bounding Box)**: 回転した長方形。重なり判定に分離軸定理を使う。

#### AABB vs AABB の重なり判定

2つの軸平行境界ボックスは、すべての軸で重なっているとき、かつそのときに限り重なる:

```
overlapX = (a.x - a.halfW < b.x + b.halfW) AND (a.x + a.halfW > b.x - b.halfW)
overlapY = (a.y - a.halfH < b.y + b.halfH) AND (a.y + a.halfH > b.y - b.halfH)
collision = overlapX AND overlapY
```

#### Circle vs Circle の重なり判定

```
dx = a.x - b.x
dy = a.y - b.y
distSquared = dx * dx + dy * dy
collision = distSquared < (a.radius + b.radius) ^ 2
```

距離の二乗同士を比較すれば、高コストな平方根演算を回避できます。

#### Separating Axis Theorem (SAT)

2つの凸形状は、少なくとも1つの軸で投影が重ならなければ衝突しません。
長方形の場合は、両方の長方形の辺法線を軸として判定します。
すべての投影が重なれば、形状は衝突しています。

#### Sweep and Prune（広域フェーズ）

すべてのオブジェクトペアを判定する（O(n^2)）代わりに、1軸上の最小端で
オブジェクトをソートします。その軸で重ならないオブジェクト同士は衝突しないため、
詳細判定から除外できます。

### 擬似コード -- 衝突判定と解決

```
// Broad phase: spatial hash or sweep-and-prune
candidates = broadPhase(allObjects)

for each pair (a, b) in candidates:
    overlap = narrowPhaseTest(a, b)

    if overlap:
        // Compute penetration vector
        penetration = computePenetration(a, b)

        // Resolve: push objects apart along the minimum penetration axis
        if a.isStatic:
            b.position += penetration
        else if b.isStatic:
            a.position -= penetration
        else:
            a.position -= penetration * 0.5
            b.position += penetration * 0.5

        // Optional: apply impulse for velocity response
        relativeVelocity = a.velocity - b.velocity
        impulse = computeImpulse(relativeVelocity, penetration.normal, a.mass, b.mass)
        a.velocity -= impulse / a.mass
        b.velocity += impulse / b.mass
```

#### 最小貫通ベクトル（AABB 向け）

```
function computePenetration(a, b):
    overlapX_left  = (a.x + a.halfW) - (b.x - b.halfW)
    overlapX_right = (b.x + b.halfW) - (a.x - a.halfW)
    overlapY_top   = (a.y + a.halfH) - (b.y - b.halfH)
    overlapY_bot   = (b.y + b.halfH) - (a.y - a.halfH)

    minOverlapX = min(overlapX_left, overlapX_right)
    minOverlapY = min(overlapY_top, overlapY_bot)

    if minOverlapX < minOverlapY:
        return Vector(sign * minOverlapX, 0)
    else:
        return Vector(0, sign * minOverlapY)
```

### 空間分割戦略

| Strategy | Best For | Description |
|---|---|---|
| **Uniform Grid** | 均一に分布したオブジェクト | 世界を固定セルに分割し、オブジェクトを所属セルに登録する。 |
| **Quadtree** | 非均一な分布 | 空間を4分木で再帰分割する。疎なシーンで効率的。 |
| **Spatial Hash** | 動的なシーン | オブジェクト位置をバケットにハッシュする。近傍探索は O(1)。 |
| **Sweep and Prune** | 動くオブジェクトが多い場合 | 軸でソートし、区間が重なるものだけ判定する。 |

### ゲーム開発での実践的な用途

- **プラットフォーマー物理**: プレイヤーと地形の衝突を解決し、着地できるようにしつつ
  壁すり抜けを防ぐ。
- **飛翔体ヒット判定**: 飛翔体（小さな AABB や円が多い）が敵や障害物に
  接触した瞬間を検出する。
- **トリガーゾーン**: プレイヤーが領域に入ったとき（物理解決なしの重なり判定）に
  イベントを発火する。
- **エンティティ積み重なり**: 複数回の反復解決で、オブジェクトが重なって積み上がる状況を処理する。

---

## Velocity と Speed

> Source: https://www.gamedev.net/tutorials/programming/math-and-physics/a-quick-lesson-in-velocity-and-speed-r6109/

### 概要

Velocity（速度ベクトル）と Speed（速さ）は、ゲーム内オブジェクトの移動における基本概念です。
**Speed** はスカラー（大きさのみ）、**velocity** はベクトル（大きさと方向）です。
この違いを理解することは、正しい移動、物理、AI ステアリング挙動の実装に不可欠です。

### 数学的 / アルゴリズム的な概念

#### 定義

- **Speed**: 方向を問わず、どれだけ速く動くかを表すスカラー量。
  ```
  speed = |velocity| = sqrt(vx^2 + vy^2)
  ```

- **Velocity**: 速さと方向の両方を表すベクトル量。
  ```
  velocity = (vx, vy)
  ```

- **Acceleration**: 時間に対する velocity の変化率。
  ```
  acceleration = (ax, ay)
  velocity += acceleration * deltaTime
  ```

#### Velocity による位置更新

各フレームで、位置は velocity にタイムステップを掛けた分だけ更新されます:

```
position.x += velocity.x * deltaTime
position.y += velocity.y * deltaTime
```

これは最も単純な一次積分法である **Euler integration** です。

#### 方向ベクトルの正規化

一定速度で特定方向へ移動するには、方向ベクトルを正規化し、
希望する speed を掛けます:

```
direction = target - current
length = sqrt(direction.x^2 + direction.y^2)
if length > 0:
    direction.x /= length
    direction.y /= length
velocity = direction * speed
```

これにより、両軸で全速移動すると実効速度が約 1.414 倍になる
「斜め移動問題」を防げます。

#### フレームレート非依存

`deltaTime` がないと移動速度はフレームレート依存になります:

```
// WRONG: frame-rate dependent
position += velocity

// CORRECT: frame-rate independent
position += velocity * deltaTime
```

`deltaTime` は前回フレーム更新からの経過時間（秒）です。

### 擬似コード -- 完全な移動更新

```
function update(entity, deltaTime):
    // Apply acceleration (gravity, thrust, friction, etc.)
    entity.velocity.x += entity.acceleration.x * deltaTime
    entity.velocity.y += entity.acceleration.y * deltaTime

    // Clamp speed to a maximum
    currentSpeed = magnitude(entity.velocity)
    if currentSpeed > entity.maxSpeed:
        entity.velocity = normalize(entity.velocity) * entity.maxSpeed

    // Apply friction / drag
    entity.velocity.x *= (1 - entity.friction * deltaTime)
    entity.velocity.y *= (1 - entity.friction * deltaTime)

    // Update position
    entity.position.x += entity.velocity.x * deltaTime
    entity.position.y += entity.velocity.y * deltaTime
```

### ゲーム開発での実践的な用途

- **キャラクター移動**: 毎フレーム velocity を適用して滑らかに移動させ、
  最大速度でクランプして一貫した操作感を保つ。
- **飛翔体**: 弾丸や矢に初期 velocity ベクトルを与え、毎フレーム位置を更新する。
- **重力**: 毎フレーム、velocity に一定の下向き加速度を与えて落下を表現する。
- **摩擦と抗力**: 減衰係数を掛けて時間とともに velocity を下げ、
  地面摩擦や空気抵抗を表現する。
- **AI ステアリング**: ターゲットへの望ましい velocity を計算し、
  現在の velocity をそこへ滑らかに近づける（seek, flee, arrive）。

---

## 物理エンジンの基礎

> Source: https://winter.dev/articles/physics-engine

### 概要

物理エンジンは、重力・衝突・剛体ダイナミクスといった現実世界の物理挙動をシミュレートし、
ゲームオブジェクトが自然に動き相互作用するようにします。物理エンジンの中核ループは、
力の適用、運動の積分、衝突検出、衝突解決で構成されます。

### 数学的 / アルゴリズム的な概念

#### 物理ループ

物理エンジンは固定タイムステップで更新します:

```
accumulator = 0
fixedDeltaTime = 1 / 60  // 60 Hz physics

function physicsUpdate(frameDeltaTime):
    accumulator += frameDeltaTime

    while accumulator >= fixedDeltaTime:
        step(fixedDeltaTime)
        accumulator -= fixedDeltaTime
```

固定タイムステップを使うことで、描画フレームレートに依存せず
決定的かつ安定したシミュレーションになります。

#### 積分法

**Semi-Implicit Euler**（symplectic Euler）-- ゲーム物理の標準:

```
velocity += acceleration * dt
position += velocity * dt
```

これは（先に位置を更新する）explicit Euler より安定しています。
velocity を先に更新してから位置更新に使うためです。

**Verlet Integration** -- velocity を明示的に保持しない代替法:

```
newPosition = 2 * position - oldPosition + acceleration * dt * dt
oldPosition = position
position = newPosition
```

Verlet は拘束（布、ラグドール）に特に有効で、
運動量を保ちながら位置を直接操作できます。

#### 剛体のプロパティ

各剛体は次を持ちます:

| Property | Description |
|---|---|
| `position` | ワールド空間での重心位置 |
| `velocity` | 線形速度ベクトル |
| `acceleration` | 全力の合計 / 質量 |
| `mass` | 線形加速度への抵抗 |
| `inverseMass` | `1 / mass`（静的オブジェクトは 0） |
| `angle` | 回転角 |
| `angularVelocity` | 回転速度 |
| `inertia` | 角加速度への抵抗 |
| `restitution` | 反発係数（0 = 反発なし、1 = 完全弾性） |
| `friction` | 表面摩擦係数 |

#### 力の蓄積

力は各フレームで蓄積され、その後加速度へ変換されます:

```
function applyForce(body, force):
    body.forceAccumulator += force

function integrate(body, dt):
    body.acceleration = body.forceAccumulator * body.inverseMass
    body.velocity += body.acceleration * dt
    body.position += body.velocity * dt
    body.forceAccumulator = (0, 0)  // reset
```

#### 衝突検出パイプライン

検出フェーズは2段階に分かれます:

1. **Broad Phase**: 境界ボリューム（AABB）と空間構造（グリッド、BVH ツリー、
   sweep-and-prune）を使って、衝突し得ないペアを高速に除外する。

2. **Narrow Phase**: 候補ペアに対して形状同士の精密判定を行い、
   実際に重なっているかと接触情報（衝突法線、貫通深度、接触点）を計算する。

#### インパルスによる衝突解決

2つの剛体が衝突したとき、衝突法線方向にインパルスを適用して
分離し、速度を調整します:

```
function resolveCollision(a, b, normal, penetration):
    // Relative velocity at the contact point
    relVel = b.velocity - a.velocity
    velAlongNormal = dot(relVel, normal)

    // Do not resolve if objects are separating
    if velAlongNormal > 0:
        return

    // Coefficient of restitution (take minimum)
    e = min(a.restitution, b.restitution)

    // Impulse magnitude
    j = -(1 + e) * velAlongNormal
    j /= a.inverseMass + b.inverseMass

    // Apply impulse
    impulse = j * normal
    a.velocity -= impulse * a.inverseMass
    b.velocity += impulse * b.inverseMass

    // Positional correction (prevent sinking)
    correction = max(penetration - slop, 0) / (a.inverseMass + b.inverseMass) * percent
    a.position -= correction * a.inverseMass * normal
    b.position += correction * b.inverseMass * normal
```

主要な定数:
- `slop`: 微小なめり込みによるジッタを防ぐための小さな許容値（例: 0.01）。
- `percent`: 通常 0.2〜0.8。位置補正をどれだけ強く適用するかを制御。

#### 回転ダイナミクス

2D 回転では、トルクは力の回転版です:

```
torque = cross(contactPoint - centerOfMass, impulse)
angularAcceleration = torque * inverseInertia
angularVelocity += angularAcceleration * dt
angle += angularVelocity * dt
```

慣性モーメントは形状によって異なります:
- **Circle**: `I = 0.5 * m * r^2`
- **Rectangle**: `I = (1/12) * m * (w^2 + h^2)`

### 擬似コード -- 完全な物理ステップ

```
function step(dt):
    // 1. Apply external forces (gravity, player input, etc.)
    for each body in world.bodies:
        if not body.isStatic:
            body.applyForce(gravity * body.mass)

    // 2. Integrate velocities and positions
    for each body in world.bodies:
        if not body.isStatic:
            body.velocity += (body.forceAccumulator * body.inverseMass) * dt
            body.position += body.velocity * dt
            body.angularVelocity += body.torque * body.inverseInertia * dt
            body.angle += body.angularVelocity * dt
            body.forceAccumulator = (0, 0)
            body.torque = 0

    // 3. Broad-phase collision detection
    pairs = broadPhase(world.bodies)

    // 4. Narrow-phase collision detection
    contacts = []
    for each (a, b) in pairs:
        contact = narrowPhase(a, b)
        if contact:
            contacts.append(contact)

    // 5. Resolve collisions (iterative solver)
    for i in range(solverIterations):   // typically 4-10 iterations
        for each contact in contacts:
            resolveCollision(contact.a, contact.b,
                             contact.normal, contact.penetration)
```

### ゲーム開発での実践的な用途

- **プラットフォーマー**: 重力、接地判定、ジャンプ弧、移動床。
- **トップダウンゲーム**: 壁沿いの滑り、攻撃ノックバック。
- **ラグドール物理**: 拘束で接続された剛体チェーン。
- **車両シミュレーション**: サスペンションスプリング、タイヤ摩擦、駆動力。
- **破壊表現**: オブジェクトを個別物理ボディの破片へ分解する。

---

## ゲーム開発のためのベクトル数学

> Source: https://www.gamedev.net/tutorials/programming/math-and-physics/vector-maths-for-game-dev-beginners-r5442/

### 概要

ベクトルはゲーム開発における数学的な土台です。ベクトルは大きさと方向の両方を持つ量を表します。
2D ゲームでは `(x, y)` の組、3D では `(x, y, z)` の組です。移動、物理、描画、AI など、
ほぼすべてのゲームシステムがベクトル演算に依存しています。

### 数学的 / アルゴリズム的な概念

#### ベクトル表現

2D ベクトル:
```
v = (x, y)
```

3D ベクトル:
```
v = (x, y, z)
```

ベクトルは位置、方向、速度、力、または大きさと方向を持つ任意の量を表現できます。

#### ベクトル加算

成分ごとの加算。位置に速度を適用する、力を合成する、などに使います。

```
a + b = (a.x + b.x, a.y + b.y)
```

**例**: キャラクターを速度で移動させる:
```
position = position + velocity * deltaTime
```

#### ベクトル減算

成分ごとの減算。ある点から別の点への方向と距離を求めるために使います。

```
a - b = (a.x - b.x, a.y - b.y)
```

**例**: 敵からプレイヤーへの方向:
```
directionToPlayer = player.position - enemy.position
```

#### スカラー倍

方向を変えずにベクトルの大きさをスケーリングします:

```
s * v = (s * v.x, s * v.y)
```

**例**: 移動速度の設定:
```
velocity = normalizedDirection * speed
```

#### 大きさ（長さ）

ベクトルの長さ。ピタゴラスの定理で計算します:

```
|v| = sqrt(v.x^2 + v.y^2)
```

3D では:
```
|v| = sqrt(v.x^2 + v.y^2 + v.z^2)
```

**最適化**: 実距離が不要で比較だけしたい場合は、
高コストな平方根を避けるために二乗長を使います:

```
|v|^2 = v.x^2 + v.y^2
```

#### 正規化

同じ方向を向く単位ベクトル（長さ1）を生成します:

```
normalize(v) = v / |v| = (v.x / |v|, v.y / |v|)
```

正規化ベクトルは純粋な方向を表します。ゼロ除算を避けるため、
割る前に必ず `|v| > 0` を確認してください。

**例**: エンティティが向いている方向を得る:
```
facing = normalize(target - self.position)
```

#### ドット積

2つのベクトルの角度関係を表すスカラー値です:

```
a . b = a.x * b.x + a.y * b.y
```

3D では:
```
a . b = a.x * b.x + a.y * b.y + a.z * b.z
```

幾何学的な解釈:
```
a . b = |a| * |b| * cos(theta)
```

ここで `theta` はベクトル間の角度です。単位ベクトル同士なら:
```
a . b = cos(theta)
```

主な性質:
- `a . b > 0`: ほぼ同じ方向（角度 < 90 度）。
- `a . b == 0`: 直交（角度 = 90 度）。
- `a . b < 0`: ほぼ逆方向（角度 > 90 度）。

**ゲーム開発での用途**:
- 視野判定: プレイヤーは敵の前方にいるか？
- ライティング: 拡散反射光強度の計算（`max(0, dot(normal, lightDir))`）。
- 射影: あるベクトルを別のベクトルへ射影する。

#### クロス積（3D）

入力2ベクトルの両方に垂直なベクトルを生成します:

```
a x b = (
    a.y * b.z - a.z * b.y,
    a.z * b.x - a.x * b.z,
    a.x * b.y - a.y * b.x
)
```

クロス積の大きさは次に等しい:
```
|a x b| = |a| * |b| * sin(theta)
```

2D における「クロス積」はスカラー（3D クロス積の z 成分）です:
```
a x b = a.x * b.y - a.y * b.x
```

**ゲーム開発での用途**:
- 巻き方向の判定（時計回り / 反時計回り）。
- ライティング用の法線計算。
- 点が線分の左側か右側かの判定。

#### 垂直ベクトル（2D）

`(x, y)` に垂直なベクトルを得るには:
```
perp = (-y, x)    // 90 degrees counter-clockwise
perp = (y, -x)    // 90 degrees clockwise
```

2D の辺や壁の法線計算に便利です。

#### 射影

ベクトル `a` をベクトル `b` に射影する:

```
proj_b(a) = (a . b / b . b) * b
```

`b` がすでに単位ベクトルなら:
```
proj_b(a) = (a . b) * b
```

**ゲーム開発での用途**:
- 面法線方向の速度成分を求める（反射/バウンド用）。
- 壁沿い移動: 速度から法線成分を引く。

#### 反射

法線 `n`（単位ベクトル）を持つ面でベクトル `v` を反射する:

```
reflected = v - 2 * (v . n) * n
```

**ゲーム開発での用途**:
- ボールの壁反射。
- 光反射の計算。
- 跳弾軌道の計算。

### 擬似コード -- Vector2D クラス

```
class Vector2D:
    x, y

    function add(other):
        return Vector2D(x + other.x, y + other.y)

    function subtract(other):
        return Vector2D(x - other.x, y - other.y)

    function scale(scalar):
        return Vector2D(x * scalar, y * scalar)

    function magnitude():
        return sqrt(x * x + y * y)

    function magnitudeSquared():
        return x * x + y * y

    function normalize():
        mag = magnitude()
        if mag > 0:
            return Vector2D(x / mag, y / mag)
        return Vector2D(0, 0)

    function dot(other):
        return x * other.x + y * other.y

    function cross(other):
        return x * other.y - y * other.x

    function perpendicular():
        return Vector2D(-y, x)

    function reflect(normal):
        d = dot(normal)
        return Vector2D(x - 2 * d * normal.x, y - 2 * d * normal.y)

    function angleTo(other):
        return acos(normalize().dot(other.normalize()))

    function distanceTo(other):
        return subtract(other).magnitude()

    function lerp(other, t):
        return Vector2D(
            x + (other.x - x) * t,
            y + (other.y - y) * t
        )
```

### ゲーム開発での実践的な用途

- **移動とステアリング**: 位置に速度ベクトルを加算し、方向ベクトルを正規化して
  speed を掛けることで一貫した移動を実現する。
- **距離チェック**: 半径判定などを高速化するため二乗長を使う
  （例: 「この敵は射程内か？」）。
- **視野判定**: エンティティの前方ベクトルとターゲット方向のドット積を使い、
  ターゲットが視野コーン内か判定する。
- **壁沿い滑り**: 速度を壁接線（法線に垂直）へ射影し、表面沿いに滑らかに移動させる。
- **反射とバウンド**: 飛翔体やボールが面に当たったとき反射式を使う。
- **補間**: 2ベクトル間の `lerp`（線形補間）を使って滑らかな移動、
  カメラ追従、アニメーションを行う。
- **回転**: 三角関数を使ってベクトルを角度分回転する:
  ```
  rotated.x = v.x * cos(angle) - v.y * sin(angle)
  rotated.y = v.x * sin(angle) + v.y * cos(angle)
  ```

---

## クイックリファレンステーブル

| Algorithm / Concept | Primary Use Case | Complexity |
|---|---|---|
| Bresenham's Line | グリッドレイキャスティング、視線判定 | レイ1本あたり O(max(dx, dy)) |
| AABB Overlap | 高速な衝突判定 | ペアあたり O(1) |
| Circle Overlap | 円形コライダー判定 | ペアあたり O(1) |
| Separating Axis Theorem | 凸多角形の衝突判定 | ペアあたり O(n)（n = 辺数） |
| Spatial Hashing | 広域フェーズの衝突カリング | 平均探索 O(1) |
| Euler Integration | シンプルな物理ステップ | ボディ・ステップごとに O(1) |
| Verlet Integration | 拘束ベース物理 | ボディ・ステップごとに O(1) |
| Impulse Resolution | 衝突応答 | O(iterations * contacts) |
| Vector Normalization | 方向抽出 | O(1) |
| Dot Product | 角度/射影クエリ | O(1) |
| Cross Product | 垂直性 / 巻き方向 | O(1) |
| Reflection | バウンド / 跳弾 | O(1) |

