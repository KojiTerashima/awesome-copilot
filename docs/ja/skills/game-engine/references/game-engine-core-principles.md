# ゲームエンジンのコア設計原則

ゲームエンジンを構築するうえでの基本アーキテクチャと設計原則を包括的にまとめたリファレンスです。モジュール性、関心の分離、コアサブシステム、実践的な実装ガイダンスを扱います。

出典: https://www.gamedev.net/articles/programming/general-and-gameplay-programming/making-a-game-engine-core-design-principles-r3210/

---

## なぜゲームエンジンを作るのか

ゲームエンジンは、ゲーム開発に必要な共通システムを抽象化した再利用可能なソフトウェアフレームワークです。プロジェクトごとにレンダリング、物理、入力、オーディオをゼロから書く代わりに、よく設計されたエンジンはそれらをモジュール化され設定可能なサブシステムとして提供します。

主な動機:
- **再利用性** -- 複数のゲームプロジェクトで同じコードベースを使える。
- **エンジンコードとゲームコードの分離** -- エンジン開発者とゲームデザイナーが独立して作業できる。
- **保守性** -- 構造化されたコードはデバッグ・拡張・最適化がしやすい。
- **拡張性** -- 既存システムを書き直さずに新機能や新プラットフォームを追加できる。

---

## コア設計原則

### モジュール性

エンジン内の主要な各システムは、明確に定義されたインターフェースを持つ独立モジュールであるべきです。モジュール同士は内部実装に直接触れるのではなく、クリーンな API を通じて連携するべきです。

**重要な理由:**
- 他システムに影響を与えず実装を差し替えられる（例: OpenGL レンダラーを Vulkan に置き換える）。
- 個々のシステムを分離してテストできる。
- チームが別々のモジュールを並行して開発できる。

**構成例:**

```
engine/
  core/           -- メモリ、ロギング、数学、ユーティリティ
  platform/       -- OS 抽象化、ウィンドウ処理、ファイル I/O
  renderer/       -- グラフィックス API、シェーダー、マテリアル
  physics/        -- 衝突、剛体ダイナミクス
  audio/          -- 音声再生、ミキシング、空間オーディオ
  input/          -- キーボード、マウス、ゲームパッド、タッチ
  scripting/      -- スクリプト言語バインディング
  scene/          -- シーングラフ、エンティティ管理
  resources/      -- アセット読み込み、キャッシュ、ストリーミング
```

### 関心の分離

各システムは、単一で明確に定義された責務を持つべきです。レンダリングロジックと物理、入力処理とゲーム状態管理を混在させるべきではありません。

**実践ガイドライン:**
- レンダラーはゲームメカニクスを知るべきではない。
- 物理エンジンはエンティティの描画方法を知るべきではない。
- 入力処理は、生のデバイスイベントをゲームコードが消費できる抽象アクションへ変換するべき。
- ゲームロジック層はエンジンの上に乗り、エンジンサービスを利用するが、エンジン自体は改変しない。

### データ駆動設計

可能な限り、挙動はハードコードされたロジックではなくデータで制御するべきです。これにより、デザイナーやアーティストはコードを再コンパイルせずにゲームの挙動を変更できます。

**データ駆動アプローチの例:**
- レベルレイアウトはコードではなくデータファイル（JSON, XML, binary）で定義する。
- エンティティのプロパティや挙動はコンポーネントデータで設定する。
- シェーダーパラメータはツールで編集可能なマテリアルプロパティとして公開する。
- アニメーション状態機械は命令的コードではなく設定で定義する。

### 依存関係の最小化

各モジュールは、できるだけ少ない他モジュールに依存するべきです。依存グラフは絡み合った網ではなく、クリーンな階層構造であるべきです。

```
Game Code
    |
    v
Engine High-Level Systems (Scene, Entity, Scripting)
    |
    v
Engine Low-Level Systems (Renderer, Physics, Audio, Input)
    |
    v
Engine Core (Memory, Math, Logging, Platform Abstraction)
    |
    v
Operating System / Hardware
```

