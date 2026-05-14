# ブラウザとエンジンのリファレンス

Web ブラウザ、レンダリング エンジン、およびブラウザ固有の情報。

## 主要なブラウザ

### Google Chrome

**エンジン**: Blink (レンダリング)、V8 (JavaScript)  
**リリース**: 2008  
**市場シェア**: ~65% (デスクトップ)  

**開発者ツール**: 
- 要素パネル
- コンソール
- ネットワークタブ
- パフォーマンスプロファイラー
- ライトハウス監査

### モジラ Firefox

**エンジン**: Gecko (レンダリング)、SpiderMonkey (JavaScript)  
**リリース**: 2004  
**市場シェア**: ~3% (デスクトップ)  

**特徴**:
- プライバシーを重視する
- コンテナタブ
- 強化された追跡保護
- 開発者版

### アップルサファリ

**エンジン**: WebKit (レンダリング)、JavaScriptCore (JavaScript)  
**リリース**: 2003  
**市場シェア**: ~20% (デスクトップ)、iOS で優勢  

**特徴**:
- エネルギー効率が高い
- プライバシー重視
- インテリジェントな追跡防止
- iOS ではブラウザのみが許可されます

### Microsoft Edge

**エンジン**: Blink (2020 年以降は Chromium ベース)  
**リリース**: 2015 (EdgeHTML)、2020 (Chromium)  

**特徴**:
- Windowsの統合
- コレクション
- 垂直タブ
- IEモード（互換性）

### オペラ

**エンジン**: 点滅  
**ベース**: クロム  

**特徴**:
- 内蔵VPN
- 広告ブロッカー
- サイドバー

## レンダリング エンジン

### 点滅

**使用者**: Chrome、Edge、Opera、Vivaldi  
**フォーク元**: WebKit (2013)  
**言語**: C++  

### ウェブキット

**使用者**: Safari  
**起源**: KHTML (KDE)  
**言語**: C++  

### ヤモリ

**使用者**: Firefox  
**開発者**: Mozilla  
**言語**: C++、Rust  

### レガシー エンジン

- **Trident**: Internet Explorer (非推奨)
- **EdgeHTML**: オリジナルの Edge (非推奨)
- **Presto**: 古い Opera (非推奨)

## JavaScript エンジン

|エンジン |ブラウザ |言語 |
|----------|----------|----------|
| V8 |クローム、エッジ | C++ |
|スパイダーモンキー | Firefox | C++、Rust |
| JavaScriptコア |サファリ | C++ |
|チャクラ | IE/エッジ (レガシー) | C++ |

### V8 の機能

- JITコンパイル
- インラインキャッシュ
- 隠しクラス
- ガベージコレクション
- WASMのサポート

## ブラウザ開発ツール

### Chrome デベロッパーツール```javascript
// Console API
console.log('message');
console.table(array);
console.time('label');
console.timeEnd('label');

// Command Line API
$() // document.querySelector()
$$() // document.querySelectorAll()
$x() // XPath query
copy(object) // Copy to clipboard
monitor(function) // Log function calls
```**パネル**:
- 要素: DOM 検査
- コンソール: JavaScript コンソール
- ソース: デバッガ
- ネットワーク: HTTP リクエスト
- パフォーマンス: プロファイリング
- メモリ: ヒープ スナップショット
- 用途: 保管、サービスワーカー
- セキュリティ: 証明書情報
- ライトハウス: 監査

### Firefox 開発ツール

**ユニークな機能**:
- CSS グリッド インスペクター
- フォントエディター
- アクセシビリティインスペクター
- ネットワークスロットリング

## ブラウザ間の互換性

### ブラウザー プレフィックス (ベンダー プレフィックス)```css
/* Legacy - use autoprefixer instead */
.element {
  -webkit-transform: rotate(45deg); /* Chrome, Safari */
  -moz-transform: rotate(45deg); /* Firefox */
  -ms-transform: rotate(45deg); /* IE */
  -o-transform: rotate(45deg); /* Opera */
  transform: rotate(45deg); /* Standard */
}
```**最新のアプローチ**: ビルド ツール (Autoprefixer) を使用する

### ユーザーエージェント文字列```javascript
// Check browser
const userAgent = navigator.userAgent;

