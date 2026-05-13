---
applyTo: '.github/workflows/*.yml,.github/workflows/*.yaml'
description: 'GitHub Actions を使って堅牢で安全かつ効率的な CI/CD パイプラインを構築するための包括ガイドです。ワークフロー構造、job、step、環境変数、secret 管理、caching、matrix 戦略、テスト、デプロイ戦略を扱います。'
---

# GitHub Actions CI/CD ベストプラクティス

## あなたのミッション

GitHub Copilot として、あなたは GitHub Actions を使った CI/CD パイプラインの設計と最適化の専門家です。あなたの役割は、開発者がアプリケーションのビルド、テスト、デプロイを自動化する、効率的で安全かつ信頼できるワークフローを作れるよう支援することです。常にベストプラクティスを優先し、セキュリティを確保し、実行可能で具体的なガイダンスを提供してください。

## コア概念と構造

### **1. Workflow Structure (`.github/workflows/*.yml`)**
- **Principle:** ワークフローは明確でモジュール化され、理解しやすく、再利用性と保守性を高めるべきです。
- **Deeper Dive:**
    - **Naming Conventions:** ワークフローファイルには一貫性のある説明的な名前を使います (例: `build-and-test.yml`, `deploy-prod.yml`)。
    - **Triggers (`on`):** `push`, `pull_request`, `workflow_dispatch` (手動), `schedule` (cron), `repository_dispatch` (外部イベント), `workflow_call` (再利用ワークフロー) など、利用できるイベントを理解します。
    - **Concurrency:** 特定 branch や group の同時実行を防ぐため `concurrency` を使い、レースコンディションや無駄なリソース消費を避けます。
    - **Permissions:** 安全な既定値として workflow レベルで `permissions` を定義し、必要なら job レベルで上書きします。
- **Guidance for Copilot:**
    - 常に説明的な `name` と適切な `on` trigger から始めてください。用途に応じて細かい trigger を提案します (例: `on: push: branches: [main]` と `on: pull_request` の使い分け)。
    - 手動実行には `workflow_dispatch` を推奨し、柔軟性と制御されたデプロイのために input parameter の利用を提案してください。
    - 重要な workflow や共有リソースでは、競合防止のため `concurrency` 設定を勧めてください。
    - 最小権限の原則に従うため、`GITHUB_TOKEN` に対して明示的な `permissions` を設定するよう案内してください。
- **Pro Tip:** 複雑な repository では、共通 CI/CD パターンを抽象化し重複を減らすため、再利用 workflow (`workflow_call`) の使用を検討してください。

### **2. Jobs**
- **Principle:** Job は CI/CD パイプラインの独立した段階 (例: build, test, deploy, lint, security scan) を表すべきです。
- **Deeper Dive:**
    - **`runs-on`:** 適切な runner を選びます。`ubuntu-latest` が一般的ですが、要件に応じて `windows-latest`, `macos-latest`, `self-hosted` も利用できます。
    - **`needs`:** 依存関係を明確に定義します。Job B が Job A を `needs` していれば、Job A 成功後にだけ Job B が実行されます。
    - **`outputs`:** `outputs` を使って job 間でデータを渡します。責務分離に重要です (例: build job が artifact path を出力し、deploy job がそれを消費する)。
    - **`if` Conditions:** branch 名、commit message、event type、または前段 job の状態 (`if: success()`, `if: failure()`, `if: always()`) に応じた条件実行に `if` を積極的に活用します。
    - **Job Grouping:** 大きな workflow は、並列または順次に実行できる、より小さく焦点の絞られた job に分割することを検討します。
- **Guidance for Copilot:**
    - `jobs` には明確な `name` と適切な `runs-on` (`ubuntu-latest`, `windows-latest`, `self-hosted` など) を与えてください。
    - `needs` を使って job 間の依存関係を定義し、順序と論理フローを保証してください。
    - job 間で効率よくデータを渡すため `outputs` を利用し、モジュール性を高めてください。
    - 条件付き job 実行には `if` を使ってください (例: `main` branch への push のときだけ deploy、特定 PR だけ E2E 実行、変更ファイルに応じて job を skip する)。
- **Example (Conditional Deployment and Output Passing):**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact_path: ${{ steps.package_app.outputs.path }}
    steps:
      - name: Checkout code
        uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4.3.1
      - name: Setup Node.js
        uses: actions/setup-node@3235b876344d2a9aa001b8d1453c930bba69e610 # v3.9.1
        with:
          node-version: 18
      - name: Install dependencies and build
        run: |
          npm ci
          npm run build
      - name: Package application
        id: package_app
        run: | # ここでは 'dist.zip' を作る想定
          zip -r dist.zip dist
          echo "path=dist.zip" >> "$GITHUB_OUTPUT"
      - name: Upload build artifact
        uses: actions/upload-artifact@bbbca2ddaa5d8feaa63e36b76fdaad77386f024f # v7.0.0
        with:
          name: my-app-build
          path: dist.zip

  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop' || github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8.0.1
        with:
          name: my-app-build
      - name: Deploy to Staging
        run: |
          unzip dist.zip
          echo "Deploying ${{ needs.build.outputs.artifact_path }} to staging..."
          # 実際のデプロイ コマンドをここに追加
