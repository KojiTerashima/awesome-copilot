# メディアとグラフィックのリファレンス

Web 用のマルチメディア コンテンツ、グラフィックス、および関連テクノロジ。

## 画像フォーマット

### JPEG/JPG

写真の非可逆圧縮。

**特徴**:
- 写真に最適
- 透明度はサポートされていません
- ファイルサイズが小さい
- 編集すると画質が劣化します

**使用方法**:```html
<img src="photo.jpg" alt="Photo">
```### PNG

透明性のあるロスレス圧縮。

**特徴**:
- アルファチャンネル（透明度）をサポート
- JPEGよりもファイルサイズが大きい
- ロゴ、グラフィック、スクリーンショットに適しています
- PNG-8 (256 色) 対 PNG-24 (1600 万色)```html
<img src="logo.png" alt="Logo">
```### WebP

圧縮率が向上した最新の形式。

**特徴**:
- JPEG/PNGより小さい
- 透明性をサポート
- アニメーションをサポート
- 古いブラウザではサポートされていません```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Fallback">
</picture>
```### AVIF

次世代の画像フォーマット。

**特徴**:
- WebP よりも優れた圧縮率
- HDRをサポート
- エンコードが遅い
- 限定的なブラウザのサポート

### GIF

アニメーション画像（限定色）。

**特徴**:
- 最大 256 色
- アニメーションをサポート
- 単純な透明度 (アルファなし)
- 最新の代替手段を検討する (ビデオ、WebP)

### SVG (スケーラブル ベクター グラフィックス)

XML ベースのベクター グラフィックス。

**特徴**:
- 品質を損なうことなく拡張可能
- シンプルなグラフィックの場合はファイル サイズが小さい
- CSS/JS で操作可能
- アニメーションのサポート```html
<!-- Inline SVG -->
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>

<!-- External SVG -->
<img src="icon.svg" alt="Icon">
```**SVG の作成**:```html
<svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Rectangle -->
  <rect x="10" y="10" width="80" height="60" fill="red" />
  
  <!-- Circle -->
  <circle cx="150" cy="40" r="30" fill="blue" />
  
  <!-- Path -->
  <path d="M10 100 L100 100 L50 150 Z" fill="green" />
  
  <!-- Text -->
  <text x="50" y="180" font-size="20">Hello SVG</text>
</svg>
```## キャンバス API

2D ラスター グラフィックス (ビットマップ)。

### 基本セットアップ```html
<canvas id="myCanvas" width="400" height="300"></canvas>
```

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Draw rectangle
ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 100, 50);

// Draw circle
ctx.beginPath();
ctx.arc(200, 150, 50, 0, Math.PI * 2);
ctx.fillStyle = 'blue';
ctx.fill();

// Draw line
ctx.beginPath();
ctx.moveTo(50, 200);
ctx.lineTo(350, 250);
ctx.strokeStyle = 'green';
ctx.lineWidth = 3;
ctx.stroke();

// Draw text
ctx.font = '30px Arial';
ctx.fillStyle = 'black';
ctx.fillText('Hello Canvas', 50, 100);

// Draw image
const img = new Image();
img.onload = () => {
  ctx.drawImage(img, 0, 0);
};
img.src = 'image.jpg';
```### Canvas メソッド```javascript
// Paths
ctx.beginPath();
ctx.moveTo(x, y);
ctx.lineTo(x, y);
ctx.arc(x, y, radius, startAngle, endAngle);
ctx.closePath();
ctx.fill();
ctx.stroke();

// Transforms
ctx.translate(x, y);
ctx.rotate(angle);
ctx.scale(x, y);
ctx.save(); // Save state
ctx.restore(); // Restore state

// Compositing
ctx.globalAlpha = 0.5;
ctx.globalCompositeOperation = 'source-over';

// Export
const dataURL = canvas.toDataURL('image/png');
canvas.toBlob(blob => {
  // Use blob
}, 'image/png');
```## WebGL

ブラウザ上の 3D グラフィックス。

**使用例**:
- 3D ビジュアライゼーション
- ゲーム
- データの視覚化
- VR/AR

**ライブラリ**:
- **Three.js**: 簡単な 3D グラフィックス
- **Babylon.js**: ゲーム エンジン
- **PixiJS**: 2D WebGL レンダラー```javascript
// Three.js example
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();

renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Create cube
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

camera.position.z = 5;

