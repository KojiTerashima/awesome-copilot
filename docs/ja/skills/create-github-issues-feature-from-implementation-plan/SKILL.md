---
name: create-github-issues-feature-from-implementation-plan
description: 'feature_request.yml または chore_request.yml template を使って、implementation plan の phase ごとに GitHub Issue を作成します。'
---

# Create GitHub Issue from Implementation Plan

`${file}` にある implementation plan から GitHub Issue を作成してください。

## Process

1. plan file を分析して phase を特定する
2. `search_issues` を使って既存 issue を確認する
3. phase ごとに `create_issue` で新規 issue を作成するか、既存 issue があれば `update_issue` で更新する
4. `feature_request.yml` または `chore_request.yml` template を使う（なければ default を使う）

## Requirements

- implementation phase ごとに issue を 1 件作成する
- 明確で構造化された title と description にする
- plan で必要な変更だけを含める
- 作成前に既存 issue と照合する

## Issue Content

- Title: implementation plan の phase 名
- Description: phase の詳細、requirements、文脈
- Labels: issue の種類に応じて適切なもの（feature/chore）
