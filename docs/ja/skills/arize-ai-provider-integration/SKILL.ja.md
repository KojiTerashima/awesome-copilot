---
name: arize-ai-provider-integration
description: "Arize AI 連携を作成・参照・更新・削除する際に **このスキルを呼び出してください**。連携の一覧表示、対応する任意の LLM プロバイダー（OpenAI、Anthropic、Azure OpenAI、AWS Bedrock、Vertex AI、Gemini、NVIDIA NIM、custom）向け連携の作成、認証情報やメタデータの更新、ax CLI を使った連携削除をカバーします。"
---

# Arize AI Integration Skill

## 概念

- **AI Integration** = Arize に登録された保存済み LLM プロバイダー認証情報。評価器がジャッジモデルを呼び出すためや、Arize の他機能がユーザーに代わって LLM を呼び出す必要がある場合に使用されます
- **Provider** = 連携の背後にある LLM サービス（例: `openAI`, `anthropic`, `awsBedrock`）
- **Integration ID** = 連携の base64 エンコード済みグローバル識別子（例: `TGxtSW50ZWdyYXRpb246MTI6YUJjRA==`）。評価器作成や他の後続処理に必要です
- **Scoping** = どのスペースまたはユーザーが連携を利用できるかを制御する可視性ルール
- **Auth type** = Arize がプロバイダーに認証する方法: `default`（プロバイダー API キー）、`proxy_with_headers`（カスタムヘッダー経由のプロキシ）、`bearer_token`（Bearer トークン認証）

## 前提条件

タスクに直接進み、必要な `ax` コマンドを実行してください。事前にバージョン、環境変数、プロファイルを確認しないでください。

`ax` コマンドが失敗した場合は、エラーに基づいて対処します:
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / API キー不足 → `ax profiles show` を実行して現在のプロファイルを確認。プロファイルがない、または API キーが誤っている場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md を使ってプロファイルを作成/更新。`.env` にもキーがない場合は、ユーザーに Arize API キー（https://app.arize.com/admin > API Keys）を尋ねる
- Space ID が不明 → `.env` の `ARIZE_SPACE_ID` を確認、または `ax spaces list -o json` を実行、またはユーザーに確認
- LLM プロバイダー呼び出しが失敗（OPENAI_API_KEY / ANTHROPIC_API_KEY 不足）→ `.env` を確認し、あれば読み込み、なければユーザーに確認

---

## AI Integration を一覧表示

スペースでアクセス可能な連携をすべて表示:

```bash
ax ai-integrations list --space-id SPACE_ID
```

名前でフィルタ（大文字小文字を区別しない部分一致）:

```bash
ax ai-integrations list --space-id SPACE_ID --name "openai"
```

大量の結果をページネーション:

```bash
# 1ページ目を取得
ax ai-integrations list --space-id SPACE_ID --limit 20 -o json

# 前回レスポンスのカーソルを使って次ページを取得
ax ai-integrations list --space-id SPACE_ID --limit 20 --cursor CURSOR_TOKEN -o json
```

**主なフラグ:**

| Flag | Description |
|------|-------------|
| `--space-id` | 連携を一覧表示する対象スペース |
| `--name` | 連携名に対する大文字小文字を区別しない部分一致フィルタ |
| `--limit` | 最大件数（1–100、デフォルト 50） |
| `--cursor` | 前回レスポンスのページネーショントークン |
| `-o, --output` | 出力形式: `table`（デフォルト）または `json` |

**レスポンス項目:**

| Field | Description |
|-------|-------------|
| `id` | Base64 の Integration ID — 後続コマンド用にこれをコピー |
| `name` | 人が読める名前 |
| `provider` | LLM プロバイダー enum（下記の Supported Providers を参照） |
| `has_api_key` | 認証情報が保存されていれば `true` |
| `model_names` | 許可されたモデル一覧。全モデル有効なら `null` |
| `enable_default_models` | このプロバイダーのデフォルトモデルを許可するか |
| `function_calling_enabled` | ツール/関数呼び出しが有効か |
| `auth_type` | 認証方式: `default`, `proxy_with_headers`, `bearer_token` |

---

## 特定の Integration を取得

```bash
ax ai-integrations get INT_ID
ax ai-integrations get INT_ID -o json
```

連携の完全な設定を確認したり、作成後に ID を確認したりするために使います。

---

## AI Integration を作成

作成前に必ず先に連携一覧を確認してください。ユーザーがすでに適切な連携を持っている可能性があります:

```bash
ax ai-integrations list --space-id SPACE_ID
```

適切な連携がない場合は作成します。必要なフラグはプロバイダーによって異なります。

### OpenAI

```bash
ax ai-integrations create \
  --name "My OpenAI Integration" \
  --provider openAI \
  --api-key $OPENAI_API_KEY
```

### Anthropic

```bash
ax ai-integrations create \
  --name "My Anthropic Integration" \
  --provider anthropic \
  --api-key $ANTHROPIC_API_KEY
```

### Azure OpenAI

