# Cost Estimator Reference

Azure の単価を月額および年額のコスト見積もりに変換するための計算式とパターン。

## 標準的な時間ベースの計算

### 1 か月あたりの時間数

Azure では、標準的な課金期間として **730 時間/月** を使用します（365 日 × 24 時間 / 12 か月）。

```
Monthly Cost = Unit Price per Hour × 730
Annual Cost  = Monthly Cost × 12
```

### よく使う乗数

| 期間 | 時間 | 計算 |
|--------|-------|-------------|
| 1 時間 | 1 | 単価 |
| 1 日 | 24 | 単価 × 24 |
| 1 週間 | 168 | 単価 × 168 |
| 1 か月 | 730 | 単価 × 730 |
| 1 年 | 8,760 | 単価 × 8,760 |

## サービス別の計算式

### Virtual Machines (Compute)

```
Monthly Cost = hourly price × 730
```

営業時間のみ稼働する VM の場合（8 時間/日、22 日/月）:
```
Monthly Cost = hourly price × 176
```

### Azure Functions

```
Execution Cost = price per execution × number of executions
Compute Cost   = price per GB-s × (memory in GB × execution time in seconds × number of executions)
Total Monthly  = Execution Cost + Compute Cost
```

無料枠: 月あたり 100 万実行および 400,000 GB-s。

### Azure Blob Storage

```
Storage Cost   = price per GB × storage in GB
Transaction Cost = price per 10,000 ops × (operations / 10,000)
Egress Cost    = price per GB × egress in GB
Total Monthly  = Storage Cost + Transaction Cost + Egress Cost
```

### Azure Cosmos DB

#### プロビジョニング済みスループット
```
Monthly Cost = (RU/s / 100) × price per 100 RU/s × 730
```

#### サーバーレス
```
Monthly Cost = (total RUs consumed / 1,000,000) × price per 1M RUs
```

### Azure SQL Database

#### DTU モデル
```
Monthly Cost = price per DTU × DTUs × 730
```

#### vCore モデル
```
Monthly Cost = vCore price × vCores × 730  +  storage price per GB × storage GB
```

### Azure Kubernetes Service (AKS)

```
Monthly Cost = node VM price × 730 × number of nodes
```

標準 tier ではコントロール プレーンは無料です。

### Azure App Service

```
Monthly Cost = plan price × 730 (for hourly-priced plans)
```

または、固定 tier プランの場合は月額固定料金。

### Azure OpenAI

```
Monthly Cost = (input tokens / 1000) × input price per 1K tokens
             + (output tokens / 1000) × output price per 1K tokens
```

## 予約価格と従量課金の比較

価格オプションを提示する際は、必ず比較を示します:

```
| Pricing Model | Monthly Cost | Annual Cost | Savings vs. PAYG |
|---------------|-------------|-------------|------------------|
| Pay-As-You-Go | $X | $Y | — |
| 1-Year Reserved | $A | $B | Z% |
| 3-Year Reserved | $C | $D | W% |
| Savings Plan (1yr) | $E | $F | V% |
| Savings Plan (3yr) | $G | $H | U% |
| Spot (if available) | $I | N/A | T% |
```

節約率の計算式:
```
Savings % = ((PAYG Price - Reserved Price) / PAYG Price) × 100
```

## コスト集計テーブルのテンプレート

結果は常にこの形式で提示します:

```markdown
| Service | SKU | Region | Unit Price | Unit | Monthly Est. | Annual Est. |
|---------|-----|--------|-----------|------|-------------|-------------|
| Virtual Machines | Standard_D4s_v5 | East US | $0.192/hr | 1 Hour | $140.16 | $1,681.92 |
```

## ヒント

- 見積もり前に必ず **利用パターン** を明確にする（24/7、営業時間内のみ、断続的など）。
- **ストレージ** では、想定データ量とアクセス パターンを確認する。
- **データベース** では、必要なスループット（RU/s、DTU、または vCore）を確認する。
- **サーバーレス** サービスでは、想定呼び出し回数と実行時間を確認する。
- 表示用の数値は小数点以下 2 桁に丸める。
- 特に指定がない限り、価格は **USD** である点に注意する。

