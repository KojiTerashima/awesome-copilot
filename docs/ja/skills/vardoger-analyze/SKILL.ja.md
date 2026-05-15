---
name: vardoger-analyze
description: "ユーザーが GitHub Copilot CLI アシスタントのパーソナライズ、Copilot のスタイルへの適応、vardoger の使用、または Copilot CLI 会話履歴の分析を要求する場合に使用します。 `~/.copilot/session-state/` にあるローカル セッション ディレクトリを読み取り、繰り返しの設定と規則を抽出し、フェンスで囲まれたパーソナライゼーション ブロックを `~/.copilot/copilot-instructions.md` に書き込みます。ローカルの `vardoger` CLI (`pipx install vardoger`) を介してユーザーのマシン上で完全に実行されます。ネットワーク呼び出しやアップロードはありません。トリガー: 「副操縦士を個人化する」、「副操縦士の履歴を分析する」、「副操縦士を私に合わせて調整する」、「vardoger を実行する」、「履歴から副操縦士の指示を更新する」、「副操縦士に私のスタイルを学習させる」。"
license: Apache-2.0
---

# Copilot CLI 履歴を分析し、パーソナライズされた指示を生成します

ローカルの `vardoger` CLI を駆動して、ユーザーの GitHub Copilot CLI 会話履歴を読み取り、行動パターンを抽出し、パーソナライゼーション ブロックを `~/.copilot/copilot-instructions.md` に書き込みます。

## 仕組み

`vardoger` は履歴をバッチで準備します。あなた (アシスタント) は、行動シグナルの各バッチを要約し、すべての要約を最終的なパーソナライゼーションに統合します。 `vardoger` は、`<!-- vardoger:start -->` / `<!-- vardoger:end -->` マーカーで囲まれた結果を書き込みます。そのため、同じファイル内の手動で作成されたルールは保持されます。

## サンドボックスに関する注意事項 (コマンドを実行する前にお読みください)

`vardoger` は、現在のワークスペースの**外部**にあるファイルの読み取りと書き込みを行います。

- `~/.copilot/session-state/` から Copilot CLI 履歴を読み取ります。
- チェックポイント状態ファイルを `~/.vardoger/state.json` (初回実行時に作成) に書き込みます。
- 最終的なパーソナライゼーションを `~/.copilot/copilot-instructions.md` に書き込みます。

ホストが `vardoger` コマンドの承認を求めたら、ワークスペースを超えた書き込みアクセスを許可します。そうしないと、サンドボックスが現在の作業ディレクトリ外への書き込みをブロックするため、最初の `vardoger prepare` 呼び出しは `PermissionError: ... ~/.vardoger/state.tmp` で失敗します。

## ワークフロー

1. `vardoger` CLI がインストールされていることを確認し、インストールされていない場合はインストール ガイダンスに従ってフェイルファストします。
2. `vardoger status --platform copilot --json` で古さをチェックし、パーソナライゼーションがまだ新しい場合は早期に停止します。
3. `vardoger prepare --platform copilot` を使用してバッチのメタデータを取得し、バッチの数を確認します。
4. バッチごとに `vardoger prepare --platform copilot --batch <N>` を実行し、動作シグナルの簡潔な箇条書きの概要を書き込みます。
5. `vardoger prepare --platform copilot --synthesize` を使用して合成プロンプトを取得します。
6. 合成プロンプトに従って、すべてのバッチ サマリーを 1 つのパーソナライゼーションに合成します。
7. パーソナライゼーションを `vardoger write --platform copilot --scope global` (または `--scope project --project <path>`) にパイプして結果を書き込みます。
8. 何が、どこに書き込まれたか、書き込みが冪等であることをユーザーに報告します。

## ステップ

### 1. vardoger がインストールされていることを確認します
```bash
if ! command -v vardoger >/dev/null 2>&1; then
  cat <<'INSTALL_EOF'
vardoger CLI is not installed.

This skill calls the `vardoger` CLI to read your Copilot CLI history and
write a personalization file, so the CLI must be on PATH.

Install options:

  # Recommended:
  pipx install vardoger

  # Or run without installing:
  uvx vardoger --help

If you do not have pipx, see https://pipx.pypa.io/stable/installation/.

Project page: https://github.com/dstrupl/vardoger

After installing, re-run the personalization request.
INSTALL_EOF
  exit 1
fi
```

### 2. 更新が必要かどうかを確認する
```bash
vardoger status --platform copilot --json
```

出力に `"is_stale": false` が表示された場合は、パーソナライゼーションが最新であることをユーザーに伝え、再実行するかどうかを尋ねます。古い場合、またはまったく生成されない場合は、分析を続行します。

### 3. バッチメタデータを取得する
```bash
vardoger prepare --platform copilot
```

これにより、`{"batches": 3, "total_conversations": 29}` のような JSON が出力されます。バッチ数に注目してください。ユーザーに「M 個のバッチで N 個の会話が見つかりました。分析中...」と伝えます。

### 4. 各バッチを要約する

1 から N までのバッチ番号ごとに、次を実行します。
```bash
vardoger prepare --platform copilot --batch 1
```

出力には、要約プロンプトとその後に続く会話データが含まれます。出力を注意深く読み、そのバッチで観察された動作シグナルの簡潔な箇条書きの要約を作成します。要約は後で使用できるようにしておいてください。

どのバッチを処理しているかをユーザーに伝えます:「N 個のバッチ 1 を分析しています...」

すべてのバッチ (`--batch 2`、`--batch 3` など) に対して繰り返します。

### 5. 合成プロンプトを表示する
```bash
vardoger prepare --platform copilot --synthesize
```

### 6. パーソナライゼーションを合成する

合成プロンプトに従って、すべてのバッチ サマリーを 1 つのパーソナライゼーションに結合します。出力は、AI アシスタントに対する実用的な指示を含むクリーンなマークダウンである必要があります。

### 7. 結果を書きます

パーソナライゼーションを `vardoger` にパイプします。
```bash
echo "YOUR_PERSONALIZATION_HERE" | vardoger write --platform copilot --scope global
```

`YOUR_PERSONALIZATION_HERE` を、生成した実際のパーソナライゼーション マークダウンに置き換えます。 `--scope global` は `~/.copilot/copilot-instructions.md` に書き込みます。代わりに `--scope project --project <path>` を使用して、書き込みの範囲を特定のリポジトリに限定してください。

### 8. ユーザーへの報告

どこに何が書かれたかをユーザーに伝えます。パーソナライゼーションを更新するためにいつでも vardoger を再実行するように要求できること、書き込みが冪等であること (フェンスで囲まれたブロックは置き換えられ、その外側にあるものはすべて保持される) について言及します。

## いつ使用するか

- ユーザーが Copilot CLI アシスタントをパーソナライズするように要求したとき。
- ユーザーが Copilot CLI の会話履歴の分析を要求したとき。
- ユーザーが「vardoger」について言及したとき。