---
name: nano-banana-pro-openrouter
description: 'Gemini 3 Pro Imageモデルを使ってOpenRouter経由で画像を生成または編集します。プロンプトのみの画像生成、画像編集、複数画像の合成に対応。1K/2K/4K出力をサポート。'
metadata:
  emoji: 🍌
  requires:
    bins:
      - uv
    env:
      - OPENROUTER_API_KEY
  primaryEnv: OPENROUTER_API_KEY
---


# Nano Banana Pro OpenRouter

## 概要

`google/gemini-3-pro-image-preview`モデルを使用してOpenRouterで画像を生成または編集します。プロンプトのみの生成、単一画像の編集、複数画像の合成に対応しています。

### プロンプトのみの生成

```
uv run {baseDir}/scripts/generate_image.py \
  --prompt "雪をかぶった山々の上に広がるシネマティックな夕焼け" \
  --filename sunset.png
```

### 単一画像の編集

```
uv run {baseDir}/scripts/generate_image.py \
  --prompt "空をドラマチックなオーロラに置き換える" \
  --input-image input.jpg \
  --filename aurora.png
```

### 複数画像の合成

```
uv run {baseDir}/scripts/generate_image.py \
  --prompt "被写体を一つのスタジオポートレートにまとめる" \
  --input-image face1.jpg \
  --input-image face2.jpg \
  --filename composite.png
```

## 解像度

- `--resolution`に`1K`、`2K`、または`4K`を指定してください。
- 指定がない場合はデフォルトで`1K`になります。

## システムプロンプトのカスタマイズ

このスキルは`assets/SYSTEM_TEMPLATE`からオプションのシステムプロンプトを読み込みます。これによりコードを変更せずに画像生成の挙動をカスタマイズできます。

## 挙動と制約

- `--input-image`を繰り返して最大3枚の入力画像を受け付けます。
- `--filename`は相対パス（現在のディレクトリに保存）または絶対パスを指定可能です。
- 複数画像が返された場合はファイル名に`-1`、`-2`などを付加します。
- 保存した各画像について`MEDIA: <path>`を出力します。画像をレスポンスに再読み込みしません。

## トラブルシューティング

スクリプトが非ゼロ終了した場合はstderrを確認し、以下の一般的な問題をチェックしてください：

| 症状 | 解決策 |
|---------|------------|
| `OPENROUTER_API_KEY is not set` | ユーザーに設定を促してください。PowerShell: `$env:OPENROUTER_API_KEY = "sk-or-..."` / bash: `export OPENROUTER_API_KEY="sk-or-..."` |
| `uv: command not found` または認識されない | macOS/Linux: <code>curl -LsSf https://astral.sh/uv/install.sh &#124; sh</code>。Windows: <code>powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 &#124; iex"</code>。その後ターミナルを再起動してください。 |
| `AuthenticationError` / HTTP 401 | キーが無効かクレジットがありません。<https://openrouter.ai/settings/keys>で確認してください。 |

一時的なエラー（HTTP 429、ネットワークタイムアウトなど）の場合は30秒後に1回だけ再試行してください。同じエラーを2回以上繰り返さず、問題をユーザーに通知してください。