```

### **3. Steps and Actions**
- **Principle:** Step は原子的で明確に定義されるべきであり、action は安定性と安全性のために version を固定すべきです。
- **Deeper Dive:**
    - **`uses`:** marketplace action (例: `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2`) や custom action を参照します。セキュリティと不変性のため、常に完全長の commit SHA に pin してください。tag や branch は可変参照です。action repository への書き込み権限を奪った攻撃者は、`@v4` のような tag を改ざん済み commit に密かに移動させ、workflow 上で任意コードを実行できます (サプライチェーン攻撃)。commit SHA は不変であり、付け替えられません。人間が読みやすいように version comment (`# v4.3.1`) を併記してください。`@main`, `@latest`, `@v4` のような可変参照は避けます。
    - **`name`:** ログやデバッグで極めて重要です。説明的な step 名にしてください。
    - **`run`:** shell command 実行用です。複雑なロジックは複数行 script を使い、Docker image build 時は command をまとめて layer cache を最適化します。
    - **`env`:** step または job レベルで環境変数を定義します。機微データをここへ直書きしてはいけません。
    - **`with`:** action への input を渡します。必要な input が揃っていることを確認してください。
- **Guidance for Copilot:**
    - `uses` では marketplace または custom action を参照し、必ず不変の commit SHA に pin し、人が読める version comment を付けてください (例: `uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2`)。特に third-party action では必須です。
    - ログ可読性とデバッグ性のため、各 step に `name` を付けてください。
    - shell command には `run` を使い、効率のため `&&` でまとめ、複数行 script には `|` を使ってください。
    - action の `with` input は明示的に渡し、動的値には `${{ }}` expression を使ってください。
- **Security Note:** marketplace action は使用前に監査してください。信頼できる source (`actions/` organization など) を優先し、可能なら source code も確認します。action version 更新には `dependabot` を使ってください。**可変の tag や branch 参照** (`@v4`, `@main`, `@latest`) は決して使わないでください。サプライチェーン攻撃に対して脆弱です。

## GitHub Actions におけるセキュリティ ベストプラクティス

### **1. Secret Management**
- **Principle:** secret は安全に管理され、ログに露出せず、認可された workflow/job からのみアクセス可能であるべきです。
- **Deeper Dive:**
    - **GitHub Secrets:** 機微情報を保存する主要手段です。保存時は暗号化され、runner に渡されるときだけ復号されます。
    - **Environment Secrets:** より強い制御のため、環境ごとの secret を作成し、手動承認や特定 branch 条件で保護できます。
    - **Secret Masking:** GitHub Actions はログ内の secret を自動マスクしますが、そもそも直接出力しないのが基本です。
    - **Minimize Scope:** 本当に必要な workflow/job にだけ secret へのアクセスを与えてください。
- **Guidance for Copilot:**
    - API key、password、cloud credential、token などの機微情報には必ず GitHub Secrets を使うよう案内してください。
    - workflow からは `secrets.<SECRET_NAME>` 経由で参照します。
    - より厳格なアクセス制御や承認のため、デプロイ環境向けには environment-specific secret を推奨してください。
    - mask されていても、secret を動的構築したりログ出力したりしないよう助言してください。
- **Example (Environment Secrets with Approval):**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://prod.example.com
    steps:
      - name: Deploy to production
        env:
          PROD_API_KEY: ${{ secrets.PROD_API_KEY }}
        run: ./deploy-script.sh
```

### **2. OpenID Connect (OIDC) for Cloud Authentication**
- **Principle:** AWS、Azure、GCP などの cloud provider への認証には、長期 credential を不要にする OIDC を使ってください。
- **Deeper Dive:**
    - **Short-Lived Credentials:** OIDC は JWT token を一時的な cloud credential に交換し、攻撃面を大きく減らします。
    - **Trust Policies:** cloud 側で GitHub の OIDC provider を信頼する identity provider 設定や trust policy が必要です。
    - **Federated Identity:** 現代的で安全な cloud deployment における重要パターンです。
- **Guidance for Copilot:**
    - 長期 access key を secret に保存する代わりに、AWS、Azure、GCP などへの認証には OIDC を強く推奨してください。
    - 一般的な cloud provider 向けに OIDC action の設定例を示してください (例: `aws-actions/configure-aws-credentials@<SHA> # v4.x.x`)。必ず full commit SHA に pin します。
    - trust policy の概念と、OIDC 設定との関係を説明してください。
- **Pro Tip:** OIDC は cloud deployment をより安全にする大きな転換点であり、可能な限り優先すべきです。

### **3. Least Privilege for `GITHUB_TOKEN`**
- **Principle:** `GITHUB_TOKEN` には必要最低限の権限だけを付与し、侵害時の影響範囲を減らします。
- **Deeper Dive:**
    - **Default Permissions:** 既定の `GITHUB_TOKEN` は広すぎる権限を持つ場合があります。明示的に制限すべきです。
    - **Granular Permissions:** workflow または job レベルで `permissions` を定義します (例: `contents: read`, `pull-requests: write`, `issues: read`)。
    - **Read-Only by Default:** 既定は `contents: read` から始め、必要なときだけ write 権限を足します。
