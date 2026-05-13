# ゲーム開発の基礎

Webゲーム開発技術、ゲームアーキテクチャ、そしてゲームループの構造を包括的に解説するリファレンスです。

Sources:
- https://developer.mozilla.org/en-US/docs/Games/Introduction
- https://developer.mozilla.org/en-US/docs/Games/Anatomy

---

## ゲーム開発のためのWeb技術

### グラフィックスとレンダリング

- **WebGL** -- OpenGL ES 2.0 をベースにした、ハードウェアアクセラレーション対応の2D/3Dグラフィックス。高性能レンダリングのためにGPUへ直接アクセスできます。
- **Canvas API** -- `<canvas>` 要素を通じた2D描画サーフェス。2Dゲーム、スプライト描画、ピクセル操作に適しています。
- **SVG** -- 解像度に依存しないビジュアルを実現する Scalable Vector Graphics。UI要素やシンプルなベクターベースのゲームに有用です。
- **HTML/CSS** -- ゲームUI、メニュー、HUD、オーバーレイを構築するための標準的なWeb技術です。

### オーディオ

- **Web Audio API** -- リアルタイム再生、音声合成、空間オーディオ、エフェクト処理、動的ミキシングをサポートする高度な音声エンジン。
- **HTML Audio Element** -- BGMや基本的な効果音の再生に適したシンプルな音声再生手段。

### 入力と操作

- **Gamepad API** -- ボタンマッピングやアナログスティック入力を含む、ゲームコントローラー／ゲームパッド対応。
- **Touch Events API** -- モバイルデバイス向けのマルチタッチ入力処理。
- **Pointer Lock API** -- ゲーム領域内にマウスカーソルを固定し、精密なカメラ／エイム操作向けに生の座標差分を提供します。
- **Device Sensors** -- モーションベース入力のための加速度センサーとジャイロスコープへのアクセス。
- **Full Screen API** -- 没入感のあるフルスクリーンのゲーム体験を有効にします。

### ネットワーキングとマルチプレイヤー

- **WebSockets API** -- リアルタイム対戦、チャット、ライブ更新のための永続的な双方向通信チャネル。
- **WebRTC API** -- 低遅延マルチプレイヤー、ボイスチャット、データチャネル向けのP2P接続。
- **Fetch API** -- ゲームアセットのダウンロード、レベルデータの読み込み、非リアルタイムなゲーム状態の送信に使うHTTPリクエスト。

### データ保存とパフォーマンス

- **IndexedDB API** -- セーブデータ、キャッシュ済みアセット、オフラインプレイ対応のためのクライアント側構造化ストレージ。
- **Typed Arrays** -- GLテクスチャ、音声サンプル、コンパクトなゲームデータ向けに、生のバイナリデータバッファへ直接アクセスできます。
- **Web Workers API** -- メインスレッドをブロックせず、重い計算（物理、経路探索、AI）をオフロードするバックグラウンドスレッド実行。

### 言語とコンパイル

- **JavaScript** -- Webゲーム開発の主要言語です。
- **C/C++ via Emscripten** -- 既存のネイティブゲームコードを JavaScript または WebAssembly にコンパイルしてWeb展開できます。
- **WebAssembly (Wasm)** -- パフォーマンスが重要なゲームコードに対して、ネイティブに近い実行速度を提供します。

---

## 作成できるゲームの種類

現代のWebプラットフォームでは、幅広い種類のゲームをサポートできます。

- 3Dアクションゲームやシューティング
- ロールプレイングゲーム（RPG）
- 2Dプラットフォーマーや横スクロールゲーム
- パズルゲームやストラテジーゲーム
- カードゲームやボードゲーム
- カジュアルゲームやモバイル向けゲーム
- リアルタイム通信を伴うマルチプレイヤー体験

---

## Webベースのゲーム開発の利点

1. **ユニバーサルな到達性** -- ゲームはブラウザを通じて、スマートフォン、タブレット、PC、スマートTVで動作します。
2. **アプリストアへの依存なし** -- ストア審査なしでWebに直接デプロイできます。
3. **収益コントロールの自由度** -- 必須の売上分配はなく、任意の決済システムを利用できます。
4. **即時アップデート** -- ストア審査待ちなしで、すぐに更新を反映できます。
5. **分析データを自前で管理可能** -- 独自にデータ収集することも、任意の分析プロバイダーを選ぶことも可能です。
6. **プレイヤーとの直接的な関係** -- 仲介者なしでプレイヤーと関われます。
7. **本質的に共有しやすい** -- 標準的なWebの仕組みでリンク共有・発見が可能です。

