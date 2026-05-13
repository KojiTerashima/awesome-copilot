# ax プロファイル設定

認証に失敗する場合（401、プロファイル未設定、APIキー未設定）にこれを参照してください。これらの確認を先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイル設定に誤りがある場合（APIキー違い、リージョン違いなど）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

出力を見て、何が設定されているかを把握します。
- `API Key: (not set)` または表示なし → キーの作成または更新が必要
- プロファイル出力がない、または "No profiles found" → まだプロファイルが存在しない
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れ
- 接続済みだがエンドポイント/リージョンが誤っている → リージョンの更新が必要

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在していて一部設定が誤っている場合は、壊れている箇所だけを修正します。

**生のAPIキー値をフラグとして渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照してください。変数がまだシェルで設定されていない場合は、先にユーザーに設定してもらってからコマンドを実行します。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（秘密情報を含まないため、そのまま実行して安全）
ax profiles update --region us-east-1b

# 両方を一度に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定したフィールドだけを変更し、それ以外の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルを完全に別の設定（別組織、別リージョン）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` 経由で参照し、生の値をインラインで書かないでください。**

```bash
# 事前に ARIZE_API_KEY がシェルで export 済みである必要があります
ax profiles create --api-key $ARIZE_API_KEY

# リージョンを指定して作成
ax profiles create --api-key $ARIZE_API_KEY --region us-east-1b

# 名前付きプロファイルを作成
ax profiles create work --api-key $ARIZE_API_KEY --region us-east-1b
```

任意の `ax` コマンドで名前付きプロファイルを使うには、`-p NAME` を追加します。
```bash
ax spans export PROJECT_ID -p work
```

## 4. APIキーの取得

**ユーザーにAPIキーをチャットへ貼り付けるよう求めてはいけません。APIキー値をログ出力、echo、表示してはいけません。**

`ARIZE_API_KEY` がまだ設定されていない場合は、ユーザーにシェルで export するよう案内します。

```bash
export ARIZE_API_KEY="..."   # ユーザーは自分の端末上でここにキーを貼り付けます
```

キーは https://app.arize.com/admin > API Keys で確認できます。**スコープ付きサービスキー**（個人ユーザーキーではない）を作成するよう推奨してください。サービスキーは個人アカウントに紐づかず、プログラム用途でより安全です。キーはスペース単位でスコープされるため、正しいスペースのキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を進めます。

## 5. 検証

作成または更新の後は毎回、次を実行します。

```bash
ax profiles show
```

APIキーとリージョンが正しいことを確認し、その後で元のコマンドを再実行します。

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
反映のためターミナルを再起動します。

## 資格情報を次回利用のために保存する

**セッションの終了時**、この会話中にユーザーが資格情報を手動で提供しており、**かつ** それらの値が保存済みプロファイルや環境変数から既に読み込まれたものではない場合は、保存を提案します。

**以下の場合はこの手順を完全にスキップします:**
- APIキーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数からすでに読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数で既に設定されていた
- ユーザーが base64 project ID のみを使用した（space ID は不要だった）

**提案方法:** **AskQuestion** を使う: *"Would you like to save your Arize credentials so you don't have to enter them next time?"*  
選択肢は `"Yes, save them"` / `"No thanks"`。

**ユーザーが yes と答えた場合:**

1. **API key** — 現在の状態確認のため `ax profiles show` を実行。次に `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します（キーは必ず事前に環境変数として export 済みであること。生キー値を渡してはいけません）。

2. **Space ID** — 環境変数として永続化するには、上記の Space ID セクションを参照してください。