// Render loop
function animate() {
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();
```## ビデオ

### HTML5 ビデオ要素```html
<video controls width="640" height="360">
  <source src="video.mp4" type="video/mp4">
  <source src="video.webm" type="video/webm">
  Your browser doesn't support video.
</video>
```**属性**:
- `controls`: 再生コントロールを表示します
- `autoplay`: 自動的に起動します
- `loop`: ビデオを繰り返します
- `muted`: 音声をミュートします
- `poster`: サムネイル画像
- `preload`: なし/メタデータ/自動

### ビデオ形式

- **MP4 (H.264)**: 広くサポートされています
- **WebM (VP8/VP9)**: オープン形式
- **Ogg (Theora)**: オープンフォーマット

### JavaScript コントロール```javascript
const video = document.querySelector('video');

// Playback
video.play();
video.pause();
video.currentTime = 10; // Seek to 10s

// Properties
video.duration; // Total duration
video.currentTime; // Current position
video.paused; // Is paused?
video.volume = 0.5; // 0.0 to 1.0
video.playbackRate = 1.5; // Speed

// Events
video.addEventListener('play', () => {});
video.addEventListener('pause', () => {});
video.addEventListener('ended', () => {});
video.addEventListener('timeupdate', () => {});
```## オーディオ

### HTML5 オーディオ要素```html
<audio controls>
  <source src="audio.mp3" type="audio/mpeg">
  <source src="audio.ogg" type="audio/ogg">
</audio>
```### オーディオ形式

- **MP3**: 広くサポートされています
- **AAC**: 高品質
- **Ogg Vorbis**: オープンフォーマット
- **WAV**: 非圧縮

### ウェブオーディオ API

高度なオーディオ処理:```javascript
const audioContext = new AudioContext();

// Load audio
fetch('audio.mp3')
  .then(response => response.arrayBuffer())
  .then(arrayBuffer => audioContext.decodeAudioData(arrayBuffer))
  .then(audioBuffer => {
    // Create source
    const source = audioContext.createBufferSource();
    source.buffer = audioBuffer;
    
    // Create gain node (volume)
    const gainNode = audioContext.createGain();
    gainNode.gain.value = 0.5;
    
    // Connect: source -> gain -> destination
    source.connect(gainNode);
    gainNode.connect(audioContext.destination);
    
    // Play
    source.start();
  });
```## レスポンシブ画像

### srcset とサイズ```html
<!-- Different resolutions -->
<img src="image-800.jpg"
     srcset="image-400.jpg 400w,
             image-800.jpg 800w,
             image-1200.jpg 1200w"
     sizes="(max-width: 600px) 100vw,
            (max-width: 900px) 50vw,
            800px"
     alt="Responsive image">

<!-- Pixel density -->
<img src="image.jpg"
     srcset="image.jpg 1x,
             image@2x.jpg 2x,
             image@3x.jpg 3x"
     alt="High DPI image">
```### 絵要素

アートディレクションとフォーマット切り替え：```html
<picture>
  <!-- Different formats -->
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  
  <!-- Different crops for mobile/desktop -->
  <source media="(max-width: 600px)" srcset="image-mobile.jpg">
  <source media="(min-width: 601px)" srcset="image-desktop.jpg">
  
  <!-- Fallback -->
  <img src="image.jpg" alt="Fallback">
</picture>
```## 画像の最適化

### ベストプラクティス

1. **正しい形式を選択**:
   - 写真: JPEG、WebP、AVIF
   - グラフィック/ロゴ: PNG、SVG、WebP
   - アニメーション: ビデオ、WebP

2. **画像を圧縮**:
   - 圧縮ツールを使用する
   - 品質とファイルサイズのバランスを取る
   - 大きな画像用のプログレッシブ JPEG

3. **レスポンシブ画像**:
   - 適切なサイズを提供する
   - srcset/pictureを使用する
   - デバイスのピクセル比を考慮する

4. **遅延読み込み**:```html
   <img src="image.jpg" loading="lazy" alt="Lazy loaded">
   ```5. **寸法**:```html
   <img src="image.jpg" width="800" height="600" alt="With dimensions">
   ```## 画像読み込みテクニック

### 遅延読み込み```html
<!-- Native lazy loading -->
<img src="image.jpg" loading="lazy" alt="Image">

<!-- Intersection Observer -->
<img data-src="image.jpg" class="lazy" alt="Image">
```

```javascript
const images = document.querySelectorAll('.lazy');
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});

images.forEach(img => observer.observe(img));
```### プログレッシブ機能強化```html
<!-- Low quality placeholder -->
<img src="image-tiny.jpg"
     data-src="image-full.jpg"
     class="blur"
     alt="Progressive image">
```## ファビコン

ウェブサイトのアイコン:```html
<!-- Standard -->
<link rel="icon" href="/favicon.ico" sizes="any">

<!-- Modern -->
<link rel="icon" href="/icon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">

<!-- Multiple sizes -->
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16.png">
```## マルチメディアのベスト プラクティス

### パフォーマンス

- ファイルサイズの最適化
- 適切な形式を使用する
- 遅延読み込みの実装
- 配信には CDN を使用します
- ビデオを圧縮する

### アクセシビリティ

- 画像に代替テキストを提供します
- ビデオにキャプション/字幕を含めます
- 音声のトランスクリプトを提供します
- 音声付きで自動再生しない
- キーボードコントロールを確実にする

### SEO

- わかりやすいファイル名
- キーワードを含む代替テキスト
- 構造化データ (schema.org)
- 画像サイトマップ

## 用語集の用語

**対象となる重要な用語**:
- アルファ
・ベースライン（イメージ）
- ベースライン (スクリプト作成)
- キャンバス
- ファビコン
- JPEG
- 可逆圧縮
- 非可逆圧縮
- PNG
- 段階的な強化
- 品質の価値観
- ラスター画像
- レンダリング
- レンダリングエンジン
- SVG
- ベクター画像
- WebGL
- WebP

## 追加のリソース

- [MDN Canvas チュートリアル](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial)
- [SVG チュートリアル](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial)
- [WebGL の基礎](https://webglfundamentals.org/)
- [レスポンシブ画像ガイド](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)
- [Web オーディオ API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)