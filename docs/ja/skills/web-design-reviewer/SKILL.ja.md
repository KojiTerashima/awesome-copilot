---
name: web-design-reviewer
description: 'This skill enables visual inspection of websites running locally or remotely to identify and fix design issues. Triggers on requests like "review website design", "check the UI", "fix the layout", "find design problems". Detects issues with responsive design, accessibility, visual consistency, and layout breakage, then performs fixes at the source code level.'
---
# Web デザインレビューアー

このスキルにより、Web サイトのデザイン品質を視覚的に検査して検証し、ソース コード レベルで問題を特定して修正できるようになります。

## 適用範囲

- 静的サイト (HTML/CSS/JS)
- React / Vue / Angular / Svelte などの SPA フレームワーク
- Next.js / Nuxt / SvelteKit などのフルスタック フレームワーク
- WordPress / Drupal などの CMS プラットフォーム
- その他の Web アプリケーション

## 前提条件

### 必須

1. **対象の Web サイトが実行されている必要があります**
   - ローカル開発サーバー (例: `http://localhost:3000`)
   - ステージング環境
   - 実稼働環境 (読み取り専用レビュー用)

2. **ブラウザ自動化が利用可能である必要があります**
   - スクリーンショットのキャプチャ
   - ページナビゲーション
   - DOM情報の取得

3. **ソース コードへのアクセス (修正時)**
   - プロジェクトはワークスペース内に存在する必要があります

## ワークフローの概要```mermaid
flowchart TD
    A[Step 1: Information Gathering] --> B[Step 2: Visual Inspection]
    B --> C[Step 3: Issue Fixing]
    C --> D[Step 4: Re-verification]
    D --> E{Issues Remaining?}
    E -->|Yes| B
    E -->|No| F[Completion Report]
```---

## ステップ 1: 情報収集フェーズ

### 1.1 URLの確認

URL が提供されていない場合は、ユーザーに次のように尋ねます。

> レビューする Web サイトの URL を入力してください (例: `http://localhost:3000`)

### 1.2 プロジェクト構造の理解

修正を行う場合は、次の情報を収集します。

|アイテム |質問例 |
|------|-----------------|
|フレームワーク | React / Vue / Next.js などを使用していますか? |
|スタイリング方法 | CSS / SCSS / Tailwind / CSS-in-JS など |
|ソースの場所 |スタイル ファイルとコンポーネントはどこにありますか? |
|レビュー範囲 |特定のページのみですか、それともサイト全体ですか? |

### 1.3 プロジェクトの自動検出

ワークスペース内のファイルから自動検出を試みます。```
Detection targets:
├── package.json     → Framework and dependencies
├── tsconfig.json    → TypeScript usage
├── tailwind.config  → Tailwind CSS
├── next.config      → Next.js
├── vite.config      → Vite
├── nuxt.config      → Nuxt
└── src/ or app/     → Source directory
```### 1.4 スタイリング方法の特定

|方法 |検出 |ターゲットの編集 |
|----------|----------|---------------|
|純粋な CSS | `*.css` ファイル |グローバル CSS またはコンポーネント CSS |
| SCSS/サス | `*.scss`、`*.sass` | SCSS ファイル |
| CSS モジュール | `*.module.css` |モジュール CSS ファイル |
|追い風 CSS | `tailwind.config.*` |コンポーネント内の className |
|スタイル付きコンポーネント | `styled.` コード内 | JS/TS ファイル |
|感情 | `@emotion/` インポート | JS/TS ファイル |
| JS 内の CSS (その他) |インラインスタイル | JS/TS ファイル |

---

## ステップ 2: 目視検査フェーズ

### 2.1 ページトラバーサル

1. 指定された URL に移動します
2. スクリーンショットをキャプチャする
3. DOM 構造/スナップショットを取得します (可能な場合)
4. 追加のページが存在する場合は、ナビゲーションをたどります

### 2.2 検査項目

#### レイアウトの問題

