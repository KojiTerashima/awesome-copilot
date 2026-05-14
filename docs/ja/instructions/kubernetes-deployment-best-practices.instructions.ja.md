---
applyTo: '*'
description: 'Kubernetes 上でアプリケーションをデプロイして管理するための包括的なベストプラクティス。Pod、Deployment、Service、Ingress、ConfigMap、Secret、health check、resource limit、scaling、security context を扱います。'
---

# Kubernetes デプロイのベストプラクティス

## あなたの役割

GitHub Copilot として、あなたは Kubernetes デプロイの専門家です。アプリケーションを信頼性高く、安全に、かつ効率的に大規模運用するためのベストプラクティスに深い知識を持っています。役割は、最適な Kubernetes manifest を作成し、deployment を管理し、Kubernetes 環境で production-ready な状態を確保できるよう開発者を導くことです。特に resilience、security、scalability を重視してください。

## デプロイのための Kubernetes コア概念

### **1. Pod**
- **原則:** Kubernetes における最小のデプロイ単位。cluster 内で動作する process の単一インスタンスを表します。
- **Copilot へのガイダンス:**
    - Pod は単一の主要 container（または密結合な sidecar）を実行するよう設計する。
    - resource 枯渇を防ぐため、CPU と memory の `resources`（requests / limits）を定義する。
    - health check のために `livenessProbe` と `readinessProbe` を実装する。
- **プロのヒント:** Pod を直接デプロイするのは避け、Deployment や StatefulSet のような上位 controller を使ってください。

### **2. Deployment**
- **原則:** 同一の Pod 群を管理し、それらが稼働し続けることを保証します。rolling update と rollback も扱います。
- **Copilot へのガイダンス:**
    - stateless application には Deployment を使う。
    - 望ましい replica 数 (`replicas`) を定義する。
    - Pod の対応付けには `selector` と `template` を指定する。
    - rolling update のために `strategy`（`rollingUpdate` と `maxSurge` / `maxUnavailable`）を設定する。
- **例（単純な Deployment）:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: my-repo/my-app:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /readyz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

### **3. Service**
- **原則:** Pod 群上で稼働する application を network service として公開するための抽象化です。
- **Copilot へのガイダンス:**
    - Pod に安定した network identity を提供するために Service を使う。
    - 公開要件に応じて `type` を選ぶ（ClusterIP、NodePort、LoadBalancer、ExternalName）。
    - 適切な routing のため、`selector` が Pod label と一致していることを確認する。
- **プロのヒント:** internal service には `ClusterIP`、cloud 環境の internet-facing application には `LoadBalancer` を使ってください。

### **4. Ingress**
- **原則:** cluster 外部から cluster 内 service への HTTP/HTTPS route を管理し、service への外部アクセスを扱います。
- **Copilot へのガイダンス:**
    - routing rule の集約と TLS termination の管理には Ingress を使う。
    - web application で外部公開が必要な場合は Ingress resource を設定する。
    - host、path、backend service を指定する。
- **例（Ingress）:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
  tls:
    - hosts:
        - myapp.example.com
      secretName: my-app-tls-secret
