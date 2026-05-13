# ax プロファイル設定

認証が失敗したとき（401、プロファイル未設定、API キー未設定）にこれを参照してください。これらの確認を先回りして実行してはいけません。

プロファイルが存在しない場合、またはプロファイルの設定が不正な場合（API キー違い、リージョン違い など）に使用します。

## 1. 現在の状態を確認する

```bash
ax profiles show
```

出力を見て、何が設定されているかを把握します。
- `API Key: (not set)` または表示されない → キーの作成/更新が必要
- プロファイルの出力がない、または "No profiles found" → まだプロファイルが存在しない
- 接続済みだが `401 Unauthorized` が出る → キーが誤っているか期限切れ
- 接続済みだがエンドポイント/リージョンが誤っている → リージョンの更新が必要

## 2. 設定ミスのあるプロファイルを修正する

プロファイルが存在するが 1 つ以上の設定が誤っている場合は、壊れている箇所だけを修正します。

**API キーの生値をフラグに直接渡してはいけません。** 必ず `ARIZE_API_KEY` 環境変数経由で参照してください。変数がシェルに未設定の場合は、先にユーザーに設定してもらってからコマンドを実行します。

```bash
# ARIZE_API_KEY がすでにシェルで export 済みの場合:
ax profiles update --api-key $ARIZE_API_KEY

# リージョンを修正（機密情報なし — 直接実行して安全）
ax profiles update --region us-east-1b

# 両方を一度に修正
ax profiles update --api-key $ARIZE_API_KEY --region us-east-1b
```

`update` は指定した項目だけを変更し、それ以外の設定はすべて保持されます。プロファイル名を指定しない場合は、アクティブなプロファイルが更新されます。

## 3. 新しいプロファイルを作成する

プロファイルが存在しない場合、または既存プロファイルをまったく別の設定（別組織、別リージョン）に向ける必要がある場合:

**キーは必ず `$ARIZE_API_KEY` で参照し、生値を直接書かないでください。**

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

## 4. API キーを取得する

**ユーザーに API キーをチャットへ貼り付けるよう依頼してはいけません。API キーの値をログ出力・echo・表示してはいけません。**

`ARIZE_API_KEY` が未設定の場合は、ユーザーにシェルで export するよう案内します。

```bash
export ARIZE_API_KEY="..."   # ユーザーが自分の端末上でここにキーを貼り付けます
```

キーは https://app.arize.com/admin > API Keys で確認できます。**スコープ付きサービスキー**（個人ユーザーキーではない）の作成を推奨してください。サービスキーは個人アカウントに紐づかず、プログラム利用でより安全です。キーは space 単位でスコープされるため、正しい space のキーをコピーしていることを確認してください。

ユーザーが変数設定済みであることを確認したら、上記の説明どおり `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します。

## 5. 検証

作成または更新の後は必ず以下を実行します。

```bash
ax profiles show
```

API キーとリージョンが正しいことを確認し、元のコマンドを再実行します。

## Space ID

space ID 用のプロファイルフラグはありません。環境変数として保存してください。

**macOS/Linux** — `~/.zshrc` または `~/.bashrc` に追加:
```bash
export ARIZE_SPACE_ID="U3BhY2U6..."
```
その後 `source ~/.zshrc`（またはターミナル再起動）を実行します。

**Windows (PowerShell):**
```powershell
[System.Environment]::SetEnvironmentVariable('ARIZE_SPACE_ID', 'U3BhY2U6...', 'User')
```
反映のためにターミナルを再起動してください。

## 今後のために認証情報を保存する

**セッションの最後**に、この会話中にユーザーが手動で認証情報を提供した場合、かつそれらの値が保存済みプロファイルや環境変数からすでに読み込まれていなかった場合は、保存を提案します。

**次の場合はこの手順を完全にスキップ:**
- API キーが既存プロファイルまたは `ARIZE_API_KEY` 環境変数からすでに読み込まれていた
- space ID が `ARIZE_SPACE_ID` 環境変数ですでに設定されていた
- ユーザーが base64 の project ID のみを使用していた（space ID は不要だった）

**提案方法:** **AskQuestion** を使い、*"Would you like to save your Arize credentials so you don't have to enter them next time?"* と質問し、選択肢は `"Yes, save them"` / `"No thanks"` とします。

**ユーザーが yes と答えた場合:**

1. **API キー** — 現在の状態確認のために `ax profiles show` を実行します。その後 `ax profiles create --api-key $ARIZE_API_KEY` または `ax profiles update --api-key $ARIZE_API_KEY` を実行します（キーはすでに環境変数として export 済みである必要があります。生値を直接渡してはいけません）。

2. **Space ID** — 環境変数として永続化する方法は、上記の Space ID セクションを参照してください。

