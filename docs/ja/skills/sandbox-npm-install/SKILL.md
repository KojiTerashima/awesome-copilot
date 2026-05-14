---
name: sandbox-npm-install
description: 'Install npm packages in a Docker sandbox environment. Use this skill whenever you need to install, reinstall, or update node_modules inside a container where the workspace is mounted via virtiofs. Native binaries (esbuild, lightningcss, rollup) crash on virtiofs, so packages must be installed on the local ext4 filesystem and symlinked back.'
---
# サンドボックス npm インストール

## このスキルを使用する場合

このスキルは次の場合に使用します。
- 新しいサンドボックス セッションで初めて npm パッケージをインストールする必要があります
- `package.json` または `package-lock.json` が変更されたため、再インストールする必要があります
- `SIGILL`、`SIGSEGV`、`mmap`、`unaligned sysNoHugePageOS` などのエラーによるネイティブ バイナリ クラッシュが発生する
- `node_modules` ディレクトリが見つからないか破損しています

## 前提条件

- virtiofs がマウントされたワークスペースを備えた Docker サンドボックス環境
- コンテナーで Node.js と npm が利用可能
- ターゲット ワークスペース内の `package.json` ファイル

## 背景

Docker サンドボックス ワークスペースは通常、**virtiofs** (ホストと Linux VM 間のファイル同期) を介してマウントされます。ネイティブ Go および Rust バイナリ (esbuild、lightningcss、rollup など) は、aarch64 上の virtiofs から実行すると mmap アライメント エラーでクラッシュします。修正するには、コンテナーのローカル ext4 ファイルシステムにインストールし、シンボリックリンクをワークスペースに戻します。

## 段階的なインストール

バンドルされているインストール スクリプトをワークスペース ルートから実行します。```bash
bash scripts/install.sh
```### 共通オプション

|オプション |説明 |
|---|---|
| `--workspace <path>` | `package.json` を含むディレクトリへのパス (省略した場合は自動検出) |
| `--playwright` | E2E テスト用に Playwright Chromium ブラウザもインストールします。

### スクリプトの動作

1. `package.json`、`package-lock.json`、および `.npmrc` (存在する場合) をローカルの ext4 ディレクトリにコピーします
2. ローカル ファイル システムで `npm ci` (ロックファイルがない場合は `npm install`) を実行します。
3. `node_modules` をワークスペースにシンボリックリンクします。
4. 既知のネイティブ バイナリ (esbuild、rollup、lightningcss、vite) が存在する場合は検証します。
5. オプションで Playwright ブラウザとシステムの依存関係をインストールします (使用可能な場合は `sudo` を使用します)

検証が失敗した場合は、スクリプトを再度実行します。初期セットアップ中にクラッシュが断続的に発生する可能性があります。

## インストール後の検証

スクリプトが完了したら、ツールチェーンが動作することを確認します。例えば：```bash
npm test             # Run project tests
npm run build        # Build the project
npm run dev          # Start dev server
```## 重要な注意事項

- ローカル インストール ディレクトリ (例: `/home/agent/project-deps`) は **container-local** であり、ホストに同期されていません。
- `node_modules` シンボリックリンクはホスト上で壊れたリンクとして表示されます。`node_modules` は通常 gitignored されるため、これは無害です
- ホスト上で `npm ci` または `npm install` を実行すると、シンボリックリンクは自然に実際のディレクトリに置き換えられます。
- `package.json` または `package-lock.json` を変更した後、インストール スクリプトを再実行します。
- マウントされたワークスペースで `npm ci` または `npm install` を直接実行しないでください。ネイティブ バイナリがクラッシュします。

## トラブルシューティング

|問題 |ソリューション |
|---|---|
|開発サーバーを実行する場合は `SIGILL` または `SIGSEGV` |インストール スクリプトを再実行します。 `npm install` をワークスペースで直接実行していないことを確認してください。
| `node_modules` がインストール後に見つかりません |シンボリックリンクが存在することを確認してください: `ls -la node_modules` |
|インストール中の権限エラー |ローカルの deps ディレクトリが現在のユーザーによって書き込み可能であることを確認してください。
|検証が断続的に失敗する |スクリプトを再度実行します。ネイティブ バイナリのクラッシュは、最初のロード時に決定的ではない可能性があります。

## Vite の互換性

プロジェクトで Vite を使用している場合は、`server.fs.allow` でシンボリックリンクされたパスを許可する必要がある場合があります。 Vite がシンボリックリンクを通じてファイルを提供できるように、シンボリックリンクターゲットの親ディレクトリ (例: `/home/agent/project-deps/`) を Vite 設定に追加します。