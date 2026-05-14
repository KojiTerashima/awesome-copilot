# Servers & Infrastructure Reference

Web サーバー、ホスティング、展開、およびインフラストラクチャの概念。

## Web サーバー

### 一般的な Web サーバー

#### Nginx

High-performance web server and reverse proxy.

**特徴**:
- 負荷分散
- リバースプロキシ
- 静的ファイルの提供
- SSL/TLS終端

**基本構成**:```nginx
server {
    listen 80;
    server_name example.com;
    
    # Serve static files
    location / {
        root /var/www/html;
        index index.html;
    }
    
    # Proxy to backend
    location /api {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # SSL configuration
    listen 443 ssl;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
}
```#### Apache HTTP サーバー

広く使用されている Web サーバー。

**特徴**:
- .htaccessのサポート
- モジュールシステム
- 仮想ホスティング

**基本的な .htaccess**:```apache
# Redirect to HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Custom error pages
ErrorDocument 404 /404.html

# Cache control
<FilesMatch "\.(jpg|jpeg|png|gif|css|js)$">
    Header set Cache-Control "max-age=31536000, public"
</FilesMatch>
```#### Node.js サーバー

**Express.js**:```javascript
const express = require('express');
const app = express();

app.use(express.json());
app.use(express.static('public'));

app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```**内蔵HTTPサーバー**:```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end('<h1>Hello World</h1>');
});

server.listen(3000);
```## ホスティング オプション

### 静的ホスティング

静的サイトの場合 (HTML、CSS、JS)。

**プラットフォーム**:
- **Vercel**: 自動展開、サーバーレス機能
- **Netlify**: ビルド自動化、エッジ機能
- **GitHub ページ**: パブリック リポジトリは無料
- **Cloudflare ページ**: 高速グローバル CDN
- **AWS S3 + CloudFront**: スケーラブル、セットアップが必要

**展開**:```bash
# Vercel
npx vercel

# Netlify
npx netlify deploy --prod

# GitHub Pages (via Git)
git push origin main
```### サービスとしてのプラットフォーム (PaaS)

マネージド アプリケーション ホスティング。

**プラットフォーム**:
- **Heraku**: 簡単な導入、アドオン
- **鉄道**: 最新の開発者エクスペリエンス
- **レンダリング**: 統合プラットフォーム
- **Google App Engine**: 自動スケーリング
- **Azure App Service**: Microsoft クラウド

**例 (Heraku)**:```bash
# Deploy
git push heroku main

# Scale
heroku ps:scale web=2

# View logs
heroku logs --tail
```### サービスとしてのインフラストラクチャ (IaaS)

仮想サーバー (より多くの制御、より多くのセットアップ)。

**プロバイダー**:
- **AWS EC2**: Amazon 仮想サーバー
- **Google Compute Engine**: Google VM
- **DigitalOcean Droplets**: シンプルな VPS
- **Linode**: 開発者向けの VPS

### コンテナ化

**ドッカー**:```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
# Build image
docker build -t my-app .

# Run container
docker run -p 3000:3000 my-app
```**Docker Compose**:```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```### Kubernetes

コンテナ オーケストレーション プラットフォーム。

**コンセプト**:
- **ポッド**: 展開可能な最小ユニット
- **サービス**: ポッドを公開する
- **展開**: レプリカの管理
- **入力**: HTTP ルーティング

## コンテンツ配信ネットワーク (CDN)

高速コンテンツ配信のための分散ネットワーク。

**利点**:
- ロード時間の短縮
- サーバー負荷の軽減
- DDoS保護
- 地理的分布

**人気の CDN**:
- **Cloudflare**: 無料利用枠、DDoS 保護
- **AWS CloudFront**: Amazon CDN
- **高速**: エッジ コンピューティング
- **Akamai**: エンタープライズ CDN

**図書館用 CDN**:```html
<!-- CDN-hosted library -->
<script src="https://cdn.jsdelivr.net/npm/vue@3/dist/vue.global.js"></script>
```## ドメイン ネーム システム (DNS)

ドメイン名を IP アドレスに変換します。

### DNS レコード

|タイプ |目的 |例 |
|------|--------|----------|
|あ | IPv4 アドレス | `example.com → 192.0.2.1` |
|ああああ | IPv6 アドレス | `example.com → 2001:db8::1` |
| CNAME |別のドメインへのエイリアス | `www → example.com` |
| MX |メールサーバー | `mail.example.com` |
| TXT |テキスト情報 | SPF、DKIM レコード |
| NS |ネームサーバー | DNS 委任 |

**DNS ルックアップ**:```bash
# Command line
nslookup example.com
dig example.com

# JavaScript (not direct DNS, but IP lookup)
fetch('https://dns.google/resolve?name=example.com')
```### DNS の伝播

DNS の変更が世界中に広がるまでの時間 (通常は 24 ～ 48 時間)。

## SSL/TLS 証明書

クライアントとサーバー間のデータを暗号化します。

### 証明書の種類

- **ドメイン検証 (DV)**: 基本、自動化
- **組織検証 (OV)**: 検証されたビジネス
- **拡張検証 (EV)**: 最高の検証

### 証明書の取得

**暗号化しましょう** (無料):```bash
# Certbot
sudo certbot --nginx -d example.com
```**Cloudflare** (Cloudflare DNS を使用すると無料)

