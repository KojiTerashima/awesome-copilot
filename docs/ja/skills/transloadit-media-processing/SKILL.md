---
name: transloadit-media-processing
description: 'Process media files (video, audio, images, documents) using Transloadit. Use when asked to encode video to HLS/MP4, generate thumbnails, resize or watermark images, extract audio, concatenate clips, add subtitles, OCR documents, or run any media processing pipeline. Covers 86+ processing robots for file transformation at scale.'
license: MIT
compatibility: Requires a free Transloadit account (https://transloadit.com/signup). Uses the @transloadit/mcp-server MCP server or the @transloadit/node CLI.
---
# Transloadit メディア処理

Transloadit のクラウド インフラストラクチャを使用して、メディア ファイルを処理、変換、エンコードします。
86 個以上の特殊な処理ロボットでビデオ、オーディオ、画像、ドキュメントをサポートします。

## このスキルを使用する場合

このスキルは、次の場合に使用します。

- ビデオを HLS、MP4、WebM、またはその他の形式にエンコードします
- ビデオからサムネイルまたはアニメーション GIF を生成
- 画像のサイズ変更、トリミング、透かし、または最適化
- 画像形式間の変換 (JPEG、PNG、WebP、AVIF、HEIF)
- オーディオの抽出またはトランスコード (MP3、AAC、FLAC、WAV)
- ビデオまたはオーディオクリップを連結します
- ビデオに字幕またはオーバーレイテキストを追加します
- OCR文書（PDF、スキャンした画像）
- 音声合成またはテキスト読み上げを実行します。
- AI ベースのコンテンツ管理またはオブジェクト検出を適用する
- 操作を連鎖させる複数ステップのメディア パイプラインを構築する

## セットアップ

### オプション A: MCP サーバー (Copilot に推奨)

Transloadit MCP サーバーを IDE 構成に追加します。これにより、エージェントに直接アクセスできるようになります。
Transloadit ツール (`create_template`、`create_assembly`、`list_assembly_notifications` など) に送信します。

**VS Code / GitHub Copilot** (`.vscode/mcp.json` またはユーザー設定):```json
{
  "servers": {
    "transloadit": {
      "command": "npx",
      "args": ["-y", "@transloadit/mcp-server", "stdio"],
      "env": {
        "TRANSLOADIT_KEY": "YOUR_AUTH_KEY",
        "TRANSLOADIT_SECRET": "YOUR_AUTH_SECRET"
      }
    }
  }
}
```https://transloadit.com/c/-/api-credentials で API 認証情報を取得します。

### オプション B: CLI

コマンドを直接実行したい場合は、次のようにします。```bash
npx -y @transloadit/node assemblies create \
  --steps '{"encoded": {"robot": "/video/encode", "use": ":original", "preset": "hls-1080p"}}' \
  --wait \
  --input ./my-video.mp4
```## コア ワークフロー

### ビデオを HLS (アダプティブ ストリーミング) にエンコードする```json
{
  "steps": {
    "encoded": {
      "robot": "/video/encode",
      "use": ":original",
      "preset": "hls-1080p"
    }
  }
}
```### ビデオからサムネイルを生成する```json
{
  "steps": {
    "thumbnails": {
      "robot": "/video/thumbs",
      "use": ":original",
      "count": 8,
      "width": 320,
      "height": 240
    }
  }
}
```### 画像のサイズ変更と透かし入れ```json
{
  "steps": {
    "resized": {
      "robot": "/image/resize",
      "use": ":original",
      "width": 1200,
      "height": 800,
      "resize_strategy": "fit"
    },
    "watermarked": {
      "robot": "/image/resize",
      "use": "resized",
      "watermark_url": "https://example.com/logo.png",
      "watermark_position": "bottom-right",
      "watermark_size": "15%"
    }
  }
}
```### ドキュメントの OCR```json
{
  "steps": {
    "recognized": {
      "robot": "/document/ocr",
      "use": ":original",
      "provider": "aws",
      "format": "text"
    }
  }
}
```### オーディオ クリップを連結する```json
{
  "steps": {
    "imported": {
      "robot": "/http/import",
      "url": ["https://example.com/clip1.mp3", "https://example.com/clip2.mp3"]
    },
    "concatenated": {
      "robot": "/audio/concat",
      "use": "imported",
      "preset": "mp3"
    }
  }
}
```## マルチステップのパイプライン

`"use"` フィールドを使用してステップを連鎖させることができます。各ステップは、前のステップの出力を参照します。```json
{
  "steps": {
    "resized": {
      "robot": "/image/resize",
      "use": ":original",
      "width": 1920
    },
    "optimized": {
      "robot": "/image/optimize",
      "use": "resized"
    },
    "exported": {
      "robot": "/s3/store",
      "use": "optimized",
      "bucket": "my-bucket",
      "path": "processed/${file.name}"
    }
  }
}
```## 主要な概念

- **アセンブリ**: 単一の処理ジョブ。 `create_assembly` (MCP) または `assemblies create` (CLI) を介して作成されます。
- **テンプレート**: Transloadit に保存されている再利用可能なステップのセット。 `create_template` (MCP) または `templates create` (CLI) を介して作成されます。
- **ロボット**: 処理ユニット (例: `/video/encode`、`/image/resize`)。完全なリストは https://transloadit.com/docs/transcoding/ でご覧ください。
- **ステップ**: パイプラインを定義する JSON オブジェクト。各キーはステップ名で、各値はロボットを構成します。
- **`:original`**: アップロードされた入力ファイルを参照します。

## ヒント

- 処理が完了するまでブロックするには、CLI で `--wait` を使用します。
- すべてのパラメータを指定するのではなく、共通の形式ターゲットに `preset` 値 (例: `"hls-1080p"`、`"mp3"`、`"webp"`) を使用します。
- `"use": "step_name"` をチェーンして、中間ダウンロードなしで複数ステップのパイプラインを構築します。
- バッチ処理の場合は、`/http/import` を使用して URL、S3、GCS、Azure、FTP、または Dropbox からファイルをプルします。
- テンプレートには、アセンブリ作成時に渡される動的な値の `${variables}` を含めることができます。