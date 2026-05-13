# 3D Web ゲーム

Web 上で 3D ゲームを構築するための包括的なリファレンスです。基礎理論、主要フレームワーク、シェーダープログラミング、衝突判定、没入型 WebXR 体験を網羅します。

Sources: [MDN Web Docs -- Games Techniques: 3D on the web](https://developer.mozilla.org/en-US/docs/Games/Techniques/3D_on_the_web)

---

## 3D 理論と基礎

どのフレームワークを使う場合でも、まず 3D レンダリングの中核概念を理解することが重要です。

### 座標系

WebGL は **右手座標系** を使用します。

- **X 軸** -- 右方向を指す
- **Y 軸** -- 上方向を指す
- **Z 軸** -- 画面の外側、つまり閲覧者に向かう方向を指す

すべての 3D オブジェクトは、この座標系を基準に配置されます。

### 頂点・辺・面・メッシュ

- **Vertex** -- 3D 空間内の点。`(x, y, z)` で定義され、追加属性として color（RGBA、値は 0.0-1.0）、normal（頂点が向いている方向でライティングに使用）、texture coordinates を持つ。
- **Edge** -- 2 つの頂点を結ぶ線。
- **Face** -- 辺で囲まれた平面（例: 3 つの頂点を結ぶ三角形）。
- **Geometry** -- 頂点・辺・面から構成される形状の構造。
- **Material** -- 色、テクスチャ、ラフネス、メタルネスなどを組み合わせた表面の見た目。
- **Mesh** -- Geometry と Material を組み合わせた、描画可能な 3D オブジェクト。

### レンダリングパイプライン

パイプラインは 3D オブジェクトを画面上の 2D ピクセルへ変換します。主な 4 段階は次のとおりです。

**1. 頂点処理（Vertex Processing）**

個々の頂点データをプリミティブ（三角形、線、点）にまとめ、変換を適用します。

- **Model transformation** -- オブジェクトをワールド空間内で配置・向き設定する。
- **View transformation** -- 仮想カメラを配置・向き設定する。
- **Projection transformation** -- カメラの視野角（FOV）、アスペクト比、near plane、far plane を定義する。
- **Viewport transformation** -- 結果を画面のビューポートへマッピングする。

**2. ラスタライズ（Rasterization）**

3D プリミティブを、ピクセルグリッドに対応した 2D フラグメントへ変換します。

**3. フラグメント処理（Fragment Processing）**

テクスチャとライティングを使って各フラグメントの最終色を決定します。

- **Textures**: 3D サーフェスに貼り付ける 2D 画像。個々のテクスチャ要素は *texel* と呼ばれます。Texture wrapping はジオメトリ周囲で画像を繰り返し、texture filtering は表示解像度とテクスチャ解像度が異なる際の縮小・拡大を処理します。
- **Lighting (Phong model)**: 光との相互作用は 4 種類 -- **diffuse**（太陽のような遠方からの指向性光）、**specular**（懐中電灯のような点光源ハイライト）、**ambient**（一定の環境光）、**emissive**（オブジェクト自身が放つ光）。

**4. 出力マージ（Output Merging）**

3D フラグメントを最終的な 2D ピクセルグリッドへ変換します。画面外のオブジェクトや遮蔽されたオブジェクトは効率化のためカリングされます。

### カメラ

カメラは、何が見えるかを定義します。

- **Position** -- 3D 空間内の位置。
- **Direction** -- カメラが向いている方向。
- **Orientation** -- 視線軸まわりの回転。

### 実践のヒント

- WebGL のサイズ・位置の値は無単位です。ミリメートル、メートル、フィートなど、何を表すかは自分で決めます。
- 実装に入る前にパイプラインの概念を理解しましょう。頂点処理とフラグメント処理の段階はシェーダーでプログラム可能です。
- どのフレームワーク（Three.js、Babylon.js、A-Frame、PlayCanvas）もこのパイプラインを抽象化しますが、基礎は共通です。

---

## フレームワーク

### Three.js

Three.js は Web 向け 3D エンジンの中でも特に人気があります。WebGL の上に高水準 API を提供し、大規模なプラグイン生態系、サンプル、コミュニティサポートがあります。

#### セットアップ

```html
<!doctype html>
<html lang="en-GB">
  <head>
    <meta charset="utf-8" />
    <title>Three.js Demo</title>
    <style>
      html, body, canvas {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
        font-size: 0;
      }
    </style>
  </head>
  <body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r79/three.min.js"></script>
    <script>
      const WIDTH = window.innerWidth;
      const HEIGHT = window.innerHeight;
      /* all code goes here */
    </script>
  </body>
</html>
```

または npm でインストールします。

```bash
npm install --save three
npm install --save-dev vite
npx vite
```

#### コアコンポーネント

**Renderer** -- ブラウザ内でシーンを表示します。

```javascript
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(WIDTH, HEIGHT);
renderer.setClearColor(0xdddddd, 1);
document.body.appendChild(renderer.domElement);
```

**Scene** -- すべての 3D オブジェクト、ライト、カメラを入れるコンテナです。

```javascript
const scene = new THREE.Scene();
```

**Camera** -- 視点を定義します（最も一般的なのは PerspectiveCamera）。

```javascript
const camera = new THREE.PerspectiveCamera(70, WIDTH / HEIGHT);
camera.position.z = 50;
scene.add(camera);
```

パラメータ: 視野角（度）、アスペクト比。ほかのカメラ種類には Orthographic や Cube があります。

#### Geometry・Material・Mesh

```javascript
// Geometry defines the shape
const boxGeometry = new THREE.BoxGeometry(10, 10, 10);
const torusGeometry = new THREE.TorusGeometry(7, 1, 16, 32);
const dodecahedronGeometry = new THREE.DodecahedronGeometry(7);

// Material defines the surface appearance
const basicMaterial = new THREE.MeshBasicMaterial({ color: 0x0095dd });   // No lighting
const phongMaterial = new THREE.MeshPhongMaterial({ color: 0xff9500 });   // Glossy
const lambertMaterial = new THREE.MeshLambertMaterial({ color: 0xeaeff2 }); // Matte

// Mesh combines geometry + material
const cube = new THREE.Mesh(boxGeometry, basicMaterial);
cube.position.set(-25, 0, 0);
cube.rotation.set(0.4, 0.2, 0);
scene.add(cube);
```

#### ライティング

```javascript
const light = new THREE.PointLight(0xffffff);
light.position.set(-10, 15, 50);
scene.add(light);
```

ほかのライト種類: Ambient、Directional、Hemisphere、Spot。

注意: `MeshBasicMaterial` はライティングの影響を受けません。ライティングされた表面には `MeshPhongMaterial` または `MeshLambertMaterial` を使います。

#### アニメーションループ

```javascript
let t = 0;
function render() {
  t += 0.01;
  requestAnimationFrame(render);

  cube.rotation.y += 0.01;                          // continuous rotation
  torus.scale.y = Math.abs(Math.sin(t));             // pulsing scale
  dodecahedron.position.y = -7 * Math.sin(t * 2);   // bobbing position

  renderer.render(scene, camera);
}
render();
```

#### 実践のヒント

- `Math.sin()` でスケールをアニメーションさせるときは、負のスケール値を避けるために `Math.abs()` を使います。
- レンダーループは `requestAnimationFrame` を使うことで、滑らかでブラウザ最適化されたフレーム更新になります。
- 完全な API は [Three.js documentation](https://threejs.org/docs/) を参照してください。

---

### Babylon.js

Babylon.js は、組み込み数学ライブラリ、物理演算サポート、豊富なドキュメントを備えた高機能 3D エンジンです。

#### セットアップ

```html
<script src="https://cdn.babylonjs.com/v7.34.1/babylon.js"></script>
<canvas id="render-canvas"></canvas>
```

#### エンジン・シーン・レンダーループ

```javascript
const canvas = document.getElementById("render-canvas");
const engine = new BABYLON.Engine(canvas);

const scene = new BABYLON.Scene(engine);
scene.clearColor = new BABYLON.Color3(0.8, 0.8, 0.8);

function renderLoop() {
  scene.render();
}
engine.runRenderLoop(renderLoop);
```

#### カメラとライティング

```javascript
const camera = new BABYLON.FreeCamera("camera", new BABYLON.Vector3(0, 0, -10), scene);
const light = new BABYLON.PointLight("light", new BABYLON.Vector3(10, 10, 0), scene);
```

#### メッシュの作成

```javascript
const box = BABYLON.Mesh.CreateBox("box", 2, scene);       // name, size, scene
const torus = BABYLON.Mesh.CreateTorus("torus", 2, 0.5, 15, scene); // name, diameter, thickness, tessellation, scene
const cylinder = BABYLON.Mesh.CreateCylinder("cylinder", 2, 2, 2, 12, 1, scene);
// name, height, topDiameter, bottomDiameter, tessellation, heightSubdivisions, scene
```

#### マテリアル

```javascript
const boxMaterial = new BABYLON.StandardMaterial("material", scene);
boxMaterial.emissiveColor = new BABYLON.Color3(0, 0.58, 0.86);
box.material = boxMaterial;
```

#### 変換とアニメーション

```javascript
box.position.x = 5;
box.rotation.x = -0.2;
box.scaling.x = 1.5;

// Animation inside render loop
let t = 0;
function renderLoop() {
  scene.render();
  t -= 0.01;
  box.rotation.y = t * 2;
  torus.scaling.z = Math.abs(Math.sin(t * 2)) + 0.5;
  cylinder.position.y = Math.sin(t * 3);
}
engine.runRenderLoop(renderLoop);
```

#### 実践のヒント

- `BABYLON` グローバルオブジェクトに、フレームワークの機能がすべて含まれています。
- `BABYLON.Vector3` と `BABYLON.Color3` は位置指定や色指定で広く使われます。
- Babylon.js には、ベクトル・色・行列向けの数学ライブラリが標準で含まれています。
- 物理演算、パーティクル、ポストプロセスなどの高度な機能は [Babylon.js documentation](https://doc.babylonjs.com/) を参照してください。

---

### A-Frame

A-Frame は Mozilla の宣言的な HTML ベースのフレームワークで、Web 上の VR/AR 体験構築に使われます。エンティティ・コンポーネントシステムを採用し、内部では WebGL で動作します。

#### セットアップ

```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>A-Frame Demo</title>
    <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
    <style>
      body { margin: 0; padding: 0; width: 100%; height: 100%; font-size: 0; }
    </style>
  </head>
  <body>
    <a-scene>
      <!-- entities go here -->
    </a-scene>
  </body>
</html>
```

`<a-scene>` 要素がルートコンテナです。A-Frame はデフォルトのカメラ、ライティング、入力コントロールを自動で含みます。

#### プリミティブとエンティティ

```html
<!-- Built-in primitive shapes -->
<a-box position="0 1 -3" rotation="0 10 0" color="#4CC3D9"></a-box>
<a-sky color="#DDDDDD"></a-sky>

<!-- Generic entity with explicit geometry and material -->
<a-entity
  geometry="primitive: torus; radius: 1; radiusTubular: 0.1; segmentsTubular: 12;"
  material="color: #EAEFF2; roughness: 0.1; metalness: 0.5;"
  rotation="10 0 0"
  position="-3 1 0">
</a-entity>
```

#### JavaScript でエンティティを作成する

```javascript
const scene = document.querySelector("a-scene");
const cylinder = document.createElement("a-cylinder");
cylinder.setAttribute("color", "#FF9500");
cylinder.setAttribute("height", "2");
cylinder.setAttribute("radius", "0.75");
cylinder.setAttribute("position", "3 1 0");
scene.appendChild(cylinder);
```

#### カメラとライティング

```html
<a-camera position="0 1 4" cursor-visible="true" cursor-color="#0095DD" cursor-opacity="0.5">
</a-camera>

<a-light type="directional" color="white" intensity="0.5" position="-1 1 2"></a-light>
<a-light type="ambient" color="white"></a-light>
```

デフォルト操作: 移動は WASD キー、視点変更はマウス。VR モードボタンは右下に表示されます。

#### アニメーション

HTML 属性による宣言的アニメーション:

```html
<a-box
  color="#0095DD"
  rotation="20 40 0"
  position="0 1 0"
  animation="property: rotation; from: 20 0 0; to: 20 360 0;
    dir: alternate; loop: true; dur: 4000; easing: easeInOutQuad;">
</a-box>
```

アニメーションプロパティ: `property`（アニメーションする属性）、`from`/`to`（開始/終了値）、`dir`（alternate または normal）、`loop`（boolean）、`dur`（ミリ秒）、`easing`（イージング関数）。

JavaScript による動的アニメーション:

```javascript
let t = 0;
function render() {
  t += 0.01;
  requestAnimationFrame(render);
  cylinder.setAttribute("position", `3 ${Math.sin(t * 2) + 1} 0`);
}
render();
```

#### 実践のヒント

- A-Frame は、馴染みのある HTML 構文で素早く VR/AR プロトタイプを作るのに最適です。
- エンティティ・コンポーネント構成は拡張しやすく、コミュニティプラグインで物理演算やゲームパッド操作などを追加できます。
- 背景色や 360 度画像には `<a-sky>` を使います。
- A-Frame はデスクトップ、モバイル（iOS/Android）、VR ヘッドセット（Meta Quest、HTC Vive）をサポートします。

---

### PlayCanvas

PlayCanvas は WebGL ゲームエンジンで、2 つのワークフローを提供します。

1. **Engine approach** -- PlayCanvas JavaScript ライブラリを HTML に直接読み込み、ゼロからコードを書く。
2. **Editor approach** -- オンラインのドラッグ＆ドロップ式ビジュアルエディタでシーンを構成する。

#### 主な機能

- エンティティ・コンポーネントシステム構成
- [ammo.js](https://github.com/kripken/ammo.js/) を基盤とした組み込み物理エンジン
- 衝突判定
- オーディオサポート
- 入力処理（キーボード、マウス、タッチ、ゲームパッド）
- リソース/アセット管理

#### 実践のヒント

- PlayCanvas は、リアルタイム共同編集が可能なオンラインエディタにより、チームでのゲーム開発に特に強みがあります。
- エンジン単体アプローチは軽量で、任意の Web ページに埋め込めます。
- エンティティ、コンポーネント、カメラ、ライト、マテリアル、アニメーションのチュートリアルは [PlayCanvas developer documentation](https://developer.playcanvas.com/) を参照してください。

---

## GLSL シェーダー

GLSL（OpenGL Shading Language）は C ライクな言語で、GPU 上で直接実行されます。これにより、レンダリングパイプラインの頂点処理とフラグメント処理をカスタム制御できます。

### シェーダーとは何か

シェーダーは CPU ではなく GPU で実行される小さなプログラムです。強い型付けを持ち、ベクトル・行列数学を多用します。WebGL で重要なのは次の 2 種類です。

- **Vertex shader** -- 頂点ごとに 1 回実行され、3D 位置を画面座標へ変換する。
- **Fragment shader**（pixel shader） -- ピクセルごとに 1 回実行され、最終 RGBA 色を決定する。

### 頂点シェーダー

頂点シェーダーの役割は、頂点の変換後位置を保持する GLSL 組み込み変数 `gl_Position` を設定することです。

```glsl
void main() {
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position.x, position.y, position.z, 1.0);
}
```

- `projectionMatrix` -- 透視投影または正射影を処理します（Three.js が提供）。
- `modelViewMatrix` -- モデル変換とビュー変換を結合します（Three.js が提供）。
- `vec4(x, y, z, w)` -- 4 成分ベクトル。位置頂点では `w` は通常 1.0。

頂点は直接操作できます。

```glsl
void main() {
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position.x + 10.0, position.y, position.z + 5.0, 1.0);
}
```

### フラグメントシェーダー

フラグメントシェーダーの役割は、RGBA 色を保持する GLSL 組み込み変数 `gl_FragColor` を設定することです。

```glsl
void main() {
  gl_FragColor = vec4(0.0, 0.58, 0.86, 1.0);
}
```

RGBA 成分は 0.0 から 1.0 の float です。Alpha 0.0 は完全透明、1.0 は完全不透明です。

### HTML と Three.js でシェーダーを使う

カスタム type 属性を持つ script タグにシェーダーソースを埋め込みます。

```html
<script id="vertexShader" type="x-shader/x-vertex">
  void main() {
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
</script>

<script id="fragmentShader" type="x-shader/x-fragment">
  void main() {
    gl_FragColor = vec4(0.0, 0.58, 0.86, 1.0);
  }
</script>
```

`ShaderMaterial` で適用します。

```javascript
const shaderMaterial = new THREE.ShaderMaterial({
  vertexShader: document.getElementById("vertexShader").textContent,
  fragmentShader: document.getElementById("fragmentShader").textContent,
});

const cube = new THREE.Mesh(boxGeometry, shaderMaterial);
```

### シェーダーパイプライン

1. **Vertex shader** が各頂点を処理し、`gl_Position` を出力する。
2. **Rasterization** が 3D 座標を 2D 画面ピクセルへマッピングする。
3. **Fragment shader** が各ピクセルを処理し、`gl_FragColor` を出力する。

### 重要概念

- **Uniforms** -- JavaScript からシェーダーへ渡す値で、1 回の draw call 内では全頂点/全フラグメントで一定（例: 光源位置、時間）。
- **Attributes** -- 頂点ごとのデータで、頂点シェーダーへ渡される（例: 位置、法線、UV 座標）。
- **Varyings** -- 頂点シェーダーからフラグメントシェーダーへ渡され、サーフェス上で補間される値。

### 実践のヒント

- シェーダーは GPU 上で実行され、CPU の計算負荷をオフロードします。これはリアルタイム性能で重要です。
- Three.js、Babylon.js などのフレームワークはシェーダー設定の多くを抽象化しますが、純粋な WebGL では大幅にボイラープレートが増えます。
- [ShaderToy](https://www.shadertoy.com/) はシェーダー例や着想を得るのに非常に有用です。
- GLSL は明示的な型宣言が必要なので、float には `1` ではなく必ず `1.0` を使ってください。

---

## 衝突判定

衝突判定は、3D オブジェクト同士が交差したタイミングを判定する技術で、ゲーム物理、インタラクション、ゲームプレイロジックの基盤です。

### 軸平行境界ボックス（AABB）

AABB は、オブジェクトを座標軸に平行な回転しない直方体で包みます。三角関数を使わず論理比較のみで判定できるため、一般的な衝突テストの中で最速です。

**制約**: AABB はオブジェクトと一緒に回転しません。回転するエンティティには、毎フレーム境界ボックスを再計算するか、代わりに境界球を使います。

#### 点 vs. AABB

3 軸すべてをチェックして、点がボックス内にあるか判定します。

```javascript
function isPointInsideAABB(point, box) {
  return (
    point.x >= box.minX &&
    point.x <= box.maxX &&
    point.y >= box.minY &&
    point.y <= box.maxY &&
    point.z >= box.minZ &&
    point.z <= box.maxZ
  );
}
```

#### AABB vs. AABB

3 軸すべてで 2 つのボックスが重なっているか判定します。

```javascript
function intersect(a, b) {
  return (
    a.minX <= b.maxX &&
    a.maxX >= b.minX &&
    a.minY <= b.maxY &&
    a.maxY >= b.minY &&
    a.minZ <= b.maxZ &&
    a.maxZ >= b.minZ
  );
}
```

### 境界球（Bounding Spheres）

境界球は回転に不変（オブジェクトがどう回っても球は同じ）なため、回転するエンティティに適しています。ただし球状でない形にはフィットしづらく、誤検知（false positive）が増えます。

#### 点 vs. 球

点と球の中心の距離が半径より小さいかを判定します。

```javascript
function isPointInsideSphere(point, sphere) {
  const distance = Math.sqrt(
    (point.x - sphere.x) ** 2 +
    (point.y - sphere.y) ** 2 +
    (point.z - sphere.z) ** 2
  );
  return distance < sphere.radius;
}
```

**性能最適化**: 2 乗距離を比較して平方根を避けます。

```javascript
const distanceSqr =
  (point.x - sphere.x) ** 2 +
  (point.y - sphere.y) ** 2 +
  (point.z - sphere.z) ** 2;
return distanceSqr < sphere.radius * sphere.radius;
```

#### 球 vs. 球

中心間距離が半径の和より小さいかを判定します。

```javascript
function intersect(sphere, other) {
  const distance = Math.sqrt(
    (sphere.x - other.x) ** 2 +
    (sphere.y - other.y) ** 2 +
    (sphere.z - other.z) ** 2
  );
  return distance < sphere.radius + other.radius;
}
```

#### 球 vs. AABB

クランプを使って AABB 上の最近接点を求め、その点との距離を判定します。

```javascript
function intersect(sphere, box) {
  const x = Math.max(box.minX, Math.min(sphere.x, box.maxX));
  const y = Math.max(box.minY, Math.min(sphere.y, box.maxY));
  const z = Math.max(box.minZ, Math.min(sphere.z, box.maxZ));

  const distance = Math.sqrt(
    (x - sphere.x) ** 2 +
    (y - sphere.y) ** 2 +
    (z - sphere.z) ** 2
  );

  return distance < sphere.radius;
}
```

### Three.js における衝突判定

Three.js には、境界ボリューム衝突判定用の `Box3` と `Sphere`、および可視化ヘルパーが組み込まれています。

#### 境界ボリュームの作成

```javascript
// Box3 from an object (recommended -- accounts for transforms and children)
const knotBBox = new THREE.Box3(new THREE.Vector3(), new THREE.Vector3());
knotBBox.setFromObject(knot);

// Sphere from geometry
const knotBSphere = new THREE.Sphere(
  knot.position,
  knot.geometry.boundingSphere.radius
);
```

**重要**: `setFromObject()` は位置・回転・スケール・子メッシュを考慮します。geometry の `boundingBox` プロパティは考慮しません。

#### 交差テスト

```javascript
// Point inside box or sphere
knotBBox.containsPoint(point);
knotBSphere.containsPoint(point);

// Box vs. box
knotBBox.intersectsBox(otherBox);

// Sphere vs. sphere
knotBSphere.intersectsSphere(otherSphere);
```

注意: `containsBox()` は片方のボックスがもう片方を完全に内包しているかを判定するもので、`intersectsBox()` とは異なります。

#### Sphere vs. Box3（カスタムパッチ）

Three.js は球とボックスの判定を標準では提供していません。手動で追加します。

```javascript
THREE.Sphere.__closest = new THREE.Vector3();
THREE.Sphere.prototype.intersectsBox = function (box) {
  THREE.Sphere.__closest.set(this.center.x, this.center.y, this.center.z);
  THREE.Sphere.__closest.clamp(box.min, box.max);
  const distance = this.center.distanceToSquared(THREE.Sphere.__closest);
  return distance < this.radius * this.radius;
};
```

#### 可視デバッグ用 BoxHelper

`BoxHelper` は任意メッシュを囲む可視ワイヤーフレーム境界ボックスを作成し、更新も簡単です。

```javascript
const knotBoxHelper = new THREE.BoxHelper(knot, 0x00ff00);
scene.add(knotBoxHelper);

// After moving or rotating the mesh, update the helper
knot.position.set(-3, 2, 1);
knot.rotation.x = -Math.PI / 4;
knotBoxHelper.update();

// Convert to Box3 for intersection tests
const box3 = new THREE.Box3();
box3.setFromObject(knotBoxHelper);
box3.intersectsBox(otherBox3);
```

BoxHelper の利点: `update()` で自動リサイズ、子メッシュを含む、可視デバッグが可能。制約: ボックスボリュームのみ（球ヘルパーはなし）。

### 物理エンジン

より高度な衝突判定と応答には物理エンジンを使います。

- **Cannon.js** -- JavaScript 向けオープンソース 3D 物理エンジン。
- **ammo.js** -- Bullet 物理ライブラリの JavaScript 移植版（PlayCanvas で利用）。

物理エンジンは、可視メッシュに紐づく *physical body* を作成し、速度・位置・回転・トルクなどのプロパティを持たせます。衝突計算には *physical shape*（ボックス、球、凸包）を使用します。

### 実践のヒント

- 軸平行で回転しないオブジェクトには AABB を使います。最速の選択肢です。
- 回転するオブジェクトには境界球を使います。球は回転に不変です。
- 複雑な形状では、複合境界ボリューム（複数プリミティブの組み合わせ）を検討します。
- ループ内での `Math.sqrt()` は避け、2 乗距離で比較してください。
- 本番ゲームでは、衝突判定を一から実装するより物理エンジンを統合するのが望ましいです。

---

## WebXR

WebXR は、ブラウザで仮想現実（VR）および拡張現実（AR）体験を構築するための最新 Web API です。非推奨となった WebVR API の後継です。

### WebXR とは

WebXR Device API は XR ハードウェア（ヘッドセット、コントローラー）へのアクセスを提供し、立体視レンダリングを可能にします。次のリアルタイムデータを取得します。

- ヘッドセットの位置と向き
- コントローラーの位置、向き、速度、加速度
- XR コントローラーからの入力イベント

### 対応デバイス

- Meta Quest
- Valve Index
- PlayStation VR (PSVR2)
- WebXR 対応ブラウザを搭載した任意のデバイス

### コア概念

すべての WebXR 体験には次の 2 つが必要です。

1. **リアルタイム位置データ** -- アプリケーションはヘッドセットとコントローラーの 3D 空間上の位置を継続的に受け取る。
2. **リアルタイム立体視レンダリング** -- アプリケーションはヘッドセット表示向けに、左右の目それぞれにわずかにオフセットした 2 つの映像を描画する。

### フレームワークサポート

主要な 3D Web フレームワークはすべて WebXR をサポートしています。

- **A-Frame** -- VR モードボタンを内蔵。宣言的な HTML ベースシーンは自動的に VR で動作。
- **Three.js** -- `renderer.xr` を通じて WebXR 統合を提供。[Three.js VR documentation](https://threejs.org/docs/#manual/en/introduction/How-to-create-VR-content) を参照。
- **Babylon.js** -- XR Experience Helper により WebXR を標準サポート。

### 関連 API

- **Gamepad API** -- 非 XR コントローラー入力（ゲームパッド、ジョイスティック）向け。
- **Device Orientation API** -- モバイルデバイスの回転検出向け。

### 設計原則

- 生のグラフィック品質やゲームプレイの複雑さより、**没入感** を優先する。
- ユーザーが体験の *一部である* と感じられる必要がある。
- VR では、不安定なフレームレートの高精細グラフィックより、安定した高フレームレートで描画された基本形状のほうが魅力的な場合がある。
- 試行錯誤は不可欠であり、実機で頻繁にテストする。

### 実践のヒント

- 素早く VR プロトタイピングするなら A-Frame から始めます。宣言的 HTML アプローチにより数分で動く VR シーンを作れます。
- レンダリングや性能をより細かく制御したい場合は Three.js または Babylon.js を使います。
- 必ず実機ヘッドセットでテストしてください。デスクトッププレビューとは体験が大きく異なります。
- 乗り物酔いを防ぐため、安定した高フレームレート（72-90+ FPS）を維持します。
- API 全体の参照は [MDN WebXR Device API](https://developer.mozilla.org/en-US/docs/Web/API/WebXR_Device_API) を確認してください。

