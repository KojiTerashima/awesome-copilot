---
applyTo: '**'
description: 'OWASP Top 10 2025 に基づく包括的な secure coding standard。55 以上の anti-pattern、検出 regex、現代的な web / backend framework 向け framework-specific 修正、AI / LLM security guidance を含む。'
---

# セキュリティ標準

web application 開発向けの包括的な security rule。すべての anti-pattern には、severity 分類、検出方法、OWASP 2025 参照、修正 code example が含まれる。

**Severity level:**

- **CRITICAL** — 悪用可能な脆弱性。merge 前に必ず修正すること。
- **IMPORTANT** — 大きなリスク。同じ sprint 内で修正すべき。
- **SUGGESTION** — defense-in-depth の改善。今後の iteration で計画する。

---

## OWASP Top 10 — 2025 クイックリファレンス

| # | Category | 主な対策 |
|---|----------|----------|
| A01 | Broken Access Control | すべての endpoint に auth middleware、RBAC、ownership check |
| A02 | Security Misconfiguration | security header、本番で debug を無効化、default credential を使わない |
| A03 | Software Supply Chain Failures *(NEW)* | `npm audit`、lockfile integrity、SBOM、SLSA provenance |
| A04 | Cryptographic Failures | password には Argon2id / bcrypt、すべて TLS、code に secret を置かない |
| A05 | Injection | parameterized query、input validation、user input を raw HTML にしない |
| A06 | Insecure Design | threat modeling、secure design pattern、abuse case testing |
| A07 | Authentication Failures | login の rate limit、安全な session management、MFA |
| A08 | Software or Data Integrity Failures | CDN script に SRI、artifact に署名、unsafe deserialization を避ける |
| A09 | Security Logging and Alerting Failures | security event を log、log に PII を入れない、correlation ID、active alerting |
| A10 | Mishandling of Exceptional Conditions *(NEW)* | すべての error を処理、本番で stack trace を出さない、fail-secure |

---

## Injection Anti-Patterns (I1-I8)

### I1: 文字列連結による SQL Injection

- **Severity**: CRITICAL
- **Detection**: `\$\{.*\}.*(?:SELECT|INSERT|UPDATE|DELETE|FROM|WHERE)`
- **OWASP**: A05

