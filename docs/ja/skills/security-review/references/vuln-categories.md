# 脆弱性カテゴリ — 詳細リファレンス

このファイルには、すべての脆弱性カテゴリの詳細な検出ガイダンスが含まれています。
これをスキャン ワークフローのステップ 4 でロードします。

---

## 1. 射出欠陥

### SQL インジェクション
**何を探すべきか:**
- SQL クエリ内の文字列の連結または補間
- 変数を使用した生の `.query()`、`.execute()`、`.raw()` 呼び出し
- ユーザー入力による ORM `whereRaw()`、`selectRaw()`、`orderByRaw()`
- 二次 SQLi: データは安全に保存され、後で安全に使用されません。
- サニタイズされていない入力で呼び出されたストアド プロシージャ

**検出信号 (すべての言語):**```
"SELECT ... " + variable
`SELECT ... ${variable}`
f"SELECT ... {variable}"
"SELECT ... %s" % variable   # Only safe with proper driver parameterization
cursor.execute("... " + input)
db.raw(`... ${req.params.id}`)
```**安全なパターン (パラメータ化):**```js
db.query('SELECT * FROM users WHERE id = ?', [userId])
User.findOne({ where: { id: userId } })  // ORM safe
```**エスカレーション チェッカー:**
- クエリ結果が別のクエリで使用されたことはありますか? (二次)
- テーブル/列名はユーザー制御ですか? (パラメータ化できません - 許可リストに登録する必要があります)

---

### クロスサイト スクリプティング (XSS)
**何を探すべきか:**
- `innerHTML`、`outerHTML`、`document.write()` (ユーザーデータあり)
- React の `dangerouslySetInnerHTML`
- エスケープなしでレンダリングするテンプレート エンジン: `{{{ var }}}` (ハンドルバー)、`!= var` (Pug)
- ユーザーデータを含む jQuery `.html()`、`.append()`
- `eval()`、`setTimeout(string)`、`setInterval(string)` (ユーザーデータ付き)
- DOM ベース: DOM に書き込まれる `location.hash`、`document.referrer`、`window.name`
- 保存された XSS: ユーザー入力は DB に保存され、後でエスケープせずにレンダリングされます。

**フレームワークによる検出:**
- **React**: `dangerouslySetInnerHTML` を除き、デフォルトでは安全です
- **Angular**: `bypassSecurityTrustHtml` を除き、デフォルトでは安全です
- **Vue**: `v-html` を除き、デフォルトでは安全です
- **バニラ JS**: すべての DOM 書き込みが疑わしい

---

### コマンドインジェクション
**探すべきもの (Node.js):**```js
exec(userInput)
execSync(`ping ${host}`)
spawn('sh', ['-c', userInput])
child_process.exec('ls ' + dir)
```**探すべきもの (Python):**```python
os.system(user_input)
subprocess.call(user_input, shell=True)
eval(user_input)
```**何を探すべきか (PHP):**```php
exec($input)
system($_GET['cmd'])
passthru($input)
`$input`  # backtick operator
```**安全な代替方法:** shell=True を指定せずに、配列形式の spawn/subprocess を使用します。コマンドにホワイトリストを使用します。

---

### サーバーサイドリクエストフォージェリ (SSRF)
**何を探すべきか:**
- URL がユーザー制御である HTTP リクエスト
- Webhook、URL プレビュー、画像取得機能
- 外部 URL を取得する PDF ジェネレーター
- ユーザーが指定した URL にリダイレクトします

**高リスクのターゲット:**
- AWS メタデータ サービス: `169.254.169.254`
- 内部サービス: `localhost`、`127.0.0.1`、`10.x.x.x`、`192.168.x.x`
- クラウドメタデータエンドポイント

**検出:**```js
fetch(req.body.url)
axios.get(userSuppliedUrl)
http.get(params.webhook)
```---

## 2. 認証とアクセス制御

### 壊れたオブジェクト レベルの認証 (BOLA / IDOR)
**何を探すべきか:**
- 所有権チェックなしで URL/パラメータから直接取得されたリソース ID
- `userId === currentUser.id` を検証せずに `findById(req.params.id)`
- 連続する数値 ID (容易に推測可能)

**脆弱なパターンの例:**```js
// VULNERABLE: no ownership check
app.get('/api/documents/:id', async (req, res) => {
  const doc = await Document.findById(req.params.id);
  res.json(doc);
});

// SAFE: verify ownership
app.get('/api/documents/:id', async (req, res) => {
  const doc = await Document.findOne({ _id: req.params.id, owner: req.user.id });
  if (!doc) return res.status(403).json({ error: 'Forbidden' });
  res.json(doc);
});
```---

