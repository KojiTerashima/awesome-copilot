# ax プロファイル設定

認証に失敗したとき（401、プロファイル未設定、API キー未設定）にこれを参照してください。これらの確認を先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイルの設定が誤っている場合（API キーの誤り、リージョンの誤りなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

何が設定されているかを理解するために、出力を確認します。
- `API Key: (not set)` または表示がない → キーを作成または更新する必要があります
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しません
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れです
- 接続済みだがエンドポイント/リージョンが誤っている → リージョンを更新する必要があります

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在していて、1 つ以上の設定が誤っている場合は、壊れている箇所だけを修正してください。

**生の API キー値をフラグとして渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照します。変数がまだシェルで設定されていない場合は、先にユーザーへ設定を案内してからコマンドを実行してください。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（機密情報を含まないため、直接実行して安全）
ax profiles update --region us-east-1b

# 両方を一度に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定したフィールドだけを変更し、他の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存のプロファイルを完全に別の設定（別組織、別リージョン）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` 経由で参照し、生の値をインラインで書かないでください。**

```bash
# 事前に ARIZE_API_KEY をシェルで export しておく必要があります
ax profiles create --api-key $ARIZE_API_KEY

# リージョン付きで作成
ax profiles create --api-key $ARIZE_API_KEY --region us-east-1b

# 名前付きプロファイルを作成
ax profiles create work --api-key $ARIZE_API_KEY --region us-east-1b
```

任意の `ax` コマンドで名前付きプロファイルを使うには、`-p NAME` を追加します。
```bash
ax spans export PROJECT_ID -p work
```

## 4. API キーの取得

**ユーザーに API キーをチャットへ貼り付けるよう依頼してはいけません。API キーの値をログ出力・echo・表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合は、ユーザーにシェルで export するよう案内してください。

```bash
export ARIZE_API_KEY="..."   # ユーザーが自身のターミナルでここにキーを貼り付けます
```

キーは https://app.arize.com/admin > API Keys で確認できます。**scoped service key**（個人ユーザーキーではない）の作成を推奨してください。サービスキーは個人アカウントに紐づかず、プログラム用途でより安全です。キーは space スコープなので、正しい space のキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` に進みます。

## 5. 検証

作成または更新の後は必ず:

```bash
ax profiles show
```

API キーとリージョンが正しいことを確認し、その後に元のコマンドを再実行します。

## Space ID

space ID 用のプロファイルフラグはありません。環境変数として保存してください。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後、`source ~/.zshrc`（またはターミナルを再起動）します。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映のためにターミナルを再起動してください。

## 今後の利用のために認証情報を保存する

**セッションの最後に**、この会話中にユーザーが認証情報を手動で提供しており、かつその値が保存済みプロファイルまたは環境変数からすでに読み込まれていなかった場合は、保存を提案してください。

**次の場合はこの処理を完全にスキップ:**
- API キーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数からすでに読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数で既に設定済みだった
- ユーザーが base64 project ID のみを使用した（space ID は不要だった）

**提案方法:** **AskQuestion** を使い、*"Would you like to save your Arize credentials so you don't have to enter them next time?"* と質問し、選択肢を `"Yes, save them"` / `"No thanks"` にします。

**ユーザーが yes と答えた場合:**

1. **API キー** — `ax profiles show` を実行して現在の状態を確認します。その後、`ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します（キーは必ず環境変数として export 済みである必要があり、生のキー値を渡してはいけません）。

2. **Space ID** — 上記の Space ID セクションを参照して、環境変数として永続化します。

