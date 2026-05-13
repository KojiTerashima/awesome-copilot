# ax プロファイル設定

認証に失敗する場合（401、プロファイル不足、API キー不足）に参照してください。これらの確認を先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイル設定が誤っている場合（API キー違い、リージョン違いなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

出力を見て、何が設定されているかを把握します。
- `API Key: (not set)` または表示自体がない → キーの作成/更新が必要
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しない
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れ
- 接続済みだがエンドポイント/リージョンが違う → リージョンの更新が必要

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在していて、1 つ以上の設定が誤っている場合は、壊れている項目だけを修正します。

**生の API キー値をフラグで渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数を参照してください。変数がまだシェルに設定されていない場合は、先に設定するようユーザーに案内してから次のコマンドを実行します。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（秘密情報は含まれないため、そのまま実行して安全）
ax profiles update --region us-east-1b

# 両方を一度に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定したフィールドのみを変更し、他の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルをまったく別の構成（別 org、別リージョン）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` 経由で参照し、生値を直接書かないでください。**

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

**ユーザーに API キーをチャットへ貼り付けるよう求めてはいけません。API キー値をログ出力・echo・表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合は、シェルで export するよう案内します。

```bash
export ARIZE_API_KEY="..."   # ユーザーは自分の端末上でここにキーを貼り付ける
```

キーは https://app.arize.com/admin > API Keys で確認できます。**スコープ付きサービスキー**（個人ユーザーキーではない）を作成することを推奨してください。サービスキーは個人アカウントに紐づかないため、プログラム用途ではより安全です。キーはスペース単位でスコープされるため、正しいスペースのキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記のとおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します。

## 5. 検証

作成または更新の後は毎回:

```bash
ax profiles show
```

API キーとリージョンが正しいことを確認してから、元のコマンドを再実行します。

## Space ID

space ID 用のプロファイルフラグはありません。環境変数として保存してください。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後 `source ~/.zshrc`（またはターミナル再起動）。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映にはターミナルの再起動が必要です。

## 今後のために認証情報を保存する

**セッションの最後**に、この会話中にユーザーが認証情報を手動で提供しており、**かつ** それらの値が保存済みプロファイルまたは環境変数から既に読み込まれていない場合は、保存するか提案してください。

**次の場合はこの手順全体をスキップ:**
- API キーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数から既に読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数で既に設定されていた
- ユーザーが base64 project ID のみを使用した（space ID は不要だった）

**提案方法:** **AskQuestion** を使用: *"Would you like to save your Arize credentials so you don't have to enter them next time?"*  
選択肢は `"Yes, save them"` / `"No thanks"`。

**ユーザーが yes と答えた場合:**

1. **API キー** — 現在の状態を確認するために `ax profiles show` を実行。その後 `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行（キーはすでに環境変数として export 済みであること。生のキー値は絶対に渡さない）。

2. **Space ID** — 上記の「Space ID」セクションを参照し、環境変数として永続化します。

