# ax プロファイル設定

認証が失敗したとき（401、プロファイル未設定、APIキー未設定）にこれを参照してください。これらの確認を先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイルの設定が誤っている場合（APIキーの誤り、リージョンの誤りなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

出力を見て、何が設定されているかを把握してください。
- `API Key: (not set)` または表示自体がない → キーを作成/更新する必要がある
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しない
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れ
- 接続済みだがエンドポイント/リージョンが違う → リージョンを更新する必要がある

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在するが1つ以上の設定が誤っている場合は、壊れている箇所だけを修正してください。

**生のAPIキー値をフラグとして渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照してください。変数がまだシェルに設定されていない場合は、先に設定するようユーザーに案内し、その後コマンドを実行します。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（秘密情報を含まないため、そのまま実行して安全）
ax profiles update --region us-east-1b

# 両方を同時に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定したフィールドだけを変更し、それ以外の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルをまったく別の設定（別組織、別リージョン）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` で参照し、生値をインラインで書かないでください。**

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

## 4. APIキーの取得

**ユーザーにAPIキーをチャットへ貼り付けるよう求めてはいけません。APIキー値をログ出力・echo・表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合は、ユーザーにシェルで export するよう案内してください。

```bash
export ARIZE_API_KEY="..."   # ユーザーが自分のターミナルでここにキーを貼り付ける
```

キーは https://app.arize.com/admin > API Keys で確認できます。**スコープ付きサービスキー**（個人ユーザーキーではない）を作成することを推奨してください。サービスキーは個人アカウントに紐づかないため、プログラム用途でより安全です。キーはスペース単位でスコープされるため、正しいスペースのキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します。

## 5. 検証

作成または更新の後は必ず次を実行します。

```bash
ax profiles show
```

APIキーとリージョンが正しいことを確認し、元のコマンドを再実行してください。

## Space ID

Space ID 用のプロファイルフラグはありません。環境変数として保存してください。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後 `source ~/.zshrc`（またはターミナル再起動）を実行します。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映のためターミナルを再起動してください。

## 今後使うために認証情報を保存する

**セッションの最後**に、この会話中にユーザーが何らかの認証情報を手動で提供しており、**かつ** それらの値が保存済みプロファイルまたは環境変数から既に読み込まれたものではない場合、保存を提案してください。

**次の場合はこの処理を完全にスキップ:**
- APIキーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数から既に読み込まれていた
- Space ID が `ARIZE_SPACE_ID` 環境変数で既に設定されていた
- ユーザーが base64 の project ID だけを使用した（space ID は不要だった）

**提案方法:** **AskQuestion** を使って、*"Would you like to save your Arize credentials so you don't have to enter them next time?"* と尋ね、選択肢は `"Yes, save them"` / `"No thanks"` にします。

**ユーザーが yes と答えた場合:**

1. **API key** — 現在の状態確認のため `ax profiles show` を実行し、その後 `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します（キーは事前に env var として export 済みである必要があります。生のキー値は絶対に渡さないでください）。

2. **Space ID** — 環境変数として永続化するため、上記の Space ID セクションを参照してください。

