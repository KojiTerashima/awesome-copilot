# ゲーム開発テクニック

MDN Web Docs から編集された、Web ベースのゲームを構築するための重要なテクニックを網羅した包括的なリファレンス。

---

## 非同期スクリプト

**出典:** [MDN - asm.js の非同期スクリプト](https://developer.mozilla.org/en-US/docs/Games/Techniques/Async_scripts)

### それは何ですか

非同期コンパイルにより、JavaScript エンジンはゲームの読み込み中にメインスレッドから asm.js コードをコンパイルし、生成されたマシン コードをキャッシュできます。これにより、後続のロード時の再コンパイルが防止され、ブラウザーに最大限の柔軟性が与えられ、コンパイル プロセスが最適化されます。

### 仕組み

スクリプトが非同期で読み込まれると、メイン スレッドがレンダリングとユーザー インタラクションの処理を継続している間、ブラウザはバックグラウンド スレッドでスクリプトをコンパイルできます。コンパイルされたコードはキャッシュされるため、今後のアクセスでは再コンパイルが完全にスキップされます。

### いつ使用するか

- asm.js コードをコンパイルする中規模または大規模なゲーム。
- 起動時のパフォーマンスが重要なゲーム (事実上すべてのゲーム)。
- ブラウザーがセッション間でコンパイルされたマシン コードをキャッシュしたい場合。

### コード例

**HTML 属性のアプローチ:**```html
<script async src="file.js"></script>
```**JavaScript の動的作成 (デフォルトは非同期):**```javascript
const script = document.createElement("script");
script.src = "file.js";
document.body.appendChild(script);
```**重要:** インライン スクリプトは、`async` 属性を使用しても非同期になりません。これらはすぐにコンパイルされて実行されます。```html
<!-- This is NOT async despite the attribute -->
<script async>
  // Inline JavaScript code
</script>
```**文字列ベースのコードの非同期コンパイルに BLOB URL を使用する:**```javascript
const blob = new Blob([codeString]);
const script = document.createElement("script");
const url = URL.createObjectURL(blob);
script.onload = script.onerror = () => URL.revokeObjectURL(url);
script.src = url;
document.body.appendChild(script);
```重要な洞察は、`src` (`innerHTML` または `textContent` ではなく) を設定すると、非同期コンパイルがトリガーされるということです。

---

## 起動パフォーマンスの最適化

**出典:** [MDN - 起動パフォーマンスの最適化](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Optimizing_startup_performance)

### それは何ですか

Web アプリケーションとゲームの起動と応答の速さを改善し、アプリ、ブラウザー、またはデバイスがフリーズしているようにユーザーに表示されるのを防ぐための戦略のコレクション。

### 仕組み

基本的な原則は、起動時にメインスレッドのブロックを回避することです。作業はバックグラウンド スレッド (Web ワーカー) にオフロードされ、スタートアップ コードは小さなマイクロタスクに分割され、メイン スレッドはユーザー イベントとレンダリングのために解放されます。イベント ループは継続的に循環し続ける必要があります。

### いつ使用するか

- 常に -- これは、すべての Web アプリケーションとゲームに共通する懸念事項です。
- 最初から非同期で構築する方が簡単なため、新しいアプリには重要です。
- 同期読み込みを想定し、リファクタリングが必要なネイティブ アプリを移植する場合に不可欠です。

### 主要なテクニック

**1. `defer` および `async`** を使用したスクリプトの読み込み

HTML 解析のブロックを防止します。```html
<script defer src="app.js"></script>
<script async src="helper.js"></script>
```**2.大量の処理を行う Web ワーカー**

データのフェッチ、デコード、計算をワーカーに移します。これにより、メイン スレッドが UI およびユーザー イベント用に解放されます。

**3.データ処理**

- カスタム実装の代わりに、ブラウザーが提供するデコーダー (画像、ビデオ) を使用します。
- データを順次ではなく、可能な限り並行して処理します。
- アセットのデコーディング (例: JPEG から生のテクスチャ データへ) をワーカーにオフロードします。

**4.リソースの読み込み**

- 起動 HTML の重要なレンダリング パスの外側にスクリプトやスタイルシートを含めないでください。必要な場合にのみロードしてください。
- リソース ヒントを使用します: `preconnect`、`preload`。

**5.コードのサイズと圧縮**

- JavaScript ファイルを縮小します。
- Gzip または Brotli 圧縮を使用します。
- データファイルを最適化して圧縮します。

**6.知覚されたパフォーマンス**

- ユーザーの関心を維持するためにスプラッシュ画面を表示します。
- 重いサイトの進行状況インジケーターを表示します。
- 絶対的な持続時間が同じであっても、時間が速く感じられるようにします。

**7. Emscripten メイン ループ ブロッカー (移植されたアプリ用)**```javascript
emscripten_push_main_loop_blocker();
// Establish functions to execute before main thread continues
// Create queue of functions called in sequence
```### パフォーマンス目標

|メトリック |ターゲット |
|---|---|
|コンテンツの初期外観 | 1～2秒 |
|ユーザーが知覚できる遅延 | 50ms以下 |
|しきい値が遅い | 200ミリ秒を超える |

古いデバイスや遅いデバイスを使用しているユーザーは、開発者よりも長い遅延を経験します。常にそれに応じて最適化してください。

---

## WebRTC データ チャネル

**出典:** [MDN - WebRTC データ チャネル](https://developer.mozilla.org/en-US/docs/Games/Techniques/WebRTC_data_channels)

### それは何ですか

WebRTC データ チャネルを使用すると、アクティブな接続を介してテキストまたはバイナリ データをピアに送信できます。ゲームのコンテキストでは、これにより、プレイヤーは中央サーバーを経由せずに、テキスト チャットやゲーム状態の同期のためにデータを相互に送信できるようになります。

### 仕組み

WebRTC は 2 つのブラウザー間にピアツーピア接続を確立します。確立されると、その接続上でデータ チャネルを開くことができます。データ チャネルには 2 つの種類があります。

**信頼できるチャネル:**
- メッセージがピアに到着することを保証します。
- メッセージの順序を維持する -- メッセージは送信されたのと同じ順序で到着します。
- TCP ソケットに似ています。

**信頼できないチャネル:**
- メッセージの配信については保証しません。
- メッセージは特定の順序で届くとは限りません。
・メッセージが全く届かない場合がございます。
- UDP ソケットに似ています。

### いつ使用するか

- **信頼できるチャネル:** ターンベースのゲーム、チャット、またはすべてのメッセージが順番に到着する必要があるシナリオ。
- **信頼性の低いチャネル:** 保証された配信よりも低レイテンシーが重要なリアルタイム アクション ゲーム (例: 古いデータが欠落データよりも悪い位置更新)。

### ゲームでの使用例

- プレイヤー間のテキストチャットコミュニケーション。
- プレイヤー間でのゲームステータス情報の交換。
- リアルタイムのゲーム状態の同期。
- 専用のゲームサーバーを必要としないピアツーピアのマルチプレイヤー。

### 実装メモ

- WebRTC API は主にオーディオおよびビデオ通信用として知られていますが、堅牢なピアツーピア データ チャネル機能も含まれています。
- 実装を簡素化し、ブラウザの違いを回避するために、ライブラリを推奨します。
- WebRTC の完全なドキュメントは、[MDN WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) で入手できます。

---

## Web ゲーム用オーディオ

**出典:** [MDN - ウェブ ゲーム用オーディオ](https://developer.mozilla.org/en-US/docs/Games/Techniques/Audio_for_Web_Games)

### それは何ですか

オーディオは Web ゲームにフィードバックと雰囲気を提供します。この手法では、デスクトップおよびモバイル プラットフォーム全体でのオーディオの実装、ブラウザーの違いへの対応、および最適化戦略について説明します。

### 仕組み

次の 2 つの主要な API が利用可能です。

1. **HTMLMediaElement** -- 基本的なオーディオ再生用の標準 `<audio>` 要素。
2. **Web Audio API** -- 動的なオーディオ操作、位置決め、正確なタイミングのための高度な API。

### いつ使用するか- シンプルでリニアな再生 (複雑な制御を必要としない BGM) には `<audio>` 要素を使用します。
- Web オーディオ API を使用して、ダイナミックな音楽、3D 空間オーディオ、正確なタイミング、リアルタイム操作を実現します。
- モバイルをターゲットにする場合、または短いサウンド効果が多数ある場合は、オーディオ スプライトを使用します。

### モバイルにおける主な課題

- **自動再生ポリシー:** ブラウザはサウンド付きの自動再生を制限します。再生は、ユーザーがクリックまたはタップして開始する必要があります。
- **音量制御:** モバイル ブラウザでは、OS レベルのユーザー制御を維持するために、プログラムによる音量制御が無効になる場合があります。
- **バッファリング/プリロード:** モバイル ブラウザでは、データ使用量を削減するために、再生開始前にバッファリングが無効になることがよくあります。

### テクニック 1: オーディオ スプライト

CSS スプライトの概念から借用した、複数のオーディオ クリップを 1 つのファイルに結合し、タイムスタンプごとに特定のセクションを再生します。

**HTML:**```html
<audio id="myAudio" src="mysprite.mp3"></audio>
<button data-start="18" data-stop="19">0</button>
<button data-start="16" data-stop="17">1</button>
<button data-start="14" data-stop="15">2</button>
<button data-start="12" data-stop="13">3</button>
<button data-start="10" data-stop="11">4</button>
<button data-start="8" data-stop="9">5</button>
<button data-start="6" data-stop="7">6</button>
<button data-start="4" data-stop="5">7</button>
<button data-start="2" data-stop="3">8</button>
<button data-start="0" data-stop="1">9</button>
```**JavaScript:**```javascript
const myAudio = document.getElementById("myAudio");
const buttons = document.getElementsByTagName("button");
let stopTime = 0;

for (const button of buttons) {
  button.addEventListener("click", () => {
    myAudio.currentTime = button.dataset.start;
    stopTime = Number(button.dataset.stop);
    myAudio.play();
  });
}

myAudio.addEventListener("timeupdate", () => {
  if (myAudio.currentTime > stopTime) {
    myAudio.pause();
  }
});
```**モバイル向けオーディオのプライミング (最初のユーザー インタラクションでトリガー):**```javascript
const myAudio = document.createElement("audio");
myAudio.src = "my-sprite.mp3";
myAudio.play();
myAudio.pause();
```### テクニック 2: Web オーディオ API マルチトラック ミュージック

別々のオーディオ トラックを正確なタイミングでロードして同期します。

**オーディオ コンテキストを作成し、ファイルをロードします:**```javascript
const audioCtx = new AudioContext();

async function getFile(filepath) {
  const response = await fetch(filepath);
  const arrayBuffer = await response.arrayBuffer();
  const audioBuffer = await audioCtx.decodeAudioData(arrayBuffer);
  return audioBuffer;
}
```**同期によるトラック再生:**```javascript
let offset = 0;

function playTrack(audioBuffer) {
  const trackSource = audioCtx.createBufferSource();
  trackSource.buffer = audioBuffer;
  trackSource.connect(audioCtx.destination);

  if (offset === 0) {
    trackSource.start();
    offset = audioCtx.currentTime;
  } else {
    trackSource.start(0, audioCtx.currentTime - offset);
  }

  return trackSource;
}
```**再生ハンドラーで自動再生ポリシーを処理します:**```javascript
playButton.addEventListener("click", () => {
  if (audioCtx.state === "suspended") {
    audioCtx.resume();
  }

  playTrack(track);
  playButton.dataset.playing = true;
});
```### テクニック 3: ビート同期したトラックの再生

シームレスなトランジションを実現するには、新しいトラックを同期して境界を越えます。```javascript
const tempo = 3.074074076; // Time in seconds of your beat/bar

if (offset === 0) {
  source.start();
  offset = context.currentTime;
} else {
  const relativeTime = context.currentTime - offset;
  const beats = relativeTime / tempo;
  const remainder = beats - Math.floor(beats);
  const delay = tempo - remainder * tempo;
  source.start(context.currentTime + delay, relativeTime + delay);
}
```### テクニック 4: 位置オーディオ (3D 空間化)

`PannerNode` を使用して 3D 空間にオーディオを配置します。

- ゲームの世界空間にオブジェクトを配置します。
- 音源の方向と動きを設定します。
- 環境効果（洞窟の残響、水中消音など）を適用します。

WebGL 3D ゲームでオーディオをビジュアル オブジェクトやプレーヤーの視点に結び付けるのに特に役立ちます。

### 意思決定マトリックス

|テクニック |いつ使用する |長所 |短所 |
|---|---|---|---|
|オーディオ スプライト |短い音が多く、モバイル | HTTP リクエストを削減し、モバイル対応 |低ビットレートではシーク精度が低下する |
|基本的な `<audio>` |シンプルなリニア再生 |幅広いサポート |制限された制御、自動再生の制限 |
|ウェブオーディオ API |ダイナミックな音楽、3D ポジショニング、正確なタイミング |フルコントロール、リアルタイム操作、同期 |より複雑なコード |
|位置オーディオ | 3D イマーシブ ゲーム |リアリズム、プレイヤーの没入感 | WebGL コンテキスト認識が必要 |

---

## 2D 衝突検出

**出典:** [MDN - 2D 衝突検出](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection)

### それは何ですか

2D 衝突検出アルゴリズムは、ゲーム エンティティがその形状タイプ (長方形から長方形、長方形から円、円から円など) に基づいて重なり合うか交差するかを判断します。ゲームでは通常、ピクセル完璧な検出ではなく、エンティティをカバーする「ヒットボックス」と呼ばれる単純な汎用形状を使用し、視覚的な精度とパフォーマンスのバランスをとります。

### 仕組み

各アルゴリズムは、2 つの形状間の幾何学的関係をチェックします。重複が検出された場合は、衝突が報告されます。アプローチは形状の種類によって異なります。

### いつ使用するか

- 回転のない単純な長方形エンティティには AABB を使用します。
- 丸いエンティティの場合、または迅速で簡単なチェックが必要な場合は、円衝突を使用します。
- 複雑な凸多角形には分離軸定理 (SAT) を使用します。
- 多数のエンティティがある場合は、広範なフェーズの縮小 (クワッド ツリー、空間ハッシュマップ) を使用します。

### アルゴリズム 1: 軸揃えバウンディング ボックス (AABB)

軸が揃った 2 つの長方形間の衝突検出 (回転なし)。長方形の 4 つの辺の間に隙間がないことを確認することで衝突を検出します。```javascript
class BoxEntity extends BaseEntity {
  width = 20;
  height = 20;

  isCollidingWith(other) {
    return (
      this.position.x < other.position.x + other.width &&
      this.position.x + this.width > other.position.x &&
      this.position.y < other.position.y + other.height &&
      this.position.y + this.height > other.position.y
    );
  }
}
```### アルゴリズム 2: サークル衝突

2 つの円間の衝突検出。 2 つの円の中心点を取得し、それらの間の距離が半径の合計より小さいかどうかを確認します。```javascript
class CircleEntity extends BaseEntity {
  radius = 10;

  isCollidingWith(other) {
    const dx =
      this.position.x + this.radius - (other.position.x + other.radius);
    const dy =
      this.position.y + this.radius - (other.position.y + other.radius);
    const distance = Math.sqrt(dx * dx + dy * dy);
    return distance < this.radius + other.radius;
  }
}
```注: 円の `x` 座標と `y` 座標は左上隅を参照するため、実際の中心を比較するには半径を追加する必要があります。

### アルゴリズム 3: 分離軸定理 (SAT)

任意の 2 つの凸多角形間の衝突を検出する衝突アルゴリズム。これは、各ポリゴンを可能なすべての軸に投影し、重なりをチェックすることによって機能します。いずれかの軸にギャップがある場合、ポリゴンは衝突していません。

SAT は実装がより複雑ですが、任意の凸多角形を処理します。

### 衝突パフォーマンス: ブロードフェーズとナローフェーズ

すべてのエンティティを他のすべてのエンティティに対してテストすると、計算コストが高くなります (O(n^2))。ゲームは衝突検出を 2 つのフェーズに分割します。

**ブロードフェーズ** -- 空間データ構造を使用して、どのエンティティが衝突している可能性があるかを迅速に特定します。
- クアッドツリー
- R ツリー
- 空間ハッシュマップ

**狭いフェーズ** -- 正確な衝突アルゴリズム (AABB、Circle、SAT) を広いフェーズの少数の候補リストにのみ適用します。

### 基本エンジン コード

**衝突視覚化用の CSS:**```css
.entity {
  display: inline-block;
  position: absolute;
  height: 20px;
  width: 20px;
  background-color: blue;
}

.movable {
  left: 50px;
  top: 50px;
  background-color: red;
}

.collision-state {
  background-color: green !important;
}
```**JavaScript 衝突チェッカーとエンティティ システム:**```javascript
const collider = {
  moveableEntity: null,
  staticEntities: [],
  checkCollision() {
    const isColliding = this.staticEntities.some((staticEntity) =>
      this.moveableEntity.isCollidingWith(staticEntity),
    );
    this.moveableEntity.setCollisionState(isColliding);
  },
};

const container = document.getElementById("container");

class BaseEntity {
  ref;
  position;
  constructor(position) {
    this.position = position;
    this.ref = document.createElement("div");
    this.ref.classList.add("entity");
    this.ref.style.left = `${this.position.x}px`;
    this.ref.style.top = `${this.position.y}px`;
    container.appendChild(this.ref);
  }
  shiftPosition(dx, dy) {
    this.position.x += dx;
    this.position.y += dy;
    this.redraw();
  }
  redraw() {
    this.ref.style.left = `${this.position.x}px`;
    this.ref.style.top = `${this.position.y}px`;
  }
  setCollisionState(isColliding) {
    if (isColliding && !this.ref.classList.contains("collision-state")) {
      this.ref.classList.add("collision-state");
    } else if (!isColliding) {
      this.ref.classList.remove("collision-state");
    }
  }
  isCollidingWith(other) {
    throw new Error("isCollidingWith must be implemented in subclasses");
  }
}

document.addEventListener("keydown", (e) => {
  e.preventDefault();
  switch (e.key) {
    case "ArrowLeft":
      collider.moveableEntity.shiftPosition(-5, 0);
      break;
    case "ArrowUp":
      collider.moveableEntity.shiftPosition(0, -5);
      break;
    case "ArrowRight":
      collider.moveableEntity.shiftPosition(5, 0);
      break;
    case "ArrowDown":
      collider.moveableEntity.shiftPosition(0, 5);
      break;
  }
  collider.checkCollision();
});
```---

## タイルマップ

**出典:** [MDN - タイルマップ](https://developer.mozilla.org/en-US/docs/Games/Techniques/Tilemaps)

### それは何ですか

タイルマップは、タイルと呼ばれる小さな規則的な形状の画像を使用してゲーム世界を構築する 2D ゲーム開発の基本的な技術です。大きなモノリシック レベルのイメージを保存する代わりに、ゲーム世界は再利用可能なタイル グラフィックスのグリッドから組み立てられ、パフォーマンスとメモリに大きなメリットをもたらします。

### 仕組み

**コア構造:**

1. **タイル アトラス (スプライトシート):** すべてのタイル イメージが 1 つのアトラス ファイルに保存されます。各タイルには、その識別子として使用されるインデックスが割り当てられます。
2. **タイルマップ データ オブジェクト:** タイル サイズ (ピクセル寸法)、画像アトラス参照、マップ寸法 (タイルまたはピクセル単位)、ビジュアル グリッド (タイル インデックスの配列)、およびオプションのロジック グリッド (衝突、パスファインディング、スポーン データ) が含まれます。

特別な値 (負の数、0、または null) は空のタイルを表します。

### いつ使用するか

- あらゆる種類の 2D ゲーム世界 (プラットフォーマー、RPG、戦略ゲーム、パズル ゲーム) の構築。
- スーパー マリオ ブラザーズ、パックマン、ゼルダ、スタークラフト、シム シティなどの古典にインスピレーションを得たゲーム。
- グリッドベースの世界がパスファインディング、衝突、またはレベル編集に論理的な利点をもたらすシナリオ。

### 静的タイルマップのレンダリング

地図が画面上に完全に収まる場合:```javascript
for (let column = 0; column < map.columns; column++) {
  for (let row = 0; row < map.rows; row++) {
    const tile = map.getTile(column, row);
    const x = column * map.tileSize;
    const y = row * map.tileSize;
    drawTile(tile, x, y);
  }
}
```### カメラを使用したタイルマップのスクロール

ワールド座標 (レベル位置) とスクリーン座標 (レンダリング位置) の間の変換:```javascript
// These functions assume camera points to top-left corner

function worldToScreen(x, y) {
  return { x: x - camera.x, y: y - camera.y };
}

function screenToWorld(x, y) {
  return { x: x + camera.x, y: y + camera.y };
}
```重要な原則: パフォーマンスを最適化するために、表示されているタイルのみをレンダリングします。レンダリング中にカメラ オフセット変換を適用します。

### タイルマップの種類

**正方形タイル (最も一般的):**
- RPG および戦略ゲーム (Warcraft 2、Final Fantasy) のトップダウン ビュー。
- プラットフォーマー (スーパー マリオ ブラザーズ) の側面図。

**アイソメトリック タイルマップ:**
- 3D 環境のような錯覚を作り出します。
- シミュレーションおよび戦略ゲーム (シムシティ 2000、ファラオ、ファイナルファンタジー タクティクス) で人気。

### レイヤー

複数のビジュアル レイヤーを使用すると、次のことが可能になります。
- さまざまな背景タイプ間でタイルを再利用します。
- キャラクターが地形の後ろまたは前に出現します (木の後ろを歩きます)。
- タイルのバリエーションが少なくなり、より豊かな世界。

例: 草、砂、またはレンガの背景上の別のレイヤーにレンダリングされた岩のタイル。

### ロジックグリッド

非ビジュアル ゲーム ロジック用の別個のグリッド:
- **衝突検出:** 歩行可能なタイルとブロックされたタイルをマークします。
- **キャラクターのスポーン:** スポーンポイントの位置を定義します。
- **経路探索:** ナビゲーション グラフを作成します。
- **タイルの組み合わせ:** 有効なパターン (テトリス、宝石で飾られた) を検出します。

### パフォーマンスの最適化

1. **表示されているタイルのみをレンダリング** -- 画面外のタイルを完全にスキップします。
2. **キャンバスに事前レンダリング** -- マップをオフスクリーンのキャンバス要素にレンダリングし、単一の操作としてブリットします。
3. **オフキャンバス バッファリング** -- スクロール中の再描画を減らすために、表示領域より大きいセクション (2x2 タイルより大きい) を描画します。
4. **チャンキング** -- 大きなタイルマップをセクション (例: 10x10 タイル チャンク) に分割し、それぞれを「大きなタイル」として事前レンダリングします。

---

## コントロール: ゲームパッド API

**出典:** [MDN - コントロール ゲームパッド API](https://developer.mozilla.org/en-US/docs/Games/Techniques/Controls_Gamepad_API)

### それは何ですか

ゲームパッド API は、プラグインなしで Web ブラウザーでゲームパッド コントローラーを検出して使用するためのインターフェイスを提供します。 JavaScript を通じてボタンの押下と軸の変更を公開し、ブラウザベースのゲームをコンソールのように制御できるようにします。

### 仕組み

2 つの基本的なイベントがコントローラーのライフサイクルを処理します。

- `gamepadconnected` -- ゲームパッドが接続されているときに発生します。
- `gamepaddisconnected` -- (物理的または非アクティブのため) 切断されたときに起動されます。

セキュリティ上の注意: イベントを発生させるには、ページが表示されている間、ユーザーによるコントローラーの操作が必要です (フィンガープリントの防止)。

**ゲームパッド オブジェクトのプロパティ:**

|プロパティ |説明 |
|---|---|
| `id` |コントローラー情報を含む文字列 |
| `index` |接続されたデバイスの一意の識別子 |
| `connected` |接続ステータスを示すブール値 |
| `mapping` |レイアウトタイプ (「標準」が一般的なオプション) |
| `axes` |アナログ スティックの位置を表す浮動小数点数 (-1 ～ 1) の配列 |
| `buttons` | `pressed` および `value` プロパティを持つ GamepadButton オブジェクトの配列 |

### いつ使用するか

- コンソール コントローラーで動作するゲームを構築する場合。
- Windows および macOS で Xbox 360、Xbox One、PS3、または PS4 コントローラーをサポートする場合。
- デュアル入力サポート (キーボード + ゲームパッド) が必要な場合。

### コード例

**基本的なセットアップ構造:**```javascript
const gamepadAPI = {
  controller: {},
  turbo: false,
  connect() {},
  disconnect() {},
  update() {},
  buttonPressed() {},
  buttons: [],
  buttonsCache: [],
  buttonsStatus: [],
  axesStatus: [],
};
```**ボタン レイアウト (Xbox 360):**```javascript
const gamepadAPI = {
  buttons: [
    "DPad-Up", "DPad-Down", "DPad-Left", "DPad-Right",
    "Start", "Back", "Axis-Left", "Axis-Right",
    "LB", "RB", "Power", "A", "B", "X", "Y",
  ],
};
```**イベントリスナー:**```javascript
window.addEventListener("gamepadconnected", gamepadAPI.connect);
window.addEventListener("gamepaddisconnected", gamepadAPI.disconnect);
```**接続および切断ハンドラー:**```javascript
connect(evt) {
  gamepadAPI.controller = evt.gamepad;
  gamepadAPI.turbo = true;
  console.log("Gamepad connected.");
},

disconnect(evt) {
  gamepadAPI.turbo = false;
  delete gamepadAPI.controller;
  console.log("Gamepad disconnected.");
},
```**更新メソッド (フレームごとに呼び出されます):**```javascript
update() {
  // Clear the buttons cache
  gamepadAPI.buttonsCache = [];

  // Move the buttons status from the previous frame to the cache
  for (let k = 0; k < gamepadAPI.buttonsStatus.length; k++) {
    gamepadAPI.buttonsCache[k] = gamepadAPI.buttonsStatus[k];
  }

  // Clear the buttons status
  gamepadAPI.buttonsStatus = [];

  // Get the gamepad object
  const c = gamepadAPI.controller || {};

  // Loop through buttons and push the pressed ones to the array
  const pressed = [];
  if (c.buttons) {
    for (let b = 0; b < c.buttons.length; b++) {
      if (c.buttons[b].pressed) {
        pressed.push(gamepadAPI.buttons[b]);
      }
    }
  }

  // Loop through axes and push their values to the array
  const axes = [];
  if (c.axes) {
    for (const ax of c.axes) {
      axes.push(ax.toFixed(2));
    }
  }

  // Assign received values
  gamepadAPI.axesStatus = axes;
  gamepadAPI.buttonsStatus = pressed;

  return pressed;
},
```**ホールドサポートによるボタン検出:**```javascript
buttonPressed(button, hold) {
  let newPress = false;
  if (gamepadAPI.buttonsStatus.includes(button)) {
    newPress = true;
  }
  if (!hold && gamepadAPI.buttonsCache.includes(button)) {
    newPress = false;
  }
  return newPress;
},
```パラメータ:
- `button` -- リッスンするボタンの名前。
- `hold` -- true の場合、ボタンの長押しは連続アクションとしてカウントされます。 false の場合、新しいプレスのみが登録されます。

**ゲームループでの使用:**```javascript
if (gamepadAPI.turbo) {
  if (gamepadAPI.buttonPressed("A", "hold")) {
    this.turbo_fire();
  }
  if (gamepadAPI.buttonPressed("B")) {
    this.managePause();
  }
}
```**閾値付きアナログスティック入力 (スティックドリフト防止):**```javascript
if (gamepadAPI.axesStatus[0].x > 0.5) {
  this.player.angle += 3;
  this.turret.angle += 3;
}
```**接続されているすべてのゲームパッドの取得:**```javascript
const gamepads = navigator.getGamepads();
// Returns an array where unavailable/disconnected slots contain null
// Example with one device at index 1: [null, [object Gamepad]]
```---

## 鮮明なピクセルアートの外観

**出典:** [MDN - 鮮明なピクセル アートの外観](https://developer.mozilla.org/en-US/docs/Games/Techniques/Crisp_pixel_art_look)

### それは何ですか

平滑化補間を行わずに個々の画像ピクセルを画面ピクセルのブロックにマッピングすることにより、高解像度ディスプレイ上でぼやけのないピクセル アートをレンダリングする技術。レトロなピクセル アートでは、拡大縮小中にハード エッジを維持する必要がありますが、最新のブラウザでは、色をブレンドしてぼかしを作成するスムージング アルゴリズムがデフォルトで使用されます。

### 仕組み

CSS `image-rendering` プロパティは、ブラウザーが画像を拡大縮小する方法を制御します。 `pixelated` に設定すると、最近傍スケーリングが強制され、バイリニアまたはバイキュービック スムージングを適用する代わりに、ピクセル アートの鮮明でブロック状の外観が維持されます。

**主要な CSS 値:**
- `pixelated` -- ピクセル アートの鮮明なエッジを保持します。
- `crisp-edges` -- 一部のブラウザでサポートされる代替手段。

### いつ使用するか

- ピクセルアートアセットを使用したレトロスタイルのゲーム。
- 意図的にブロック状のピクセル化されたビジュアル スタイルが必要なゲーム。
- 小さなスプライト画像を大きな表示サイズに拡大縮小する場合。

### テクニック 1: CSS を使用して `<img>` 要素をスケーリングする```html
<img
  src="character.png"
  alt="pixel art character, upscaled with CSS, appearing crisp" />
```

```css
img {
  width: 48px;
  height: 136px;
  image-rendering: pixelated;
}
```### テクニック 2: キャンバス内の鮮明なピクセル アート

キャンバスの `width`/`height` 属性を元のピクセル アートの解像度に設定し、CSS `width`/`height` を使用してスケーリングします (例: 4 倍のスケール: 128 ピクセルから 512 ピクセルの CSS 幅)。```html
<canvas id="game" width="128" height="128">A cat</canvas>
```

```css
canvas {
  width: 512px;
  height: 512px;
  image-rendering: pixelated;
}
```

```javascript
const ctx = document.getElementById("game").getContext("2d");

const image = new Image();
image.onload = () => {
  ctx.drawImage(image, 0, 0);
};
image.src = "cat.png";
```### テクニック 3: 補正を伴う任意のキャンバス スケーリング

非整数のスケール係数の場合、画像ピクセルは整数倍でキャンバス ピクセルに揃える必要があります。```javascript
const ctx = document.getElementById("game").getContext("2d");
ctx.scale(0.8, 0.8);

const image = new Image();
image.onload = () => {
  // Correct formula: dWidth = sWidth / xScale * n (where n is an integer)
  ctx.drawImage(image, 0, 0, 128, 128, 0, 0, 128 / 0.8, 128 / 0.8);
};
image.src = "cat.png";
````drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)`を使用する場合:
- `dWidth` は `sWidth / xScale * n` と等しくなければなりません
- `dHeight` は `sHeight / yScale * m` と等しくなければなりません
- `n` と `m` は正の整数 (1、2、3 など)

### 既知の制限事項

**devicePixelRatio のずれ:** `devicePixelRatio` が整数ではない場合 (例: ブラウザーのズームが 110% の場合)、CSS ピクセルがデバイス ピクセルに完全にマッピングできないため、ピクセルが不均一にレンダリングされる可能性があります。これにより、外観が不均一になり、簡単な解決策はありません。

### ベストプラクティス

1. 可能な限り、整数のスケール係数 (2x、3x、4x) を使用します。
2. アスペクト比を維持します -- 幅と高さを均等に拡大します。
3. さまざまなブラウザーのズーム レベルでテストします。
4. 分数のキャンバス スケール係数やdrawImage の寸法を避けてください。
5. アクセシビリティのために、キャンバス要素に説明的な `aria-label` 属性を含めます。