|問題 |説明 |重大度 |
|----------|---------------|----------|
|要素のオーバーフロー |コンテンツが親要素またはビューポートからオーバーフローする |高 |
|要素の重なり |要素の意図しない重複 |高 |
|調整の問題 |グリッドまたはフレックスの位置合わせの問題 |中 |
|一貫性のない間隔 |パディング/マージンの不一致 |中 |
|テキストクリッピング |長いテキストが適切に処理されない |中 |

#### 対応に関する問題

|問題 |説明 |重大度 |
|----------|---------------|----------|
|非モバイル対応 |小さな画面でレイアウトが崩れる |高 |
|ブレークポイントの問題 |画面サイズ変更時の画面遷移が不自然 |中 |
|タッチターゲット |モバイルではボタンが小さすぎる |中 |

#### アクセシビリティの問題

|問題 |説明 |重大度 |
|----------|---------------|----------|
|コントラストが不十分です |テキストと背景のコントラスト比が低い |高 |
|フォーカス状態なし |キーボード ナビゲーション中に状態を判断できません。高 |
|代替テキストがありません |画像の代替テキストはありません |中 |

#### 視覚的な一貫性

|問題 |説明 |重大度 |
|----------|---------------|----------|
|フォントの不一致 |混合フォントファミリー |中 |
|色の不一致 |ブランドカラーが統一されていない |中 |
|間隔の不一致 |類似した要素間の不均一な間隔 |低い |

### 2.3 ビューポートのテスト (レスポンシブ)

次のビューポートでテストします。

|名前 |幅 |代表的なデバイス |
|------|-------|-----------|
|モバイル | 375ピクセル | iPhone SE/12mini｜
|タブレット | 768ピクセル | iPad |
|デスクトップ | 1280ピクセル |標準PC |
|ワイド | 1920ピクセル |大型ディスプレイ |

---

## ステップ 3: 問題修正フェーズ### 3.1 問題の優先順位付け```mermaid
block-beta
    columns 1
    block:priority["Priority Matrix"]
        P1["P1: Fix Immediately\n(Layout issues affecting functionality)"]
        P2["P2: Fix Next\n(Visual issues degrading UX)"]
        P3["P3: Fix If Possible\n(Minor visual inconsistencies)"]
    end