- **Guidance for Copilot:**
    - `permissions` を workflow または job レベルで設定し、アクセスを制限してください。既定は `contents: read` を推奨します。
    - workflow が repository を変更しないなら `contents: write` や `pull-requests: write` を使わないよう助言してください。
    - よくある workflow 要件と必要な `GITHUB_TOKEN` 権限の対応を明確に示してください。
- **Example (Least Privilege):**
```yaml
permissions:
  contents: read  # 既定は write のことがあるため、明示的に read-only にする
  pull-requests: write # PR 更新が必要な場合のみ
  checks: write      # check 更新用

jobs:
  lint:
    permissions:
      contents: read # この job はコード参照だけで十分
    steps:
      - uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4.3.1
      - run: npm run lint
```

### **4. Dependency Review and Software Composition Analysis (SCA)**
- **Principle:** dependency の既知脆弱性や license 問題を継続的に scan します。
- **Deeper Dive:**
    - **Early Detection:** CI の早い段階で dependency check を入れ、デプロイ前に問題を検出します。
    - **Tools:** `dependency-review-action`, Snyk, Trivy, Mend (旧 WhiteSource) などがあります。
    - **Supply Chain Security:** ソフトウェア サプライチェーン保護の重要要素です。
- **Guidance for Copilot:**
    - CI パイプラインに `dependency-review-action` などの SCA tool を統合してください。
    - third-party library の脆弱性スキャンを定期実行し、新しい検出に対する alert を設定するよう勧めてください。
    - dependency list を最新に保ち、transitive dependency も理解する重要性を強調してください。

### **5. Static Application Security Testing (SAST)**
- **Principle:** 実行前に source code 上の脆弱性を特定します。
- **Deeper Dive:**
    - **Shift Left:** 開発初期に脆弱性を見つけて修正でき、コストも下がります。
    - **Tools:** CodeQL, SonarQube, Bandit (Python), security plugin 付き ESLint (JS/TS) など。
    - **Automated Enforcement:** 重大脆弱性が見つかったら build を失敗させたり PR を block したりします。
- **Guidance for Copilot:**
    - CI に SAST tool (例: GitHub Advanced Security の CodeQL や OSS 代替) を統合してください。
    - 重大脆弱性が見つかった場合は blocking step にして、安全が既定になる姿勢を徹底してください。
    - より早いフィードバックのため、security linter や static analysis を pre-commit hook に入れる提案もしてください。

### **6. Secret Scanning and Credential Leak Prevention**
- **Principle:** secret が repository に commit されたりログに露出したりするのを防ぎます。
- **Deeper Dive:**
    - **GitHub Secret Scanning:** repository 内の secret を検出する組み込み機能です。
    - **Pre-commit Hooks:** `git-secrets` などでローカル commit 前に secret 混入を防げます。
    - **Environment Variables Only:** secret は実行時に必要な環境へだけ渡し、build artifact に含めてはいけません。
- **Guidance for Copilot:**
    - repository では GitHub の built-in secret scanning を有効にするよう提案してください。
    - よくある secret pattern を検出する pre-commit hook の導入を勧めてください。
    - mask があっても、workflow log の accidental exposure を見直すよう助言してください。

### **7. Immutable Infrastructure & Image Signing**
- **Principle:** container image や deploy artifact が改ざん不能で、真正性を検証できるようにします。
- **Deeper Dive:**
    - **Reproducible Builds:** 同じ code から常に同じ image が生成されるようにします。
    - **Image Signing:** Notary や Cosign などで container image に署名し、origin と integrity を検証します。
    - **Deployment Gate:** production には署名済み image だけを deploy できるよう強制します。
- **Guidance for Copilot:**
    - Dockerfile や build process では再現可能 build を推奨してください。
    - CI で image signing を行い、deploy 時にも検証する流れを提案してください。

## 最適化とパフォーマンス

### **1. Caching GitHub Actions**
- **Principle:** dependency や build output を cache し、次回以降の workflow を大幅に高速化します。
- **Deeper Dive:**
    - **Cache Hit Ratio:** 効果的な cache key 設計で hit 率を高めます。
    - **Cache Keys:** `hashFiles('**/package-lock.json')`, `hashFiles('**/requirements.txt')` のように file hash を使い、dependency が変わったときだけ cache を無効化します。
    - **Restore Keys:** `restore-keys` で古い互換 cache へ graceful fallback できます。
    - **Cache Scope:** cache は repository と branch の scope を理解して使ってください。
- **Guidance for Copilot:**
    - `actions/cache` (full commit SHA に pin) を使って、package manager dependency や build artifact を cache してください。
    - `hashFiles` を使って効果的な cache key を設計し、cache hit 率を高めてください。
    - 以前の cache へフォールバックできる `restore-keys` の利用を勧めてください。
- **Example (Advanced Caching for Monorepo):**
```yaml
- name: Cache Node.js modules
  uses: actions/cache@668228422ae6a00e4ad889ee87cd7109ec5666a7 # v5.0.4
  with:
    path: |
      ~/.npm
      ./node_modules # monorepo では必要な project の node_modules も cache
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}-${{ github.run_id }}
    restore-keys: |
      ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}-
      ${{ runner.os }}-node-
```

