# HTTP とネットワークのリファレンス

HTTP プロトコル、ネットワーキングの概念、Web 通信に関する包括的なリファレンス。

## HTTP (ハイパーテキスト転送プロトコル)

クライアントとサーバー間でハイパーテキストを転送するためのプロトコル。ウェブ上のデータ通信の基礎。

### HTTP バージョン

- **HTTP/1.1** (1997): テキストベースの永続的な接続、パイプライン化
- **HTTP/2** (2015): バイナリ プロトコル、多重化、サーバー プッシュ、ヘッダー圧縮
- **HTTP/3** (2022): QUIC (UDP) に基づいており、パフォーマンスが向上し、パケット損失の処理が改善されています。

## リクエストメソッド

|方法 |目的 |冪等 |安全 |キャッシュ可能 |
|----------|-----------|---------------|------|-----------|
|入手 |リソースを取得 |はい |はい |はい |
|投稿 |リソースの作成 |いいえ |いいえ |めったに |
|置く |リソースの更新/置換 |はい |いいえ |いいえ |
|パッチ |部分更新 |いいえ |いいえ |いいえ |
|削除 |リソースの削除 |はい |いいえ |いいえ |
|頭 | GET に似ていますが、本文はありません |はい |はい |はい |
|オプション |許可されたメソッドを取得する |はい |はい |いいえ |
|接続 |トンネルを確立する |いいえ |いいえ |いいえ |
|トレース |エコーリクエスト |はい |はい |いいえ |

**安全**: サーバーの状態を変更しません  
**冪等**: 複数の同一のリクエストは単一のリクエストと同じ効果があります。

## ステータスコード

### 1xx 情報

|コード |メッセージ |意味 |
|------|--------|----------|
| 100 |続ける |クライアントはリクエストを続行する必要があります |
| 101 |プロトコルの切り替え |サーバー切り替えプロトコル |

### 2xx 成功

|コード |メッセージ |意味 |
|------|--------|----------|
| 200 | OK |リクエストは成功しました |
| 201 |作成された |リソースが作成されました |
| 202 |承認済み |受け入れられましたが処理されていません |
| 204 |コンテンツなし |成功しましたが、返されるコンテンツがありません |
| 206 |部分的なコンテンツ |部分リソース (範囲リクエスト) |

### 3xx リダイレクト

|コード |メッセージ |意味 |
|------|--------|----------|
| 301 |永久に移動されました |リソースが永久に移動されました |
| 302 |見つかりました |一時的なリダイレクト |
| 303 |その他を見る |異なる URI での応答 |
| 304 |変更されていません |リソースは変更されていません (キャッシュ) |
| 307 |一時的なリダイレクト | 302 と同様ですが、メソッドを保持します。
| 308 |永続的なリダイレクト | 301 と同様ですが、メソッドを保持します。

### 4xx クライアント エラー|コード |メッセージ |意味 |
|------|--------|----------|
| 400 |不正なリクエスト |無効な構文 |
| 401 |不正 |認証が必要です |
| 403 |禁止 |アクセスが拒否されました |
| 404 |見つかりません |リソースが見つかりません |
| 405 |許可されていないメソッド |メソッドはサポートされていません |
| 408 |リクエストのタイムアウト |リクエストに時間がかかりすぎました |
| 409 |紛争 |リクエストが状態と競合しています |
| 410 |消えた |リソースが永久になくなりました |
| 413 |ペイロードが大きすぎます |リクエスト本文が大きすぎます |
| 414 | URI が長すぎます | URI が長すぎます |
| 415 |サポートされていないメディア タイプ |サポートされていないメディア タイプ |
| 422 |処理できないエンティティ |セマンティック エラー |
| 429 |リクエストが多すぎます |レート制限を超えました |

### 5xx サーバー エラー

|コード |メッセージ |意味 |
|------|--------|----------|
| 500 |内部サーバーエラー |一般的なサーバー エラー |
| 501 |未実装 |メソッドはサポートされていません |
| 502 |不正なゲートウェイ |上流からの無効な応答 |
| 503 |サービスが利用できません |サーバーが一時的に利用不可 |
| 504 |ゲートウェイのタイムアウト |アップストリームのタイムアウト |
| 505 | HTTP バージョンはサポートされていません | HTTP バージョンはサポートされていません |

