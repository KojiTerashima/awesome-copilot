---
name: acreadiness-policy
description: 'ユーザーが AgentRC ポリシーを選択、作成、または適用できるようにします。ポリシーは、無関係なチェックの無効化、影響/レベルの上書き、合格率しきい値の設定、またはチームの上書きによる組織のベースラインの連鎖によって、準備状況スコアをカスタマイズします。ユーザーが厳密モード、AI のみのスコアリング、カスタムの重み、CI ゲーティングについて質問する場合、または組織全体の標準化を希望する場合に使用します。'
argument-hint: "[show | new <name> | apply <path-or-pkg>] — e.g. /acreadiness-policy show, /acreadiness-policy new strict-frontend"
---

# /acreadiness-policy — AgentRC ポリシー

このスキルは、ユーザーが準備状況の **ポリシー**、**厳密モード**、**カスタム スコアリング**、**チェックの無効化**、**組織標準**、または **CI ゲーティング**について質問した場合に使用します。

ポリシーは、AgentRC が準備状況をスコアリングする方法をカスタマイズする 3 つのオプション セクション (`criteria`、`extras`、`thresholds`) を含む小さな JSON ファイルです。

## 組み込みの例

AgentRC には `examples/policies/` の 3 つのサンプル ポリシーが同梱されています。

|ポリシー |何をするのか |
|---|---|
| `strict.json` | 100% の合格率、主要基準への影響を高める |
| `ai-only.json` |すべてのリポジトリの健全性チェックを無効にし、AI ツールに重点を置きます |
| `repo-health-only.json` | AI チェックを無効にし、従来の品質に重点を置く |

カスタム ポリシーを作成する前の開始点としてこれらを推奨します。

## ポリシースキーマ
```jsonc
{
  "name": "my-policy",
  "criteria": {
    "disable":  ["env-example", "observability", "dependabot"],
    "override": {
      "readme":      { "impact": "high", "level": 2 },
      "lint-config": { "title": "Linter required" }
    }
  },
  "extras": {
    "disable": ["pre-commit"]
  },
  "thresholds": {
    "passRate": 0.9
  }
}
```

### 衝撃荷重

|影響 |重量 |
|---|---|
|クリティカル | 5 |
|高い | 4 |
|中 | 3 |
|低い | 2 |
|情報 | 0 |

@@コード0@@。グレード: **A** ≥ 0.9、**B** ≥ 0.8、**C** ≥ 0.7、**D** ≥ 0.6、**F** < 0.6。

## サブコマンド

### @@コード0@@
現在有効なポリシーをリストします (`agentrc.config.json` `policies` 配列から、またはなし)。

### @@コード0@@
適切なデフォルトを使用して `policies/<name>.json` を足場にします。ユーザーに次の手順を説明します。
1. **無効にするもの** — スタックに無関係なピラーまたはエクストラ (例: 静的サイトの `observability` を無効にする)。
2. **上げる内容** — 必須のもの (例: `readme`、`codeowners`) については `impact` を `high` または `critical` にオーバーライドします。
3. **合格率のしきい値** - 一般的な組織ベースライン: `0.7` (寛容)、`0.85` (標準)、`1.0` (厳格)。
4. `agentrc.config.json` からポリシーを参照します。
   ```json
   { "policies": ["./policies/<name>.json"] }
   ```

### @@コード0@@
`agentrc readiness --json --policy <source>` を実行し、`assess` スキル / `ai-readiness-reporter` エージェントに引き渡してレポートを再レンダリングします。チェーンをサポートします。```bash
npx -y github:microsoft/agentrc readiness --json --policy ./org-baseline.json,./team-frontend.json
```

## CI ゲーティング

ポリシーを `--fail-level` と組み合わせて、CI の最低成熟度レベルを強制します。
```yaml
- run: npx -y github:microsoft/agentrc readiness --policy ./policies/strict.json --fail-level 3
```

## 高度な

JSON ポリシーはしきい値を無効化、上書き、設定できますが、**新しい条件を追加することはできません**。新しい検出ロジックについては、AgentRC の TypeScript プラグイン システム (`docs/dev/plugins.md`) をユーザーに示します。

## 運用ルール

- **サイレントにピラーを無効にしないでください。** ユーザーが `observability` を無効にしたい場合は、トレードオフを確認して説明してください。
- **無効化よりも `impact` のオーバーライドを優先します。** 無効化するとギャップが完全に非表示になります。オーバーライドすると、引き続きレポートに表示されます。
- **追加機能は有効のままにすることをお勧めします。** 追加料金はかかりません。スコアには影響しません。
- **階層化を提案する** — ほとんどの組織は、ベースライン ポリシーと `--policy a.json,b.json` でチェーンされたチームごとのオーバーライドを必要としています。