モジュール間の循環依存は設計不良の兆候であり、排除すべきです。

---

## Entity-Component-System (ECS) パターン

ECS は、継承よりもコンポジションを重視する、現代のゲームエンジンで広く採用されているアーキテクチャパターンです。

### コア概念

- **Entity** -- ゲームオブジェクトを表す一意識別子（多くの場合は整数 ID）。Entity 自体は挙動やデータを持たない。
- **Component** -- Entity に付与される単純なデータコンテナ。各 Component 型は Entity の状態の一側面（position, velocity, sprite, health など）を保持する。
- **System** -- 特定の Component 集合を持つすべての Entity を処理する関数またはオブジェクト。ロジックは System に、データは Component に置く。

### 継承より ECS を使う理由

従来のオブジェクト指向継承は、硬直した深い階層を生みます:

```
GameObject
  -> MovableObject
    -> Character
      -> Player
      -> Enemy
        -> FlyingEnemy
        -> GroundEnemy
```

このアプローチの問題点:
- 複数ブランチの特性を組み合わせる新しい Entity 型を追加するには、階層の再構成や多重継承が必要になる。
- 深い階層は脆く、基底クラスの変更がすべての派生先に波及する。
- クラスに未使用の挙動が時間とともに蓄積する。

ECS はコンポジションでこれらを解決します:

```javascript
// An entity is just an ID
const player = world.createEntity();

// Attach components to define what it is
world.addComponent(player, new Position(100, 200));
world.addComponent(player, new Velocity(0, 0));
world.addComponent(player, new Sprite("player.png"));
world.addComponent(player, new Health(100));
world.addComponent(player, new PlayerInput());

// A "flying enemy" is just a different combination of components
const flyingEnemy = world.createEntity();
world.addComponent(flyingEnemy, new Position(400, 50));
world.addComponent(flyingEnemy, new Velocity(0, 0));
world.addComponent(flyingEnemy, new Sprite("bat.png"));
world.addComponent(flyingEnemy, new Health(30));
world.addComponent(flyingEnemy, new AIBehavior("patrol_fly"));
world.addComponent(flyingEnemy, new Flying());
```

### System は Component を処理する

```javascript
// Movement system: processes all entities with Position + Velocity
function movementSystem(world, deltaTime) {
  for (const [entity, pos, vel] of world.query(Position, Velocity)) {
    pos.x += vel.x * deltaTime;
    pos.y += vel.y * deltaTime;
  }
}

// Render system: processes all entities with Position + Sprite
function renderSystem(world, context) {
  for (const [entity, pos, sprite] of world.query(Position, Sprite)) {
    context.drawImage(sprite.image, pos.x, pos.y);
  }
}

// Gravity system: only affects entities with Velocity but NOT Flying
function gravitySystem(world, deltaTime) {
  for (const [entity, vel] of world.query(Velocity).without(Flying)) {
    vel.y += 9.8 * deltaTime;
  }
}
```

### ECS の利点

- **柔軟なコンポジション** -- コードを変更せず、Component を組み合わせて任意の Entity 型を作れる。
- **キャッシュ効率の高いデータ配置** -- メモリ上で Component を連続配置すると CPU キャッシュ性能が向上する。
- **並列性** -- 異なる Component 集合を扱う System は並列実行できる。
- **シリアライズしやすい** -- Component は単純データなので、セーブ/ロードを素直に実装できる。

---

## エンジンのコアサブシステム

### メモリ管理

ゲームエンジンの性能において、独自のメモリ管理は極めて重要です。デフォルトアロケータ（malloc/new）は汎用目的であり、ゲームワークロード向けに最適化されていません。

**一般的な割り当て戦略:**

