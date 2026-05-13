---
name: create-github-pull-request-from-specification
description: 'pull_request_template.md template を使って、specification file から feature request 用の GitHub Pull Request を作成します。'
---

# Create GitHub Pull Request from Specification

`${workspaceFolder}/.github/pull_request_template.md` にある specification に対する GitHub Pull Request を作成してください。

## Process

1. `${workspaceFolder}/.github/pull_request_template.md` の specification file template を `search` tool で分析し、requirement を抽出する。
2. `${input:targetBranch}` に対して `create_pull_request` tool で pull request draft template を作成し、同時に `get_pull_request` で current branch の pull request が既に存在しないことを確認する。存在する場合は step 4 に進み、step 3 はスキップする。
3. `get_pull_request_diff` tool を使って pull request の変更内容を取得し、変更情報を分析する。
4. 前 step で作成した pull request の body と title を `update_pull_request` tool で更新する。step 1 で取得した template 情報を反映して、必要に応じて body と title を更新する。
5. `update_pull_request` tool を使って draft から ready for review に切り替え、pull request の state を更新する。
6. `get_me` で pull request を作成した人の username を取得し、`update_issue` tool でその pull request に割り当てる。
7. 作成した Pull request の URL をユーザーへ返す。

## Requirements
- specification 全体に対して pull request は 1 件のみ
- specification を識別できる明確な title/pull_request_template.md にする
- pull_request_template.md に十分な情報を埋める
- 作成前に既存 pull request と照合する