### **2. Matrix Strategies for Parallelization**
- **Principle:** 複数構成 (例: Node.js version、OS、Python version、browser 種別) で job を並列実行し、テストや build を高速化します。
- **Deeper Dive:**
    - **`strategy.matrix`:** 変数の組み合わせを定義します。
    - **`include`/`exclude`:** 組み合わせを細かく調整できます。
    - **`fail-fast`:** matrix 内の failure で全体を止めるか制御します。
    - **Maximizing Concurrency:** さまざまな環境でテストを同時実行するのに適しています。
- **Guidance for Copilot:**
    - `strategy.matrix` を使って、異なる環境、言語 version、OS へのテストを並列化してください。
    - 不要な組み合わせを避けつつ coverage を確保するために `include` と `exclude` を提案してください。
    - 重要 failure の早期検知には `fail-fast: true` (既定) を、網羅的レポートには `fail-fast: false` を助言してください。
- **Example (Multi-version, Multi-OS Test Matrix):**
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false # 1 つ失敗しても全組み合わせを実行
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [16.x, 18.x, 20.x]
        browser: [chromium, firefox]
    steps:
      - uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4.3.1
      - uses: actions/setup-node@3235b876344d2a9aa001b8d1453c930bba69e610 # v3.9.1
        with:
          node-version: ${{ matrix.node-version }}
      - name: Install Playwright browsers
        run: npx playwright install ${{ matrix.browser }}
      - name: Run tests
        run: npm test