- **Stack Allocator** -- 一時的でフレーム単位のデータ向けに高速な LIFO 割り当て。各フレーム末尾でスタックポインタをリセットする。
- **Pool Allocator** -- 同一型オブジェクト（entities, components, particles）向けの固定サイズブロック割り当て。断片化ゼロ。
- **Frame Allocator** -- 毎フレームリセットする線形アロケータ。フレーム内の一時データに最適。
- **Double-Buffered Allocator** -- 2 つのフレームアロケータをフレームごとに交互利用し、前フレームのデータを保持できる。

```cpp
// Conceptual frame allocator
class FrameAllocator {
    char* buffer;
    size_t offset;
    size_t capacity;

public:
    void* allocate(size_t size) {
        void* ptr = buffer + offset;
        offset += size;
        return ptr;
    }

    void reset() {
        offset = 0;  // All allocations freed instantly
    }
};
```

### リソース管理

リソースマネージャーは、ゲームアセットの読み込み、キャッシュ、ライフタイム管理を担います。

**主な責務:**
- **非同期読み込み** -- ゲームループの停止を避けるため、バックグラウンドスレッドでアセットを読み込む。
- **参照カウント** -- 何個のシステムがアセットを使用中か追跡し、参照がなくなれば解放する。
- **キャッシュ** -- 最近使ったアセットをメモリに保持し、重複したディスク読み込みを避ける。
- **ホットリロード** -- 開発中にディスク上の変更を検知し、実行時に再読み込みする。
- **リソースハンドル** -- 生ポインタではなくハンドル（ID やスマートポインタ）で参照する。

```javascript
class ResourceManager {
  constructor() {
    this.cache = new Map();
    this.loading = new Map();
  }

  async load(path) {
    // Return cached resource if available
    if (this.cache.has(path)) {
      return this.cache.get(path);
    }

    // Avoid duplicate loads
    if (this.loading.has(path)) {
      return this.loading.get(path);
    }

    // Start async load
    const promise = this._loadFromDisk(path).then(resource => {
      this.cache.set(path, resource);
      this.loading.delete(path);
      return resource;
    });

    this.loading.set(path, promise);
    return promise;
  }

  unload(path) {
    this.cache.delete(path);
  }
}
```

### レンダリングパイプライン

レンダリングサブシステムは、ゲームの視覚状態を画面上のピクセルへ変換します。

**典型的なレンダリングパイプラインの段階:**

1. **シーン走査** -- シーングラフをたどる、または ECS から描画可能 Entity を問い合わせる。
2. **Frustum culling** -- カメラ視野外のオブジェクトを除外する。
3. **Occlusion culling** -- 他のジオメトリに隠れたオブジェクトを除外する。
4. **ソート** -- マテリアル、深度、透明度要件で描画順を整える。
5. **バッチ化** -- 同一マテリアルのオブジェクトをまとめ、draw call と状態変更を最小化する。
6. **頂点処理** -- 頂点をモデル空間からスクリーン空間へ変換する（vertex shader）。
7. **ラスタライズ** -- 三角形をフラグメント（ピクセル）へ変換する。
8. **フラグメント処理** -- ライティング、テクスチャ、エフェクトで最終ピクセル色を計算する（fragment shader）。
9. **ポストプロセス** -- bloom、tone mapping、anti-aliasing などのスクリーンスペース効果を適用する。

**Render command パターン:**

draw call を直接発行する代わりに、送信前にソート・バッチ化できる render command のリストを構築します:

```javascript
class RenderCommand {
  constructor(mesh, material, transform, sortKey) {
    this.mesh = mesh;
    this.material = material;
    this.transform = transform;
    this.sortKey = sortKey;
  }
}

class Renderer {
  constructor() {
    this.commandQueue = [];
  }

  submit(command) {
    this.commandQueue.push(command);
  }

  flush(context) {
    // Sort by material to minimize state changes
    this.commandQueue.sort((a, b) => a.sortKey - b.sortKey);

    for (const cmd of this.commandQueue) {
      this._bindMaterial(cmd.material);
      this._setTransform(cmd.transform);
      this._drawMesh(cmd.mesh, context);
    }

    this.commandQueue.length = 0;
  }
}
```

