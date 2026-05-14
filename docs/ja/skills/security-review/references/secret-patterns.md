# 秘密および資格情報の検出パターン

ステップ 3 (秘密および暴露スキャン) 中にこのファイルをロードします。

---

## 信頼性の高い秘密のパターン

これらのパターンは、ほとんどの場合、本当の秘密を示しています。

### API キーとトークン```regex
# OpenAI
sk-[a-zA-Z0-9]{48}

# Anthropic
sk-ant-[a-zA-Z0-9\-_]{90,}

# AWS Access Key
AKIA[0-9A-Z]{16}

# AWS Secret Key (look for near AWS_ACCESS_KEY_ID assignment)
[0-9a-zA-Z/+]{40}

# GitHub Token
gh[pousr]_[a-zA-Z0-9]{36,}
github_pat_[a-zA-Z0-9]{82}

# Stripe
sk_live_[a-zA-Z0-9]{24,}
rk_live_[a-zA-Z0-9]{24,}

# Twilio Account SID
AC[a-z0-9]{32}
# Twilio API Key
SK[a-z0-9]{32}

# SendGrid
SG\.[a-zA-Z0-9\-_.]{66}

# Slack
xoxb-[0-9]+-[0-9]+-[a-zA-Z0-9]+
xoxp-[0-9]+-[0-9]+-[0-9]+-[a-zA-Z0-9]+
xapp-[0-9]+-[A-Z0-9]+-[0-9]+-[a-zA-Z0-9]+

# Google API Key
AIza[0-9A-Za-z\-_]{35}

# Google OAuth
[0-9]+-[0-9A-Za-z_]{32}\.apps\.googleusercontent\.com

# Cloudflare (near CF_API_TOKEN)
[a-zA-Z0-9_\-]{37}

# Mailgun
key-[a-zA-Z0-9]{32}

# Heroku
[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}
```### 秘密鍵```regex
-----BEGIN (RSA |EC |OPENSSH |DSA |PGP )?PRIVATE KEY( BLOCK)?-----
-----BEGIN CERTIFICATE-----
```### データベース接続文字列```regex
# MongoDB
mongodb(\+srv)?:\/\/[^:]+:[^@]+@

# PostgreSQL / MySQL
(postgres|postgresql|mysql):\/\/[^:]+:[^@]+@

# Redis with password
redis:\/\/:[^@]+@

# Generic connection string with password
(connection[_-]?string|connstr|db[_-]?url).*password=
```### ハードコードされたパスワード (変数名シグナル)```regex
# Variable names that suggest secrets
(password|passwd|pwd|secret|api_key|apikey|auth_token|access_token|private_key)
  \s*[=:]\s*["'][^"']{8,}["']
```---

## エントロピーベースの検出

代入コンテキスト内の 20 文字を超える文字列リテラルに適用されます。
高エントロピー (シャノン エントロピー > 4.5 ビット/文字) + 長さ > 20 = 秘密である可能性があります。```
Calculate entropy: -sum(p * log2(p)) for each character frequency p
Threshold: > 4.5 bits/char AND > 20 chars AND assigned to a variable
```除外すべき一般的な誤検知:
- ロレム・イプサムテキスト
- HTML/CSSコンテンツ
- Base64 でエンコードされた非機密設定 (ただし、フラグとメモ)
- UUID/GUID (エントロピーは高いがフォーマットは認識可能)

---

## 決してコミットしてはいけないファイル

これらのファイルがリポジトリ ルートに存在するか、git によって追跡されているかどうかをフラグします。```
.env
.env.local
.env.production
.env.staging
*.pem
*.key
*.p12
*.pfx
id_rsa
id_ed25519
credentials.json
service-account.json
gcp-key.json
secrets.yaml
secrets.json
config/secrets.yml
````.gitignore` もチェックしてください。シークレット ファイル パターンが .gitignore にない場合は、それにフラグを立てます。

---

## CI/CD および IaC の秘密のリスク

### GitHub アクション — 次のパターンにフラグを立てます。```yaml
# Hardcoded values in env: blocks (should use ${{ secrets.NAME }})
env:
  API_KEY: "actual-value-here"   # VULNERABLE

# Printing secrets
- run: echo ${{ secrets.MY_SECRET }}   # leaks to logs
```### Docker — 以下にフラグを立てます。```dockerfile
# Secrets in ENV (persisted in image layers)
ENV AWS_SECRET_KEY=actual-value

# Secrets passed as build args (visible in image history)
ARG API_KEY=actual-value
```### Terraform — 以下にフラグを立てます。```hcl
# Hardcoded sensitive values (should use var or data source)
password = "hardcoded-password"
access_key = "AKIAIOSFODNN7EXAMPLE"
```---

## 安全なパターン (フラグを立てないでください)

これらは意図的なプレースホルダーです。認識してスキップしてください。```
"your-api-key-here"
"<YOUR_API_KEY>"
"${API_KEY}"
"${process.env.API_KEY}"
"os.environ.get('API_KEY')"
"REPLACE_WITH_YOUR_KEY"
"xxx...xxx"
"sk-..." (in documentation/comments)
```