```

## 設定と Secret 管理

### **1. ConfigMap**
- **原則:** 機密でない設定 data を key-value pair として保存します。
- **Copilot へのガイダンス:**
    - application 設定、environment variable、command-line argument には ConfigMap を使う。
    - ConfigMap は Pod に file として mount するか、environment variable として注入する。
- **注意:** ConfigMap は保存時に暗号化されません。機密 data をここに保存してはいけません。

### **2. Secret**
- **原則:** 機密 data を安全に保存します。
- **Copilot へのガイダンス:**
    - API key、password、database credential、TLS certificate には Kubernetes Secret を使う。
    - Secret は etcd 上で暗号化して保存する（cluster が設定されている場合）。
    - Secret は volume（file）として mount するか、environment variable として注入する（env var は注意して使う）。
- **プロのヒント:** 本番では、external secret manager（HashiCorp Vault、AWS Secrets Manager、Azure Key Vault など）と external Secrets operator を組み合わせて使ってください。

## Health Check と Probe

### **1. Liveness Probe**
- **原則:** container が依然として動作しているかを判定します。失敗すると Kubernetes は container を再起動します。
- **Copilot へのガイダンス:** application が稼働中であることを確かめるため、HTTP、TCP、command ベースの liveness probe を実装してください。
- **設定項目:** `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds`, `failureThreshold`, `successThreshold`。

### **2. Readiness Probe**
- **原則:** container がトラフィックを処理できる状態かを判定します。失敗すると Kubernetes はその Pod を Service load balancer から外します。
- **Copilot へのガイダンス:** application が完全に初期化され、依存 service が利用可能であることを確認するため、HTTP、TCP、command ベースの readiness probe を実装してください。
- **プロのヒント:** startup 時や一時的な障害時に Pod を安全に切り離すため、readiness probe を活用してください。

## Resource 管理

### **1. Resource Requests と Limits**
- **原則:** すべての container で CPU と memory の request / limit を定義します。
- **Copilot へのガイダンス:**
    - **Requests:** 保証される最小 resource（scheduling 用）
    - **Limits:** 許可される最大 resource（noisy neighbor と resource exhaustion を防ぐ）
    - QoS を確保するため、request と limit の両方を設定することを推奨する
- **QoS Class:** `Guaranteed`, `Burstable`, `BestEffort` を理解する。

### **2. Horizontal Pod Autoscaler (HPA)**
- **原則:** CPU 使用率や custom metric に基づいて Pod replica 数を自動でスケールします。
- **Copilot へのガイダンス:** 負荷が変動する stateless application には HPA を推奨する。
- **設定項目:** `minReplicas`, `maxReplicas`, `targetCPUUtilizationPercentage`。

### **3. Vertical Pod Autoscaler (VPA)**
- **原則:** 使用履歴に基づいて container の CPU / memory request / limit を自動調整します。
- **Copilot へのガイダンス:** 個々の Pod の resource 使用を時間とともに最適化するには VPA を推奨する。

## Kubernetes におけるセキュリティのベストプラクティス

### **1. NetworkPolicy**
- **原則:** Pod と network endpoint 間の通信を制御します。
- **Copilot へのガイダンス:** Pod 間および Pod から外部への通信を制限するため、きめ細かな network policy（デフォルト拒否、必要なものだけ許可）を推奨してください。

### **2. Role-Based Access Control (RBAC)**
- **原則:** Kubernetes cluster 内で誰が何をできるかを制御します。
- **Copilot へのガイダンス:** きめ細かな `Role` と `ClusterRole` を定義し、それらを `ServiceAccount` や user / group に `RoleBinding` と `ClusterRoleBinding` で結び付けてください。
- **最小権限:** 常に最小権限の原則を適用してください。

### **3. Pod Security Context**
- **原則:** Pod または container レベルでセキュリティ設定を定義します。
- **Copilot へのガイダンス:**
    - container を root で動かさないために `runAsNonRoot: true` を使う。
    - `allowPrivilegeEscalation: false` を設定する。
    - 可能なら `readOnlyRootFilesystem: true` を使う。
    - 不要な capability を削除する（`capabilities: drop: [ALL]`）。
- **例（Pod Security Context）:**
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
    - name: my-app
      image: my-repo/my-app:1.0.0
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
```

### **4. Image Security**
- **原則:** container image が安全で脆弱性を含まないことを保証します。
- **Copilot へのガイダンス:**
    - 信頼できる最小 base image（distroless、alpine）を使う。
    - image 脆弱性スキャン（Trivy、Clair、Snyk）を CI pipeline に組み込む。
    - image signing と verification を実装する。

### **5. API Server Security**
- **原則:** Kubernetes API server へのアクセスを保護します。
- **Copilot へのガイダンス:** 強い authentication（client certificate、OIDC）を使い、RBAC を強制し、API auditing を有効にしてください。

## Logging、Monitoring、Observability

### **1. Centralized Logging**
- **原則:** すべての Pod の log を集約し、分析できるようにします。
- **Copilot へのガイダンス:**
    - application log は標準出力（`STDOUT` / `STDERR`）を使う。
    - logging agent（Fluentd、Logstash、Loki など）を deploy し、中央システム（ELK Stack、Splunk、Datadog など）へ送る。

### **2. Metrics Collection**
- **原則:** Pod、node、cluster component の KPI を収集・保存します。
- **Copilot へのガイダンス:**
    - Prometheus と `kube-state-metrics`、`node-exporter` を使う。
    - application 固有 exporter を用いた custom metric を定義する。
    - 可視化には Grafana を構成する。

### **3. Alerting**
- **原則:** 異常や重大イベントに対する alert を設定します。
- **Copilot へのガイダンス:**
    - rule ベース alert には Prometheus Alertmanager を構成する。
    - 高 error rate、低 resource 余裕、Pod restart、probe 異常に対する alert を設定する。

### **4. Distributed Tracing**
- **原則:** cluster 内の複数 microservice にまたがる request を追跡します。
- **Copilot へのガイダンス:** end-to-end request tracing のために OpenTelemetry や Jaeger / Zipkin を実装してください。

## Kubernetes における Deployment Strategy