### 物理統合

物理サブシステムは、物理挙動をシミュレートし衝突を検出します。

**主要な設計上の考慮点:**

- **固定タイムステップ** -- 物理更新は描画フレームレートから独立した固定レート（例: 50 Hz）で行うべき。これにより決定論的なシミュレーション挙動を保証できる。
- **衝突フェーズ** -- まず broad phase（空間分割、境界ボリューム階層）で非衝突ペアを高速に除外し、その後 narrow phase で正確な交差判定を行う。
- **物理ワールドの分離** -- 物理ワールドはゲーム Entity とは分離された独自表現（physics body）を持つべき。同期ステップで両者を対応付ける。

```javascript
class PhysicsWorld {
  constructor(fixedTimestep = 1 / 50) {
    this.fixedTimestep = fixedTimestep;
    this.accumulator = 0;
    this.bodies = [];
  }

  update(deltaTime) {
    this.accumulator += deltaTime;

    while (this.accumulator >= this.fixedTimestep) {
      this.step(this.fixedTimestep);
      this.accumulator -= this.fixedTimestep;
    }
  }

  step(dt) {
    // Integrate velocities
    for (const body of this.bodies) {
      body.velocity.y += body.gravity * dt;
      body.position.x += body.velocity.x * dt;
      body.position.y += body.velocity.y * dt;
    }

    // Detect and resolve collisions
    this.broadPhase();
    this.narrowPhase();
    this.resolveCollisions();
  }
}
```

### 入力システム

入力システムは、生のハードウェアイベントをゲームに意味のあるアクションへ変換します。

**レイヤード設計:**

1. **Hardware Layer** -- OS から生イベント（キー押下、マウス移動、ボタン押下）を受け取る。
2. **Mapping Layer** -- 設定可能なバインディングで生入力を名前付きアクションへ変換する（例: "Space" -> "Jump", "W" -> "MoveForward"）。
3. **Action Layer** -- ゲームコードが参照する抽象アクションを提供し、特定ハードウェア入力から完全に分離する。

```javascript
class InputManager {
  constructor() {
    this.bindings = new Map();
    this.actionStates = new Map();
  }

  bind(action, key) {
    this.bindings.set(key, action);
  }

  handleKeyDown(event) {
    const action = this.bindings.get(event.code);
    if (action) {
      this.actionStates.set(action, true);
    }
  }

  handleKeyUp(event) {
    const action = this.bindings.get(event.code);
    if (action) {
      this.actionStates.set(action, false);
    }
  }

  isActionActive(action) {
    return this.actionStates.get(action) || false;
  }
}

// Usage
const input = new InputManager();
input.bind("Jump", "Space");
input.bind("MoveLeft", "KeyA");
input.bind("MoveRight", "KeyD");

// In game update:
if (input.isActionActive("Jump")) {
  player.jump();
}
```

### イベントシステム

イベントシステムは、直接参照なしでエンジンサブシステムとゲームコード間の疎結合な通信を可能にします。

**Publish-subscribe パターン:**

```javascript
class EventBus {
  constructor() {
    this.listeners = new Map();
  }

  on(eventType, callback) {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    this.listeners.get(eventType).push(callback);
  }

  off(eventType, callback) {
    const callbacks = this.listeners.get(eventType);
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index !== -1) callbacks.splice(index, 1);
    }
  }

  emit(eventType, data) {
    const callbacks = this.listeners.get(eventType);
    if (callbacks) {
      for (const callback of callbacks) {
        callback(data);
      }
    }
  }
}

// Usage
const events = new EventBus();

events.on("collision", (data) => {
  console.log(`${data.entityA} collided with ${data.entityB}`);
});

events.on("entityDestroyed", (data) => {
  spawnExplosion(data.position);
  addScore(data.points);
});

// Emit from physics system
events.emit("collision", { entityA: player, entityB: wall });
```

