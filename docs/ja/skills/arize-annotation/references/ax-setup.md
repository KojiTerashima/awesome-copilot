# ax CLI — トラブルシューティング

`ax` コマンドが失敗した場合にのみ、これを参照してください。これらの確認を事前に実行してはいけません。

## まずバージョンを確認

`ax` がインストールされている場合（`command not found` ではない場合）、必ず詳細調査の前に `ax --version` を実行してください。バージョンは `0.8.0` 以上である必要があります。多くのエラーは古いインストールが原因です。バージョンが古すぎる場合は、以下の **バージョンが古すぎる** を参照してください。

## `ax: command not found`

**macOS/Linux:**
1. よくある場所を確認: `~/.local/bin/ax`, `~/Library/Python/*/bin/ax`
2. インストール: `uv tool install arize-ax-cli`（推奨）、`pipx install arize-ax-cli`、または `pip install arize-ax-cli`
3. 必要に応じて PATH に追加: `export PATH="$HOME/.local/bin:$PATH"`

**Windows (PowerShell):**
1. 確認: `Get-Command ax` または `where.exe ax`
2. よくある場所: `%APPDATA%\Python\Scripts\ax.exe`, `%LOCALAPPDATA%\Programs\Python\Python*\Scripts\ax.exe`
3. インストール: `pip install arize-ax-cli`
4. PATH に追加: `$env:PATH = "$env:APPDATA\Python\Scripts;$env:PATH"`

## バージョンが古すぎる（0.8.0 未満）

アップグレード: `uv tool install --force --reinstall arize-ax-cli`、`pipx upgrade arize-ax-cli`、または `pip install --upgrade arize-ax-cli`

## SSL/証明書エラー

- macOS: `export SSL_CERT_FILE=/etc/ssl/cert.pem`
- Linux: `export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`
- フォールバック: `export SSL_CERT_FILE=$(python -c "import certifi; print(certifi.where())")`

## サブコマンドが認識されない

ax をアップグレード（上記参照）するか、利用可能な最も近い代替手段を使用してください。

## それでも失敗する場合

作業を止めて、ユーザーに助けを求めてください。

