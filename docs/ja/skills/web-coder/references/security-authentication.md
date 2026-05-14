# セキュリティと認証のリファレンス

Web セキュリティ、認証、暗号化、安全なコーディングの実践に関する包括的なリファレンス。

## Web セキュリティの基礎

### CIA トライアド

情報セキュリティの中核原則:
- **機密性**: 許可された当事者のみがデータにアクセス可能
- **完全性**: データは正確で変更されないままです。
- **可用性**: 必要なときにシステムとデータにアクセス可能

### セキュリティヘッダー```http
# Content Security Policy
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com 'nonce-<random-base64-value>'; style-src 'self' 'nonce-<random-base64-value>'; object-src 'none'

# HTTP Strict Transport Security
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

# X-Frame-Options (clickjacking protection)
X-Frame-Options: DENY

# X-Content-Type-Options (MIME sniffing)
X-Content-Type-Options: nosniff

# X-XSS-Protection (legacy, use CSP instead)
X-XSS-Protection: 1; mode=block

# Referrer-Policy
Referrer-Policy: strict-origin-when-cross-origin

# Permissions-Policy
Permissions-Policy: geolocation=(), microphone=(), camera=()
```### CSP (コンテンツ セキュリティ ポリシー)

XSS およびデータ インジェクション攻撃を軽減します。

**指示**:
- `default-src`: 他のディレクティブのフォールバック
- `script-src`: JavaScript ソース
- `style-src`: CSS ソース
- `img-src`: 画像ソース
- `font-src`: フォントソース
- `connect-src`: Fetch/XMLHttpRequest の宛先
- `frame-src`: iframe ソース
- `object-src`: プラグインのソース

**値**:
- `'self'`: 同じ起源
- `'none'`: すべてブロック
- `'unsafe-inline'`: インライン スクリプト/スタイルを許可します (回避)
- `'unsafe-eval'`: eval() を許可します (回避します)
- `https:`: HTTPS ソースのみ
- `https://example.com`: 特定のドメイン

## HTTPS と TLS

### TLS (トランスポート層セキュリティ)

クライアントとサーバーの間で転送されるデータを暗号化します。

**TLS ハンドシェイク**:
1. Client Hello (サポートされているバージョン、暗号スイート)
2. Server Hello (選択したバージョン、暗号スイート)
3. サーバー証明書
4. 鍵交換
5. 完了（接続が確立されました）

**バージョン**:
- TLS 1.0、1.1 (非推奨)
- TLS 1.2 (現在の標準)
- TLS 1.3 (最新、高速)

### SSL証明書

**タイプ**:
- **ドメイン検証済み (DV)**: 基本的な検証
- **組織検証済み (OV)**: ビジネス検証
- **拡張検証 (EV)**: 厳格な検証

**認証局**: 証明書を発行する信頼できるエンティティ

**自己署名**: ブラウザーによって信頼されていません (開発/テストのみ)

### HSTS (HTTP Strict Transport Security)

ブラウザに HTTPS の使用を強制します。```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```- `max-age`: 持続時間 (秒)
- `includeSubDomains`: すべてのサブドメインに適用
- `preload`: ブラウザーのプリロード リストに送信します

## 認証

### 認証と認可

- **認証**: 身元を確認します (「あなたは誰ですか?」)
- **権限**: 権限を確認します (「何ができますか?」)

### 一般的な認証方法

#### 1. セッションベースの認証```javascript
// Login
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  // Verify credentials
  if (verifyCredentials(username, password)) {
    req.session.userId = user.id;
    res.json({ success: true });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Protected route
app.get('/profile', requireAuth, (req, res) => {
  const user = getUserById(req.session.userId);
  res.json(user);
});

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy();
  res.json({ success: true });
});
```**長所**: シンプル、サーバー制御セッション  
**短所**: ステートフル、スケーラビリティの問題、CSRF の脆弱性

#### 2. トークンベースの認証 (JWT)```javascript
// Login
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  if (verifyCredentials(username, password)) {
    const token = jwt.sign(
      { userId: user.id, role: user.role },
      SECRET_KEY,
      { expiresIn: '1h' }
    );
    res.json({ token });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Protected route
app.get('/profile', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    const user = getUserById(decoded.userId);
    res.json(user);
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
});
```**長所**: ステートレス、スケーラブル、ドメイン間で動作します。  
**短所**: 有効期限が切れる前に取り消すことができず、サイズのオーバーヘッドが発生します

#### 3.OAuth 2.0

委任されたアクセスのための承認フレームワーク。

**役割**:
- **リソース所有者**: エンドユーザー
- **クライアント**: アクセスを要求しているアプリケーション
- **認可サーバー**: トークンを発行します。
- **リソース サーバー**: 保護されたリソースをホストします。

**フローの例** (認証コード):
1. クライアントは認証サーバーにリダイレクトします
2. ユーザーが認証し、許可を与える
3. 認証サーバーはコードをリダイレクトして返します
4. クライアントはアクセス トークンのコードを交換します
5. クライアントはトークンを使用してリソースにアクセスします

#### 4. 多要素認証 (MFA)

複数の検証要素が必要です。
- **知っていること**: パスワード
- **お持ちのもの**: 電話機、ハードウェア トークン
- **あなたそのもの**: 生体認証

### パスワードセキュリティ```javascript
const bcrypt = require('bcrypt');

