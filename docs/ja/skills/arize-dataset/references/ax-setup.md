# ax CLI — トラブルシューティング

`ax` コマンドが失敗した場合にのみ、これを参照してください。これらのチェックを事前に実行してはいけません。

## まずバージョンを確認する

`ax` がインストールされている場合（`command not found` ではない場合）、さらに調査する前に必ず `ax --version` を実行してください。バージョンは `0.8.0` 以上である必要があります。多くのエラーは古いインストールが原因です。バージョンが古すぎる場合は、以下の **Version too old** を参照してください。

## `ax: command not found`

**macOS/Linux:**
1. 一般的な場所を確認: `~/.local/bin/ax`, `~/Library/Python/*/bin/ax`
2. インストール: `uv tool install arize-ax-cli`（推奨）、`pipx install arize-ax-cli`、または `pip install arize-ax-cli`
3. 必要に応じて PATH に追加: `export PATH="$HOME/.local/bin:$PATH"`

**Windows (PowerShell):**
1. 確認: `Get-Command ax` または `where.exe ax`
2. 一般的な場所: `%APPDATA%\Python\Scripts\ax.exe`, `%LOCALAPPDATA%\Programs\Python\Python*\Scripts\ax.exe`
3. インストール: `pip install arize-ax-cli`
4. PATH に追加: `$env:PATH = "$env:APPDATA\Python\Scripts;$env:PATH"`

## Version too old（0.8.0 未満）

アップグレード: `uv tool install --force --reinstall arize-ax-cli`, `pipx upgrade arize-ax-cli`, または `pip install --upgrade arize-ax-cli`

## SSL/証明書エラー

- macOS: `export SSL_CERT_FILE=/etc/ssl/cert.pem`
- Linux: `export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`
- フォールバック: `export SSL_CERT_FILE=$(python -c "import certifi; print(certifi.where())")`

## サブコマンドが認識されない

ax をアップグレードする（上記参照）か、利用可能な最も近い代替手段を使用してください。

## まだ失敗する場合

停止して、ユーザーに助けを求めてください。