**遅延イベント:**

性能と決定性のために、イベントはフレーム中にキューへ積み、更新サイクル内の特定タイミングで配信できます:

```javascript
class DeferredEventBus extends EventBus {
  constructor() {
    super();
    this.eventQueue = [];
  }

  queue(eventType, data) {
    this.eventQueue.push({ type: eventType, data });
  }

  dispatchQueued() {
    for (const event of this.eventQueue) {
      this.emit(event.type, event.data);
    }
    this.eventQueue.length = 0;
  }
}
```

### シーン管理

シーンマネージャーはゲームコンテンツを論理的なグループに整理し、異なるゲーム状態間の遷移を管理します。

**一般的なパターン:**

- **Scene graph** -- 子の transform が親に相対となる階層ツリー。親を動かすとすべての子も動く。
- **Scene stack** -- シーンを push/pop できる。ポーズメニューはゲームプレイの上に積み、閉じるとゲームプレイに戻る。
- **Scene loading** -- シーンが読み込むアセットと Entity を定義し、シーンマネージャーが読み込み・初期化・クリーンアップを調整する。

```javascript
class SceneManager {
  constructor() {
    this.scenes = new Map();
    this.activeScene = null;
  }

  register(name, scene) {
    this.scenes.set(name, scene);
  }

  async switchTo(name) {
    if (this.activeScene) {
      this.activeScene.onExit();
      this.activeScene.unloadResources();
    }

    this.activeScene = this.scenes.get(name);
    await this.activeScene.loadResources();
    this.activeScene.onEnter();
  }

  update(deltaTime) {
    if (this.activeScene) {
      this.activeScene.update(deltaTime);
    }
  }

  render(context) {
    if (this.activeScene) {
      this.activeScene.render(context);
    }
  }
}
```

---

## プラットフォーム抽象化

よく設計されたエンジンは、プラットフォーム固有コードを統一インターフェースの背後に隠蔽します。これにより、複数 OS、グラフィックス API、ハードウェア構成でエンジンを動作させられます。

**抽象化が必要な領域:**

| 項目 | 例 |
|---|---|
| ウィンドウ管理 | Win32, X11, Cocoa, SDL, GLFW |
| グラフィックス API | OpenGL, Vulkan, DirectX, Metal, WebGL |
| ファイル I/O | POSIX, Win32, 仮想ファイルシステム |
| スレッド | pthreads, Win32 threads, Web Workers |
| オーディオ出力 | WASAPI, CoreAudio, ALSA, Web Audio |
| 入力デバイス | DirectInput, XInput, evdev, Gamepad API |

```javascript
// Abstract file system interface
class FileSystem {
  async readFile(path) { throw new Error("Not implemented"); }
  async writeFile(path, data) { throw new Error("Not implemented"); }
  async exists(path) { throw new Error("Not implemented"); }
}

// Web implementation
class WebFileSystem extends FileSystem {
  async readFile(path) {
    const response = await fetch(path);
    return response.arrayBuffer();
  }
}

// Node.js implementation
class NodeFileSystem extends FileSystem {
  async readFile(path) {
    const fs = require("fs").promises;
    return fs.readFile(path);
  }
}
```

---

## 初期化とシャットダウンの順序

エンジンサブシステムは依存順に初期化し、逆順でシャットダウンする必要があります。

**典型的な初期化シーケンス:**

1. コアシステム（ロギング、メモリ、設定）
2. プラットフォーム層（ウィンドウ作成、入力デバイス）
3. レンダリングシステム（グラフィックスコンテキスト、デフォルトリソース）
4. オーディオシステム
5. 物理システム
6. リソースマネージャー（デフォルト/共有アセットの読み込み）
7. シーンマネージャー
8. スクリプティングシステム
9. ゲーム固有の初期化

