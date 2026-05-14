# ダウンロード

PDFtk は Windows 用のインストーラーを提供します。多くの Linux ディストリビューションでは、パッケージ マネージャーを使用してダウンロードしてインストールできる PDFtk パッケージが提供されています。

## Microsoft Windows

次のコマンドを使用して、Windows 10 および 11 用の PDFtk Server インストーラーをダウンロードします。```bash
winget install --id PDFLabs.PDFtk.Server
```次に、インストーラーを実行します。```bash
.\pdftk_server-2.02-win-setup.exe
```インストール後、コマンド プロンプトを開き、「`pdftk`」と入力して Enter キーを押します。 PDFtk は、簡単な使用法情報を表示して応答します。

## Linux

Debian/Ubuntu ベースのディストリビューションの場合:```bash
sudo apt-get install pdftk
```Red Hat/Fedora ベースのディストリビューションの場合:```bash
sudo dnf install pdftk
```## PDFtk サーバー GPL ライセンス

PDFtk サーバー (pdftk) はパブリック ドメイン ソフトウェアではありません。 [GNU General Public License (GPL) バージョン 2](https://www.pdflabs.com/docs/pdftk-license/gnu_general_public_license_2.txt) に基づいて無料でインストールして使用できます。 PDFtk はサードパーティのライブラリを使用します。 [これらのライブラリのライセンスとソース コードは、ここで説明されています](https://www.pdflabs.com/docs/pdftk-license/) のサードパーティ マテリアルの下にあります。

## PDFtk サーバー再配布ライセンス

PDFtk Server を独自のソフトウェアの一部として配布する予定がある場合は、PDFtk Server 再配布ライセンスが必要になります。この規則の例外は、ソフトウェアが GPL または別の互換ライセンスに基づいて一般にライセンスされている場合です。

商用再配布ライセンスを使用すると、ライセンスの条項に従って、無制限の数の PDFtk Server バイナリを 1 つの異なる商用製品の一部として配布できます。ライセンス全文をお読みください:

[PDFtk サーバー再配布ライセンス (PDF)](https://pdflabs.onfastspring.com/pdftk-server)

現在 $995 で入手可能:

[PDFtkサーバー再配布ライセンス](https://www.pdflabs.com/docs/pdftk-license/)

## ソースから PDFtk サーバーを構築する

PDFtk Server はソース コードからコンパイルできます。 PDFtk サーバーは、[Debian](https://packages.debian.org/search?keywords=pdftk)、[Ubuntu Linux](https://packages.ubuntu.com/search?keywords=pdftk)、[FreeBSD](https://www.freshports.org/print/pdftk/)、Slackware Linux、SuSE、Solaris 上でコンパイルおよび実行できることが知られています。 [HP-UX](http://hpux.connect.org.uk/hppd/hpux/Text/pdftk-1.45/)。

ソースをダウンロードして解凍します。```bash
curl -LO https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/pdftk-2.02-src.zip
unzip pdftk-2.02-src.zip
````license_gpl_pdftk/readme.txt` で [pdftk ライセンス情報](https://www.pdflabs.com/docs/pdftk-license/) を確認します。

プラットフォームに提供されている Makefile を確認し、`TOOLPATH` および `VERSUFF` が gcc/gcj/libgcj のインストールに適合していることを確認します。 `apropos gcc` を実行して `gcc-4.5` のような結果が返された場合は、`VERSUFF` を `-4.5` に設定します。 `TOOLPATH` はおそらく設定する必要はありません。

`pdftk` サブディレクトリに移動し、次を実行します。```bash
cd pdftk
make -f Makefile.Debian
```必要に応じて、プラットフォームの Makefile ファイル名を置き換えます。

PDFtk は、gcc/gcj/libgcj バージョン 3.4.5、4.4.1、4.5.0、および 4.6.3 を使用して構築されています。 PDFtk 1.4x は、libgcj 機能がないため、gcc 3.3.5 でのビルドに失敗します。 gcc 3.3 以前を使用している場合は、代わりに [pdftk 1.12](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/pdftk-1.12.tar.gz) をビルドしてみてください。