# Step 1a: Entry Point & Execution Flow

アプリケーションがどのように起動し、実際のユーザーがどのように呼び出すのかを特定します。

---

## What to investigate

### 1. How the software runs

entry point は何か? どう起動するか? CLI か、server か、library function か? 必要な引数、config file、environment variable は何か?

確認するポイント:

- `if __name__ == "__main__"` block
- framework の entry point (FastAPI の `app`、Flask の `app`、Django の `manage.py`)
- `pyproject.toml` の CLI entry point (`[project.scripts]`)
- 起動コマンドが分かる Docker/compose 設定

### 2. The real user entry point

実際のユーザーや client はどうアプリを呼び出すのか? eval が対象にすべきのはこれであり、request pipeline を迂回する内側の function ではありません。

- **Web server**: どの HTTP endpoint が user input を受けるか? method は何か (GET/POST)? request body の shape は?
- **CLI**: ユーザーはどの command-line argument を渡すか?
- **Library/function**: 呼び出し側はどの function を import して呼ぶか? 引数は何か?

### 3. Environment and configuration

- アプリに必要な env var は何か? (API key、database URL、feature flag など)
- 読み込む config file は何か?
- 妥当な default があるものは何で、明示設定が必須なのは何か?

---

## Output: `pixie_qa/01-entry-point.md`

調査結果をこのファイルに書いてください。焦点は entry point と execution flow だけに絞ります。

### Template

```markdown
# Entry Point & Execution Flow

## How to run

<アプリを起動するコマンド、必要な env var、config file>

## Entry point

- **File**: <例: app.py, main.py>
- **Type**: <FastAPI server / CLI / standalone function / etc.>
- **Framework**: <FastAPI, Flask, Django, none>

## User-facing endpoints / interface

<ユーザーがアプリとやり取りする各方法について:>

- **Endpoint / command**: <例: POST /chat, python main.py --query "...">
- **Input format**: <request body shape, CLI args, function params>
- **Output format**: <response shape, stdout format, return type>

## Environment requirements

| Variable | Purpose | Required? | Default |
| -------- | ------- | --------- | ------- |
| ...      | ...     | ...       | ...     |
```