if (userAgent.includes('Firefox')) {
  // Firefox-specific code
} else if (userAgent.includes('Chrome')) {
  // Chrome-specific code
}

// Better: Feature detection
if ('serviceWorker' in navigator) {
  // Modern browser
}
```### グレースフル デグラデーションとプログレッシブ エンハンスメント

**グレースフル デグラデーション**: 現代向けにビルド、古い向けにデグレード```css
.container {
  display: grid; /* Modern browsers */
  display: block; /* Fallback */
}
```**プログレッシブな強化**: ベースを構築し、最新のものに合わせて強化します```css
.container {
  display: block; /* Base */
}

@supports (display: grid) {
  .container {
    display: grid; /* Enhancement */
  }
}
```## ブラウザの機能

### サービスワーカー

オフライン機能用のバックグラウンド スクリプト

**サポートされている**: すべての最新ブラウザ

### Webアセンブリ

Web 用のバイナリ命令フォーマット

**サポートされている**: すべての最新ブラウザ

### Web コンポーネント

カスタム HTML 要素

**サポートされている**: すべての最新ブラウザ (ポリフィル付き)

### WebRTC

リアルタイム通信

**サポートされている**: すべての最新ブラウザ

## ブラウザストレージ

|ストレージ |サイズ |有効期限 |範囲 |
|----------|------|---------------|----------|
|クッキー | 4KB |設定可能 |ドメイン |
|ローカルストレージ | 5～10MB |決して |由来 |
|セッションストレージ | 5～10MB |タブを閉じる |由来 |
|インデックス付きDB | 50MB以上 |決して |由来 |

## モバイルブラウザ

### iOS サファリ

- iOS ではブラウザのみが許可されます
- すべての iOS ブラウザは WebKit を使用します
- デスクトップSafariとは異なります

### Chrome モバイル (Android)

- ブリンクエンジン
- デスクトップ Chrome に似ています

### サムスンインターネット

- クロムベース
- Samsung デバイスで人気

## ブラウザ市場シェア (2026 年)

**デスクトップ**:
- クロム: ~65%
- サファリ: ~20%
- エッジ: ~5%
- Firefox: ~3%
- その他: ~7%

**モバイル**:
- クロム: ~65%
- サファリ: ~25%
- Samsung インターネット: ~5%
- その他: ~5%

## ブラウザのテスト

### ツール

- **BrowserStack**: クラウド ブラウザのテスト
- **Sauce Labs**: 自動テスト
- **CrossBrowserTesting**: ライブ テスト
- **LambdaTest**: クロスブラウザーテスト

### 仮想マシン

- **VirtualBox**: 無料の仮想化
- **Parallels**: Mac 仮想化
- **Windows Dev VM**: 無料の Windows VM

## 開発者向け機能

### Chromium ベースの開発者の機能

- **リモート デバッグ**: モバイル デバイスをデバッグします
- **ワークスペース**: ファイルを直接編集します
- **スニペット**: 再利用可能なコード スニペット
- **対象範囲**: 未使用コードの検出

### Firefox 開発者版

- **CSS グリッド インスペクター**
- **フレックスボックスインスペクター**
- **フォントパネル**
- **アクセシビリティ監査**

## ブラウザ拡張機能

### マニフェスト V3 (モダン)```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0",
  "permissions": ["storage", "activeTab"],
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content.js"]
  }]
}
```## 用語集の用語

**対象となる重要な用語**:
- アップルサファリ
- 点滅
- 点滅要素
- ブラウザ
- コンテキストの閲覧
- クロム
- 開発者ツール
- エンジン
- Firefox OS
- ヤモリ
- Google Chrome
- JavaScriptエンジン
- マイクロソフトエッジ
- Microsoft Internet Explorer
- モジラ Firefox
- ネットスケープナビゲーター
- オペラブラウザ
- プレスト
- レンダリングエンジン
- トライデント
- ユーザーエージェント
- ベンダープレフィックス
- ウェブキット

## 追加のリソース

- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- [Firefox 開発者ツール](https://firefox-source-docs.mozilla.org/devtools-user/)
- [Safari Web インスペクター](https://developer.apple.com/safari/tools/)
- [使用できますか](https://caniuse.com/)
- [ブラウザ市場シェア](https://gs.statcounter.com/)