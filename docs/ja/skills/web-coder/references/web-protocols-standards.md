# Web プロトコルと標準のリファレンス

Web を管理する組織、仕様、標準。

## 標準化団体

### W3C (ワールドワイドウェブコンソーシアム)

Web 標準を開発する国際コミュニティ。

**主要な基準**:
- HTML
- CSS
- XML
- SVG
- WCAG (アクセシビリティ)
- Web API

**ウェブサイト**: https://www.w3.org/

### WHATWG (Web ハイパーテキスト アプリケーション テクノロジ ワーキング グループ)

HTML と DOM の生活標準を維持するコミュニティ。

**主要な基準**:
- HTML リビング スタンダード
- DOM リビング スタンダード
- フェッチ標準
- URL標準

**ウェブサイト**: https://whatwg.org/

### IETF (インターネット エンジニアリング タスク フォース)

インターネット標準を開発します。

**主要な基準**:
- HTTP
-TLS
- TCP/IP
- DNS
- WebRTCプロトコル

**ウェブサイト**: https://www.ietf.org/

### ECMAインターナショナル

情報システムの標準化団体。

**主要な基準**:
- ECMAScript (JavaScript)
- JSON

**ウェブサイト**: https://www.ecma-international.org/

### TC39 (技術委員会 39)

ECMAScript 標準化委員会。

**提案段階**:
- **ステージ 0**: ストローパーソン
- **ステージ 1**: 提案
- **ステージ 2**: ドラフト
- **ステージ 3**: 候補者
- **ステージ 4**: 完了 (次のバージョンに含まれます)

### IANA (インターネット割り当て番号局)

インターネットプロトコルリソースを調整します。

**責任**:
- MIME タイプ
- ポート番号
- プロトコルパラメータ
- TLD (トップレベル ドメイン)

### ICANN (割り当てられた名前と番号のためのインターネット会社)

DNS と IP アドレスを調整します。

## ウェブ標準

### HTML標準

**HTML5 の機能**:
- 意味要素 (`<article>`、`<section>` など)
- オーディオおよびビデオ要素
- キャンバスとSVG
- フォームの強化
- LocalStorage と SessionStorage
- ウェブワーカー
- 地理位置情報 API

### CSS仕様

**CSS モジュール** (各仕様はモジュールです):
- CSS セレクター レベル 4
- CSS フレックスボックス レベル 1
- CSS グリッド レベル 2
- CSSアニメーション
- CSSトランジション
- CSS カスタム プロパティ

### JavaScript 標準**ECMAScript のバージョン**:
- **ES5** (2009): 厳密モード、JSON
- **ES6/ES2015**: クラス、モジュール、アロー関数、プロミス
- **ES2016**: Array.includes()、べき乗演算子 (`**`)
- **ES2017**: async/await、Object.values/entries
- **ES2018**: オブジェクトのレスト/スプレッド、非同期反復
- **ES2019**: Array. flat()、Object.fromEntries
- **ES2020**: オプションのチェイニング、ヌル合体、BigInt
- **ES2021**: 論理割り当て、Promise.any
- **ES2022**: 最上位の待機、クラス フィールド
- **ES2023**: Array.findLast()、Object.groupBy

### Web API仕様

**一般的な API**:
- DOM (ドキュメント オブジェクト モデル)
- APIの取得
- サービスワーカー
- ウェブストレージ
- インデックス付きDB
- WebRTC
- WebGL
- ウェブオーディオAPI
- 支払いリクエストAPI
- Web認証API

## 仕様

### 規範的 vs 非規範的

- **規格**: 準拠のために必要です
- **非規範**: 参考のみ (例、メモ)

### 仕様のライフサイクル

1. **編集者草案**: 作業中です
2. **作業草案**: コミュニティによるレビュー
3. **推奨事項**: 実装とテスト
4. **推奨案**: 最終レビュー
5. **W3C 勧告**: 公式規格

## ブラウザの互換性

### 特徴検出```javascript
// Check feature support
if ('serviceWorker' in navigator) {
  // Use service workers
}

if (window.IntersectionObserver) {
  // Use Intersection Observer
}

if (CSS.supports('display', 'grid')) {
  // Use CSS Grid
}
```### ベースラインの互換性

新しく標準化された機能により、ブラウザの幅広いサポートが実現します。

**広く利用可能**: Firefox、Chrome、Edge、Safari のサポート

### ポリフィル

古いブラウザで最新の機能を提供するコード:```javascript
// Promise polyfill
if (!window.Promise) {
  window.Promise = PromisePolyfill;
}

// Fetch polyfill
if (!window.fetch) {
  window.fetch = fetchPolyfill;
}
```### プログレッシブ機能強化

基本的なブラウザ向けに構築し、最新のブラウザ向けに強化します。```css
/* Base styles */
.container {
  display: block;
}

/* Enhanced for Grid support */
@supports (display: grid) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}
```## IDL (インターフェース定義言語)

**WebIDL**: Web API を定義します。```webidl
interface Element : Node {
  readonly attribute DOMString? tagName;
  DOMString? getAttribute(DOMString qualifiedName);
  undefined setAttribute(DOMString qualifiedName, DOMString value);
};
```## 知っておくべき仕様

- **HTML リビング スタンダード**
- **CSS 仕様** (モジュール式)
- **ECMAScript 言語仕様**
- **HTTP/1.1 (RFC 9112)**
- **HTTP/2 (RFC 9113)**
- **HTTP/3 (RFC 9114)**
- **TLS 1.3 (RFC 8446)**
- **WebSocket プロトコル (RFC 6455)**
- **CORS (フェッチ標準)**
- **サービスワーカー**
- **Web 認証 (WebAuthn)**

## 用語集の用語

**対象となる重要な用語**:
- ベースライン (互換性)
- BCP 47 言語タグ
- ECMA
- ECMAScript
- HTML5
- イアナ
- ICANN
- IDL
- IETF
- ISO
- ITU
- 非規範的
- 規範的
- ポリフィル
- シム
- 仕様
- W3C
- ワイ
- WCAG
- 何WG
- ウェブ標準
- WebIDL

## 追加のリソース

- [W3C標準](https://www.w3.org/TR/)
- [WHATWG 生活基準](https://spec.whatwg.org/)
- [MDN Web ドキュメント](https://developer.mozilla.org/)
- [使用できますか](https://caniuse.com/)
- [TC39 プロポーザル](https://github.com/tc39/proposals)