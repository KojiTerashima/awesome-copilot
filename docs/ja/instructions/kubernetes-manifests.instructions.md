---
applyTo: 'k8s/**/*.yaml,k8s/**/*.yml,manifests/**/*.yaml,manifests/**/*.yml,deploy/**/*.yaml,deploy/**/*.yml,charts/**/templates/**/*.yaml,charts/**/templates/**/*.yml'
description: 'ラベル付け規約、security context、pod security、resource management、probe、validation command を含む Kubernetes YAML manifest のベストプラクティス'
---

# Kubernetes Manifest の指示

## あなたの役割

一貫したラベル付け、適切な resource management、包括的な health check を備えた、セキュリティ・信頼性・運用性を重視する production-ready な Kubernetes manifest を作成してください。

## ラベル付け規約

**必須ラベル**（Kubernetes 推奨）:
- `app.kubernetes.io/name`: アプリケーション名
- `app.kubernetes.io/instance`: インスタンス識別子
- `app.kubernetes.io/version`: バージョン
- `app.kubernetes.io/component`: コンポーネントの役割
- `app.kubernetes.io/part-of`: アプリケーショングループ
- `app.kubernetes.io/managed-by`: 管理ツール

**追加ラベル**:
- `environment`: 環境名
- `team`: 所有チーム
- `cost-center`: 請求用

**有用な annotation**:
- ドキュメントと所有者情報
- Monitoring: `prometheus.io/scrape`, `prometheus.io/port`, `prometheus.io/path`
- 変更追跡: git commit、deployment 日時

## SecurityContext のデフォルト

**Pod レベル**:
- `runAsNonRoot: true`
- `runAsUser` と `runAsGroup`: 具体的な ID を指定
- `fsGroup`: ファイルシステムグループ
- `seccompProfile.type: RuntimeDefault`

**Container レベル**:
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`（書き込みが必要なディレクトリは tmpfs mount を併用）
- `capabilities.drop: [ALL]`（必要なものだけを追加）

## Pod Security Standards

Pod Security Admission を使用してください:
- **Restricted**（本番推奨）: セキュリティ強化を強制
- **Baseline**: 最低限のセキュリティ要件
- namespace レベルで適用

## Resource Requests と Limits

**常に定義するもの**:
- Requests: 保証される最小値（scheduling 用）
- Limits: 許可される最大値（枯渇防止）

**QoS Class**:
- **Guaranteed**: requests == limits（重要なアプリ向け）
- **Burstable**: requests < limits（柔軟な resource use）
- **BestEffort**: resource 未定義（本番では避ける）

## Health Probe

**Liveness**: 異常な container を再起動する
**Readiness**: トラフィックのルーティングを制御する
**Startup**: 起動の遅いアプリケーションを保護する

それぞれに対して、適切な delay、period、timeout、threshold を設定してください。

## Rollout Strategy

**Deployment Strategy**:
- `RollingUpdate` と `maxSurge`, `maxUnavailable`
- downtime を避けるには `maxUnavailable: 0` を設定

**高可用性**:
- 最低 2〜3 replica
- Pod Disruption Budget (PDB)
- anti-affinity rule（node / zone を分散）
- 変動負荷向けに Horizontal Pod Autoscaler (HPA)

## 検証コマンド

**デプロイ前**:
- `kubectl apply --dry-run=client -f manifest.yaml`
- `kubectl apply --dry-run=server -f manifest.yaml`
- `kubeconform -strict manifest.yaml`（schema validation）
- `helm template ./chart | kubeconform -strict`（Helm 用）

**ポリシー検証**:
- OPA Conftest、Kyverno、Datree

## Rollout と Rollback

**デプロイ**:
- `kubectl apply -f manifest.yaml`
- `kubectl rollout status deployment/NAME`

**ロールバック**:
- `kubectl rollout undo deployment/NAME`
- `kubectl rollout undo deployment/NAME --to-revision=N`
- `kubectl rollout history deployment/NAME`

**再起動**:
- `kubectl rollout restart deployment/NAME`

## Manifest チェックリスト

- [ ] Labels: 標準ラベルが適用されている
- [ ] Annotations: ドキュメント情報と monitoring 設定がある
- [ ] Security: runAsNonRoot、readOnlyRootFilesystem、capabilities drop が設定されている
- [ ] Resources: requests と limits が定義されている
- [ ] Probes: liveness、readiness、startup が設定されている
- [ ] Images: 具体的な tag を使っている（`:latest` を使わない）
- [ ] Replicas: 本番向けに最低 2〜3 が設定されている
- [ ] Strategy: 適切な surge/unavailable で RollingUpdate が設定されている
- [ ] PDB: 本番向けに定義されている
- [ ] Anti-affinity: HA のために設定されている
- [ ] Graceful shutdown: terminationGracePeriodSeconds が設定されている
- [ ] Validation: dry-run と kubeconform に通っている
- [ ] Secrets: ConfigMaps ではなく Secrets resource を使っている
- [ ] NetworkPolicy: 最小権限アクセスになっている（必要な場合）

## ベストプラクティス要約

1. 標準ラベルと annotation を使う
2. 常に non-root で実行し、不要な capability を落とす
3. resource requests と limits を定義する
4. 3 種類すべての probe を実装する
5. image tag は具体的なバージョンに固定する
6. HA のために anti-affinity を設定する
7. Pod Disruption Budget を設定する
8. unavailability を 0 にした rolling update を使う
9. apply 前に manifest を検証する
10. 可能なら read-only root filesystem を有効にする
