---
name: create-github-issues-for-unmet-specification-requirements
description: 'feature_request.yml template を使って、specification file にある未実装 requirement 用の GitHub Issue を作成します。'
---

# Create GitHub Issues for Unmet Specification Requirements

`${file}` にある specification 内の未実装 requirement に対する GitHub Issue を作成してください。

## Process

1. specification file を分析して、すべての requirement を抽出する
2. requirement ごとの codebase 実装状況を確認する
3. 重複を避けるため `search_issues` で既存 issue を検索する
4. 未実装 requirement ごとに `create_issue` で新規 issue を作成する
5. `feature_request.yml` template を使う（なければ default を使う）

## Requirements

- specification の未実装 requirement ごとに issue を 1 件作成する
- requirement ID と説明の対応関係を明確にする
- 実装ガイダンスと acceptance criteria を含める
- 作成前に既存 issue と照合する

## Issue Content

- Title: requirement ID と簡潔な説明
- Description: 詳細 requirement、実装方法、文脈
- Labels: feature、enhancement（必要に応じて）

## Implementation Check

- 関連する code pattern を codebase から検索する
- `/spec/` directory にある関連 specification file を確認する
- requirement が部分的に実装済みでないことを確認する