### HTTPS 構成```nginx
# Nginx HTTPS
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```## ロードバランシング

トラフィックを複数のサーバーに分散します。

### 負荷分散アルゴリズム

- **ラウンドロビン**: サーバー間でローテーションします
- **最小接続数**: 接続数が最も少ないサーバーに送信します
- **IP ハッシュ**: クライアント IP に基づいたルート
- **重み付け**: サーバーの容量は異なります

**Nginx ロード バランサー**:```nginx
upstream backend {
    server server1.example.com weight=3;
    server server2.example.com;
    server server3.example.com;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```## リバースプロキシ

リクエストをバックエンドサーバーに転送するサーバー。

**利点**:
- 負荷分散
- SSL終端
- キャッシング
- セキュリティ (バックエンドを非表示)

**Nginx リバース プロキシ**:```nginx
server {
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```## キャッシュ戦略

### ブラウザのキャッシュ```http
Cache-Control: public, max-age=31536000, immutable
```### サーバー側のキャッシュ

**Redis**:```javascript
const redis = require('redis');
const client = redis.createClient();

// Cache data
await client.set('user:1', JSON.stringify(user), {
  EX: 3600 // Expire after 1 hour
});

// Retrieve cached data
const cached = await client.get('user:1');
```### CDN キャッシング

エッジロケーションにキャッシュされた静的アセット。

## 環境変数

ハードコーディングなしの構成。```bash
# .env file
DATABASE_URL=postgresql://localhost/mydb
API_KEY=secret-key-here
NODE_ENV=production
```

```javascript
// Access in Node.js
require('dotenv').config();
const dbUrl = process.env.DATABASE_URL;
```**ベストプラクティス**:
- .env を Git にコミットしないでください
- .env.example をテンプレートとして使用する
- 環境ごとに異なる値
- 安全なシークレット値

## 導入戦略

### 継続的展開 (CD)

コードがプッシュされると自動的にデプロイされます。

**GitHub アクション**:```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - run: npm run deploy
```### ブルーグリーン展開

2 つの同一の環境、スイッチ トラフィック。

### カナリアのデプロイメント

ユーザーのサブセットに段階的に展開します。

### ローリング展開

インスタンスを段階的に更新します。

## プロセスマネージャー

アプリケーションを実行し続けます。

### PM2```bash
# Start application
pm2 start app.js

# Start with name
pm2 start app.js --name my-app

# Cluster mode (use all CPUs)
pm2 start app.js -i max

# Monitor
pm2 monit

# Restart
pm2 restart my-app

# Stop
pm2 stop my-app

# Logs
pm2 logs

# Startup script (restart on reboot)
pm2 startup
pm2 save
```### システムド

Linux サービスマネージャー。```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node App

[Service]
ExecStart=/usr/bin/node /path/to/app.js
Restart=always
User=nobody
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```## 監視とロギング

### アプリケーションの監視

- **New Relic**: APM、モニタリング
- **Datadog**: インフラストラクチャ監視
- **Grafana**: 視覚化
- **Prometheus**: メトリクスの収集

### ログの集約

- **Elasticsearch + Kibana**: ログの検索と視覚化
- **Splunk**: エンタープライズ ログ管理
- **Papertrail**: クラウド ロギング

### 稼働時間の監視

- **UptimeRobot**: 無料の稼働時間チェック
- **Pingdom**: 監視サービス
- **StatusCake**: Web サイト監視

## セキュリティのベストプラクティス

### サーバーの強化

- ソフトウェアを常に最新の状態に保つ
- ファイアウォールを使用する (ufw、iptables)
- root SSH ログインを無効にする
- SSH キー (パスワードではない) を使用します。
- ユーザー権限を制限する
- 定期的なバックアップ

### アプリケーションのセキュリティ

- どこでも HTTPS を使用する
- レート制限を実装する
- すべての入力を検証します
- セキュリティヘッダーを使用する
- 依存関係を常に最新の状態に保つ
- 定期的なセキュリティ監査

## バックアップ戦略

### データベースのバックアップ```bash
# PostgreSQL
pg_dump dbname > backup.sql

# MySQL
mysqldump -u user -p dbname > backup.sql

# MongoDB
mongodump --db mydb --out /backup/
```### 自動バックアップ

- 毎日のバックアップ
- 複数の保存期間
- オフサイトストレージ
- 定期的に復元テストを行う

## スケーラビリティ

### 垂直スケーリング

サーバーのリソース (CPU、RAM) を増やします。

**長所**: シンプル  
**短所**: 制限があり、高価です

### 水平方向のスケーリング

さらにサーバーを追加します。

**長所**: 無制限のスケーリング  
**短所**: 複雑でロードバランサが必要

### データベースのスケーリング

- **レプリケーション**: リードレプリカ
- **シャーディング**: データベース間でデータを分割します。
- **キャッシュ**: データベースの負荷を軽減します。

## 用語集の用語

**対象となる重要な用語**:
- アパッチ
- 帯域幅
- CDN
- クラウドコンピューティング
- CNAME
- DNS
- ドメイン
- ドメイン名
- ファイアウォール
- ホスト
- ホットリンク
- IPアドレス
- ISP
- レイテンシー
- ローカルホスト
- Nginx
- 起源
- ポート
- プロキシサーバー
- 往復時間 (RTT)
- サーバー
- サイト
- TLD
- Webサーバー
- ウェブサイト

## 追加のリソース

- [Nginxドキュメント](https://nginx.org/en/docs/)
- [Docker ドキュメント](https://docs.docker.com/)
- [AWS ドキュメント](https://docs.aws.amazon.com/)
- [暗号化してみよう](https://letsencrypt.org/)
- [PM2 ドキュメント](https://pm2.keymetrics.io/docs/)