// Hash password
async function hashPassword(password) {
  const saltRounds = 10;
  return await bcrypt.hash(password, saltRounds);
}

// Verify password
async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}
```**ベストプラクティス**:
- ✅ bcrypt、scrypt、または Argon2 を使用する
- ✅ 最低 8 文字 (12 文字以上を推奨)
- ✅ 文字の混合が必要です
- ✅ レート制限を実装する
- ✅ 失敗後にアカウント ロックアウトを使用する
- ❌ プレーンテキストのパスワードは決して保存しないでください
- ❌ パスワードの長さを制限しないでください（正当な範囲内）
- ❌ パスワードを電子メールで送信しないでください

## 一般的な脆弱性

### XSS (クロスサイト スクリプティング)

悪意のあるスクリプトを Web ページに挿入します。

**タイプ**:
1. **保存された XSS**: データベースに保存された悪意のあるスクリプト
2. **反映された XSS**: URL 内のスクリプトが応答に反映されました
3. **DOM ベースの XSS**: クライアント側のスクリプト操作

**予防**:```javascript
// ❌ Vulnerable
element.innerHTML = userInput;

// ✅ Safe
element.textContent = userInput;

// ✅ Escape HTML
function escapeHTML(str) {
  return str.replace(/[&<>"']/g, (match) => {
    const map = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#39;'
    };
    return map[match];
  });
}

// ✅ Use DOMPurify for rich content
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```### CSRF (クロスサイト リクエスト フォージェリ)

ユーザーを騙して望ましくないアクションを実行させます。

**防止**：```javascript
// CSRF token
app.get('/form', (req, res) => {
  const csrfToken = generateToken();
  req.session.csrfToken = csrfToken;
  res.render('form', { csrfToken });
});

app.post('/transfer', (req, res) => {
  if (req.body.csrfToken !== req.session.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  // Process request
});

// SameSite cookie attribute
Set-Cookie: sessionId=abc; SameSite=Strict; Secure; HttpOnly
```### SQL インジェクション

悪意のある SQL コードの挿入。

**防止**：```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE username = '${username}'`;

// ✅ Parameterized queries
const query = 'SELECT * FROM users WHERE username = ?';
db.execute(query, [username]);

// ✅ ORM/Query builder
const user = await User.findOne({ where: { username } });
```### CORS の構成ミス```javascript
// ❌ Vulnerable (allows any origin)
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

// ✅ Whitelist specific origins
const allowedOrigins = ['https://example.com'];
if (allowedOrigins.includes(origin)) {
  res.setHeader('Access-Control-Allow-Origin', origin);
  res.setHeader('Access-Control-Allow-Credentials', 'true');
}
```### クリックジャッキング

ユーザーをだまして非表示の要素をクリックさせる。

**防止**：```http
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN

# Or with CSP
Content-Security-Policy: frame-ancestors 'none'
Content-Security-Policy: frame-ancestors 'self'
```### ファイルアップロードの脆弱性```javascript
// Validate file type
const allowedTypes = ['image/jpeg', 'image/png'];
if (!allowedTypes.includes(file.mimetype)) {
  return res.status(400).json({ error: 'Invalid file type' });
}

// Check file size
const maxSize = 5 * 1024 * 1024; // 5MB
if (file.size > maxSize) {
  return res.status(400).json({ error: 'File too large' });
}

// Sanitize filename
const sanitizedName = file.name.replace(/[^a-z0-9.-]/gi, '_');

// Store outside web root
const uploadPath = '/secure/uploads/' + sanitizedName;

// Use random filenames
const filename = crypto.randomBytes(16).toString('hex') + path.extname(file.name);
```## 暗号化

### 暗号化とハッシュ化

- **暗号化**: 可逆的 (キーを使用して復号化)
- **ハッシュ**: 一方向変換

### 対称暗号化

暗号化と復号化に同じキーを使用します。```javascript
const crypto = require('crypto');

function encrypt(text, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return iv.toString('hex') + ':' + encrypted;
}

function decrypt(text, key) {
  const parts = text.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encrypted = parts[1];
  const decipher = crypto.createDecipheriv('aes-256-cbc', key, iv);
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```### 公開鍵暗号化

暗号化 (公開) と復号化 (秘密) に異なるキー。

**使用例**:
- TLS/SSL証明書
- デジタル署名
- SSHキー

### ハッシュ関数```javascript
const crypto = require('crypto');

// SHA-256
const hash = crypto.createHash('sha256').update(data).digest('hex');

// HMAC (keyed hash)
const hmac = crypto.createHmac('sha256', secretKey).update(data).digest('hex');
```### デジタル署名

信頼性と完全性を検証します。```javascript
const { privateKey, publicKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048
});

// Sign
const sign = crypto.createSign('SHA256');
sign.update(data);
const signature = sign.sign(privateKey, 'hex');

// Verify
const verify = crypto.createVerify('SHA256');
verify.update(data);
const isValid = verify.verify(publicKey, signature, 'hex');
```## 安全なコーディングの実践

### 入力の検証```javascript
// Validate email
function isValidEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// Validate and sanitize
function sanitizeInput(input) {
  // Remove dangerous characters
  return input.replace(/[<>\"']/g, '');
}

// Whitelist approach
function isValidUsername(username) {
  return /^[a-zA-Z0-9_]{3,20}$/.test(username);
}
```### 出力エンコーディング

コンテキストに基づいてデータをエンコードします。
- **HTML コンテキスト**: エスケープ `< > & " '`
- **JavaScript コンテキスト**: JSON.stringify() を使用します。
- **URL コンテキスト**: encodeURIComponent() を使用します。
- **CSS コンテキスト**: 特殊文字をエスケープします

### 安全なストレージ```javascript
// ❌ Don't store sensitive data in localStorage
localStorage.setItem('token', token); // XSS can access

// ✅ Use HttpOnly cookies
res.cookie('token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 3600000
});

// ✅ For sensitive client-side data, encrypt first
const encrypted = encrypt(sensitiveData, encryptionKey);
sessionStorage.setItem('data', encrypted);
```### レート制限```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later'
});