```typescript
// 悪い例
const unsafeResult = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// 良い例 — parameterized query
const safeResult = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

### I2: NoSQL Injection (MongoDB Operator Injection)

- **Severity**: CRITICAL
- **Detection**: `\{\s*\$(?:gt|gte|lt|lte|ne|in|nin|regex|where|exists)`
- **OWASP**: A05

```typescript
// 悪い例 — 攻撃者が { "password": { "$gt": "" } } を送る
const user = await User.findOne({ username: req.body.username, password: req.body.password });

// 良い例 — input 型を検証して cast する
const username = String(req.body.username);
const password = String(req.body.password);
const user = await User.findOne({ username });
const valid = user && await verifyPassword(user.passwordHash, password);
```

### I3: Command Injection (user input を使う exec)

- **Severity**: CRITICAL
- **Detection**: `(?:exec|execSync|execFile|execFileSync)\s*\(.*(?:req\.|params\.|query\.|body\.)`
- **OWASP**: A05

```typescript
// 悪い例 — shell 補間、sync call は event loop を block する
import { execFileSync } from 'node:child_process';
const unsafeOutput = execFileSync('sh', ['-c', `ls -la ${req.query.dir}`]);

// 良い例 — async execFile、argument array、shell なし、time / output 制限あり
import { execFile } from 'node:child_process';
import { promisify } from 'node:util';
const pExecFile = promisify(execFile);

const dir = String(req.query.dir ?? '');
if (!dir || dir.startsWith('-')) throw new Error('Invalid directory');
const { stdout: safeOutput } = await pExecFile('ls', ['-la', '--', dir], {
  timeout: 5_000,      // ハングした process は早く失敗させる
  maxBuffer: 1 << 20,  // 1 MiB 上限で memory 枯渇を防ぐ
});

// 最良 — 上記 async / bounded call に加えて allowlist validation を使う
const allowedDirs = ['/data', '/public'];
if (!allowedDirs.includes(dir)) throw new Error('Invalid directory');
```

server handler では `execFileSync` より async な `execFile` / `spawn` を優先する。sync 版は Node の event loop を block し、DoS 影響を増幅し得る。常に `timeout` と `maxBuffer` を渡して実行を制限すること。

### I4: sanitize されていない HTML 描画による XSS

- **Severity**: CRITICAL
- **Detection**: `(?:v-html|\[innerHTML\]|dangerouslySetInner|bypassSecurityTrust)`
- **OWASP**: A05

すべての frontend framework に適用される。各 framework には、既定の XSS 防御を迂回する API が存在する:

- **React**: raw な user content を使う `dangerouslySetInnerHTML` prop
- **Angular**: sanitize されていない input に対する `[innerHTML]` binding または `bypassSecurityTrustHtml`
- **Vue**: user-controlled な content を使う `v-html` directive

```typescript
// 良い例 — raw HTML を描画する前に DOMPurify で sanitize する
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userContent);

// 最良 — HTML が不要なら text interpolation を使う
// React:   {userContent}
// Angular: {{ userContent }}
// Vue:     {{ userContent }}
```

### I5: user-controlled URL による SSRF

- **Severity**: CRITICAL
- **Detection**: `fetch\((?:req\.|params\.|query\.|body\.|url|href)`
- **OWASP**: A01

```typescript
// 悪い例
const data = await fetch(req.body.url);

// 良い例 — scheme allowlist + hostname allowlist + DNS / IP validation (TOCTOU note を参照)
import { promises as dns } from 'node:dns';

function isPrivateIP(ip: string): boolean {
  // IPv4-mapped IPv6 を正規化 (例: ::ffff:127.0.0.1 → 127.0.0.1)
  const normalized = ip.startsWith('::ffff:') ? ip.slice(7) : ip;
  // IPv4 private / reserved / loopback range
  if (/^(10\.|172\.(1[6-9]|2\d|3[01])\.|192\.168\.|127\.|0\.|169\.254\.)/.test(normalized)) return true;
  // IPv6 loopback、link-local (fe80::/10)、unique-local
  if (/^(::1|fe[89ab]|fc|fd)/i.test(normalized)) return true;
  return false;
}

const parsed = new URL(req.body.url);
if (parsed.protocol !== 'https:') throw new Error('Only HTTPS allowed');
const allowedHosts = ['api.example.com', 'cdn.example.com'];
if (!allowedHosts.includes(parsed.hostname)) throw new Error('Host not allowed');
// 複数 IP を通じた DNS rebinding を防ぐため、すべての A / AAAA record を解決する
const resolved = await dns.lookup(parsed.hostname, { all: true });
if (resolved.length === 0 || resolved.some(({ address }) => isPrivateIP(address))) {
  throw new Error('Private or reserved IPs not allowed');
}
// Note: 本番では、この check と fetch() の間の TOCTOU rebinding を防ぐため、
// HTTP client で解決済み IP を pin すること。undici Agent docs を参照。
const data = await fetch(parsed.toString(), { redirect: 'error' });
```

### I6: File 操作における Path Traversal

- **Severity**: CRITICAL
- **Detection**: `(?:readFile|readFileSync|createReadStream|path\.join)\s*\(.*(?:req\.|params\.|query\.|body\.)`
- **OWASP**: A01

```typescript
// 悪い例
const file = fs.readFileSync(`/data/${req.params.filename}`);

// 良い例 — 許可 directory 内に収まるよう resolve して検証する
import path from 'path';
const basePath = '/data';
const filePath = path.resolve(basePath, req.params.filename);
if (!filePath.startsWith(basePath + path.sep)) throw new Error('Path traversal detected');
const file = fs.readFileSync(filePath);
```

### I7: Template Injection

- **Severity**: CRITICAL
- **Detection**: `(?:render|compile|template)\s*\(.*(?:req\.|params\.|query\.|body\.)`
- **OWASP**: A05

```typescript
// 悪い例 — template source に user input を使う
const html = ejs.render(req.body.template, data);

// 良い例 — template は事前定義し、user input は data としてのみ渡す
const html = ejs.renderFile('./templates/page.ejs', { content: req.body.content });
```

### I8: XXE Injection (XML External Entity)

- **Severity**: CRITICAL
- **Detection**: `(?:parseXml|DOMParser|xml2js|libxmljs).*(?:req\.|body\.|file)`
- **OWASP**: A05

```typescript
// 良い例 — XML parser で external entity を無効にする
import { XMLParser } from 'fast-xml-parser';
const parser = new XMLParser({
  allowBooleanAttributes: true,
  processEntities: false,
  htmlEntities: false,
});
const result = parser.parse(req.body.xml);
```

---

## Authentication Anti-Patterns (AU1-AU8)

### AU1: JWT Algorithm Confusion (alg:none)

- **Severity**: CRITICAL
- **Detection**: `jwt\.verify\((?![^)]*\balgorithms\b)[^)]*\)`
- **OWASP**: A07

```typescript
// 悪い例 — "none" を含む任意の algorithm を受け入れてしまう
const decoded = jwt.verify(token, secret);

// 良い例 — 特定 algorithm を強制する
const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

### AU2: 期限チェックのない JWT

- **Severity**: CRITICAL
- **Detection**: `jwt\.sign\((?![^)]*\b(?:expiresIn|exp)\b)[^)]*\)`
- **OWASP**: A07

```typescript
// 悪い例 — token が期限切れにならない
const token = jwt.sign({ userId: user.id }, secret);

// 良い例 — 短命 token
const token = jwt.sign({ userId: user.id }, secret, { expiresIn: '15m' });
```

### AU3: localStorage に保存された JWT

- **Severity**: IMPORTANT
- **Detection**: `localStorage\.setItem\(.*(?:token|jwt|auth|session)`
- **OWASP**: A07

```typescript
// 悪い例 — XSS で参照できる
localStorage.setItem('accessToken', token);

// 良い例 — server が設定する httpOnly cookie
res.cookie('token', token, { httpOnly: true, secure: true, sameSite: 'strict' });
```

### AU4: password に plaintext / 高速 hash を使用 (MD5 / SHA-1 / SHA-256)

- **Severity**: CRITICAL
- **Detection**: `(?:createHash|md5|sha1|sha256)\s*\(.*password`
- **OWASP**: A04

```typescript
// 悪い例 — 高速 hash、salt なし
const sha256Hash = crypto.createHash('sha256').update(password).digest('hex');

// 良い例 — Argon2id (OWASP 推奨)
import { hash as argon2Hash, argon2id } from 'argon2';
const hashed = await argon2Hash(password, { type: argon2id, memoryCost: 65536, timeCost: 3 });
```

### AU5: login に brute-force 防御がない

- **Severity**: CRITICAL
- **Detection**: `(?:post|router\.post)\s*\(\s*['"]\/(?:login|signin|auth|register|reset)`
- **OWASP**: A07

```typescript
// 悪い例 — rate limit なし
app.post('/api/auth/login', loginHandler);

// 良い例
import rateLimit from 'express-rate-limit';
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 5 });
app.post('/api/auth/login', authLimiter, loginHandler);
```

### AU6: login 時に session regeneration がない (Session Fixation)

- **Severity**: IMPORTANT
- **Detection**: `(?:session|req\.session)\s*\.\s*(?:userId|user|authenticated)\s*=`
- **OWASP**: A07

```typescript
// 良い例 — fixation 防止のため、login 成功後に session ID を再生成する
req.session.regenerate((err) => {
  if (err) return next(err);
  req.session.userId = user.id;
  req.session.save(next);
});
```

関連事項: password 変更や権限昇格時には、user の他の active session も無効化する (例: `tokenVersion` column を増やし、古い version を持つ session を拒否する、または session store を走査して該当 user の entry を破棄する)。

### AU7: state parameter のない OAuth

- **Severity**: CRITICAL
- **Detection**: `authorize\?(?![^\n#]*\bstate=)[^\n#]*`
- **OWASP**: A07

```typescript
// 良い例 — CSRF 防御のため state parameter を含める
const state = crypto.randomBytes(32).toString('hex');
session.oauthState = state;
const authUrl = `https://provider.com/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&state=${state}`;
```

### AU8: public OAuth client に PKCE がない

- **Severity**: IMPORTANT
- **Detection**: `(?:authorization_code|code).*(?!.*code_challenge)`
- **OWASP**: A07

すべての public client (SPA、mobile) で、S256 challenge method による PKCE (Proof Key for Code Exchange) を使うこと。

---

## Authorization Anti-Patterns (AZ1-AZ6)

### AZ1: 新 endpoint に auth middleware がない

- **Severity**: CRITICAL
- **Detection**: `(?:app|router)\.\w+\s*\(\s*['"]\/api\/(?:admin|users|settings)`
- **OWASP**: A01

```typescript
// 悪い例
router.delete('/api/users/:id', deleteUser);

// 良い例
router.delete('/api/users/:id', authenticate, authorize('admin'), deleteUser);
```

### AZ2: client-side のみの authorization

- **Severity**: CRITICAL
- **Detection**: server-side check を伴わない component guard
- **OWASP**: A01

frontend guard は UX 用でしかない。**必ず server 側でも検証すること。**

### AZ3: IDOR (Insecure Direct Object Reference)

- **Severity**: CRITICAL
- **Detection**: ownership check なしの `params\.(?:id|userId|orderId)`
- **OWASP**: A01

```typescript
// 良い例 — ownership を検証する
router.get('/api/orders/:orderId', authenticate, async (req, res) => {
  const order = await Order.findById(req.params.orderId);
  if (!order || order.userId !== req.user.id) {
    return res.status(404).json({ error: 'Not found' });
  }
  res.json(order);
});
```

### AZ4: Mass Assignment

- **Severity**: CRITICAL
- **Detection**: `(?:create|update|findOneAndUpdate)\s*\(\s*req\.body\s*\)`
- **OWASP**: A01

```typescript
// 悪い例
await User.findByIdAndUpdate(id, req.body);

// 良い例 — 許可 field を明示的に選ぶ
const { name, email, avatar } = req.body;
await User.findByIdAndUpdate(id, { name, email, avatar });
```

### AZ5: role parameter による privilege escalation

- **Severity**: CRITICAL
- **Detection**: `req\.body\.role|req\.body\.isAdmin|req\.body\.permissions`
- **OWASP**: A01

```typescript
// 良い例 — input から role を受け取らない
const { name, email, password } = req.body;
const user = await User.create({ name, email, password, role: 'user' });
```

### AZ6: 機密操作で再認証がない

- **Severity**: IMPORTANT
- **Detection**: `(?:delete|destroy|remove).*(?:account|user|organization)` without re-auth
- **OWASP**: A01

account 削除、email 変更、その他の sensitive operation では current password を要求すること。

---

## Secrets Anti-Patterns (S1-S6)

### S1: hardcoded API key / token

- **Severity**: CRITICAL
- **Detection**: `(?:password|secret|api_key|token|apiKey)\s*[:=]\s*['"][A-Za-z0-9+/=]{8,}['"]`
- **OWASP**: A04

```typescript
// 悪い例
const API_KEY = 'sk_live_abc123def456';

// 良い例
const API_KEY = process.env.API_KEY;
```

### S2: Git に commit された .env

- **Severity**: CRITICAL
- **Detection**: `git ls-files .env` (空でなければならない)
- **OWASP**: A04

```gitignore
# .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
```

### S3: client に露出した server secret

- **Severity**: CRITICAL
- **Detection**: `NEXT_PUBLIC_.*(?:SECRET|PRIVATE|PASSWORD|KEY(?!.*PUBLIC))`
- **OWASP**: A02

```bash
# 悪い例
NEXT_PUBLIC_DATABASE_URL=postgresql://...

# 良い例
DATABASE_URL=postgresql://...
NEXT_PUBLIC_API_URL=https://api.example.com
```

Angular では、client bundle に含まれる `environment.ts` file に secret を置いてはならない。

### S4: 設定内の default credential

- **Severity**: CRITICAL
- **Detection**: `(?:admin|root|default|test).*(?:password|pass|pwd)\s*[:=]\s*['"](?:admin|root|password|1234|test)`
- **OWASP**: A02

environment variable と、その zod schema による validation を使うこと。

### S5: CI / CD pipeline log 内の secret

- **Severity**: IMPORTANT
- **Detection**: `(?:echo|console\.log|print).*(?:\$SECRET|\$TOKEN|\$PASSWORD|process\.env)`
- **OWASP**: A09

CI では masked secret を使う。secret を含む environment variable を echo してはならない。

### S6: error response / stack trace 内の sensitive data

- **Severity**: IMPORTANT
- **Detection**: `(?:stack|trace|query|sql).*(?:res\.json|res\.send|c\.JSON)`
- **OWASP**: A10

```typescript
// 良い例 — client には汎用 error、詳細は log にのみ残す
app.use((err, req, res, _next) => {
  logger.error({ err, path: req.path, method: req.method });
  const isDev = process.env.NODE_ENV === 'development';
  res.status(500).json({
    error: 'Internal Server Error',
    ...(isDev && { message: err.message }),
  });
});
```

---

## Headers Anti-Patterns (H1-H8)

### H1: Content-Security-Policy がない

- **Severity**: IMPORTANT
- **Detection**: `Content-Security-Policy` header が存在しない
- **OWASP**: A02

### H2: unsafe-inline と unsafe-eval を含む CSP

- **Severity**: IMPORTANT
- **Detection**: `Content-Security-Policy.*(?:'unsafe-inline'|'unsafe-eval')`
- **OWASP**: A02

nonce ベース CSP を使う: `script-src 'self' 'nonce-{SERVER_GENERATED}'`

### H3: Strict-Transport-Security がない

- **Severity**: IMPORTANT
- **Detection**: `Strict-Transport-Security` header が存在しない
- **OWASP**: A02

値: `max-age=31536000; includeSubDomains; preload`

### H4: X-Content-Type-Options がない

- **Severity**: IMPORTANT
- **Detection**: `X-Content-Type-Options: nosniff` が存在しない
- **OWASP**: A02

### H5: X-Frame-Options がない

- **Severity**: IMPORTANT
- **Detection**: `X-Frame-Options` header が存在しない
- **OWASP**: A02

値: `DENY`。あわせて `Content-Security-Policy: frame-ancestors 'none'` も設定する。

### H6: 緩すぎる Referrer-Policy

- **Severity**: SUGGESTION
- **Detection**: `Referrer-Policy.*(?:unsafe-url|no-referrer-when-downgrade)`
- **OWASP**: A02

使うべき値: `strict-origin-when-cross-origin`

### H7: Permissions-Policy がない

- **Severity**: SUGGESTION
- **Detection**: `Permissions-Policy` header が存在しない
- **OWASP**: A02

値: `camera=(), microphone=(), geolocation=(), payment=()`

### H8: credential 付き CORS wildcard

- **Severity**: CRITICAL
- **Detection**: `(?:cors|Access-Control-Allow-Origin).*\*`
- **OWASP**: A02

```typescript
// 良い例
app.use(cors({
  origin: ['https://app.example.com', 'https://staging.example.com'],
  credentials: true,
}));
```

---

## Frontend Anti-Patterns (FE1-FE8)

### FE1: sanitize されていない HTML 描画

- **Severity**: CRITICAL
- **Detection**: DOMPurify なしの `(?:innerHTML|v-html|dangerouslySetInner)`
- **OWASP**: A05

user-controlled な HTML を描画する前には、常に DOMPurify で sanitize すること。I4 を参照。

### FE2: user input を使う動的 code evaluation

- **Severity**: CRITICAL
- **Detection**: `eval\s*\(`
- **OWASP**: A05

代わりに JSON.parse などの structured data parser を使う。

### FE3: origin validation のない postMessage

- **Severity**: IMPORTANT
- **Detection**: `addEventListener\s*\(\s*['"]message['"].*(?!.*origin)`
- **OWASP**: A01

```typescript
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://trusted.example.com') return;
  processData(event.data);
});
```

### FE4: Prototype Pollution

- **Severity**: IMPORTANT
- **Detection**: `(?:__proto__|constructor\.prototype|Object\.assign)\s*.*(?:req\.|body\.|query\.)`
- **OWASP**: A05

object に merge する前に、user input の key を検証し、filter すること。

### FE5: Open Redirect

- **Severity**: IMPORTANT
- **Detection**: `(?:window\.location|location\.href|router\.push)\s*=\s*(?:req\.|params\.|query\.)`
- **OWASP**: A01

```typescript
// 良い例 — relative path のみ許可
const redirect = new URLSearchParams(window.location.search).get('redirect');
if (redirect?.startsWith('/') && !redirect.startsWith('//')) {
  window.location.href = redirect;
}
```

### FE6: localStorage 内の sensitive data

- **Severity**: IMPORTANT
- **Detection**: `localStorage\.setItem\(.*(?:token|session|credit|ssn|password)`
- **OWASP**: A07

token には httpOnly cookie を使う。

### FE7: CSRF Token がない

- **Severity**: IMPORTANT
- **Detection**: CSRF token または SameSite cookie がない POST / PUT / DELETE form
- **OWASP**: A01

double-submit cookie または synchronizer token を使う。Next.js Server Actions には Origin header による built-in CSRF 防御がある。

### FE8: client-only の input validation

- **Severity**: IMPORTANT
- **Detection**: frontend のみで form validation を行っている
- **OWASP**: A05

server 側でも **必ず** 検証すること。zod、joi、class-validator を使う。

---

## Dependencies Anti-Patterns (D1-D5)

### D1: 既知の脆弱性を持つ dependency

- **Severity**: CRITICAL
- **Detection**: `npm audit --audit-level=high` が non-zero で終了する
- **OWASP**: A03

### D2: lockfile の不整合

- **Severity**: IMPORTANT
- **Detection**: `npm ci` が失敗する
- **OWASP**: A08

### D3: Typosquatting リスク

- **Severity**: IMPORTANT
- **Detection**: 新 dependency 名の手動 review
- **OWASP**: A03

### D4: 新 dependency の postinstall script

- **Severity**: IMPORTANT
- **Detection**: 新 dependency の package.json に `"postinstall"`
- **OWASP**: A03

### D5: 本番で pin されていない version

- **Severity**: SUGGESTION
- **Detection**: `":\s*["']\*["']|":\s*["']latest["']`
- **OWASP**: A03

---

## API Anti-Patterns (AP1-AP6)

### AP1: rate limiting のない新 endpoint

- **Severity**: IMPORTANT
- **OWASP**: A05

### AP2: depth limit のない GraphQL

- **Severity**: IMPORTANT
- **Detection**: depth / complexity limit のない `new ApolloServer`
- **OWASP**: A05

```typescript
import depthLimit from 'graphql-depth-limit';
const server = new ApolloServer({
  schema,
  validationRules: [depthLimit(5)],
  introspection: process.env.NODE_ENV !== 'production',
});
```

### AP3: validation のない file upload

- **Severity**: IMPORTANT
- **Detection**: type / size check のない `multer|formidable|busboy`
- **OWASP**: A05

```typescript
const upload = multer({
  dest: 'uploads/',
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowed.includes(file.mimetype));
  },
});
```

### AP4: signature verification のない webhook

- **Severity**: CRITICAL
- **OWASP**: A08

必ず webhook signature (Stripe、GitHub HMAC など) を検証すること。

### AP5: 内部情報を露出する API

- **Severity**: IMPORTANT
- **Detection**: `(?:stack|trace|query|sql).*(?:res\.json|res\.send)`
- **OWASP**: A10

### AP6: request body size limit がない

- **Severity**: IMPORTANT
- **Detection**: `limit` なしの `express\.json\(\)`
- **OWASP**: A05

```typescript
app.use(express.json({ limit: '100kb' }));
```

---

## AI / LLM Security Anti-Patterns (AI1-AI3)

### AI1: user input による prompt injection

- **Severity**: CRITICAL
- **Detection**: sanitize なしで LLM prompt に連結された user input
- **OWASP**: A05 (Injection)

```typescript
// 悪い例 — prompt に user input を直接埋め込む
const response = await llm.complete(`Summarize this: ${userInput}`);

// 良い例 — system / user message を分離した structured input
const response = await llm.complete({
  system: "You are a summarization assistant. Only summarize the provided text.",
  user: userInput,
});
```

### AI2: sanitize なしで SQL / shell に使われる LLM output

- **Severity**: CRITICAL
- **Detection**: validation なしで `db.query()`、`exec()`、template literal に渡される LLM response
- **OWASP**: A05 (Injection)

LLM output を安全だと信頼してはならない。**信頼できない user input と同様に扱う** こと。query は parameterize し、shell argument は escape し、render 前に HTML を sanitize する。

### AI3: LLM response の output validation がない

- **Severity**: IMPORTANT
- **Detection**: schema validation なしで render / 実行される LLM response
- **OWASP**: A08 (Software or Data Integrity Failures)

LLM output は application logic で使う前に、期待 schema (Zod、JSON Schema) に対して検証すること。期待 structure に一致しない response は拒否する。

---

## Logging Anti-Patterns (L1-L4)

### L1: security event が log されていない

- **Severity**: IMPORTANT
- **OWASP**: A09

log すべき対象: auth failure、access denied、rate limit hit、input validation failure、password change。

### L2: log 内の sensitive data

- **Severity**: CRITICAL
- **Detection**: `(?:log|logger)\.\w+\(.*(?:password|token|secret|ssn|credit)`
- **OWASP**: A09

```typescript
import pino from 'pino';
const logger = pino({ redact: ['req.headers.authorization', 'req.body.password'] });
```

### L3: Trace ID がない

- **Severity**: SUGGESTION
- **OWASP**: A09

### L4: Log Injection

- **Severity**: IMPORTANT
- **Detection**: `console\.log\(.*\+.*(?:req\.|user\.|body\.)`
- **OWASP**: A09

string concatenation ではなく、structured logging (JSON、自動 escape) を使う。

---

## Framework-Specific: React / Next.js (RX1-RX4)

### RX1: auth なしの Server Action

- **Severity**: CRITICAL
- **Detection**: `auth()` または session check のない `'use server'` function
- **OWASP**: A01

```typescript
'use server';
import { auth } from '@/auth';
export async function deleteUser(id: string) {
  const session = await auth();
  if (!session?.user || session.user.role !== 'admin') throw new Error('Unauthorized');
  await db.user.delete({ where: { id } });
}
```

### RX2: client で `NEXT_PUBLIC_` なしの process.env を参照

- **Severity**: IMPORTANT
- **Detection**: `NEXT_PUBLIC_` なしで `process.env` に触る `'use client'` file
- **OWASP**: A02

### RX3: data を漏らす RSC serialization

- **Severity**: IMPORTANT
- **OWASP**: A01

DB object を Client Component に渡す前に、必要な field のみ選択すること。

### RX4: API route を保護しない middleware.ts

- **Severity**: IMPORTANT
- **Detection**: `/api/` を含まない `config.matcher`
- **OWASP**: A01

---

## Framework-Specific: Angular (NG1-NG3)

### NG1: user input に対する bypassSecurityTrustHtml

- **Severity**: CRITICAL
- **Detection**: `bypassSecurityTrust(?:Html|Script|Style|Url|ResourceUrl)`
- **OWASP**: A05

`bypassSecurityTrust` を呼ぶ前に、**まず** DOMPurify で sanitize すること。

### NG2: Template Expression Injection

- **Severity**: IMPORTANT
- **OWASP**: A05

user-controlled な template に対して JitCompilerFactory を使ってはならない。

### NG3: auth を付与しない HttpInterceptor

- **Severity**: IMPORTANT
- **OWASP**: A07

auth token には一元化された `HttpInterceptorFn` を使う。

---

## Framework-Specific: Express (EX1-EX4)

### EX1: helmet.js がない

- **Severity**: IMPORTANT
- **OWASP**: A02

```typescript
import helmet from 'helmet';
app.use(helmet());
app.disable('x-powered-by');
```

### EX2: body size limit のない express.json()

- **Severity**: IMPORTANT
- **OWASP**: A05

```typescript
app.use(express.json({ limit: '100kb' }));
```

### EX3: secure flag のない cookie

- **Severity**: IMPORTANT
- **OWASP**: A07

```typescript
res.cookie('session', value, {
  httpOnly: true, secure: true, sameSite: 'strict', maxAge: 3600000, path: '/',
});
```

### EX4: stack trace を露出する error handler

- **Severity**: IMPORTANT
- **OWASP**: A10

error detail は development mode でのみ露出させること。

---

## Framework-Specific: Go (GO1-GO3)

### GO1: security 操作での math/rand

- **Severity**: CRITICAL
- **Detection**: security-related file 内の `math/rand` import
- **OWASP**: A04

暗号学的に安全な乱数には `crypto/rand` を使う。

### GO2: TLS InsecureSkipVerify

- **Severity**: CRITICAL
- **Detection**: `InsecureSkipVerify:\s*true`
- **OWASP**: A04

代わりに system CA pool (既定) を使う。

### GO3: SQL での文字列補間

- **Severity**: CRITICAL
- **Detection**: `fmt\.Sprintf\s*\(.*(?:SELECT|INSERT|UPDATE|DELETE|FROM|WHERE)`
- **OWASP**: A05

```go
// 良い例 — parameterized
db.Where("id = ?", userID).Find(&user)
```

---

## Security Headers Template

### helmet.js (Express)

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'"],
      connectSrc: ["'self'"],
      frameAncestors: ["'none'"],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
  frameguard: { action: 'deny' },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
}));
app.disable('x-powered-by');
```

---

## JWT Validation Checklist

1. 期待する algorithm で signature を検証する — `alg: none` は拒否する
2. algorithm を強制する: `algorithms: ['RS256']` または `['ES256']`
3. `exp` を確認する — 期限切れ token は拒否する
4. `iat` を確認する — 古すぎる時刻に発行された token は拒否する
5. `aud` を確認する — この service 向けでない token は拒否する
6. `iss` を確認する — 未知 issuer の token は拒否する
7. httpOnly cookie に保存する — localStorage は使わない
8. 短命 access token (15 分) と refresh token rotation を使う
9. 署名 key は定期的に rotate する

---

## Secure Cookie Flag

```
Set-Cookie: session=value; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| Flag | Purpose | 使う場面 |
|------|---------|----------|
| `HttpOnly` | JavaScript から参照不可にする (XSS による token 窃取を防ぐ) | 常に |
| `Secure` | HTTPS 上でのみ送信する | 常に |
| `SameSite=Strict` | same-site request でのみ送信する (最強の CSRF 防御) | auth / session cookie |
| `SameSite=Lax` | top-level navigation では送信される (中程度の CSRF 防御) | cross-site top-level navigation が必要な cookie (例: OAuth 戻り) |
| `Path=/` | cookie の scope を制限する | 常に |
| `Max-Age` | 明示的な有効期限 (`Expires` より推奨) | 常に |

---

## Security Checklist

### Authentication と Session
- [ ] password は Argon2id または bcrypt (cost >= 12) で hash されている
- [ ] JWT は RS256 / ES256 で署名され、verify 時に algorithm を強制している
- [ ] access token は 15 分以内で期限切れになる
- [ ] refresh token は one-time use、rotation され、httpOnly cookie に保存されている
- [ ] login、registration、password reset に rate limiting がある
- [ ] authentication 後に session が再生成される
- [ ] privileged account に MFA が利用できる

### Authorization
- [ ] すべての API endpoint に auth middleware がある
- [ ] すべての resource access に ownership check がある (IDOR 防止)
- [ ] server-side authorization を行っている (frontend guard は UX 用のみ)
- [ ] mass assignment を防いでいる (明示的 field selection)
- [ ] sensitive operation に再認証が必要

### Input と Output
- [ ] すべての user input が server 側で validation されている (zod / joi / class-validator)
- [ ] すべての database 操作に parameterized query を使っている
- [ ] user content を描画するときは HTML output を sanitize (DOMPurify) している
- [ ] 本番の error response に stack trace が出ていない

### Secret
- [ ] source code に hardcoded secret がない
- [ ] `.env` file が `.gitignore` に入っている
- [ ] server secret が client に露出していない (`NEXT_PUBLIC_` を secret に使っていない)
- [ ] startup 時に environment variable が validation されている

### Header
- [ ] Content-Security-Policy が設定されている (nonce ベース推奨)
- [ ] preload 付き Strict-Transport-Security
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY
- [ ] Referrer-Policy: strict-origin-when-cross-origin
- [ ] 未使用 API を制限する Permissions-Policy
- [ ] CORS が既知 origin に限定されている

### Dependency
- [ ] CI で `npm audit` (または同等) が通っている
- [ ] lockfile が commit され、`npm ci` で検証されている
- [ ] 新 dependency に typosquatting や postinstall script がないか review している
- [ ] 本番で wildcard や "latest" version を使っていない

### Logging
- [ ] security event が log されている (auth failure、access denied、rate limit など)
- [ ] log に sensitive data (password、token、PII) がない
- [ ] correlation ID を含む structured logging を使っている
- [ ] 異常パターンに対する alert が設定されている