## HTTP ヘッダー

### リクエストヘッダー```http
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json, text/plain
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Authorization: Bearer token123
Cookie: sessionId=abc123
If-None-Match: "etag-value"
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
Origin: https://example.com
Referer: https://example.com/page
```**一般的なリクエスト ヘッダー**:
- `Accept`: クライアントが受け入れるメディア タイプ
- `Accept-Encoding`: エンコード形式(圧縮)
- `Accept-Language`: 優先言語
- `Authorization`: 認証資格情報
- `Cache-Control`: ディレクティブのキャッシュ
- `Cookie`: サーバーに送信された Cookie
- `Content-Type`: リクエストボディの種類
- `Host`: ターゲットホストとポート
- `If-Modified-Since`: 条件付きリクエスト
- `If-None-Match`: 条件付きリクエスト(ETag)
- `Origin`: リクエストの送信元 (CORS)
- `Referer`: 前のページの URL
- `User-Agent`: クライアント情報

### 応答ヘッダー```http
HTTP/1.1 200 OK
Date: Mon, 04 Mar 2026 12:00:00 GMT
Server: nginx/1.18.0
Content-Type: application/json; charset=utf-8
Content-Length: 348
Content-Encoding: gzip
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Mon, 04 Mar 2026 11:00:00 GMT
Access-Control-Allow-Origin: *
Set-Cookie: sessionId=xyz789; HttpOnly; Secure; SameSite=Strict
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```**一般的な応答ヘッダー**:
- `Access-Control-*`: CORS ヘッダー
- `Cache-Control`: ディレクティブのキャッシュ
- `Content-Encoding`: コンテンツの圧縮
- `Content-Length`: 本体サイズ (バイト単位)
- `Content-Type`: ボディのメディアタイプ
- `Date`: 応答日時
- `ETag`: リソースのバージョン識別子
- `Expires`: 有効期限
- `Last-Modified`: 最終更新日
- `Location`: リダイレクト URL
- `Server`: サーバー ソフトウェア
- `Set-Cookie`: Cookie を設定します
- `Strict-Transport-Security`: HSTS
- `X-Content-Type-Options`: MIME タイプ スニッフィング
- `X-Frame-Options`: クリックジャッキング保護

## CORS (クロスオリジンリソース共有)

クロスオリジンリクエストを許可するメカニズム。

### 簡単なリクエスト

次の場合に自動的に許可されます。
- メソッド: GET、HEAD、または POST
- 安全なヘッダーのみ
- Content-Type: `application/x-www-form-urlencoded`、`multipart/form-data`、または `text/plain`

### プリフライトリクエスト

複雑なリクエストの場合、ブラウザは最初に OPTIONS リクエストを送信します。```http
OPTIONS /api/users HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```### CORS ヘッダー

**リクエスト**:
- `Origin`: リクエストの送信元
- `Access-Control-Request-Method`: 意図した方法
- `Access-Control-Request-Headers`: 意図されたヘッダー

**応答**:
- `Access-Control-Allow-Origin`: 許可されたオリジン (* または特定)
- `Access-Control-Allow-Methods`: 許可されるメソッド
- `Access-Control-Allow-Headers`: 許可されるヘッダー
- `Access-Control-Allow-Credentials`: 資格情報を許可します
- `Access-Control-Max-Age`: プリフライト キャッシュ期間
- `Access-Control-Expose-Headers`: クライアントがアクセスできるヘッダー

## キャッシング

### キャッシュ制御ディレクティブ

**ディレクティブのリクエスト**:
- `no-cache`: キャッシュを使用する前にサーバーで検証します
- `no-store`: キャッシュをまったく行わない
- `max-age=N`: 最大経過時間 (秒単位)
- `max-stale=N`: 古い応答を最大 N 秒間受け入れます
- `min-fresh=N`: 少なくとも N 秒間は新鮮です
- `only-if-cached`: キャッシュされた応答のみを使用します

**応答ディレクティブ**:
- `public`: 任意のキャッシュでキャッシュ可能
- `private`: ブラウザのみでキャッシュ可能
- `no-cache`: 使用前に検証する必要があります
- `no-store`: キャッシュしないでください
- `max-age=N`: N 秒間新鮮
- `s-maxage=N`: 共有キャッシュの最大保存期間
- `must-revalidate`: 古い場合は検証する必要があります
- `immutable`: 内容は変更されません

### 例```http
# Cache for 1 hour
Cache-Control: public, max-age=3600