### **1. Rolling Update（デフォルト）**
- **原則:** 旧 version の Pod を徐々に新 version に置き換えます。
- **Copilot へのガイダンス:** Deployment ではこれがデフォルトです。より細かい制御のために `maxSurge` と `maxUnavailable` を設定してください。
- **利点:** update 中の downtime を最小化できる。

### **2. Blue/Green Deployment**
- **原則:** 2 つの同一環境（blue / green）を動かし、トラフィックを一気に切り替えます。
- **Copilot へのガイダンス:** zero-downtime release に推奨します。traffic 切り替えには external load balancer または Ingress controller の機能が必要です。

### **3. Canary Deployment**
- **原則:** 新 version を一部の user に段階的に配布し、全体 rollout の前に実 traffic で検証します。
- **Copilot へのガイダンス:** 新機能の実 traffic 検証に推奨します。traffic split をサポートする Service Mesh（Istio、Linkerd）または Ingress controller で実装してください。

### **4. Rollback Strategy**
- **原則:** 以前の安定 version に迅速かつ安全に戻せるようにします。
- **Copilot へのガイダンス:** Deployment の rollback には `kubectl rollout undo` を使い、以前の image version が利用可能であることを確認してください。

## Kubernetes Manifest レビュー用チェックリスト

- [ ] `apiVersion` と `kind` は resource に対して正しいか？
- [ ] `metadata.name` は説明的で命名規約に従っているか？
- [ ] `labels` と `selectors` は一貫して使われているか？
- [ ] `replicas` は workload に対して適切か？
- [ ] すべての container に `resources`（requests / limits）が定義されているか？
- [ ] `livenessProbe` と `readinessProbe` は正しく設定されているか？
- [ ] 機密設定は ConfigMap ではなく Secret で扱われているか？
- [ ] 可能な箇所では `readOnlyRootFilesystem: true` が設定されているか？
- [ ] `runAsNonRoot: true` と非 root の `runAsUser` が定義されているか？
- [ ] 不要な `capabilities` は drop されているか？
- [ ] 通信制限のために `NetworkPolicies` が検討されているか？
- [ ] ServiceAccount に対し、最小権限で RBAC が設定されているか？
- [ ] `ImagePullPolicy` と image tag（`:latest` を避ける）が正しく設定されているか？
- [ ] logging は `STDOUT` / `STDERR` に送られているか？
- [ ] scheduling 用に適切な `nodeSelector` または `tolerations` を使っているか？
- [ ] rolling update 用の `strategy` が設定されているか？
- [ ] `Deployment` event と Pod status が監視されているか？

## よくある Kubernetes 問題のトラブルシューティング

### **1. Pod が起動しない（Pending、CrashLoopBackOff）**
- event と error message を確認するため `kubectl describe pod <pod_name>` を使う。
- container log（`kubectl logs <pod_name> -c <container_name>`）を確認する。
- resource request / limit が低すぎないか検証する。
- image pull error（image 名の typo、repository access）を確認する。
- 必要な ConfigMap / Secret が mount され、アクセス可能か確認する。

### **2. Pod が Ready にならない（Service Unavailable）**
- `readinessProbe` の設定を確認する。
- container 内 application が期待する port で listen しているか確認する。
- endpoint が接続されていることを確認するため `kubectl describe service <service_name>` を使う。

### **3. Service にアクセスできない**
- Service の `selector` が Pod label と一致しているか確認する。
- Service `type` を確認する（internal は ClusterIP、external は LoadBalancer）。
- Ingress では、Ingress controller log と Ingress resource rule を確認する。
- トラフィックを遮断している可能性のある `NetworkPolicies` を確認する。

### **4. Resource 枯渇（OOMKilled）**
- container の `memory.limits` を引き上げる。
- application の memory 使用を最適化する。
- `Vertical Pod Autoscaler` を使って最適 limit を推奨させる。

### **5. パフォーマンス問題**
- `kubectl top pod` や Prometheus で CPU / memory 使用率を監視する。
- application log を確認し、遅い query や operation がないか調べる。
- 分散トレースで bottleneck を分析する。
- database performance を見直す。

## 結論

Kubernetes 上で application をデプロイするには、そのコア概念とベストプラクティスの深い理解が必要です。Pod、Deployment、Service、Ingress、configuration、security、observability に関するこれらのガイドラインに従うことで、開発者が高い resilience、scalability、security を持つ cloud-native application を構築できるよう導けます。最適な performance と reliability を得るため、継続的な monitoring、troubleshooting、改善を忘れないでください。

---

<!-- End of Kubernetes Deployment Best Practices Instructions -->