---

## ゲームループの構造

すべてのゲームは、次のステップを繰り返す連続サイクルで動作します。

1. **Present** -- 現在のゲーム状態をプレイヤーに表示する。
2. **Accept** -- ユーザー入力（キーボード、マウス、ゲームパッド、タッチ）を受け取る。
3. **Interpret** -- 生の入力を意味のあるゲーム内アクションに変換する。
4. **Calculate** -- アクション、物理、AI、時間に基づいてゲーム状態を更新する。
5. **Repeat** -- 更新後の状態を表示するために先頭へ戻る。

ゲームは **event-driven**（ターン制のようにプレイヤー操作待ち）にも、**per-frame**（メインループで連続更新）にもできます。

---

## requestAnimationFrame でゲームループを構築する

### 基本のメインループ

```javascript
window.main = () => {
  window.requestAnimationFrame(main);

  // Your game logic here: update state, render frame
};

main(); // Start the cycle
```

重要なポイント:
- `requestAnimationFrame()` はコールバックをブラウザの再描画スケジュール（通常 60 Hz）に同期させます。
- 利用可能な計算時間を最大化するため、ループ処理を行う **前に** 次フレームを予約します。

### 自己完結型メインループ（IIFE）

```javascript
;(() => {
  function main() {
    window.requestAnimationFrame(main);

    // Game logic here
  }

  main();
})();
```

### 停止可能なメインループ

```javascript
;(() => {
  function main() {
    MyGame.stopMain = window.requestAnimationFrame(main);

    // Game logic here
  }

  main();
})();

// To stop the loop:
window.cancelAnimationFrame(MyGame.stopMain);
```

---

## タイミングとフレームレート

### DOMHighResTimeStamp

`requestAnimationFrame` はコールバックに `DOMHighResTimeStamp` を渡し、1ミリ秒の1/1000単位までの高精度な時刻を提供します。

```javascript
;(() => {
  function main(tFrame) {
    MyGame.stopMain = window.requestAnimationFrame(main);

    // tFrame is a high-resolution timestamp in milliseconds
    // Use it for delta-time calculations
  }

  main();
})();
```

### フレーム時間の予算

60 Hz では、各フレームで利用できる処理時間はおよそ **16.67ms** です。ブラウザのフレームサイクルは次の通りです。

1. 新しいフレーム開始（前フレームが画面表示済み）
2. `requestAnimationFrame` のコールバックを実行
3. ガベージコレクションとフレームごとのブラウザ処理を実行
4. VSync まで待機して繰り返し

---

## シンプルな更新・描画パターン

ゲームが目標フレームレートを維持できる場合の最も単純なアプローチ:

```javascript
;(() => {
  function main(tFrame) {
    MyGame.stopMain = window.requestAnimationFrame(main);

    update(tFrame); // Process game logic
    render();       // Draw the frame
  }

  main();
})();
```

前提:
- 各フレームで入力処理と状態更新を時間予算内に完了できる。
- シミュレーションはディスプレイのリフレッシュレート（通常 ~60 FPS）と同じ速度で動作する。
- フレーム補間は不要。

---

## 固定タイムステップによる更新と描画の分離

可変リフレッシュレートへの堅牢な対応と、一貫したシミュレーション挙動のための構成:

```javascript
;(() => {
  function main(tFrame) {
    MyGame.stopMain = window.requestAnimationFrame(main);
    const nextTick = MyGame.lastTick + MyGame.tickLength;
    let numTicks = 0;

    // Calculate how many simulation updates are needed
    if (tFrame > nextTick) {
      const timeSinceTick = tFrame - MyGame.lastTick;
      numTicks = Math.floor(timeSinceTick / MyGame.tickLength);
    }

    queueUpdates(numTicks);
    render(tFrame);
    MyGame.lastRender = tFrame;
  }

  function queueUpdates(numTicks) {
    for (let i = 0; i < numTicks; i++) {
      MyGame.lastTick += MyGame.tickLength;
      update(MyGame.lastTick);
    }
  }

  MyGame.lastTick = performance.now();
  MyGame.lastRender = MyGame.lastTick;
  MyGame.tickLength = 50; // 20 Hz simulation rate (50ms per tick)

  setInitialState();
  main(performance.now());
})();
```