# Don't cache
Cache-Control: no-store

# Cache in browser only, revalidate after 1 hour
Cache-Control: private, max-age=3600, must-revalidate

# Cache forever (with versioned URLs)
Cache-Control: public, max-age=31536000, immutable
```### 条件付きリクエスト

効率的なキャッシュのために ETags または Last-Modified を使用します。```http
GET /resource HTTP/1.1
If-None-Match: "etag-value"
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
```変更されていない場合:```http
HTTP/1.1 304 Not Modified
ETag: "etag-value"
```## クッキー```http
# Server sets cookie
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; Secure; SameSite=Strict; Max-Age=3600

# Client sends cookie
Cookie: sessionId=abc123; userId=456
```### クッキーの属性

- `Path=/`: Cookie パスのスコープ
- `Domain=example.com`: Cookie ドメインのスコープ
- `Max-Age=N`: N 秒後に期限切れになります
- `Expires=date`: 特定の日付で期限切れになります
- `Secure`: HTTPS 経由でのみ送信されます
- `HttpOnly`: JavaScript からはアクセスできません
- `SameSite=Strict|Lax|None`: CSRF 保護

## REST (表現型状態転送)

Web サービスのアーキテクチャ スタイル。

### REST の原則

1. **クライアントとサーバー**: 懸念事項の分離
2. **ステートレス**: 各リクエストには必要な情報がすべて含まれています
3. **キャッシュ可能**: 応答はキャッシュ可能性を定義する必要があります
4. **統一インターフェイス**: 標準化された通信
5. **階層化システム**: クライアントはエンドサーバーに接続されているかどうかを知りません
6. **コード オン デマンド** (オプション): サーバーは実行可能コードを送信できます。

### RESTful API 設計```
GET    /users           # List users
GET    /users/123       # Get user 123
POST   /users           # Create user
PUT    /users/123       # Update user 123 (full)
PATCH  /users/123       # Update user 123 (partial)
DELETE /users/123       # Delete user 123

GET    /users/123/posts # List posts by user 123
GET    /posts?author=123 # Alternative: filter posts
```### HTTP コンテンツ ネゴシエーション```http
# Client requests JSON
Accept: application/json

# Server responds with JSON
Content-Type: application/json

# Client can accept multiple formats
Accept: application/json, application/xml;q=0.9, text/plain;q=0.8
```## ネットワーキングの基礎

### TCP (伝送制御プロトコル)

信頼性の高いデータ配信を保証する接続指向プロトコル。

**TCP ハンドシェイク** (3 方向):
1. クライアント → サーバー: SYN
2. サーバー → クライアント: SYN-ACK
3. クライアント → サーバー: ACK

**特徴**:
- 確実な配信（再送信）
- 注文されたデータ
- エラーチェック
- フロー制御
- 接続指向

### UDP (ユーザー データグラム プロトコル)

高速データ伝送のためのコネクションレス型プロトコル。

**特徴**:
- 高速 (ハンドシェイクなし)
- 配送保証なし
- 注文なし
- オーバーヘッドの低減
- ストリーミング、ゲーム、DNS に使用

### DNS (ドメインネームシステム)

ドメイン名を IP アドレスに変換します。```
example.com → 93.184.216.34
```**DNS レコードの種類**:
- `A`：IPv4アドレス
- `AAAA`: IPv6アドレス
- `CNAME`: 正規名（エイリアス）
- `MX`：メール交換
- `TXT`: テキストレコード
- `NS`: ネームサーバー

### IP アドレス指定

**IPv4**: `192.168.1.1` (32 ビット)  
**IPv6**: `2001:0db8:85a3:0000:0000:8a2e:0370:7334` (128 ビット)

### ポート

- **既知のポート** (0-1023):
  - 80: HTTP
  - 443: HTTPS
  - 21: FTP
  - 22: SSH
  - 25: SMTP
  - 53:DNS
- **登録済みポート** (1024-49151)
- **動的ポート** (49152-65535)

### 帯域幅と遅延

**帯域幅**: 単位時間あたりに転送されるデータ量 (Mbps、Gbps)  
**Latency**: データ送信の遅延時間 (ミリ秒)

**ラウンドトリップ時間 (RTT)**: リクエストがサーバーに到達し、レスポンスが返されるまでの時間

## Webソケット

単一の TCP 接続を介した全二重通信。```javascript
// Client
const ws = new WebSocket('wss://example.com/socket');