app.use('/api/', limiter);

// Stricter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true
});

app.use('/api/login', authLimiter);
```### エラー処理```javascript
// ❌ Expose internal details
catch (error) {
  res.status(500).json({ error: error.message });
}

// ✅ Generic error message
catch (error) {
  console.error(error); // Log internally
  res.status(500).json({ error: 'Internal server error' });
}
```## セキュリティテスト

### ツール
- **OWASP ZAP**: セキュリティ スキャナー
- **Burp Suite**: Web 脆弱性スキャナー
- **nmap**: ネットワーク スキャナー
- **SQLMap**: SQL インジェクション テスト
- **Nikto**: Web サーバー スキャナー

### チェックリスト
- [ ] あらゆる場所で HTTPS が適用される
- [ ] セキュリティヘッダーが設定されました
- [ ] 認証は安全に実装されています
- [ ] すべてのエンドポイントで承認がチェックされました
- [ ] 入力の検証とサニタイズ
- [ ] 出力エンコーディング
- [ ] CSRF保護
- [ ] SQL インジェクションの防止
- [ ] XSS 防止
- [ ] レート制限
- [ ] 安全なセッション管理
- [ ] パスワードを安全に保管
- [ ] ファイルアップロードのセキュリティ
- [ ] エラー処理により情報が漏洩しない
- [ ] 依存関係は最新です
- [ ] セキュリティのログ記録と監視

## 用語集の用語

**対象となる重要な用語**:
- 認証
- 認証者
- 認証局
- チャレンジレスポンス認証
- CIA
- 暗号
- 暗号スイート
- 暗号文
- 資格情報
- クロスサイト リクエスト フォージェリ (CSRF)
- クロスサイトスクリプティング (XSS)
- 暗号解析
- 暗号化
- 復号化
- サービス拒否 (DoS)
- デジタル証明書
- デジタル署名
- 分散型サービス拒否 (DDoS)
- 暗号化
- フェデレーション ID
- 指紋採取
- ファイアウォール
- HSTS
- アイデンティティプロバイダー (IdP)
- ミットM
- 多要素認証
- ノンス
- オワスプ
- 平文
- 最小特権の原則
- 特権付き
- 公開鍵暗号化
- 信頼当事者
- リプレイ攻撃
- 塩
- 安全なコンテキスト
- セキュア ソケット レイヤ (SSL)
- セッションハイジャック
- 署名（セキュリティ）
- SQLインジェクション
- 対称キー暗号化
- トランスポート層セキュリティ (TLS)

## 追加のリソース

- [OWASP トップ 10](https://owasp.org/www-project-top-ten/)
- [MDN Web セキュリティ](https://developer.mozilla.org/en-US/docs/Web/Security)
- [セキュリティヘッダー](https://securityheaders.com/)
- [SSL Labs](https://www.ssllabs.com/)