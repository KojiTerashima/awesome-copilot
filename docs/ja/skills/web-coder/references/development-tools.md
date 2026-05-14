# 開発ツールのリファレンス

Web 開発のためのツールとワークフロー。

## バージョン管理

### Git

分散バージョン管理システム。

**基本的なコマンド**:```bash
# Initialize repository
git init

# Clone repository
git clone https://github.com/user/repo.git

# Check status
git status

# Stage changes
git add file.js
git add . # All files

# Commit
git commit -m "commit message"

# Push to remote
git push origin main

# Pull from remote
git pull origin main

# Branches
git branch feature-name
git checkout feature-name
git checkout -b feature-name # Create and switch

# Merge
git checkout main
git merge feature-name

# View history
git log
git log --oneline --graph
```**ベストプラクティス**:
- 意味のあるメッセージを頻繁にコミットする
- 機能にブランチを使用する
- 押す前に引く
- コミットする前に変更を確認する
- 生成されたファイルには .gitignore を使用します

### GitHub/GitLab/Bitbucket

コラボレーション機能を備えた Git ホスティング プラットフォーム:
- プルリクエスト / マージリクエスト
- コードレビュー
- 問題の追跡
- CI/CDの統合
- プロジェクト管理

## パッケージマネージャー

### npm (ノード パッケージ マネージャー)```bash
# Initialize project
npm init
npm init -y # Skip prompts

# Install dependencies
npm install package-name
npm install -D package-name # Dev dependency
npm install -g package-name # Global

# Update packages
npm update
npm outdated

# Run scripts
npm run build
npm test
npm start

# Audit security
npm audit
npm audit fix
```**package.json**:```json
{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "build": "webpack",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "webpack": "^5.75.0"
  }
}
```### 糸

npm のより高速な代替手段:```bash
yarn add package-name
yarn remove package-name
yarn upgrade
yarn build
```### pnpm

効率的なパッケージマネージャー (ディスクスペースの節約):```bash
pnpm install
pnpm add package-name
pnpm remove package-name
```## ビルドツール

### ウェブパック

モジュールバンドラー:```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    path: __dirname + '/dist',
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ]
};
```### ヴァイテ

高速で最新のビルド ツール:```bash
# Create project
npm create vite@latest my-app

# Dev server
npm run dev

# Build
npm run build
```### 小包

ゼロ構成バンドラー:```bash
parcel index.html
parcel build index.html
```## タスク ランナー

### npm スクリプト```json
{
  "scripts": {
    "dev": "webpack serve --mode development",
    "build": "webpack --mode production",
    "test": "jest",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```## テストフレームワーク

### 冗談

JavaScript テスト フレームワーク:```javascript
// sum.test.js
const sum = require('./sum');

describe('sum function', () => {
  test('adds 1 + 2 to equal 3', () => {
    expect(sum(1, 2)).toBe(3);
  });
  
  test('handles negative numbers', () => {
    expect(sum(-1, -2)).toBe(-3);
  });
});
```### ヴィテスト

Vite を利用したテスト (Jest 互換):```javascript
import { describe, test, expect } from 'vitest';

describe('math', () => {
  test('addition', () => {
    expect(1 + 1).toBe(2);
  });
});
```### 劇作家

エンドツーエンドのテスト:```javascript
import { test, expect } from '@playwright/test';

test('homepage has title', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
});
```## リンターとフォーマッタ

### ESLint

JavaScript リンター:```javascript
// .eslintrc.js
module.exports = {
  extends: ['eslint:recommended'],
  rules: {
    'no-console': 'warn',
    'no-unused-vars': 'error'
  }
};
```### より美しく

コードフォーマッタ:```json
// .prettierrc
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```### スタイルリント

CSS リンター:```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "indentation": 2,
    "color-hex-length": "short"
  }
}
```## IDE とエディター

### Visual Studio コード

**主な機能**:
- インテリセンス
- デバッグ
- Gitの統合
- 拡張機能マーケットプレイス
- 端末の統合

**人気の拡張機能**:
- ESLint
- より美しく
- ライブサーバー
- GitLens
- パスインテリセンス

### ウェブストーム

JetBrains による Web 開発用のフル機能の IDE。

### 崇高なテキスト

軽量で高速なテキストエディター。

### Vim/Neovim

ターミナルベースのエディター (学習曲線が急です)。

## TypeScript

JavaScript の型付きスーパーセット:```typescript
// types.ts
interface User {
  id: number;
  name: string;
  email?: string; // Optional
}

function getUser(id: number): User {
  return { id, name: 'John' };
}

// Generics
function identity<T>(arg: T): T {
  return arg;
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```## 継続的インテグレーション (CI/CD)

### GitHub アクション```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
```### その他の CI/CD プラットフォーム

- **GitLab 内**
- **CircleCI**
- **トラビスはこちら**
- **ジェンキンス**

## デバッグ

### ブラウザ開発ツール```javascript
// Debugging statements
debugger; // Pause execution
console.log('value:', value);
console.error('error:', error);
console.trace(); // Stack trace
```### Node.js のデバッグ```bash
# Built-in debugger
node inspect app.js

# Chrome DevTools
node --inspect app.js
node --inspect-brk app.js # Break on start
```## パフォーマンスプロファイリング

### Chrome DevTools のパフォーマンス

- CPUアクティビティを記録する
- フレームチャートの分析
- ボトルネックを特定する

### 灯台```bash
# CLI
npm install -g lighthouse
lighthouse https://example.com

# DevTools
Open Chrome DevTools > Lighthouse tab
```## モニタリング

### エラー追跡

- **セントリー**: エラー監視
- **ロールバー**: リアルタイムのエラー追跡
- **バグ**: エラー監視

### 分析

- **Google アナリティクス**
- **もっともらしい**: プライバシーに配慮
- **Piwik**: 自己ホスト型

### RUM (リアルユーザーモニタリング)

- **スピードカーブ**
- **ニューレリック**
- **データドッグ**

## 開発者のワークフロー

### 一般的なワークフロー

1. **セットアップ**: リポジトリのクローンを作成し、依存関係をインストールします
2. **開発**: コードを記述し、開発サーバーを実行します。
3. **テスト**: 単体テスト/統合テストを実行します。
4. **Lint/フォーマット**: コードの品質をチェックする
5. **コミット**: Git のコミットとプッシュ
6. **CI/CD**: 自動化されたテストと展開
7. **デプロイ**: 本番環境へのプッシュ

### 環境変数```bash
# .env
DATABASE_URL=postgres://localhost/db
API_KEY=secret-key-here
NODE_ENV=development
```

```javascript
// Access in Node.js
const dbUrl = process.env.DATABASE_URL;
```## 用語集の用語

**対象となる重要な用語**:
- バン
- 継続的インテグレーション
- デノ
- 開発者ツール
- フォーク
- ファズテスト
- Git
- IDE
- Node.js
- レポ
- Rsync
- SCM
- SDK
- 煙テスト
- SVN
- TypeScript

## 追加のリソース

- [Git ドキュメント](https://git-scm.com/doc)
- [npmドキュメント](https://docs.npmjs.com/)
- [Webpack ガイド](https://webpack.js.org/guides/)
- [Jest ドキュメント](https://jestjs.io/docs/getting-started)
- [TypeScript ハンドブック](https://www.typescriptlang.org/docs/handbook/intro.html)