---
name: publish-to-pages
description: 'Publish presentations and web content to GitHub Pages. Converts PPTX, PDF, HTML, or Google Slides to a live GitHub Pages URL. Handles repo creation, file conversion, Pages enablement, and returns the live URL. Use when the user wants to publish, deploy, or share a presentation or HTML file via GitHub Pages.'
---
# ページに公開する

プレゼンテーションや Web コンテンツを一度に GitHub Pages に公開します。

## 1. 前提条件の確認

これらをサイレントに実行します。表面的なエラーのみ:「」バッシュ
コマンド -v gh >/dev/null || echo "欠落: gh CLI — https://cli.github.com からインストール"
gh 認証ステータス &>/dev/null || echo "欠落: gh が認証されていません — 'gh auth login' を実行してください"
コマンド -v python3 >/dev/null || echo "欠落: python3 (PPTX 変換に必要)"
「」`poppler-utils` はオプションです (`pdftoppm` による PDF 変換)。それをブロックしないでください。

## 2. 入力検出

ユーザーが提供したものから入力タイプを決定します。

|入力 |検出 |
|------|-----------|
| HTML ファイル |拡張子 `.html` または `.htm` |
| PPTX ファイル |拡張子 `.pptx` |
| PDFファイル |拡張子 `.pdf` |
| Google スライドの URL | URL には `docs.google.com/presentation` が含まれています |

**リポ名**が指定されていない場合は、ユーザーに尋ねます。デフォルト: 拡張子なしのファイル名。

## 3. 変換

### 大きなファイルの処理

どちらの変換スクリプトも、大きなファイルを自動的に検出し、**外部アセット モード**に切り替えます。
- **PPTX:** ファイルが 20MB を超えるか、画像が 50 枚を超える → 画像は `assets/` に別のファイルとして保存されます
- **PDF:** ファイル >20MB または >50 ページ → ページ PNG は `assets/` に保存されます
- ファイルが 150MB を超えると警告が表示されます (PPTX では代わりに PDF パスが提案されます)

これにより、個々のファイルは GitHub の 100MB 制限を十分に下回ります。小さなファイルでも、単一の自己完結型 HTML が生成されます。

`--external-assets` または `--no-external-assets` を使用して動作を強制できます。

### HTML
変換は必要ありません。ファイルを `index.html` として直接使用します。

### PPTX
変換スクリプトを実行します。「」バッシュ
python3 SKILL_DIR/scripts/convert-pptx.py INPUT_FILE /tmp/output.html
# 大きなファイルの場合は、外部アセットを強制します。
python3 SKILL_DIR/scripts/convert-pptx.py INPUT_FILE /tmp/output.html --external-assets
「」`python-pptx` が欠落している場合は、ユーザーに次のように伝えます: `pip install python-pptx`

### PDF
付属のスクリプトを使用して変換します (`pdftoppm` の場合は `poppler-utils` が必要です):「」バッシュ
python3 SKILL_DIR/scripts/convert-pdf.py INPUT_FILE /tmp/output.html
# 大きなファイルの場合は、外部アセットを強制します。
python3 SKILL_DIR/scripts/convert-pdf.py INPUT_FILE /tmp/output.html --external-assets
「」各ページは PNG としてレンダリングされ、スライド ナビゲーションを備えた HTML に埋め込まれます。
`pdftoppm` がない場合は、ユーザーに次のように伝えます: `apt install poppler-utils` (macOS では `brew install poppler`)。

### Google スライド
1. URL からプレゼンテーション ID を抽出します (`/d/` と `/` の間の長い文字列)
2. PPTX としてダウンロードします。「」バッシュ
カール -L "https://docs.google.com/presentation/d/PRESENTATION_ID/export/pptx" -o /tmp/slides.pptx
「」3. 次に、上記の変換スクリプトを使用して PPTX を変換します。

## 4. 出版

### 可視性
リポジトリはデフォルトで**パブリック**に作成されます。ユーザーが `private` を指定する場合 (またはプライベート リポジトリが必要な場合)、`--private` を使用します。ただし、プライベート リポジトリの GitHub Pages には Pro、Team、または Enterprise プランが必要であることに注意してください。

＃＃＃ 公開「」バッシュ
bash SKILL_DIR/scripts/publish.sh /path/to/index.html REPO_NAME public "説明"
「」ユーザーが要求した場合は、`public` の代わりに `private` を渡します。

スクリプトはリポジトリを作成し、`index.html` (存在する場合はさらに `assets/`) をプッシュし、GitHub Pages を有効にします。

**注意:** 外部アセット モードを使用すると、出力 HTML は `assets/` 内のファイルを参照します。パブリッシュ スクリプトは、HTML ファイルとともに `assets/` ディレクトリを自動的に検出してコピーします。 HTML ファイルとその `assets/` ディレクトリが同じ親ディレクトリにあることを確認してください。

## 5. 出力

ユーザーに次のように伝えます。
- **リポジトリ:** `https://github.com/USERNAME/REPO_NAME`
- **ライブ URL:** `https://USERNAME.github.io/REPO_NAME/`
- **注意:** ページが公開されるまでに 1 ～ 2 分かかります。

## エラー処理

- **リポジトリは既に存在します:** 数値 (`my-slides-2`) または日付 (`my-slides-2026`) を追加することを提案します。
- **ページの有効化に失敗します:** それでもリポジトリ URL が返されます。ユーザーはリポジトリ設定でページを手動で有効にすることができます。
- **PPTX 変換が失敗します:** `pip install python-pptx` を実行するようにユーザーに指示します。
- **PDF 変換が失敗する:** `poppler-utils` (`apt install poppler-utils` または `brew install poppler`) をインストールすることをお勧めします。
- **Google スライドのダウンロードが失敗する:** プレゼンテーションは一般公開されていない可能性があります。 PPTX を表示可能にするか、PPTX を手動でダウンロードするようにユーザーに依頼します。