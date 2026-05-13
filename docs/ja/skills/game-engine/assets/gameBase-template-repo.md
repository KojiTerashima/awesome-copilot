# GameBase テンプレートリポジトリ

**Haxe** と **Heaps** ゲームエンジンで構築する 2D ゲームプロジェクト向けの、高機能で思想がはっきりしたスターターテンプレートです。*Dead Cells* のリード開発者である **Sebastien Benard**（deepnight）が作成・保守しています。GameBase は、エンティティ管理、LDtk によるレベル統合、レンダリングパイプライン、ゲームループアーキテクチャを備えた本番実績のある土台を提供し、開発者がボイラープレートを省いてゲーム固有ロジックの実装にすぐ入れるように設計されています。

**Repository:** [github.com/deepnight/gameBase](https://github.com/deepnight/gameBase)
**Author:** [Sebastien Benard / deepnight](https://deepnight.net)
**Technology:** Haxe + Heaps (HashLink or JS targets)
**Level editor integration:** [LDtk](https://ldtk.io)

---

## 目的

GameBase は「空のプロジェクト」問題を解決するために存在します。レンダリング、エンティティシステム、カメラ制御、デバッグオーバーレイ、レベル読み込みをゼロから構築する代わりに、開発者はこのリポジトリをクローンして、すぐにゲーム固有のメカニクス実装を始められます。ここには、商用ゲーム開発、とくに *Dead Cells* 開発で洗練されたパターンが反映されています。

主な利点:
- グリッドベース座標とサブピクセル精度を備えた事前構築済みエンティティシステム
- ビジュアルなレベル設計のための LDtk レベルエディタ統合
- 組み込みのデバッグツールとオーバーレイ
- 固定ステップ更新によるフレームレート非依存のゲームループ
- 追従、シェイク、ズーム、クランプに対応したカメラシステム
- 設定可能な Controller/入力管理
- Heaps によるスケーラブルなレンダリングパイプライン

---

## リポジトリ構成

```
gameBase/
  src/
    game/
      App.hx              -- Application entry point and initialization
      Game.hx             -- Main game process, holds level and entities
      Entity.hx           -- Base entity class with grid coords, velocity, animation
      Level.hx            -- Level loading and collision map from LDtk
      Camera.hx           -- Camera follow, shake, zoom, clamping
      Fx.hx               -- Visual effects (particles, flashes, etc.)
      Types.hx            -- Enums, typedefs, and constants
      en/
        Hero.hx            -- Player entity (example implementation)
        Mob.hx             -- Enemy entity (example implementation)
    import.hx             -- Global imports (available everywhere)
  res/
    atlas/                 -- Sprite sheets and texture atlases
    levels/                -- LDtk level project files
    fonts/                 -- Bitmap fonts
  .ldtk                   -- LDtk project file (root)
  build.hxml              -- Haxe compiler configuration
  Makefile                -- Build/run shortcuts
  README.md
```

---

## 主要ファイルとその役割

### `src/game/App.hx` -- アプリケーションエントリポイント

`dn.Process` を継承したメインアプリケーションクラス。以下を担当します:
- ウィンドウ/表示の初期化
- シーン管理（ルートシーングラフ）
- グローバル入力コントローラの設定
- デバッグ切り替えとコンソール

```haxe
class App extends dn.Process {
  public static var ME : App;

  override function init() {
    ME = this;
    // Initialize rendering, controller, assets
    new Game();
  }
}
```

### `src/game/Game.hx` -- ゲームプロセス

アクティブなゲームセッションを管理します:
- 現在の `Level` への参照を保持
- すべてのアクティブな `Entity` インスタンスを管理（グローバル連結リスト経由）
- ポーズ、ゲームオーバー、リスタートのロジックを処理
- カメラとエフェクトを統括

```haxe
class Game extends dn.Process {
  public var level : Level;
  public var hero : en.Hero;
  public var fx : Fx;
  public var camera : Camera;

  public function new() {
    super(App.ME);
    level = new Level();
    fx = new Fx();
    camera = new Camera();
    hero = new en.Hero();
  }
}
```

### `src/game/Entity.hx` -- ベースエンティティ

中核となるエンティティクラスで、以下の機能を備えます:
- **グリッドベース位置管理:** `cx`, `cy`（整数セル座標）に加え、`xr`, `yr`（0.0〜1.0 のセル内比率）で滑らかなサブピクセル移動を実現
- **速度と摩擦:** `dx`, `dy`（速度）と設定可能な `frictX`, `frictY`
- **重力:** エンティティごとに任意の重力設定
- **スプライト管理:** Heaps の `h2d.Anim` または `dn.heaps.HSprite` によるアニメーションスプライト
- **ライフサイクル:** `update()`, `fixedUpdate()`, `postUpdate()`, `dispose()`
- **衝突ヘルパー:** レベルの衝突マップに対する `hasCollision(cx, cy)` チェック

```haxe
class Entity {
  // Grid position
  public var cx : Int = 0;   // Cell X
  public var cy : Int = 0;   // Cell Y
  public var xr : Float = 0.5; // X ratio within cell (0..1)
  public var yr : Float = 1.0; // Y ratio within cell (0..1)

  // Velocity
  public var dx : Float = 0;
  public var dy : Float = 0;

  // Pixel position (computed)
  public var attachX(get,never) : Float;
  inline function get_attachX() return (cx + xr) * Const.GRID;
  public var attachY(get,never) : Float;
  inline function get_attachY() return (cy + yr) * Const.GRID;

  // Physics step
  public function fixedUpdate() {
    xr += dx;
    dx *= frictX;

    // X collision
    if (xr > 1) { cx++; xr--; }
    if (xr < 0) { cx--; xr++; }

    yr += dy;
    dy *= frictY;

    // Y collision
    if (yr > 1) { cy++; yr--; }
    if (yr < 0) { cy--; yr++; }
  }
}
```

### `src/game/Level.hx` -- レベル管理

LDtk プロジェクトファイルからレベルデータを読み込み、管理します:
- タイルレイヤー、エンティティレイヤー、int grid レイヤーを解析
- 衝突グリッド（`hasCollision(cx, cy)`）を構築
- レベル構造を問い合わせるためのヘルパーメソッドを提供

```haxe
class Level {
  var data : ldtk.Level;
  var collisions : Map<Int, Bool>;

  public function new(ldtkLevel) {
    data = ldtkLevel;
    // Parse IntGrid layer for collision marks
    for (cy in 0...data.l_Collisions.cHei)
      for (cx in 0...data.l_Collisions.cWid)
        if (data.l_Collisions.getInt(cx, cy) == 1)
          collisions.set(coordId(cx, cy), true);
  }

  public inline function hasCollision(cx:Int, cy:Int) : Bool {
    return collisions.exists(coordId(cx, cy));
  }
}
```

### `src/game/Camera.hx` -- カメラシステム

以下を提供します:
- **ターゲット追従:** 設定可能なデッドゾーン付きでエンティティを滑らかに追従
- **シェイク:** 減衰付きの画面揺れ
- **ズーム:** 動的なズームイン/ズームアウト
- **クランプ:** カメラをレベル境界内に維持

### `src/game/Fx.hx` -- エフェクトシステム

パーティクルおよびビジュアルエフェクト管理:
- パーティクルプール
- 画面フラッシュ
- スローモーションヘルパー
- カラーオーバーレイエフェクト

---

## 技術スタック

### Haxe

複数ターゲットにコンパイル可能なクロスプラットフォーム高級言語:
- **HashLink (HL):** デスクトップ向けネイティブバイトコード VM（主要開発ターゲット）
- **JavaScript (JS):** ブラウザ/Web ターゲット
- **C/C++:** HXCPP 経由でネイティブビルド

### Heaps (Heaps.io)

高性能なクロスプラットフォーム 2D/3D ゲームエンジン:
- OpenGL/DirectX/WebGL による GPU アクセラレーテッドレンダリング
- `h2d.Object` 階層によるシーングラフアーキテクチャ
- スプライトバッチングとテクスチャアトラス
- ビットマップフォントレンダリング
- 入力抽象化

### LDtk

Sebastien Benard が作成した、モダンなオープンソース 2D レベルエディタ:
- ビジュアルなタイルベースレベル設計
- 衝突やメタデータのための IntGrid レイヤー
- ゲームオブジェクト配置用のエンティティレイヤー
- オートタイルルール
- プロジェクトファイルから自動生成される Haxe API

---

## セットアップ手順

### 前提条件

1. **Haxe をインストール** (4.0+): [haxe.org](https://haxe.org/download/)
2. **HashLink をインストール**（デスクトップターゲット用）: [hashlink.haxe.org](https://hashlink.haxe.org/)
3. **LDtk をインストール**（レベル編集用）: [ldtk.io](https://ldtk.io/)

### はじめ方

```bash
# Clone the repository
git clone https://github.com/deepnight/gameBase.git my-game
cd my-game

# Install Haxe dependencies
haxelib install heaps
haxelib install deepnightLibs
haxelib install ldtk-haxe-api

# Build and run (HashLink target)
haxe build.hxml
hl bin/client.hl

# Or use the Makefile (if available)
make run
```

### スタートポイントとして使う

1. **クローンまたはテンプレートを使用** -- Fork はせず、ゲーム名の新しいディレクトリにクローンします。
2. **パッケージ名を変更** -- `src/game/` の package 宣言とプロジェクト参照をゲームに合わせて更新します。
3. **`build.hxml` を編集** -- 必要に応じてメインクラス、出力パス、ターゲットを調整します。
4. **LDtk でレベル設計** -- `.ldtk` ファイルを開き、レイヤーとエンティティを定義してエクスポートします。
5. **エンティティを実装** -- `src/game/en/` に `Entity` を継承した新しいエンティティクラスを作成します。
6. **反復改善** -- デバッグコンソール（ゲーム内で切り替え）を使ってライブで確認・調整します。

---

## ビルドターゲット

| Target | Command | Output | Use Case |
|--------|---------|--------|----------|
| HashLink | `haxe build.hxml` | `bin/client.hl` | 開発、デスクトップリリース |
| JavaScript | `haxe build.js.hxml` | `bin/client.js` | Web/ブラウザビルド |
| DirectX/OpenGL | Via HL native | Native executable | 本番デスクトップリリース |

---

## デバッグ機能

GameBase には組み込みのデバッグツールがあります:
- **デバッグオーバーレイ:** キーで切り替え、エンティティ境界・グリッド・速度・衝突マップを表示
- **コンソール:** フラグ切り替え、テレポート、エンティティ生成ができるゲーム内コマンドコンソール
- **FPS カウンター:** フレームレートと更新レートの可視モニター
- **プロセスインスペクタ:** アクティブなプロセスとその階層を表示

---

## ゲームループアーキテクチャ

GameBase は固定タイムステップのゲームループパターンを採用しています:

```
Each frame:
  1. preUpdate()    -- Input polling, pre-frame logic
  2. fixedUpdate()  -- Physics, movement, collisions (fixed timestep)
     - May run 0-N times per frame to catch up
  3. update()       -- General per-frame logic
  4. postUpdate()   -- Sprite position sync, camera update, rendering prep
```

これにより、フレームレートに関係なく物理挙動の一貫性を保ちつつ、レンダリングや視覚更新は滑らかさを維持できます。

---

## エンティティのライフサイクル

```
Constructor  -->  init()  -->  [game loop: fixedUpdate/update/postUpdate]  -->  dispose()
```

- **Constructor:** 初期位置を設定し、スプライトを作成して、グローバルエンティティリストに登録
- **fixedUpdate():** 物理ステップ（速度、摩擦、重力、衝突）
- **update():** AI、ステートマシン、アニメーショントリガー
- **postUpdate():** スプライト位置をグリッド座標に同期し、視覚効果を適用
- **dispose():** エンティティリストから削除し、スプライトを破棄して参照をクリーンアップ

