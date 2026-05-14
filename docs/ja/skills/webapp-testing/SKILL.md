---
name: webapp-testing
description: Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs.
---
# Web アプリケーションのテスト

このスキルにより、Playwright オートメーションを使用したローカル Web アプリケーションの包括的なテストとデバッグが可能になります。

可能であれば、Playwright MCP サーバーを使用して作業を行う必要があります。 MCP サーバーが利用できない場合は、Playwright がインストールされているローカル Node.js 環境でコードを実行できます。

## このスキルを使用する場合

このスキルは、次の場合に使用します。

- 実際のブラウザでフロントエンド機能をテストする
- UIの動作とインタラクションを検証する
- Web アプリケーションの問題をデバッグする
- ドキュメント化またはデバッグのためにスクリーンショットをキャプチャする
- ブラウザコンソールのログを検査する
- フォームの送信とユーザー フローを検証する
- ビューポート間でレスポンシブ デザインを確認する

## 前提条件

- Node.js がシステムにインストールされている
- ローカルで実行されている Web アプリケーション (またはアクセス可能な URL)
- Playwright が存在しない場合は自動的にインストールされます

## コア機能

### 1. ブラウザの自動化

- URL に移動します
- ボタンとリンクをクリックします
- フォームフィールドに記入します
- ドロップダウンを選択します
- ダイアログとアラートを処理する

### 2. 検証

- 要素の存在をアサート
- テキストの内容を確認する
- 要素の可視性を確認する
- URLを検証する
- 応答動作をテストする

### 3. デバッグ

- スクリーンショットをキャプチャする
- コンソールログの表示
- ネットワークリクエストを検査する
- 失敗したテストをデバッグする

## 使用例

### 例 1: 基本的なナビゲーション テスト```javascript
// Navigate to a page and verify title
await page.goto("http://localhost:3000");
const title = await page.title();
console.log("Page title:", title);
```### 例 2: フォームの対話```javascript
// Fill out and submit a form
await page.fill("#username", "testuser");
await page.fill("#password", "password123");
await page.click('button[type="submit"]');
await page.waitForURL("**/dashboard");
```### 例 3: スクリーンショットのキャプチャ```javascript
// Capture a screenshot for debugging
await page.screenshot({ path: "debug.png", fullPage: true });
```## ガイドライン

1. **アプリが実行中であることを常に確認してください** - テストを実行する前に、ローカル サーバーにアクセスできることを確認してください
2. **明示的な待機を使用する** - 要素またはナビゲーションが完了するまで待ってから対話します。
3. **失敗時にスクリーンショットを取得** - 問題のデバッグに役立つようにスクリーンショットを取得します。
4. **リソースをクリーンアップ** - 完了したら必ずブラウザを閉じてください
5. **タイムアウトを適切に処理する** - 遅い操作に対して適切なタイムアウトを設定する
6. **段階的にテスト** - 複雑なフローの前に単純な対話から開始します
7. **セレクターを賢く使用する** - CSS クラスよりも data-testid またはロールベースのセレクターを優先します

## 一般的なパターン

### パターン: 要素を待つ```javascript
await page.waitForSelector("#element-id", { state: "visible" });
```### パターン: 要素が存在するかどうかを確認する```javascript
const exists = (await page.locator("#element-id").count()) > 0;
```### パターン: コンソール ログを取得する```javascript
page.on("console", (msg) => console.log("Browser log:", msg.text()));
```### パターン: エラーの処理```javascript
try {
  await page.click("#button");
} catch (error) {
  await page.screenshot({ path: "error.png" });
  throw error;
}
```## 制限事項

- Node.js環境が必要です
- ネイティブ モバイル アプリをテストできません (代わりに React Native Testing Library を使用してください)
- 複雑な認証フローで問題が発生する可能性があります
- 一部の最新のフレームワークでは、特定の構成が必要な場合があります

## ヘルパー関数

[`test-helper.js`](./assets/test-helper.js) では、要素の待機、スクリーンショットのキャプチャ、エラーの処理などの一般的なタスクを簡素化するいくつかのヘルパー関数が利用できます。これらの関数をテストにインポートして使用すると、可読性と保守性が向上します。