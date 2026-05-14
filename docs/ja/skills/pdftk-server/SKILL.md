---
name: pdftk-server
description: 'Skill for using the command-line tool pdftk (PDFtk Server) for working with PDF files. Use when asked to merge PDFs, split PDFs, rotate pages, encrypt or decrypt PDFs, fill PDF forms, apply watermarks, stamp overlays, extract metadata, burst documents into pages, repair corrupted PDFs, attach or extract files, or perform any PDF manipulation from the command line.'
---
# PDFtkサーバー

PDFtk Server は、PDF ドキュメントを操作するためのコマンドライン ツールです。さまざまな方法で PDF の結合、分割、回転、暗号化、復号化、透かし、スタンプ、フォームへの入力、メタデータの抽出、操作を行うことができます。

## このスキルを使用する場合

- 複数の PDF ファイルを 1 つに結合または結合する
- PDF を個々のページに分割またはバーストする
- PDF ページの回転
- PDF ファイルの暗号化または復号化
- FDF/XFDF データから PDF フォームフィールドに入力
- 背景の透かしまたは前景スタンプの適用
- PDF メタデータ、ブックマーク、またはフォーム フィールド情報の抽出
- 破損したPDFファイルを修復する
- PDFに埋め込まれたファイルの添付または抽出
- PDF から特定のページを削除する
- 偶数/奇数ページを別々にスキャンして丁合する
- PDF ページ ストリームの圧縮または解凍

## 前提条件

- PDFtkサーバーがシステムにインストールされている必要があります
  - **Windows**: `winget install --id PDFLabs.PDFtk.Server`
  - **macOS**: `brew install pdftk-java`
  - **Linux (Debian/Ubuntu)**: `sudo apt-get install pdftk`
  - **Linux (Red Hat/Fedora)**: `sudo dnf install pdftk`
- ターミナルまたはコマンド プロンプトへのアクセス
- `pdftk --version` を実行してインストールを確認します。

## 段階的なワークフロー

### 複数の PDF を結合する```bash
pdftk file1.pdf file2.pdf cat output merged.pdf
```ハンドルを使用してさらに制御するには:```bash
pdftk A=file1.pdf B=file2.pdf cat A B output merged.pdf
```### PDF を個別のページに分割する```bash
pdftk input.pdf burst
```### 特定のページを抽出する

1 ～ 5 ページと 10 ～ 15 ページを抜粋します。```bash
pdftk input.pdf cat 1-5 10-15 output extracted.pdf
```### 特定のページを削除する

13 ページを削除します。```bash
pdftk input.pdf cat 1-12 14-end output output.pdf
```### ページを回転する

すべてのページを時計回りに 90 度回転します。```bash
pdftk input.pdf cat 1-endeast output rotated.pdf
```### PDF を暗号化する

所有者パスワードとユーザー パスワードを 128 ビット暗号化 (デフォルト) で設定します。```bash
pdftk input.pdf output secured.pdf owner_pw mypassword user_pw userpass
```### PDF を復号化する

既知のパスワードを使用して暗号化を削除します。```bash
pdftk secured.pdf input_pw mypassword output unsecured.pdf
```### PDF フォームに記入します

FDF ファイルからフォーム フィールドにデータを入力し、それ以上の編集を防ぐためにフラット化します。```bash
pdftk form.pdf fill_form data.fdf output filled.pdf flatten
```### 背景透かしを適用する

単一ページの PDF を入力の各ページの後ろに配置します (入力は透明である必要があります)。```bash
pdftk input.pdf background watermark.pdf output watermarked.pdf
```### オーバーレイにスタンプを付ける

単一ページの PDF を入力の各ページの上に配置します。```bash
pdftk input.pdf stamp overlay.pdf output stamped.pdf
```### メタデータの抽出

ブックマーク、ページメトリクス、ドキュメント情報をエクスポートします。```bash
pdftk input.pdf dump_data output metadata.txt
```### 破損した PDF を修復する

壊れた PDF を pdftk に渡して自動修復を試みます。```bash
pdftk broken.pdf output fixed.pdf
```### スキャンしたページを照合する

偶数ページと奇数ページを別々にスキャンしてインターリーブします。```bash
pdftk A=even.pdf B=odd.pdf shuffle A B output collated.pdf
```## トラブルシューティング

|問題 |ソリューション |
|------|----------|
| `pdftk` コマンドが見つかりません |インストールを確認します。 pdftk がシステムの PATH | にあることを確認してください。
| PDF を復号化できません | `input_pw` 経由で正しい所有者またはユーザーのパスワードを指定していることを確認してください。
|出力ファイルが空か破損しています。入力ファイルの整合性をチェックします。最初に `pdftk input.pdf output repaired.pdf` を実行してみてください。
|入力後にフォームフィールドが表示されない | `flatten` フラグを使用してフィールドをページ コンテンツにマージします。
|ウォーターマークが表示されない |入力 PDF に透明な領域があることを確認してください。不透明なオーバーレイには `stamp` を使用します。
|アクセス許可拒否エラー |入力パスと出力パスのファイル権限を確認する |

## 参考文献

`references/` フォルダーにバンドルされている参照ドキュメント:

- [pdftk-man-page.md](references/pdftk-man-page.md) - すべての操作、オプション、および構文を含む完全なマニュアル リファレンス
- [pdftk-cli-examples.md](references/pdftk-cli-examples.md) - 一般的なタスクのための実践的なコマンドラインの例
- [download.md](references/download.md) - すべてのプラットフォームのインストールとダウンロードの手順
- [pdftk-server-license.md](references/pdftk-server-license.md) - PDFtk サーバーのライセンス情報
- [ third-party-materials.md ](references/ third-party-materials.md) - サードパーティ ライブラリのライセンス