利点:
- **決定論的シミュレーション** -- 表示リフレッシュレートに関係なく、ゲームロジックは固定周波数で実行されます。
- **滑らかな描画** -- 見た目を滑らかにするため、描画時にシミュレーション状態間を補間できます。
- **移植性の高い挙動** -- 60 Hz、120 Hz、144 Hz のディスプレイでも同じ挙動になります。

---

## 代替アーキテクチャパターン

### 更新に setInterval を分離使用

```javascript
// Game logic updates at a fixed rate
setInterval(() => {
  update();
}, 50); // 20 Hz

// Rendering synchronized to display
requestAnimationFrame(function render(tFrame) {
  requestAnimationFrame(render);
  draw();
});
```

欠点: `setInterval` はタブが非表示でも実行され続け、リソースを浪費します。

### 更新に Web Worker を使用

```javascript
// Heavy game logic runs in a background thread
const updateWorker = new Worker('game-update-worker.js');

requestAnimationFrame(function render(tFrame) {
  requestAnimationFrame(render);
  updateWorker.postMessage({ ticks: numTicksNeeded });
  draw();
});
```

利点: メインスレッドをブロックしません。物理演算が重いゲームやAI負荷の高いゲームに最適です。  
欠点: worker とメインスレッド間の通信オーバーヘッドがあります。

### requestAnimationFrame で Web Worker を駆動

```javascript
;(() => {
  function main(tFrame) {
    MyGame.stopMain = window.requestAnimationFrame(main);

    // Signal worker to compute updates
    updateWorker.postMessage({
      lastTick: MyGame.lastTick,
      numTicks: calculatedNumTicks
    });

    render(tFrame);
  }

  main();
})();
```

利点: 従来型タイマーに依存せず、worker が並列で計算を実行します。

---

## タブのフォーカス喪失への対処

ブラウザタブがフォーカスを失うと、`requestAnimationFrame` は大幅に遅くなるか完全に停止します。対策:

| Strategy | Description | Best For |
|---|---|---|
| Treat gap as pause | 経過時間をスキップし、更新しない | シングルプレイヤーゲーム |
| Simulate the gap | 復帰時に未処理更新をすべて実行 | シンプルなシミュレーション |
| Sync from server/peer | 権威的な状態を取得する | マルチプレイヤーゲーム |

フォーカス復帰イベント後の `numTicks` 値を監視してください。非常に大きな値であれば、ゲームが停止状態だった可能性が高く、欠落フレームをすべて再計算するのではなく特別な処理が必要です。

---

## タイミング手法の比較

| Approach | Pros | Cons |
|---|---|---|
| Simple update/render per frame | 実装が容易で応答性が高い | 低速／高速ハードウェアで破綻しやすい |
| Fixed timestep + interpolation | 一貫したシミュレーションと滑らかな表示 | 実装が複雑になる |
| Quality scaling | フレームレートを動的に維持できる | 適応的な品質制御システムが必要 |

---

## パフォーマンスのベストプラクティス

- **フレームに非クリティカルなコードを分離** し、UI・ネットワーク応答・その他の非同期処理にはイベントやコールバックを使う。
- **Web Workers を活用** して、物理、経路探索、AI など計算コストの高い処理をオフロードする。
- **GPUアクセラレーションを活用** するため、レンダリングに WebGL を使う。
- **フレーム予算内に収める** -- 60 FPS を維持するには update + render 時間を 16.67ms 未満に保つ。
- **ガベージコレクション負荷を抑える** ため、オブジェクト再利用とフレームごとの割り当て回避を徹底する。
- **タイミング戦略は早期に設計する** -- 開発途中でゲームループのアーキテクチャを変更するのは難しく、エラーの原因になりやすい。

---

## 人気の3Dフレームワークとライブラリ

- **Three.js** -- 大規模なエコシステムを持つ汎用3Dライブラリ。
- **Babylon.js** -- 物理、オーディオ、シーン管理を備えたフル機能の3Dゲームエンジン。
- **A-Frame** -- Three.js を基盤とした宣言的な3D/VRフレームワーク。
- **PlayCanvas** -- ビジュアルエディタを備えたクラウドホスト型3Dゲームエンジン。
- **Phaser** -- 物理演算と入力処理を備えた人気の2Dゲームフレームワーク。

