# ax プロファイル設定

認証に失敗したとき（401、プロファイル未設定、APIキー未設定）にこれを参照してください。これらのチェックを先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイル設定が誤っている場合（APIキー違い、リージョン違いなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

何が設定されているかを把握するため、出力内容を確認します。
- `API Key: (not set)` または表示自体がない → キーを作成/更新する必要があります
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しません
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れです
- 接続済みだが endpoint/region が誤っている → region の更新が必要です

## 2. 設定ミスのあるプロファイルを修正する

プロファイルは存在するが1つ以上の設定が誤っている場合は、壊れている箇所だけを修正してください。

**生の API キー値をフラグで渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照してください。変数がシェルにまだ設定されていない場合は、先にユーザーへ設定を案内し、その後でコマンドを実行します。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# region を修正（秘密情報を含まないため、そのまま実行して安全）
ax profiles update --region us-east-1b

# 両方を同時に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定したフィールドのみを変更し、それ以外の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルを完全に別の設定（別 org、別 region）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` 経由で参照し、生の値をインラインで書かないでください。**

```bash
# 事前に ARIZE_API_KEY がシェルで export されている必要があります
ax profiles create --api-key $ARIZE_API_KEY

# region を指定して作成
ax profiles create --api-key $ARIZE_API_KEY --region us-east-1b

# 名前付きプロファイルを作成
ax profiles create work --api-key $ARIZE_API_KEY --region us-east-1b
```

任意の `ax` コマンドで名前付きプロファイルを使うには、`-p NAME` を追加します。
```bash
ax spans export PROJECT_ID -p work
```

## 4. API キーの取得

**ユーザーに API キーをチャットへ貼り付けるよう依頼してはいけません。API キー値をログ出力・echo・表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合は、ユーザーにシェルで export してもらいます。

```bash
export ARIZE_API_KEY="..."   # ユーザーが自身のターミナルでここにキーを貼り付けます
```

キーは https://app.arize.com/admin > API Keys で確認できます。**scoped service key**（個人ユーザーキーではない）の作成を推奨してください。service key は個人アカウントに紐づかず、プログラム用途でより安全です。キーは space ごとに分かれているため、正しい space のキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` に進みます。

## 5. 検証

create または update の後は必ず以下を実行します。

```bash
ax profiles show
```

API キーと region が正しいことを確認し、元のコマンドを再実行してください。

## Space ID

space ID にはプロファイル用フラグがありません。環境変数として保存してください。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後 `source ~/.zshrc`（またはターミナル再起動）を実行します。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映にはターミナルの再起動が必要です。

## 今後のために認証情報を保存する

**セッション終了時**に、この会話の中でユーザーが認証情報を手動入力しており、かつその値が保存済みプロファイルまたは環境変数から読み込まれたものではない場合は、保存するか提案してください。

**以下の場合はこの手順を完全にスキップ:**
- API キーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数からすでに読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数ですでに設定済みだった
- ユーザーが base64 の project ID のみを使っていた（space ID が不要だった）

**提案方法:** **AskQuestion** を使用: *"Would you like to save your Arize credentials so you don't have to enter them next time?"*  
選択肢は `"Yes, save them"` / `"No thanks"`。

**ユーザーが yes と答えた場合:**

1. **API key** — `ax profiles show` を実行して現在状態を確認。その後 `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します（キーは必ず事前に環境変数として export 済みであること。生キー値は絶対に渡さない）。

2. **Space ID** — 環境変数として永続化する方法は上記 Space ID セクションを参照してください。