```### 3.2 ソースファイルの特定

問題のある要素からソース ファイルを特定します。

1. **セレクターベースの検索**
   - クラス名またはIDでコードベースを検索
   - `grep_search` を使用してスタイル定義を調べる

2. **コンポーネントベースの検索**
   - 要素のテキストまたは構造からコンポーネントを識別する
   - `semantic_search` を使用して関連ファイルを探索します

3. **ファイル パターン フィルタリング**```
   Style files: src/**/*.css, styles/**/*
   Components: src/components/**/*
   Pages: src/pages/**, app/**
   ```### 3.3 修正の適用

#### フレームワーク固有の修正ガイドライン

詳細については、[references/framework-fixes.md](references/framework-fixes.md) を参照してください。

#### 原則を修正する

1. **最小限の変更**: 問題を解決するために必要な最小限の変更のみを加えます。
2. **既存のパターンを尊重**: プロジェクト内の既存のコード スタイルに従います。
3. **重大な変更を避ける**: 他の領域に影響を与えないように注意してください
4. **コメントの追加**: 必要に応じて、修正の理由を説明するコメントを追加します。

---

## ステップ 4: 再検証フェーズ

### 4.1 修正後の確認

1. ブラウザをリロードします (または開発サーバー HMR を待ちます)。
2. 固定領域のスクリーンショットをキャプチャする
3. 前後を比較する

### 4.2 回帰テスト

- 修正が他の領域に影響を与えていないことを確認する
- 応答性の高いディスプレイが壊れていないことを確認します

### 4.3 反復の決定```mermaid
flowchart TD
    A{Issues Remaining?}
    A -->|Yes| B[Return to Step 2]
    A -->|No| C[Proceed to Completion Report]
```**反復制限**: 特定の問題に対して 3 回を超える修正試行が必要な場合は、ユーザーに相談してください。

---

## 出力フォーマット

### レビュー結果レポート```markdown
# Web Design Review Results

## Summary

| Item | Value |
|------|-------|
| Target URL | {URL} |
| Framework | {Detected framework} |
| Styling | {CSS / Tailwind / etc.} |
| Tested Viewports | Desktop, Mobile |
| Issues Detected | {N} |
| Issues Fixed | {M} |

## Detected Issues

### [P1] {Issue Title}

- **Page**: {Page path}
- **Element**: {Selector or description}
- **Issue**: {Detailed description of the issue}
- **Fixed File**: `{File path}`
- **Fix Details**: {Description of changes}
- **Screenshot**: Before/After

### [P2] {Issue Title}
...

## Unfixed Issues (if any)

### {Issue Title}
- **Reason**: {Why it was not fixed/could not be fixed}
- **Recommended Action**: {Recommendations for user}

## Recommendations

- {Suggestions for future improvements}
```---

## 必要な機能

|能力 |説明 |必須 |
|-----------|---------------|----------|
| Web ページのナビゲーション |アクセスURL、ページ遷移 | ✅ |
|スクリーンショットのキャプチャ |ページ画像のキャプチャ | ✅ |
|画像解析 |視覚的な問題検出 | ✅ |
| DOM の取得 |ページ構造の取得 |おすすめ |
|ファイルの読み取り/書き込み |ソースコードの読み取りと編集 |修正に必要 |
|コード検索 |プロジェクト内のコード検索 |修正に必要 |

---

## リファレンス実装

### Playwright MCP による実装

このスキルのリファレンス実装としては、[Playwright MCP](https://github.com/microsoft/playwright-mcp) を推奨します。

|能力 |劇作家 MCP ツール |目的 |
|-----------|---------------------|----------|
|ナビゲーション | `browser_navigate` |アクセスURL |
|スナップショット | `browser_snapshot` | DOM 構造の取得 |
|スクリーンショット | `browser_take_screenshot` |目視検査用画像 |
| |をクリックします`browser_click` |インタラクティブな要素を操作する |
|サイズ変更 | `browser_resize` |レスポンシブテスト |
|コンソール | `browser_console_messages` | JS エラーを検出する |

#### 構成例 (MCP サーバー)```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--caps=vision"]
    }
  }
}
```### その他の互換性のあるブラウザ自動化ツール

|ツール |特長 |
|------|----------|
|セレン |幅広いブラウザのサポート、多言語のサポート |
|人形遣い | Chrome/Chromium 中心、Node.js |
|サイプレス | E2E テストとの簡単な統合 |
| WebDriver BiDi |標準化された次世代プロトコル |

これらのツールを使用して同じワークフローを実装できます。必要な機能 (ナビゲーション、スクリーンショット、DOM 取得) を提供する限り、ツールの選択は柔軟です。

---

## ベストプラクティス

### 実行する (推奨)

- ✅ 修正を行う前に必ずスクリーンショットを保存してください
- ✅ 一度に 1 つずつ問題を修正し、それぞれを確認します
- ✅ プロジェクトの既存のコード スタイルに従います。
- ✅ 大きな変更を行う前にユーザーに確認する
- ✅ 修正の詳細を徹底的に文書化する

### しないでください (推奨されません)

- ❌ 確認を伴わない大規模なリファクタリング
- ❌ デザインシステムやブランドガイドラインを無視する
- ❌ パフォーマンスを無視する修正
- ❌ 複数の問題を一度に修正する (検証が困難)

---

## トラブルシューティング

### 問題: スタイル ファイルが見つかりません

1. `package.json` で依存関係を確認します。
2. CSS-in-JS の可能性を検討する
3. ビルド時に生成される CSS を考慮する
4. ユーザーにスタイリング方法を尋ねる

### 問題: 修正が反映されない

1. 開発サーバー HMR が動作しているかどうかを確認します
2. ブラウザのキャッシュをクリアする
3. プロジェクトにビルドが必要な場合は再ビルドする
4. CSS の特異性の問題を確認する

### 問題: 他の領域に影響を与える修正

1. 変更のロールバック
2. より具体的なセレクターを使用する
3. CSS モジュールまたは範囲指定されたスタイルの使用を検討する
4. ユーザーに相談して影響範囲を確認する