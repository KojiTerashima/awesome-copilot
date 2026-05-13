# ax プロファイル設定

認証に失敗したとき（401、プロファイル未設定、APIキー未設定）にこれを参照してください。これらの確認を事前に能動的に実行してはいけません。

プロファイルが存在しない場合、またはプロファイル設定が不正な場合（APIキー違い、リージョン違いなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

出力を確認して、何が設定されているかを把握します。
- `API Key: (not set)` と表示される、または欠落している → キーを作成/更新する必要があります
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しません
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れです
- 接続済みだがエンドポイント/リージョンが誤っている → リージョンを更新する必要があります

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在し、1つ以上の設定が誤っている場合は、不具合のある箇所だけを修正します。

**生のAPIキー値をフラグに直接渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照してください。シェルに変数が未設定の場合は、先にユーザーへ設定を案内してから次のコマンドを実行します。

```bash
# すでにシェルで ARIZE_API_KEY が export されている場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（秘密情報を含まないため、そのまま実行して安全）
ax profiles update --region us-east-1b

# 両方を同時に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定した項目だけを変更し、それ以外の設定は保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルを完全に別の構成（別組織、別リージョン）に向ける必要がある場合:

**必ずキーは `$ARIZE_API_KEY` で参照し、生の値をインラインで書かないでください。**

```bash
# 事前にシェルで ARIZE_API_KEY が export されている必要があります
ax profiles create --api-key $ARIZE_API_KEY

# リージョンを指定して作成
ax profiles create --api-key $ARIZE_API_KEY --region us-east-1b

# 名前付きプロファイルを作成
ax profiles create work --api-key $ARIZE_API_KEY --region us-east-1b
```

任意の `ax` コマンドで名前付きプロファイルを使うには、`-p NAME` を付けます。
```bash
ax spans export PROJECT_ID -p work
```

## 4. APIキーの取得

**ユーザーにAPIキーをチャットへ貼り付けるよう依頼してはいけません。APIキー値をログ出力、echo、表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合、ユーザーにシェルで次を export するよう案内してください。

```bash
export ARIZE_API_KEY="..."   # ユーザーが自分の端末でここにキーを貼り付けます
```

キーは https://app.arize.com/admin > API Keys で確認できます。**scoped service key**（個人ユーザーキーではない）の作成を推奨してください。service key は個人アカウントに紐づかず、プログラム利用でより安全です。キーはスペース単位のため、正しいスペースのキーをコピーしていることを確認してください。

ユーザーが変数設定を確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します。

## 5. 検証

作成または更新の後は必ず:

```bash
ax profiles show
```

APIキーとリージョンが正しいことを確認してから、元のコマンドを再実行してください。

## Space ID

space ID 用のプロファイルフラグはありません。環境変数として保存します。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後 `source ~/.zshrc`（またはターミナルを再起動）。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映するにはターミナルを再起動してください。

## 今後使うために認証情報を保存する

**セッションの最後**に、この会話中でユーザーが認証情報を手動提供しており、かつそれらの値が保存済みプロファイルや環境変数から読み込まれたものではない場合は、保存するかどうかを提案してください。

**次の場合はこの手順を完全にスキップ:**
- APIキーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数からすでに読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数ですでに設定済みだった
- ユーザーが base64 project ID のみを使用した（space ID が不要だった）

**提案方法:** **AskQuestion** を使用: *"Would you like to save your Arize credentials so you don't have to enter them next time?"*  
選択肢は `"Yes, save them"` / `"No thanks"`。

**ユーザーが yes と答えた場合:**

1. **API key** — 現在の状態を確認するため `ax profiles show` を実行。次に `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行（キーは事前に環境変数として export 済みである必要があります。生のキー値を渡してはいけません）。

2. **Space ID** — 環境変数として永続化する方法は、上記「Space ID」セクションを参照してください。