ws.onopen = () => {
  console.log('Connected');
  ws.send('Hello server!');
};

ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

ws.onerror = (error) => {
  console.error('Error:', error);
};

ws.onclose = () => {
  console.log('Disconnected');
};

// Close connection
ws.close();
```**使用例**: チャット、リアルタイム更新、ゲーム、共同編集

## サーバー送信イベント (SSE)

サーバーは HTTP 経由で更新をクライアントにプッシュします。```javascript
// Client
const eventSource = new EventSource('/events');

eventSource.onmessage = (event) => {
  console.log('New message:', event.data);
};

eventSource.addEventListener('custom-event', (event) => {
  console.log('Custom event:', event.data);
});

eventSource.onerror = (error) => {
  console.error('Error:', error);
};

// Close connection
eventSource.close();
```

```http
// Server response
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: First message

data: Second message

event: custom-event
data: Custom message data
```## ベストプラクティス

### やるべきこと
- ✅ どこでも HTTPS を使用する
- ✅ 適切なキャッシュ戦略を実装する
- ✅ 適切な HTTP メソッドを使用する
- ✅ 意味のあるステータスコードを返す
- ✅ レート制限を実装する
- ✅ 圧縮（gzip、brotli）を使用する
- ✅ 適切な CORS ヘッダーを設定する
- ✅ 適切なエラー処理を実装する
- ✅ 接続プーリングを使用する
- ✅ ネットワークパフォーマンスを監視

### やってはいけないこと
- ❌ 機密データには HTTP を使用する
- ❌ CORS セキュリティを無視する
- ❌ 間違ったステータス コードを返す (エラーの場合は 200)
- ❌ 機密データをキャッシュする
- ❌ 大きな非圧縮応答を送信する
- ❌ SSL/TLS 証明書の検証をスキップ
- ❌ 資格情報を URL に保存する
- ❌ エラー時に内部サーバーの詳細が公開される
- ❌ 同期リクエストを使用する

## 用語集の用語

**対象となる重要な用語**:
- アヤックス
- ALPN
- 帯域幅
- キャッシュ可能
- クッキー
- コルス
- CORS セーフリストに登録されたリクエスト ヘッダー
- CORS セーフリストに登録された応答ヘッダー
- クローラー
- 有効接続タイプ
- フェッチディレクティブ
- メタデータリクエストヘッダーの取得
- 禁止されたリクエストヘッダー
- 禁止されたレスポンスヘッダ名
- FTP
- 一般ヘッダー
- HOL ブロック
- HTTP
- HTTPコンテンツ
- HTTPヘッダー
- HTTP/2
- HTTP/3
- HTTPS
- HTTPS RR
- べき等
- IMAP
- レイテンシー
- パケット
- ポップ3
- プロキシサーバー
- クイック
- レート制限
- リクエストヘッダー
- レスポンスヘッダー
- 休憩
- 往復時間 (RTT)
- RTCP
- RTP
- 安全 (HTTP メソッド)
- SMTP
- TCP
- TCPハンドシェイク
- TCP スロースタート
- UDP
- WebSocket

## 追加のリソース

- [MDN HTTP ガイド](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [HTTP/2仕様](https://http2.github.io/)
- [HTTP/3 の説明](https://http3-explained.haxx.se/)
- [REST API チュートリアル](https://restfulapi.net/)