```bash
ax ai-integrations create \
  --name "My Azure OpenAI Integration" \
  --provider azureOpenAI \
  --api-key $AZURE_OPENAI_API_KEY \
  --base-url "https://my-resource.openai.azure.com/"
```

### AWS Bedrock

AWS Bedrock は API キーではなく IAM ロールベース認証を使用します。Arize が引き受けるロールの ARN を指定してください:

```bash
ax ai-integrations create \
  --name "My Bedrock Integration" \
  --provider awsBedrock \
  --role-arn "arn:aws:iam::123456789012:role/ArizeBedrockRole"
```

### Vertex AI

Vertex AI は GCP サービスアカウント認証情報を使用します。GCP プロジェクトとリージョンを指定してください:

```bash
ax ai-integrations create \
  --name "My Vertex AI Integration" \
  --provider vertexAI \
  --project-id "my-gcp-project" \
  --location "us-central1"
```

### Gemini

```bash
ax ai-integrations create \
  --name "My Gemini Integration" \
  --provider gemini \
  --api-key $GEMINI_API_KEY
```

### NVIDIA NIM

```bash
ax ai-integrations create \
  --name "My NVIDIA NIM Integration" \
  --provider nvidiaNim \
  --api-key $NVIDIA_API_KEY \
  --base-url "https://integrate.api.nvidia.com/v1"
```

### Custom (OpenAI 互換エンドポイント)

```bash
ax ai-integrations create \
  --name "My Custom Integration" \
  --provider custom \
  --base-url "https://my-llm-proxy.example.com/v1" \
  --api-key $CUSTOM_LLM_API_KEY
```

### Supported Providers

| Provider | Required extra flags |
|----------|---------------------|
| `openAI` | `--api-key <key>` |
| `anthropic` | `--api-key <key>` |
| `azureOpenAI` | `--api-key <key>`, `--base-url <azure-endpoint>` |
| `awsBedrock` | `--role-arn <arn>` |
| `vertexAI` | `--project-id <gcp-project>`, `--location <region>` |
| `gemini` | `--api-key <key>` |
| `nvidiaNim` | `--api-key <key>`, `--base-url <nim-endpoint>` |
| `custom` | `--base-url <endpoint>` |

### 任意プロバイダーで使えるオプションフラグ

| Flag | Description |
|------|-------------|
| `--model-names` | 許可するモデル名のカンマ区切りリスト。省略すると全モデルを許可 |
| `--enable-default-models` / `--no-default-models` | プロバイダーのデフォルトモデル一覧を有効/無効化 |
| `--function-calling` / `--no-function-calling` | ツール/関数呼び出しサポートを有効/無効化 |

### 作成後

返却された Integration ID（例: `TGxtSW50ZWdyYXRpb246MTI6YUJjRA==`）を控えてください。評価器作成や他の後続コマンドで必要です。見逃した場合は取得します:

```bash
ax ai-integrations list --space-id SPACE_ID -o json
# または、ID がわかっている場合:
ax ai-integrations get INT_ID
```

---

## AI Integration を更新

`update` は部分更新です。指定したフラグのみ変更されます。省略した項目はそのまま維持されます。

```bash
# 名前変更
ax ai-integrations update INT_ID --name "New Name"

# API キーをローテーション
ax ai-integrations update INT_ID --api-key $OPENAI_API_KEY

# モデル一覧を変更
ax ai-integrations update INT_ID --model-names "gpt-4o,gpt-4o-mini"

# ベース URL を更新（Azure、custom、NIM 向け）
ax ai-integrations update INT_ID --base-url "https://new-endpoint.example.com/v1"
```

`create` で受け付けるフラグはすべて `update` にも渡せます。

---

## AI Integration を削除

**警告:** 削除は永続的です。この連携を参照する評価器は実行できなくなります。

```bash
ax ai-integrations delete INT_ID --force
```

即時削除ではなく確認プロンプトを出す場合は `--force` を省略してください。

---

## トラブルシューティング

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `401 Unauthorized` | API キーがこのスペースへのアクセス権を持っていない可能性があります。https://app.arize.com/admin > API Keys でキーと Space ID を確認 |
| `No profile found` | `ax profiles show --expand` を実行; `ARIZE_API_KEY` 環境変数を設定するか `~/.arize/config.toml` を記述 |
| `Integration not found` | `ax ai-integrations list --space-id SPACE_ID` で確認 |
| 作成後に `has_api_key: false` | 認証情報が保存されていません。正しい `--api-key` または `--role-arn` で `update` を再実行 |
| 評価器実行が LLM エラーで失敗する | `ax ai-integrations get INT_ID` で連携の認証情報を確認; 必要なら API キーをローテーション |
| `provider` の不一致 | 作成後に provider は変更不可。削除して正しい provider で再作成 |

---

## 関連スキル

- **arize-evaluator**: AI Integration を使う LLM-as-judge 評価器を作成 → `arize-evaluator` を使用
- **arize-experiment**: AI Integration を利用する評価器を使って実験を実行 → `arize-experiment` を使用

---

## 認証情報を今後の利用のために保存

references/ax-profiles.md の「§ Save Credentials for Future Use」を参照してください。

