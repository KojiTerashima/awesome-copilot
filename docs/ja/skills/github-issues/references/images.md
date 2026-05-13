# Issues とコメント内の画像

CLI 経由で GitHub の issue 本文やコメントに画像をプログラム的に埋め込む方法。

## 方法（信頼性順）

### 1. GitHub Contents API（プライベートリポジトリに推奨）

同じリポジトリ内のブランチに画像ファイルを push し、認証済みの閲覧者が表示できる URL で参照します。

**Step 1: ブランチを作成**

```bash
# デフォルトブランチの SHA を取得
SHA=$(gh api repos/{owner}/{repo}/git/ref/heads/main --jq '.object.sha')

# 新しいブランチを作成
gh api repos/{owner}/{repo}/git/refs -X POST \
  -f ref="refs/heads/{username}/images" \
  -f sha="$SHA"
```

**Step 2: Contents API 経由で画像をアップロード**

```bash
# 画像を Base64 エンコードしてアップロード
BASE64=$(base64 -i /path/to/image.png)

gh api repos/{owner}/{repo}/contents/docs/images/my-image.png \
  -X PUT \
  -f message="Add image" \
  -f content="$BASE64" \
  -f branch="{username}/images" \
  --jq '.content.path'
```

画像ごとに繰り返します。Contents API はファイルごとにコミットを作成します。

**Step 3: markdown で参照**

```markdown
![Description](https://github.com/{owner}/{repo}/raw/{username}/images/docs/images/my-image.png)
```

> **重要:** `raw.githubusercontent.com` ではなく、`github.com/{owner}/{repo}/raw/{branch}/{path}` 形式を使ってください。`raw.githubusercontent.com` の URL はプライベートリポジトリで 404 を返します。`github.com/.../raw/...` 形式が機能するのは、閲覧者がログイン済みでリポジトリへのアクセス権を持っている場合、ブラウザが認証クッキーを送信するためです。

**Pros:** 閲覧者がアクセス権を持つ任意のリポジトリで使える、画像がバージョン管理に残る、有効期限がない。  
**Cons:** コミットが作成される、閲覧者の認証が必要、リポジトリにアクセスできないユーザーやメール通知では画像が表示されない。

### 2. Gist ホスティング（公開画像のみ）

画像を gist のファイルとしてアップロードします。公開して問題ない画像にのみ使えます。

```bash
# プレースホルダーファイル付きの gist を作成
gh gist create --public -f description.md <<< "Image hosting gist"

# 注意: gh gist edit はバイナリファイルをサポートしていません。
# gist にバイナリ内容を追加するには API を使う必要があります。
```

> **制限:** CLI 経由の gist ではバイナリファイルのアップロードをサポートしていません。Base64 エンコードしてテキストとして保存する必要がありますが、画像としては表示されません。推奨しません。

### 3. ブラウザアップロード（表示の信頼性が最も高い）

恒久的な画像 URL を得る最も確実な方法は GitHub Web UI です。

1. ブラウザで issue/comment を開く
2. コメントエディタに画像をドラッグ＆ドロップまたは貼り付け
3. GitHub が恒久的な `https://github.com/user-attachments/assets/{UUID}` URL を生成
4. これらの URL はリポジトリアクセスがない人にも有効で、メール通知でも表示される

> **API でできない理由:** GitHub の `upload/policies/assets` エンドポイントはブラウザセッション（CSRF トークン + クッキー）を必要とします。API トークンで呼ぶと HTML のエラーページが返ります。`user-attachments` URL を生成する公開 API はありません。

## スクリーンショットをプログラム的に撮る

ローカル Chrome と `puppeteer-core` を使って HTML モックアップのスクリーンショットを撮れます。

```javascript
const puppeteer = require('puppeteer-core');

const browser = await puppeteer.launch({
  executablePath: '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
  defaultViewport: { width: 900, height: 600, deviceScaleFactor: 2 }
});

const page = await browser.newPage();
await page.setContent(htmlString);

// 特定の要素をスクリーンショット
const elements = await page.$$('.section');
for (let i = 0; i < elements.length; i++) {
  await elements[i].screenshot({ path: `mockup-${i + 1}.png` });
}

await browser.close();
```

> **Note:** ネットワーク分離により、MCP Playwright は localhost に接続できない場合があります。代わりにローカルの Chrome インストールと puppeteer-core を使ってください。

## クイックリファレンス

| Method | Private repos | Permanent | No auth needed | API-only |
|--------|:---:|:---:|:---:|:---:|
| Contents API + `github.com/raw/` | ✅ | ✅ | ❌ | ✅ |
| Browser drag-drop (`user-attachments`) | ✅ | ✅ | ✅ | ❌ |
| `raw.githubusercontent.com` | ❌ (404) | ✅ | ❌ | ✅ |
| Gist | Public only | ✅ | ✅ | ❌ (no binary) |

## よくある落とし穴

- **`raw.githubusercontent.com` はプライベートリポジトリで 404 を返します。** URL に有効なトークンがあっても同様です。GitHub の CDN は認証ヘッダーを通しません。
- **API のダウンロード URL は一時的です。** `gh api repos/.../contents/...` の `download_url` に含まれる URL は期限付きトークンを含みます。
- **`upload/policies/assets` はブラウザセッションが必要です。** CLI からこのエンドポイントを呼び出そうとしないでください。
- **大きなファイルの Base64 エンコード** は API のペイロード上限に達する可能性があります。Contents API のファイルサイズ上限は約 100MB ですが、Base64 エンコードしたペイロードでは実用上の上限はより低くなります。
- **メール通知** では認証が必要な画像は表示されません。メールでの可読性が重要なら、ブラウザアップロード方式を使ってください。

