# PDFtkサーバーマニュアルリファレンス

- **`pdftk` バージョン 2.02**
- [バージョン履歴](https://www.pdflabs.com/docs/pdftk-version-history/) で変更を確認してください。
- 最新のドキュメントについては、[サーバーマニュアル](https://www.pdflabs.com/docs/pdftk-man-page/) を参照してください。

## 概要

PDFtk は、PDF ドキュメントを操作するためのコマンドライン ユーティリティです。 PDF ファイルの結合、分割、回転、暗号化、復号化、透かし入れ、フォーム入力、メタデータ抽出などの操作が可能になります。

## 概要```
pdftk [input PDF files | - | PROMPT]
      [input_pw <passwords>]
      [<operation>] [<operation arguments>]
      [output <filename | - | PROMPT>]
      [encrypt_40bit | encrypt_128bit]
      [allow <permissions>]
      [owner_pw <password>] [user_pw <password>]
      [compress | uncompress]
      [flatten] [need_appearances]
      [verbose] [dont_ask | do_ask]
```## 入力オプション

**入力 PDF ファイル**: 1 つ以上の PDF を指定します。標準入力には `-` を、対話型入力には `PROMPT` を使用します。ファイルには、操作時の参照用にハンドル (単一の大文字) を割り当てることができます。```
pdftk A=file1.pdf B=file2.pdf cat A B output merged.pdf
```**パスワードの入力** (`input_pw`): 暗号化された PDF の場合、ファイル ハンドルに関連付けられた所有者パスワードまたはユーザー パスワードを、または入力順序別に指定します。```
pdftk A=secured.pdf input_pw A=foopass cat output unsecured.pdf
```## コアオペレーション

### cat - 連結して作成する

オプションで回転を使用して、ページを結合、分割、または並べ替えます。ページ範囲、逆順 (プレフィックス `r`)、ページ修飾子 (`even`/`odd`)、および回転 (コンパスの方向 `north`、`south`、`east`、`west`、`left`、`right`、`down`) をサポートします。

ページ範囲の構文: `[handle][begin[-end[qualifier]]][rotation]````
pdftk A=in1.pdf B=in2.pdf cat A1-7 B1-5 A8 output combined.pdf
```### シャッフル - ページを照合する

各入力範囲から順番に 1 ページを取得し、インターリーブされた結果を生成します。別々にスキャンした奇数ページと偶数ページを照合する場合に便利です。```
pdftk A=even.pdf B=odd.pdf shuffle A B output collated.pdf
```### バースト - 個別のページに分割

1 つの PDF をページごとに 1 つのファイルに分割します。出力ファイルには、`printf` 形式の形式を使用して名前が付けられます (デフォルト: `pg_%04d.pdf`)。```
pdftk input.pdf burst output page_%02d.pdf
```### 回転 - ページを回転します

ドキュメントの順序を維持しながら、指定したページを回転します。 `cat` と同​​じページ範囲構文を使用します。```
pdftk in.pdf cat 1-endeast output rotated.pdf
```###generate_fdf - フォームデータの抽出

PDF フォームから FDF ファイルを作成し、現在のフィールド値を取得します。```
pdftk form.pdf generate_fdf output form_data.fdf
```### fill_form - フォームフィールドに値を入力します

FDF または XFDF データ ファイルから PDF フォーム フィールドに入力します。```
pdftk form.pdf fill_form data.fdf output filled.pdf flatten
```### 背景 - コンテンツの背後にウォーターマークを適用します

単一ページの PDF を入力の各ページの背景 (透かし) として適用します。最良の結果を得るには、入力 PDF の背景が透明である必要があります。```
pdftk input.pdf background watermark.pdf output watermarked.pdf
```### multibackground - 複数ページの透かしを適用する

`background` と似ていますが、背景 PDF の対応するページを入力内の一致するページに適用します。```
pdftk input.pdf multibackground watermarks.pdf output watermarked.pdf
```### スタンプ - コンテンツの上部にオーバーレイ

単一ページの PDF を入力の各ページの上にスタンプします。オーバーレイ PDF が不透明または透明度がない場合は、`background` の代わりにこれを使用します。```
pdftk input.pdf stamp overlay.pdf output stamped.pdf
```### マルチスタンプ - 複数ページのオーバーレイ

`stamp` と似ていますが、スタンプ PDF の対応するページを入力内の一致するページに適用します。```
pdftk input.pdf multistamp overlays.pdf output stamped.pdf
```### dump_data - メタデータのエクスポート

PDF メタデータ、ブックマーク、ページ メトリックをテキスト ファイルに出力します。```
pdftk input.pdf dump_data output metadata.txt
```### dump_data_utf8 - メタデータのエクスポート (UTF-8)

`dump_data` と同じですが、UTF-8 でエンコードされたテキストを出力します。```
pdftk input.pdf dump_data_utf8 output metadata_utf8.txt
```### dump_data_fields - フォームフィールド情報の抽出

タイプ、名前、値などのフォームフィールド情報をレポートします。```
pdftk form.pdf dump_data_fields output fields.txt
```### dump_data_fields_utf8 - フォームフィールド情報の抽出 (UTF-8)

`dump_data_fields` と同じですが、UTF-8 でエンコードされたテキストを出力します。

### dump_data_annots - 注釈の抽出

PDF の注釈情報をレポートします。```
pdftk input.pdf dump_data_annots output annots.txt
```### update_info - メタデータの変更

PDF メタデータとブックマークをテキスト ファイル (`dump_data` 出力と同じ形式) から更新します。```
pdftk input.pdf update_info metadata.txt output updated.pdf
```### update_info_utf8 - メタデータの変更 (UTF-8)

`update_info` と同じですが、UTF-8 でエンコードされた入力が必要です。

###attach_files - ファイルの埋め込み

オプションで特定のページにファイルを PDF に添付します。```
pdftk input.pdf attach_files table.html graph.png to_page 6 output output.pdf
```### unpack_files - 添付ファイルの抽出

PDF から添付ファイルを抽出します。```
pdftk input.pdf unpack_files output /path/to/output/
```## 出力オプション

|オプション |説明 |
|----------|---------------|
| `output <filename>` |出力ファイルを指定します。標準出力には `-` を、対話型には `PROMPT` を使用します。 |
| `encrypt_40bit` | 40 ビット RC4 暗号化を適用する |
| `encrypt_128bit` | 128 ビット RC4 暗号化を適用します (パスワード設定時のデフォルト)。
| `owner_pw <password>` |所有者パスワードを設定する |
| `user_pw <password>` |ユーザーパスワードを設定する |
| `allow <permissions>` |特定の権限を付与します (以下を参照)。
| `compress` |ページストリームを圧縮する |
| `uncompress` |ページ ストリームを解凍する (デバッグに便利) |
| `flatten` |フォームフィールドをページコンテンツにフラット化する |
| `need_appearances` |フィールドの外観を再生成する信号ビューア |
| `keep_first_id` |最初の入力からドキュメント ID を保持する |
| `keep_final_id` |最後に入力したドキュメント ID を保持する |
| `drop_xfa` | XFA フォーム データを削除する |
| `verbose` |詳細な操作出力を有効にする |
| `dont_ask` |対話型プロンプトを抑制する |
| `do_ask` |対話型プロンプトを有効にする |

## 権限

暗号化する場合は `allow` キーワードとともに使用します。利用可能な権限:

|許可 |説明 |
|-----------|---------------|
| `Printing` |高品質の印刷を許可する |
| `DegradedPrinting` |低品質の印刷を許可する |
| `ModifyContents` |コンテンツの変更を許可する |
| `Assembly` |ドキュメントのアセンブリを許可する |
| `CopyContents` |コンテンツのコピーを許可する |
| `ScreenReaders` |スクリーン リーダーへのアクセスを許可する |
| `ModifyAnnotations` |注釈の変更を許可する |
| `FillIn` |フォームへの入力を許可する |
| `AllFeatures` |すべての権限を付与する |

## 重要なメモ

- ページ番号は 1 から始まります。最終ページには `end` キーワードを使用してください
- 単一の PDF を操作する場合、ハンドルはオプションです。
- フィルター モード (操作が指定されていない) は、再構築せずに出力オプションを適用します。
- 逆ページ参照では、`r` プレフィックスを使用します (例: `r1` = 最後のページ、`r2` = 最後から 2 番目)
- `background` 操作には透過的な入力が必要です。不透明なオーバーレイ PDF には `stamp` を使用してください
- 出力ファイル名は入力ファイル名と一致できません