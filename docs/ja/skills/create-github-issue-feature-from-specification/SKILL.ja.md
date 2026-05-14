---
name: create-github-issue-feature-from-specification
description: 'feature_request.yml template を使って、specification file から feature request 用の GitHub Issue を作成します。'
---

# Create GitHub Issue from Specification

`${file}` にある specification から GitHub Issue を作成してください。

## Process

1. specification file を分析して requirement を抽出する
2. `search_issues` を使って既存 issue を確認する
3. `create_issue` で新規 issue を作成するか、既存 issue があれば `update_issue` で更新する
4. `feature_request.yml` template を使う（なければ default を使う）

## Requirements

- specification 全体に対して issue は 1 件のみ
- specification を識別できる明確な title
- specification で要求されている変更だけを含める
- 作成前に既存 issue と照合する

## Issue Content

- Title: specification から得た feature 名
- Description: 問題の説明、提案する解決策、文脈
- Labels: feature、enhancement（必要に応じて）
