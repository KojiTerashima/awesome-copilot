---
applyTo: '**/.copilot-tracking/changes/*.md'
description: '段階的な追跡と変更記録を伴って task plan を実装するための指示 - microsoft/edge-ai 提供'
---

# タスク計画実装指示

`.copilot-tracking/plans/**` と `.copilot-tracking/details/**` にある対象 task plan を実装すること。目標は、plan file 内の各 step を段階的かつ完全に実装し、指定されたすべての要件を満たす高品質で動作する software を作ることである。

実装進捗は、`.copilot-tracking/changes/**` にある対応する changes file で必ず追跡しなければならない。

## 中核となる実装プロセス

### 1. Plan の分析と準備

**実装開始前に必ず完了すること:**
- **必須**: scope、objective、全 phase、すべての checklist item を含む完全な plan file を読んで理解する
- **必須**: 対応する changes file を最後まで完全に読んで理解する。context に欠落がある場合は、`read_file` を使って file 全体を再読する
- **必須**: plan 内で言及されているすべての参照 file を特定し、context のために調べる
- **必須**: 現在の project structure と convention を理解する

### 2. 体系的な実装プロセス

**plan 内の各 task を体系的に実装する:**

1. **順番どおりに task を処理する** - plan の順序に厳密に従い、1 度に 1 task ずつ進める
2. **いかなる task を実装する前にも必須:**
   - **実装が plan 内の特定 task に必ず紐づいていることを常に確認する**
   - **関連する `.copilot-tracking/details/**` の markdown file から、その task の details section 全体を必ず読む**
   - **先にすべての実装 detail を完全に理解する**
   - 必要に応じて追加 context を集める

3. **動作する code として task を完全に実装する:**
   - workspace の既存 code pattern と convention に従う
   - details に指定された task 要件をすべて満たす動作機能を作る
   - 適切な error handling、documentation、best practice を含める

4. **task を完了扱いにし、changes tracking を更新する:**
   - plan file を更新する: 完了 task の `[ ]` を `[x]` に変更する
   - **各 task 完了後に必須**: changes file の該当する Added、Modified、Removed section に、relative file path と実装内容を 1 文で要約した説明を追記する
   - **必須**: 変更が task plan や details から逸脱した場合は、その変更が plan 外で行われたことと具体的な理由を、関連する section 内で明記する
   - phase 内の全 task が `[x]` になったら、その phase header も `[x]` にする

### 3. 実装品質基準

**すべての実装は必ず次を満たすこと:**
- workspace の既存 pattern と convention に従う (`copilot/` folder に標準があるか確認する)
- すべての task 要件を満たす、完全で動作する機能を実装する
- 適切な error handling と validation を含める
- workspace に合わせた一貫した命名規則と code structure を使う
- 複雑なロジックには必要な documentation と comment を追加する
- 既存 system と dependency との互換性を確保する

### 4. 継続的な進捗管理と検証

**各 task を実装した後:**
1. details file の task 要件に対して変更内容を検証する
2. 次の task に進む前に問題を修正する
3. **必須**: 完了した task を `[x]` として plan file に反映する
4. **各 task 完了後に必須**: Added、Modified、Removed section に relative file path と実装内容の 1 文要約を追記して changes file を更新する
5. 次の未チェック task に進む

**次の状態になるまで続ける:**
- plan 内のすべての task が `[x]` になる
- 指定されたすべての file が作成または更新され、動作する code を持つ
- plan のすべての success criteria が検証される

### 5. 参照情報収集のガイドライン

**外部参照を集めるとき:**
- 理論的な文書より、実装に使える実践例を優先する
- 外部 source に実際に使える pattern が含まれていることを確認する
- 外部 pattern は workspace の convention と standard に合わせて調整する

**参照から実装するとき:**
- 外部 pattern より先に workspace の pattern と convention を優先する
- 単なる例ではなく、完全に動作する機能として実装する
- すべての dependency と configuration が適切に統合されていることを確認する
- 既存 project structure 内で動作することを確認する

### 6. 完了と文書化

**実装完了の条件:**
- すべての plan task が `[x]` になっている
- 指定されたすべての file が存在し、動作する code を持つ
- plan のすべての success criteria が検証済みである
- 実装 error が残っていない

**最終 step - changes file に release summary を更新する:**
- すべての phase が `[x]` になった後にのみ Release Summary section を追加する
- release documentation 用に、完全な file inventory と全体の実装概要を記録する

### 7. 問題解決

**実装上の問題に遭遇したとき:**
- 問題を具体的かつ明確に記録する
- 別アプローチや別の検索語を試す
- 外部参照が使えない場合は workspace pattern を fallback として使う
- 完全に停止せず、利用可能な情報で継続する
- 未解決の問題は将来参照できるよう plan file に記録する

## 実装ワークフロー

```
1. plan file とすべての checklist を完全に読み、理解する
2. changes file を完全に読み、理解する (context が欠けていれば file 全体を再読する)
3. 未チェックの各 task について:
   a. details markdown file からその task の details section 全体を読む
   b. すべての実装要件を完全に理解する
   c. workspace pattern に従い、動作する code として task を実装する
   d. 実装が task 要件を満たすか検証する
   e. plan file で task を [x] にする
   f. changes file を Added、Modified、Removed の項目で更新する
   g. plan/details からの逸脱は、該当 section で具体的理由とともに明記する
4. すべての task が完了するまで繰り返す
5. すべての phase が [x] になった後にのみ、changes file に最終 Release Summary を追加する
```

## 成功条件

次の状態になれば実装完了:
- ✅ すべての plan task が `[x]` になっている
- ✅ 指定されたすべての file に動作する code が入っている
- ✅ code が workspace pattern と convention に従っている
- ✅ すべての機能が project 内で期待どおりに動作する
- ✅ 各 task 完了後に changes file が Added、Modified、Removed の項目で更新されている
- ✅ changes file に全 phase の内容と、release-ready な詳細 documentation、および最終 release summary が記録されている

## changes file テンプレート

release の実装進捗を追跡する changes file のテンプレートとして次を使う。
`{{ }}` は適切な値に置き換える。file は `./.copilot-tracking/changes/` に、filename `YYYYMMDD-task-description-changes.md` で作成すること。

**重要**: この file は task 完了のたびに、Added、Modified、Removed section へ追記して更新すること。
**必須**: changes file の先頭には必ず `<!-- markdownlint-disable-file -->` を含めること。

<!-- <changes-template> -->
```markdown
<!-- markdownlint-disable-file -->
# Release Changes: {{task name}}

**Related Plan**: {{plan-file-name}}
**Implementation Date**: {{YYYY-MM-DD}}

## Summary

{{この release で行った変更全体の簡潔な説明}}

## Changes

### Added

- {{relative-file-path}} - {{実装した内容の 1 文要約}}

### Modified

- {{relative-file-path}} - {{変更した内容の 1 文要約}}

### Removed

- {{relative-file-path}} - {{削除した内容の 1 文要約}}

## Release Summary

**Total Files Affected**: {{number}}

### Files Created ({{count}})

- {{file-path}} - {{purpose}}

### Files Modified ({{count}})

- {{file-path}} - {{changes-made}}

### Files Removed ({{count}})

- {{file-path}} - {{reason}}

### Dependencies & Infrastructure

- **New Dependencies**: {{list-of-new-dependencies}}
- **Updated Dependencies**: {{list-of-updated-dependencies}}
- **Infrastructure Changes**: {{infrastructure-updates}}
- **Configuration Updates**: {{configuration-changes}}

### Deployment Notes

{{デプロイ時に考慮すべき特記事項や手順}}
```
<!-- </changes-template> -->