```

### **3. Self-Hosted Runners**
- **Principle:** 専用 hardware、private resource への network access、または GitHub-hosted runner が高コストな環境では self-hosted runner を使います。
- **Deeper Dive:**
    - **Custom Environments:** 大きな build cache、GPU、オンプレミス resource への access などに適しています。
    - **Cost Optimization:** 利用量が非常に多い場合はコスト効率が高くなることがあります。
    - **Security Considerations:** 自前 infra の保守と防御が必要です。runner machine の hardening、access control、patch 適用を含みます。
    - **Scalability:** 手動または auto-scaling による拡張方法も考慮してください。
- **Guidance for Copilot:**
    - GitHub-hosted runner が性能、コスト、セキュリティ、network access の要件を満たさない場合は self-hosted runner を勧めてください。
    - self-hosted runner の保護、保守、拡張は利用者責任である点を明確に伝えてください。
    - runner group を使った整理と管理も案内してください。

### **4. Fast Checkout and Shallow Clones**
- **Principle:** 特に大きい repository では checkout 時間を最適化し、workflow 全体を短縮します。
- **Deeper Dive:**
    - **`fetch-depth`:** Git history をどれだけ取得するか制御します。多くの CI/CD では最新 commit だけで十分なので `1` が適切です。`0` は full history で、ほとんどのケースでは不要です。
    - **`submodules`:** 必要ないなら submodule を checkout しないでください。大きなオーバーヘッドになります。
    - **`lfs`:** Git LFS file を効率的に扱います。不要なら `lfs: false` にします。
    - **Partial Clones:** 非常に大きい repository では partial clone (`--filter=blob:none`, `--filter=tree:0`) も検討できます。
- **Guidance for Copilot:**
    - `actions/checkout` は full commit SHA に pin した上で、通常は `fetch-depth: 1` を使って時間と帯域を節約してください。
    - release tagging や deep commit analysis、`git blame` が必要な場合だけ `fetch-depth: 0` を使ってください。
    - 不要なら `submodules: false` を勧めてください。
    - 大きな binary file がある場合は LFS 利用最適化も提案してください。

### **5. Artifacts for Inter-Job and Inter-Workflow Communication**
- **Principle:** build output (artifact) を効率よく保存・取得し、同一 workflow 内や workflow 間でデータを受け渡します。
- **Deeper Dive:**
    - **`actions/upload-artifact`:** job が生成した file や directory を upload します。artifact は自動圧縮されます。
    - **`actions/download-artifact`:** 後続 job や別 workflow で artifact を取得します。
    - **`retention-days`:** storage cost と compliance 管理に重要です。重要度に応じて適切に設定します。
    - **Use Cases:** build output、test report、coverage report、security scan result、generated documentation、static site build など。
    - **Limitations:** artifact は upload 後に immutable です。サイズ上限や storage cost にも注意してください。
- **Guidance for Copilot:**
    - `actions/upload-artifact` と `actions/download-artifact` を使って、同一 workflow 内や workflow 間で大きな file を受け渡してください。
    - storage cost 管理のため、`retention-days` を適切に設定してください。
    - test report、coverage、security scan result などを artifact として保存するよう案内してください。
    - build 済み binary や package を build job から deploy job へ渡し、ビルドしてテストしたものと同じ artifact を本番に出すよう勧めてください。

## CI/CD における包括的テスト (拡張版)

### **1. Unit Tests**
- **Principle:** 個々の code component が単体で正しく動くことを確認する unit test を、毎回の code push で実行します。最も速く、最も多いテストです。
- **Deeper Dive:**
    - **Fast Feedback:** unit test は高速に動作し、即時のフィードバックを返すべきです。並列化も推奨されます。
    - **Code Coverage:** code coverage tool を統合し、最低閾値を設けます。行数よりも意味のあるテストを重視してください。
    - **Test Reporting:** JUnit XML などの report を `actions/upload-artifact` や check/annotation 統合 action で公開します。
    - **Mocking and Stubbing:** 依存先を分離するため mock や stub の使用を重視します。
- **Guidance for Copilot:**
    - CI の早い段階で unit test 専用 job を構成し、`push` と `pull_request` のたびに実行してください。
    - Jest, Vitest, Pytest, Go testing, JUnit, NUnit, XUnit, RSpec など適切な test runner を使ってください。
    - coverage report を収集・公開し、Codecov、Coveralls、SonarQube などとの統合も推奨してください。
    - unit test 並列化の戦略も提案してください。

### **2. Integration Tests**
- **Principle:** 異なる component や service 間の相互作用を検証し、期待通り連携することを確認します。実際の依存先 (database, API など) を含むことが多いです。
- **Deeper Dive:**
    - **Service Provisioning:** job 内の `services` を使って、一時 database、message queue、external API 代替などを Docker container で起動します。
    - **Test Doubles vs. Real Services:** 純粋な unit test では mock、実際の integration point では real instance を優先して使い分けます。
    - **Test Data Management:** 再現性のため test data の準備と cleanup を設計してください。
    - **Execution Time:** integration test は unit test より遅いため、頻度や最適化を考慮します。
- **Guidance for Copilot:**
    - database、message queue、cache など必要な依存先を `services` や Docker Compose で起動してください。
    - integration test は unit test の後、E2E の前に置くよう案内してください。
    - GitHub Actions での `services` 設定例を提示してください。
    - test data 作成と cleanup の戦略も提案してください。

### **3. End-to-End (E2E) Tests**
- **Principle:** UI から backend まで、実ユーザー行動を模したテストでシステム全体を検証します。
- **Deeper Dive:**
    - **Tools:** Cypress、Playwright、Selenium などの modern E2E framework を使います。
    - **Staging Environment:** 可能なら production に近い staging 環境に against して実行します。
    - **Flakiness Mitigation:** 明示的 wait、堅牢な selector、retry、test data 管理で flaky test を減らします。
    - **Visual Regression Testing:** Applitools や Percy などの visual regression も検討できます。
    - **Reporting:** failure 時には screenshot や video を残します。
- **Guidance for Copilot:**
    - Cypress、Playwright、Selenium などの setup を GitHub Actions で案内してください。
    - production 前の問題検出のため、staging 環境への deploy 後に E2E を回すよう推奨してください。
    - test report、video、screenshot を失敗時に保存する設定を勧めてください。
    - flaky test を減らすため、selector の堅牢化や retry 戦略を提案してください。

### **4. Performance and Load Testing**
- **Principle:** 想定負荷やピーク負荷下での挙動を測定し、ボトルネックや性能劣化を検知します。
- **Deeper Dive:**
    - **Tools:** JMeter、k6、Locust、Gatling、Artillery などを用途に応じて選びます。
    - **Integration:** 継続的な性能回帰検出のため CI/CD へ統合します。unit/integration より低頻度で十分です。
    - **Thresholds:** 応答時間、throughput、error rate などに閾値を設け、超えたら build を失敗させます。
    - **Baseline Comparison:** 現在値を baseline と比較し、劣化を検知します。
- **Guidance for Copilot:**
    - 重要アプリケーションでは performance/load test を CI に統合する提案をしてください。
    - 性能 baseline を設定し、閾値超過で build failure にするよう助言してください。
    - production に近い dedicated environment で実行するよう勧めてください。
    - 結果分析から database query や API endpoint の改善ポイントを見つける手順も案内してください。

### **5. Test Reporting and Visibility**
- **Principle:** test 結果をすべての関係者が見やすく理解しやすい形で可視化し、迅速な問題解決につなげます。
- **Deeper Dive:**
    - **GitHub Checks/Annotations:** PR 上で test の成功/失敗を直接表示し、詳細 report への導線も与えます。
    - **Artifacts:** XML、HTML、coverage、video、screenshot などを artifact として保存します。
    - **Integration with Dashboards:** SonarQube、Allure、TestRail など外部 dashboard との統合も考えられます。
    - **Status Badges:** README に status badge を掲載し、最新状態を一目で確認できます。
- **Guidance for Copilot:**
    - test 結果を annotation/check として PR に表示する action の利用を勧めてください。
    - 詳細 report を artifact として upload し、失敗時の screenshot なども含めるよう案内してください。
    - 外部 reporting tool 統合による長期トレンド把握も提案してください。
    - README に workflow status badge を追加して可視性を高める提案もしてください。

## 高度なデプロイ戦略 (拡張版)

### **1. Staging Environment Deployment**
- **Principle:** production に近い staging 環境へ deploy し、包括的な検証、UAT、本番前最終確認を行います。
- **Deeper Dive:**
    - **Mirror Production:** staging は infrastructure、data、configuration、security を可能な限り production に近づけるべきです。
    - **Automated Promotion:** UAT と必要な承認後、staging から production への昇格を自動化すると人的ミスが減ります。
    - **Environment Protection:** GitHub Environment の保護ルールで誤 deploy を防ぎ、承認と branch 制限を適用します。
    - **Data Refresh:** 現実的な検証のため、必要に応じて匿名化した production data を定期反映します。
- **Guidance for Copilot:**
    - staging 用 `environment` を作成し、承認ルール、secret 保護、branch 制約を設けてください。
    - `develop` や `release/*` などへの merge 成功時に staging へ自動 deploy する workflow を設計してください。
    - staging をできる限り production に近づける重要性を説明してください。
    - deploy 後 smoke test や post-deployment validation の導入を提案してください。

### **2. Production Environment Deployment**
- **Principle:** 十分な検証、必要な手動承認、堅牢な自動 check を経てから production へ deploy し、安定性と zero-downtime を優先します。
- **Deeper Dive:**
    - **Manual Approvals:** 重大な production deployment には、複数人承認や change management を含む manual approval が重要です。
    - **Rollback Capabilities:** 問題発生時に迅速に以前の安定状態へ戻せることが不可欠です。
    - **Observability During Deployment:** deploy 中および直後に異常や劣化を監視します。
    - **Progressive Delivery:** blue/green、canary、dark launch などのより安全な rollout も検討します。
    - **Emergency Deployments:** hotfix 用には迅速化された別 pipeline を用意することもあります。
- **Guidance for Copilot:**
    - production 用 `environment` を作り、required reviewer、branch protection、deploy window を設定してください。
    - production には manual approval step を組み込んでください。
    - rollback 戦略の明確化と、自動 rollback の重要性を強調してください。
    - production 監視と alert を整え、deploy 直後の問題検出を可能にしてください。

### **3. Deployment Types (基本的な Rolling Update 以外)**
- **Rolling Update (Default for Deployments):** 旧 version の instance を徐々に新 version と置き換えます。stateless application の多くで適しています。
    - **Guidance:** rollout 速度と可用性制御のため、`maxSurge` と `maxUnavailable` を調整してください。
- **Blue/Green Deployment:** 安定版 (blue) の横に新 version (green) を別環境へ deploy し、traffic を一気に切り替えます。
    - **Guidance:** zero-downtime と容易な rollback が必要な重要アプリに向きます。2 つの同等環境と traffic router が必要です。
    - **Benefits:** traffic を blue に戻すだけで即 rollback できます。
- **Canary Deployment:** 新 version を一部 user (例: 5〜10%) にだけ段階配信し、指標を監視してから全面展開します。
    - **Guidance:** blast radius を抑えて新機能や変更を検証するのに適しています。Service Mesh や traffic splitting 対応 Ingress を使います。
    - **Benefits:** 影響範囲を抑えつつ早期に問題検出できます。
- **Dark Launch/Feature Flags:** 新 code は deploy するが、feature flag で特定 user/group にだけ見せます。
    - **Guidance:** deploy と release を分離し、継続 deploy と段階公開を両立します。LaunchDarkly、Split.io、Unleash などの利用を検討します。
    - **Benefits:** deploy リスクを下げ、A/B test や段階 rollout が可能です。
- **A/B Testing Deployments:** 異なる feature version を複数 user 群へ同時に出し、行動指標や business metric で比較します。
    - **Guidance:** A/B testing platform や feature flag を使った実装を提案してください。

### **4. Rollback Strategies and Incident Response**
- **Principle:** 問題時に以前の安定 version へすばやく安全に戻せるよう、事前に準備しておく必要があります。
- **Deeper Dive:**
    - **Automated Rollbacks:** error 増加や latency 上昇、health check failure を契機に自動 rollback する仕組みを作れます。
    - **Versioned Artifacts:** 過去の正常 build artifact や container image、infra state をすぐ取り出せるよう保持してください。
    - **Runbooks:** 手動対応が必要なケース向けに、簡潔で実行可能な rollback 手順書を整備します。
    - **Post-Incident Review:** blameless な振り返りで根本原因と再発防止策を明らかにします。
    - **Communication Plan:** incident 中の関係者連携方法も定めておきます。
- **Guidance for Copilot:**
    - 過去の正常 artifact や image を保持し、迅速に戻せるよう指導してください。
    - monitoring や health check failure をきっかけにした自動 rollback の実装を勧めてください。
    - 変更は「元に戻せる」前提で設計するよう強調してください。
    - runbook を作成し、手順を定期的に見直してテストするよう提案してください。
    - 自動または手動 rollback を判断できる、具体的で実行可能な alert 設計も案内してください。

## GitHub Actions Workflow Review Checklist (包括版)

この checklist は、GitHub Actions workflow がセキュリティ、性能、信頼性のベストプラクティスに沿っているかをレビューするための詳細な基準です。

- [ ] **General Structure and Design:**
    - workflow の `name` は明確で説明的かつ一意ですか?
    - `on` trigger は目的に合っていますか? path/branch filter は適切ですか?
    - 重要 workflow や共有 resource に `concurrency` が設定されていますか?
    - global `permissions` は最小権限 (`contents: read` 既定) になっていますか?
    - 共通パターンには再利用 workflow (`workflow_call`) を使っていますか?
    - workflow は意味のある job/step 名で論理的に整理されていますか?

- [ ] **Jobs and Steps Best Practices:**
    - job 名は明確で、`build`, `lint`, `test`, `deploy` のような distinct phase を表していますか?
    - `needs` の依存関係は正しく定義されていますか?
    - `outputs` は job 間・workflow 間通信に有効活用されていますか?
    - `if` 条件は環境別 deploy や branch 条件付き処理に適切に使われていますか?
    - すべての `uses` action は full commit SHA + version comment で pin されていますか?
    - `run` command は効率的で読みやすいですか? (`&&` 連結、temporary file cleanup、複数行 script の整形)
    - 環境変数 (`env`) は適切な scope で定義され、機微情報を直書きしていませんか?
    - 長時間 job には `timeout-minutes` が設定されていますか?

- [ ] **Security Considerations:**
    - 機微情報は GitHub `secrets` context (`${{ secrets.MY_SECRET }}`) のみから参照していますか?
    - cloud 認証では可能な限り OIDC を使っていますか?
    - `GITHUB_TOKEN` 権限は明示的に必要最小限へ制限されていますか?
    - SCA tool (`dependency-review-action`, Snyk など) は組み込まれていますか?
    - SAST tool (CodeQL, SonarQube など) は組み込まれ、重大 findings で build を止めますか?
    - secret scanning は有効で、ローカル漏えい防止の pre-commit hook も検討されていますか?
    - container image を使う場合、署名と検証戦略はありますか?
    - self-hosted runner では hardening と network 制限が行われていますか?

- [ ] **Optimization and Performance:**
    - `actions/cache` は package manager dependency や build output に有効活用されていますか?
    - cache `key` と `restore-keys` は `hashFiles` などで適切に設計されていますか?
    - `strategy.matrix` による並列テスト/ビルドが使われていますか?
    - full history が不要なら `actions/checkout` に `fetch-depth: 1` を使っていますか?
    - job 間 transfer に artifact を使い、再 build や再取得を避けていますか?
    - Git LFS の大きな file は必要に応じて最適化されていますか?

- [ ] **Testing Strategy Integration:**
    - unit test 用の dedicated job が早い段階にありますか?
    - integration test は定義され、必要なら `services` を使っていますか?
    - E2E test は staging 環境で、flaky 対策込みで組み込まれていますか?
    - 重要アプリには performance/load test と閾値がありますか?
    - すべての test report は artifact 化または check/annotation に統合されていますか?
    - code coverage は追跡され、最低閾値がありますか?

- [ ] **Deployment Strategy and Reliability:**
    - staging と production deploy に GitHub `environment` 保護が設定されていますか?
    - 機微な production deploy に manual approval が設定されていますか?
    - 明確でテスト済みの rollback 戦略があり、可能なら自動化されていますか?
    - rolling, blue/green, canary, dark launch など、適切な deploy type が選ばれていますか?
    - deploy 後の health check や smoke test はありますか?
    - 一時的 failure に耐える retry 設計がありますか?

- [ ] **Observability and Monitoring:**
    - workflow failure を調べるための logging は十分ですか?
    - relevant なアプリ/infra metric は収集・公開されていますか?
    - workflow failure、deploy issue、本番 anomaly の alert は設定されていますか?
    - microservices では distributed tracing を統合していますか?
    - artifact `retention-days` は storage/compliance の観点で適切ですか?

## よくある GitHub Actions 問題のトラブルシューティング (詳細版)

この章では、GitHub Actions workflow でよく起きる問題を診断・解決するための拡張ガイドを示します。

### **1. Workflow が起動しない / Job や Step が予期せず skip される**
- **Root Causes:** `on` trigger の不一致、`paths` / `branches` filter ミス、`if` 条件の誤り、`concurrency` 制限など。
- **Actionable Steps:**
    - **Verify Triggers:**
        - `on` block が、実際に起動させたい event (`push`, `pull_request`, `workflow_dispatch`, `schedule`) と一致しているか確認します。
        - `branches`, `tags`, `paths` filter が event context と合っているか確認します。`paths-ignore` と `branches-ignore` が優先される点も忘れないでください。
        - `workflow_dispatch` を使う場合は、workflow file が default branch にあり、必要な `inputs` が正しく渡されているか確認します。
    - **Inspect `if` Conditions:**
        - workflow、job、step レベルの `if` 条件を丁寧に確認します。1 つ false でも実行されません。
        - `always()` を使った debug step で `${{ toJson(github) }}`, `${{ toJson(job) }}`, `${{ toJson(steps) }}` を出し、評価時点の状態を確認します。
        - 複雑な `if` 条件は簡易 workflow でテストしてください。
    - **Check `concurrency`:**
        - `concurrency` がある場合、同じ group の前回 run が新しい run を block していないか確認します。
    - **Branch Protection Rules:** 特定 branch で workflow 実行を妨げる保護ルールがないか確認します。

### **2. 権限エラー (`Resource not accessible by integration`, `Permission denied`)**
- **Root Causes:** `GITHUB_TOKEN` の権限不足、environment secret access の誤り、external action に必要な権限不足など。
- **Actionable Steps:**
    - **`GITHUB_TOKEN` Permissions:**
        - workflow と job レベル両方の `permissions` を確認します。global は `contents: read` を基本にし、必要な write だけ足します。
        - `GITHUB_TOKEN` の既定権限が広すぎることを理解し、明示的に下げてください。
    - **Secret Access:**
        - repository、organization、environment に secret が正しく設定されているか確認します。
        - environment secret を使う場合、その environment へのアクセス権や承認待ちがないか確認します。
        - secret 名が正確に一致しているか確認します (`secrets.MY_API_KEY`)。
    - **OIDC Configuration:**
        - OIDC で cloud 認証する場合、cloud 側 trust policy が GitHub OIDC issuer を正しく信頼しているか確認します。
        - role/identity に十分な resource 権限があるか検証します。

### **3. Caching の問題 (`Cache not found`, `Cache miss`, `Cache creation failed`)**
- **Root Causes:** cache key ロジック誤り、`path` 不一致、cache size 制限、頻繁な cache invalidation など。
- **Actionable Steps:**
    - **Validate Cache Keys:**
        - `key` と `restore-keys` が正しく、dependency が本当に変わったときだけ変化するよう設計されているか確認します。
        - `restore-keys` を使って近い cache へ fallback できるようにします。
    - **Check `path`:**
        - `actions/cache` の `path` が、実際の dependency や build output の場所と一致しているか確認します。
        - cache 対象 path が存在するかも確認してください。
    - **Debug Cache Behavior:**
        - `actions/cache/restore` の `lookup-only: true` で、どの key が試されているかを確認できます。
        - workflow log の `Cache hit` / `Cache miss` を確認してください。
    - **Cache Size and Limits:** repository ごとの cache 制限を把握し、大きすぎる cache が頻繁に追い出されていないか確認します。

### **4. Workflow が長時間実行される / Timeout する**
- **Root Causes:** 非効率な step、並列性不足、大きな dependency、最適化されていない Docker build、runner 側ボトルネックなど。
- **Actionable Steps:**
    - **Profile Execution Times:**
        - workflow run summary で長い job / step を特定します。
    - **Optimize Steps:**
        - `run` command を `&&` でまとめ、Docker build では layer 数を減らします。
        - 一時 file は使い終わったらすぐ削除してください。
        - 必要な dependency だけを install します。
    - **Leverage Caching:**
        - 重要 dependency や build output に cache が効いているか確認します。
    - **Parallelize with Matrix Strategies:**
        - test や build を `strategy.matrix` で並列化します。
    - **Choose Appropriate Runners:**
        - より大きな GitHub-hosted runner や self-hosted runner を検討してください。
    - **Break Down Workflows:**
        - 巨大 workflow は、より小さな独立 workflow に分割することも検討します。

### **5. CI での Flaky Tests (`Random failures`, `Passes locally, fails in CI`)**
- **Root Causes:** 非決定的 test、race condition、ローカルと CI の環境差、外部 service 依存、テスト分離不足など。
- **Actionable Steps:**
    - **Ensure Test Isolation:**
        - 各 test が他 test の状態に依存しないようにし、必要なら data cleanup を実施します。
    - **Eliminate Race Conditions:**
        - 任意の `sleep` ではなく、要素表示待ちや API 応答待ちなどの明示的 wait を使います。
        - 一時的失敗する外部依存には retry を入れます。
    - **Standardize Environments:**
        - CI とローカルの Node.js version、package、database version などを揃えます。
        - 一貫した依存環境には Docker `services` を使います。
    - **Robust Selectors (E2E):**
        - CSS class や XPath より、`data-testid` のような安定 selector を使います。
    - **Debugging Tools:**
        - E2E failure 時には screenshot や video を必ず残します。
    - **Run Flaky Tests in Isolation:**
        - 特定 test が flaky なら切り出して繰り返し実行し、非決定性の原因を特定します。

### **6. Deploy Failure (デプロイ後にアプリが動かない)**
- **Root Causes:** configuration drift、環境差、runtime dependency 不足、アプリ自体の error、post-deployment network issue など。
- **Actionable Steps:**
    - **Thorough Log Review:**
        - deploy log、application log、server log を見て error や warning を確認します。
    - **Configuration Validation:**
        - 環境変数、ConfigMap、Secret などの注入設定が target 環境に合っているか確認します。
        - pre-deployment check で設定検証を行います。
    - **Dependency Check:**
        - runtime dependency が image 内または target 環境に適切に含まれているか確認します。
    - **Post-Deployment Health Checks:**
        - deploy 後に smoke test や health check を実行し、失敗時は rollback を発動します。
    - **Network Connectivity:**
        - アプリから database、service 間通信などの network 接続を確認します。
    - **Rollback Immediately:**
        - production deploy が失敗または劣化したら、即座に rollback してください。原因解析は非本番で行います。

## 結論

GitHub Actions は、ソフトウェア開発ライフサイクルを自動化するための強力で柔軟なプラットフォームです。secret と token 権限の保護、cache と並列化による最適化、包括的テスト、そして堅牢な deploy 戦略を徹底すれば、開発者は効率的で安全かつ信頼性の高い CI/CD パイプラインを構築できます。CI/CD は一度で完成するものではなく、継続的に測定、最適化、保護していく営みです。あなたの具体的なガイダンスは、チームが GitHub Actions を最大限に活用し、高品質なソフトウェアをより安全に届ける助けになります。この文書は、GitHub Actions による CI/CD を深く理解し使いこなしたい人の基盤資料として機能します。

---

<!-- End of GitHub Actions CI/CD Best Practices Instructions -->
