# PDFtk CLI の例

PDFtk はコマンドライン プログラムです。これらの例を実行するときは、コンピューターのターミナルまたはコマンド プロンプトを使用してください。

## スキャンしたページを部単位で照合する

スキャンした偶数ページと奇数ページを 1 つのドキュメントにインターリーブします。```bash
pdftk A=even.pdf B=odd.pdf shuffle A B output collated.pdf
```奇数ページが逆順の場合:```bash
pdftk A=even.pdf B=odd.pdf shuffle A Bend-1 output collated.pdf
```## PDF を復号化する

パスワードを使用して PDF から暗号化を削除します。```bash
pdftk secured.pdf input_pw foopass output unsecured.pdf
```## 128 ビット強度を使用して PDF を暗号化する

所有者パスワード暗号化を適用します。```bash
pdftk 1.pdf output 1.128.pdf owner_pw foopass
```PDF を開くときにもパスワードを要求します。```bash
pdftk 1.pdf output 1.128.pdf owner_pw foo user_pw baz
```印刷を許可しながら暗号化します。```bash
pdftk 1.pdf output 1.128.pdf owner_pw foo user_pw baz allow printing
```## PDF に参加する

複数の PDF を 1 つに結合します。```bash
pdftk in1.pdf in2.pdf cat output out1.pdf
```明示的な制御のためのハンドルの使用:```bash
pdftk A=in1.pdf B=in2.pdf cat A B output out1.pdf
```ワイルドカードを使用してディレクトリ内のすべての PDF をマージします。```bash
pdftk *.pdf cat output combined.pdf
```## 特定のページを削除する

ドキュメントから 13 ページを除外します。```bash
pdftk in.pdf cat 1-12 14-end output out1.pdf
```ハンドルの使用:```bash
pdftk A=in1.pdf cat A1-12 A14-end output out1.pdf
```## 40 ビット暗号化を適用する

40 ビット強度でマージおよび暗号化します。```bash
pdftk 1.pdf 2.pdf cat output 3.pdf encrypt_40bit owner_pw foopass
```## パスワードで保護されているファイルに参加する

暗号化された入力のパスワードを指定します。```bash
pdftk A=secured.pdf 2.pdf input_pw A=foopass cat output 3.pdf
```## PDF ページ ストリームを解凍する

検査またはデバッグのために内部ストリームを解凍します。```bash
pdftk doc.pdf output doc.unc.pdf uncompress
```## 破損した PDF を修復する

壊れた PDF を pdftk に渡して修復を試みます。```bash
pdftk broken.pdf output fixed.pdf
```## PDF を個別のページにバーストする

各ページを独自のファイルに分割します。```bash
pdftk in.pdf burst
```暗号化と制限付き印刷によるバースト:```bash
pdftk in.pdf burst owner_pw foopass allow DegradedPrinting
```## PDF メタデータ レポートを生成する

ブックマーク、メタデータ、ページメトリクスをエクスポートします。```bash
pdftk in.pdf dump_data output report.txt
```## ページを回転する

最初のページを時計回りに 90 度回転します。```bash
pdftk in.pdf cat 1east 2-end output out.pdf
```すべてのページを 180 度回転します。```bash
pdftk in.pdf cat 1-endsouth output out.pdf
```## データから PDF フォームに記入する

FDF ファイルからフォームフィールドにデータを入力します。```bash
pdftk form.pdf fill_form data.fdf output filled_form.pdf
```入力後にフォームを平坦化します (さらなる編集を防ぎます):```bash
pdftk form.pdf fill_form data.fdf output filled_form.pdf flatten
```## 背景の透かしを適用する

すべてのページの後ろに透かしをスタンプします。```bash
pdftk input.pdf background watermark.pdf output watermarked.pdf
```## オーバーレイを上部にスタンプします

すべてのページの上にオーバーレイ PDF を適用します。```bash
pdftk input.pdf stamp overlay.pdf output stamped.pdf
```## PDF にファイルを添付する

ファイルを添付ファイルとして埋め込む:```bash
pdftk input.pdf attach_files table.html graph.png output output.pdf
```## PDF から添付ファイルを抽出する

すべての埋め込みファイルを解凍します。```bash
pdftk input.pdf unpack_files output /path/to/output/
```