### JWT の脆弱性
**何を探すべきか:**
- `alg: "none"` が受け入れられました
- 弱いシークレットまたはハードコーディングされたシークレット: `secret`、`password`、`1234`
- 有効期限 (`exp` クレーム) の検証なし
- アルゴリズムの混乱 (RS256 → HS256 のダウングレード)
- JWT は `localStorage` に保存されます (XSS リスク。httpOnly Cookie を推奨)

**検出:**```js
jwt.verify(token, secret, { algorithms: ['HS256'] })  // Check algorithms array
jwt.decode(token)  // WARNING: decode does NOT verify signature
```---

### 認証/認可がありません
**何を探すべきか:**
- 管理エンドポイントまたは機密性の高いエンドポイントに認証ミドルウェアがない
- `app.use(authMiddleware)` の後とその前に定義されたルート
- 機能フラグまたはデバッグエンドポイントは運用環境で公開されたままになります
- GraphQL リゾルバーにフィールド レベルでの認証チェックが欠落している

---

### CSRF
**何を探すべきか:**
- CSRFトークンを使用しない状態変更操作(POST/PUT/DELETE)
- SameSite 属性のない認証のために Cookie のみに依存する API
- セッション Cookie に `SameSite=Strict` または `SameSite=Lax` がありません

---

## 3. 秘密と機密データの暴露

### コード内の秘密
次のようなパターンを探します。```
API_KEY = "sk-..."
password = "hunter2"
SECRET = "abc123"
private_key = "-----BEGIN RSA PRIVATE KEY-----"
aws_secret_access_key = "wJalrXUtn..."
```エントロピーヒューリスティック: 代入コンテキストで文字の多様性が高い文字列 > 20 文字
変数名にそう記載されていない場合でも、秘密である可能性があります。

### ログ/エラーメッセージ内```js
console.log('User password:', password)
logger.info({ user, token })   // token shouldn't be logged
res.status(500).json({ error: err.stack })  // stack traces expose internals
```### API レスポンス内の機密データ
- `password_hash`、`ssn`、`credit_card` を含む完全なユーザー オブジェクトを返す
- エラー応答に内部 ID またはシステム パスを含める

---

## 4. 暗号化

### 弱いアルゴリズム
|アルゴリズム |問題 | |で置き換えます
|----------|----------|--------------|
| MD5 |セキュリティのために壊れています | SHA-256 または bcrypt (パスワード) |
| SHA-1 |衝突攻撃 | SHA-256 |
| DES / 3DES |弱い鍵のサイズ | AES-256-GCM |
| RC4 |壊れた | AES-GCM |
| ECBモード | IV なし、パターンが見える |ランダム IV を使用した GCM または CBC |

### 弱いランダム性```js
// VULNERABLE
Math.random()                    // not cryptographically secure
Date.now()                       // predictable
Math.random().toString(36)       // weak token generation

// SAFE
crypto.randomBytes(32)           // Node.js
secrets.token_urlsafe(32)        // Python
```### パスワードのハッシュ化```python
# VULNERABLE
hashlib.md5(password.encode()).hexdigest()
hashlib.sha256(password.encode()).hexdigest()

# SAFE
bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))
argon2.hash(password)
```---

## 5. 安全でない依存関係

### フラグを立てる対象:
- インストールされているバージョン範囲内の既知の CVE を含むパッケージ
- セキュリティアップデートが適用されずに 2 年以上放置されたパッケージ
- 定められた目的に対して非常に広範な権限を持つパッケージ
- 既知の不正なパッケージを引き込む推移的な依存関係
- 現在より大幅に遅れているピン留めされたバージョン (パッチが適用されていない脆弱性の可能性)

### 高リスクパッケージの監視リスト: `references/vulnerable-packages.md` を参照

---

## 6. ビジネスロジック

### 競合状態 (TOCTOU)```js
// VULNERABLE: check then act without atomic lock
const balance = await getBalance(userId);
if (balance >= amount) {
  await deductBalance(userId, amount);  // race condition between check and deduct
}

// SAFE: use atomic DB transaction or optimistic locking
await db.transaction(async (trx) => {
  const user = await User.query(trx).forUpdate().findById(userId);
  if (user.balance < amount) throw new Error('Insufficient funds');
  await user.$query(trx).patch({ balance: user.balance - amount });
});
```### レート制限がありません
次のようなエンドポイントにフラグを立てます。
- 認証資格情報を受け入れる (ログイン、2FA)
- 電子メールまたはSMSを送信する
- 負荷の高い操作を実行する
- ユーザー列挙の公開 (パスワードのリセット、登録)

---

## 7. パストラバーサル```python
# VULNERABLE
filename = request.args.get('file')
with open(f'/var/uploads/{filename}') as f:  # ../../../../etc/passwd

# SAFE
filename = os.path.basename(request.args.get('file'))
safe_path = os.path.join('/var/uploads', filename)
if not safe_path.startswith('/var/uploads/'):
    abort(400)
```