**シャットダウンではこの順序を逆転** し、依存先が先に破棄されないようにします。

```javascript
class Engine {
  async initialize() {
    this.logger = new Logger();
    this.config = new Config("engine.json");
    this.platform = new Platform();
    await this.platform.createWindow(this.config.window);

    this.renderer = new Renderer(this.platform.canvas);
    this.audio = new AudioSystem();
    this.physics = new PhysicsWorld();
    this.resources = new ResourceManager();
    this.input = new InputManager(this.platform.window);
    this.events = new EventBus();
    this.scenes = new SceneManager();

    this.logger.info("Engine initialized");
  }

  shutdown() {
    this.scenes.cleanup();
    this.resources.unloadAll();
    this.input.cleanup();
    this.physics.cleanup();
    this.audio.cleanup();
    this.renderer.cleanup();
    this.platform.cleanup();
    this.logger.info("Engine shutdown complete");
  }

  run() {
    let lastTime = performance.now();

    const loop = (currentTime) => {
      const deltaTime = (currentTime - lastTime) / 1000;
      lastTime = currentTime;

      this.input.poll();
      this.physics.update(deltaTime);
      this.scenes.update(deltaTime);
      this.events.dispatchQueued();
      this.scenes.render(this.renderer);
      this.renderer.present();

      requestAnimationFrame(loop);
    };

    requestAnimationFrame(loop);
  }
}
```

---

## パフォーマンス原則

### 早すぎる抽象化を避ける

モジュール性は重要ですが、実要件を理解する前の過度なインターフェース設計は不要な複雑さを招きます。まずはシンプルで具体的な実装から始め、実際のユースケースが要求した段階で抽象化へリファクタリングしましょう。

### 最適化の前にプロファイリングする

最適化に時間を使う前に、プロファイリングツールで実際のボトルネックを測定してください。どこで時間が使われているかという直感はしばしば誤ります。

### データ指向設計

オブジェクト指向の抽象化より、データへのアクセス方法に合わせてデータを配置します。同型 Component をメモリ上に連続配置する（Array of Structures ではなく Structure of Arrays）と、CPU キャッシュヒット率が劇的に向上します。

```javascript
// Array of Structures (cache-unfriendly for position-only iteration)
const entities = [
  { position: {x: 0, y: 0}, sprite: "hero.png", health: 100 },
  { position: {x: 5, y: 3}, sprite: "bat.png", health: 30 },
];

// Structure of Arrays (cache-friendly for position-only iteration)
const positions = { x: [0, 5], y: [0, 3] };
const sprites = ["hero.png", "bat.png"];
const healths = [100, 30];
```

### ホットパスでの割り当て最小化

フレームごとの更新中に新規オブジェクト生成やメモリ確保を避けます。バッファの事前確保、オブジェクトプール利用、一時オブジェクトの再利用を行います。

### 操作のバッチ化

文脈切り替え、draw call 準備、キャッシュミスによるオーバーヘッドを減らすため、類似処理をまとめます。次の型へ進む前に、ある型の Entity をまとめて処理します。

---

## 主要原則の要約

| 原則 | 説明 |
|---|---|
| モジュール性 | クリーンなインターフェースを持つ独立サブシステム |
| 関心の分離 | 各システムは単一責務を持つ |
| データ駆動設計 | ハードコードロジックではなくデータで挙動を制御 |
| 継承よりコンポジション | 柔軟な Entity 構築のための ECS パターン |
| 依存関係の最小化 | クリーンで階層的な依存グラフ |
| プラットフォーム抽象化 | プラットフォーム固有コードを統一インターフェースで扱う |
| 固定タイムステップ物理 | フレームレート非依存の決定論的シミュレーション |
| イベント駆動通信 | publish-subscribe による疎結合な連携 |
| データ指向パフォーマンス | アクセスパターンに合わせてメモリ配置を最適化 |
| 最適化前の測定 | 実際のボトルネックをプロファイルして特定する |

