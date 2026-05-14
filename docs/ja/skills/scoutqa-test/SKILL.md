---
name: scoutqa-test
description: 'This skill should be used when the user asks to "test this website", "run exploratory testing", "check for accessibility issues", "verify the login flow works", "find bugs on this page", or requests automated QA testing. Triggers on web application testing scenarios including smoke tests, accessibility audits, e-commerce flows, and user flow validation using ScoutQA CLI. Use this skill proactively after implementing web application features to verify they work correctly.'
---
# ScoutQA テストスキル

`scoutqa` CLI を使用して、Web アプリケーションに対して AI を利用した探索的テストを実行します。

**ScoutQA は、自律的に調査、問題の発見、機能の検証ができるインテリジェントなテスト パートナーと考え​​てください**。テストを複数の並行実行 ScoutQA に委任して、時間を節約しながらカバレッジを最大化します。

## このスキルを使用する場合

このスキルは 2 つのシナリオで使用します。

1. **ユーザーがテストを要求した** - ユーザーが Web サイトのテストまたは機能の検証を明示的に要求した場合
2. **プロアクティブな検証** - Web 機能を実装した後、テストを自動的に実行して、実装が正しく動作することを検証します。

**プロアクティブな使用例:**

- ログインフォーム実装後 → 認証フローのテスト
- フォーム検証追加後 → 検証ルールとエラー処理を確認する
- チェックアウト フローを構築した後 → エンドツーエンドの購入プロセスをテストする
- バグ修正後 → 修正が機能し、他の機能が損なわれていないことを確認します。

**ベスト プラクティス**: Web 機能の実装が完了したら、他のタスクを続行しながら、バックグラウンドで ScoutQA テストを積極的に開始して機能することを確認します。

## テストの実行

### テストワークフロー

このチェックリストをコピーして、進捗状況を追跡します。

テストの進行状況:

- [ ] 明確な期待を伴う特定のテスト プロンプトを作成します
- [ ] スカウトqaコマンドをバックグラウンドで実行します
- [ ] 実行IDとブラウザURLをユーザーに通知
- [ ] 結果の抽出と分析

**ステップ 1: 特定のテスト プロンプトを作成する**

ガイドラインについては、以下の「効果的なプロンプトの作成」セクションを参照してください。

**ステップ 2:scoutqa コマンドを実行します**

**重要**: Bash ツールのタイムアウト パラメーター (5000ms = 5 秒) を使用して、実行の詳細をキャプチャします。

Bash ツールを呼び出すときは、`timeout: 5000` をパラメータとして設定します。

- これは、Claude Code の Bash ツールの組み込みタイムアウト パラメーターです (Unix `timeout` コマンドではありません)。
- 5 秒後、Bash ツールはタスク ID で制御を返し、プロセスはバックグラウンドで実行を続けます。
- これは、プロセスを強制終了する Unix `timeout` とは異なります。ここではプロセスは実行され続けます。
- 最初の 5 秒は、ScoutQA の出力から実行 ID とブラウザ URL をキャプチャします。
- テストは、バックグラウンド タスクを使用して、ScoutQA のインフラストラクチャ上でリモートで実行され続けます。

```bash
scoutqa --url "https://example.com" --prompt "Your test instructions"
```最初の数秒で、コマンドは次のように出力します。

- **実行ID** (例: `019b831d-xxx`)
- **ブラウザ URL** (例: `https://app.scoutqa.ai/t/019b831d-xxx`)
- テストの進行状況を示す最初のツール呼び出し

5 秒のタイムアウトの後、Bash ツールはタスク ID を返し、コマンドはバックグラウンドで実行を続けます。テストの実行中に他のタスクに取り組むことができます。タイムアウトは、最初の出力 (実行 ID とブラウザ URL) をキャプチャするためだけです。テストはバックグラウンド タスクとしてローカルで実行され続け、ScoutQA のインフラストラクチャでリモートで実行され続けます。

**ステップ 3: ユーザーに実行 ID とブラウザ URL を通知します**

Bash ツールがタスク ID を返した後 (最初の 5 秒間の実行の詳細をキャプチャした後)、ユーザーに次のことを通知します。

- ブラウザで進行状況を監視できるようにするための ScoutQA 実行 ID とブラウザ URL
- 後でローカル コマンド出力を確認する場合のバックグラウンド タスク ID

他の作業を続けている間も、テストはバックグラウンドで実行され続けます。

**ステップ 4: 結果の抽出と分析**

完全な形式については、以下の「結果の表示」セクションを参照してください。

### コマンドオプション

- `--url` (必須): テストする Web サイトの URL (`localhost` / `127.0.0.1` をサポート)
- `--prompt` (必須): 自然言語テストの手順
- `--project-id` (オプション): 追跡するプロジェクトに関連付けます。
- `-v, --verbose` (オプション): 内部呼び出しを含むすべてのツール呼び出しを表示します。

### ローカルテストのサポート

ScoutQA は、`localhost` および `127.0.0.1` URL の自律的なテストをサポートしています。手動セットアップは必要ありません。```bash
# Seamlessly test a locally running app when you're developing your app
scoutqa --url "http://localhost:3000" --prompt "Test the registration form"
```### 各コマンドをいつ使用するか

**新しいテストを開始しますか?** → `scoutqa --url --prompt` を使用してください
**既知の問題を確認していますか?** → `scoutqa issue-verify --issue-id <id>` を使用してください
**実行から問題 ID を検索しますか?** → `scoutqa list-issues --execution-id <id>` を使用します
**エージェントにはさらに詳しいコンテキストが必要ですか?** → `scoutqa send-message` を使用します (「スタックした実行のフォローアップ」を参照)

## 効果的なプロンプトを作成する

規範的な手順ではなく、**何を探索し検証するか**に焦点を当てます。 ScoutQA はテスト方法を自律的に決定します。

**例：ユーザー登録の流れ**```bash
scoutqa --url "https://example.com" --prompt "
Explore the user registration flow. Test form validation edge cases,
verify error handling, and check accessibility compliance.
"
```**例: E コマース チェックアウト**```bash
scoutqa --url "https://shop.example.com" --prompt "
Test the checkout flow. Verify pricing calculations, cart persistence,
payment options, and mobile responsiveness.
"
```**例: 包括的なカバレッジのための並列テストの実行**

1 つのメッセージで複数の Bash ツール呼び出しを行い、それぞれの Bash ツールの `timeout` パラメーターを `5000` (ミリ秒) に設定して、複数のテストを並行して起動します。```bash
# Test 1: Authentication & security
scoutqa --url "https://app.example.com" --prompt "
Explore authentication: login/logout, session handling, password reset,
and security edge cases.
"

# Test 2: Core features (runs in parallel)
scoutqa --url "https://app.example.com" --prompt "
Test dashboard and main user workflows. Verify data loading,
CRUD operations, and search functionality.
"

# Test 3: Accessibility (runs in parallel)
scoutqa --url "https://app.example.com" --prompt "
Conduct accessibility audit: WCAG compliance, keyboard navigation,
screen reader support, color contrast.
"
```**実装**: 3 つの Bash ツール呼び出しで 1 つのメッセージを送信します。 Bash ツールを呼び出すたびに、`timeout` パラメーターを `5000` ミリ秒に設定します。 5 秒後、各 Bash 呼び出しはタスク ID を返しますが、プロセスはバックグラウンドで実行を続けます。これにより、初期出力の各テストから実行 ID とブラウザ URL がキャプチャされ、その後 3 つすべてが並行して実行され続けます (ScoutQA インフラストラクチャ上のローカルおよびリモートの両方でバックグラウンド タスクとして)。

**重要なガイドライン:**

- **テスト方法**ではなく、**何をテストするか**を説明します (ScoutQA が手順を理解します)
- 目標、エッジケース、懸念事項に焦点を当てる
- 異なるテスト領域に対して複数の並列実行を実行する
- ScoutQA を信頼して自律的に問題を調査し発見する
- スカウトqa コマンドを呼び出すときは、Bash ツールの `timeout` パラメータを常に `5000` ミリ秒に設定します (プロセスがバックグラウンドで継続している間、5 秒後に制御が戻ります)。
- 並列テストの場合は、単一のメッセージで複数の Bash ツール呼び出しを実行します。
- 覚えておいてください: Bash ツールのタイムアウト ≠ Unix タイムアウト コマンド (Bash タイムアウトはバックグラウンドでプロセスを続行しますが、Unix タイムアウトはプロセスを強制終了します)

### 一般的なテストのシナリオ

**展開後の煙テスト:**```bash
scoutqa --url "$URL" --prompt "
Smoke test: verify critical functionality works after deployment.
Check homepage, navigation, login/logout, and key user flows.
"
```**アクセシビリティ監査:**```bash
scoutqa --url "$URL" --prompt "
Audit accessibility: WCAG 2.1 AA compliance, keyboard navigation,
screen reader support, color contrast, and semantic HTML.
"
```**電子商取引テスト:**```bash
scoutqa --url "$URL" --prompt "
Explore e-commerce functionality: product search/filtering,
cart operations, checkout flow, and pricing calculations.
"
```**SaaS アプリケーション:**```bash
scoutqa --url "$URL" --prompt "
Test SaaS app: authentication, dashboard, CRUD operations,
permissions, and data integrity.
"
```**フォームの検証:**```bash
scoutqa --url "$URL" --prompt "
Test form validation: edge cases, error handling, required fields,
format validation, and successful submission.
"
```**モバイルの応答性:**```bash
scoutqa --url "$URL" --prompt "
Check mobile experience: responsive layout, navigation,
touch interactions, and viewport behavior.
"
```**既知の問題の検証:**```bash
# First, find issue IDs from a previous execution
scoutqa list-issues --execution-id <executionId>

# Then verify the issue (creates a new verification execution automatically)
scoutqa issue-verify --issue-id <issueId>
````issue-verify` コマンドは次のことを行います。

1. 問題の検証実行を作成する
2. 実行IDとブラウザURLを表示する
3. エージェントの検証の進行状況をリアルタイムでストリーミングします。
4. 結果へのリンクを含む完了概要を表示する

**機能検証 (実装後):**```bash
scoutqa --url "$URL" --prompt "
Verify the new [feature name] works correctly. Test core functionality,
edge cases, error handling, and integration with existing features.
"
```**例: 機能のコーディング後のプロアクティブなテスト**

ユーザー登録フォームを実装した後、それが機能することを自動的に検証します。```bash
scoutqa --url "http://localhost:3000/register" --prompt "
Test the newly implemented registration form. Verify:
- Form validation (email format, password strength, required fields)
- Error messages display correctly
- Successful registration flow
- Edge cases (duplicate emails, special characters, etc.)
"
```これにより、実装がコンテキスト内で新しいうちに問題が即座に捕捉されます。

## リストの問題

以前の実行で見つかった問題を参照するには、`scoutqa list-issues` を使用します。これは、`issue-verify` で使用する問題 ID を見つけるのに役立ちます。```bash
scoutqa list-issues --execution-id <executionId>
```**オプション:**

- `--execution-id` (必須): 実行 ID (`/t/<executionId>` URL または CLI 出力から)

**出力例:**```
Showing 3 issues:

🔴 019c-abc1
   Login button unresponsive on mobile
   Severity: critical | Category: usability | Status: open

🟠 019c-abc2
   Missing form validation on email field
   Severity: high | Category: functional | Status: open

🟡 019c-abc3
   Color contrast insufficient on footer links
   Severity: medium | Category: accessibility | Status: resolved
```## 結果の提示

### 即時プレゼンテーション (テスト開始後)

scoutqa コマンドを実行した直後に、実行の詳細をユーザーに表示します。```markdown
**ScoutQA Test Started**

Execution ID: `019b831d-xxx`
View Live: https://app.scoutqa.ai/t/019b831d-xxx

The test is running remotely. You can view real-time progress in your browser at the link above while I continue with other tasks.
```### 最終結果 (完了後)

実行が完了したら、次の形式を使用して結果を表示します。```markdown
**ScoutQA Test Results**

Execution ID: `ex_abc123`

**Issues Found:**

[High] Accessibility: Missing alt text on logo image

- Impact: Screen readers cannot describe the logo
- Location: Header navigation

[Medium] Usability: Submit button not visible on mobile viewport

- Impact: Users cannot complete form on mobile devices
- Location: Contact form, bottom of page

[Low] Functional: Search returns no results for valid queries

- Impact: Search feature appears broken
- Location: Main search bar

**Summary:** Found 3 issues across accessibility, usability, and functional categories. See full interactive report with screenshots at the URL above.
```常に以下を含めてください:

- **実行 ID** (例: `ex_abc123`) 参照用
- **見つかった問題** (重大度、カテゴリ (アクセシビリティ、使いやすさ、機能)、影響、および場所)

## スタックした実行のフォローアップ

リモート エージェントがスタックした場合、または説明が必要な場合は、`send-message` を使用して続行します。```bash
# Example: Agent is stuck at login, user provides credentials
scoutqa send-message --execution-id ex_abc123 --prompt "
Use these test credentials: username: testuser@example.com, password: TestPass123
"

# Example: Agent asks which flow to test next
scoutqa send-message --execution-id ex_abc123 --prompt "
Focus on the checkout flow next, skip the wishlist feature
"
```## テスト結果の確認

ScoutQA テストは、ScoutQA のインフラストラクチャ上でリモートで実行されます。短いタイムアウトでテストを開始して実行 ID を取得した後、次のようにします。

1. テストは引き続きリモートで実行されます (ローカルのバックグラウンドではなく)
2. すぐに他の作業を続けることができます
3. 後で結果を確認するには、テスト開始時に指定されたブラウザの URL にアクセスします。
4. または、`scoutqa get-execution --execution-id <id>` を使用して CLI 経由で結果を取得します

**ベスト プラクティス**: Bash ツールの `timeout` パラメーターを `5000` ミリ秒に設定してテストを開始します。 5 秒後、Bash ツールはタスク ID と実行の詳細 (実行 ID とブラウザ URL) を含む制御を返し、テストはバックグラウンドで実行を続けます。その後、他の作業を続行し、必要に応じて ScoutQA の Web サイトまたは CLI 経由で結果を確認できます。

## トラブルシューティング

|問題 |ソリューション |
| ------------------------------ | -------------------------------------------------------- |
| `command not found: scoutqa` | CLI をインストールします: `npm i -g @scoutqa/cli@latest` |
|認証の有効期限が切れた / 不正です | `scoutqa auth login` を実行します。
|テストがハングするか入力が必要です | `scoutqa send-message --execution-id` を使用してください。
|テスト結果を確認する |ブラウザの URL または `scoutqa get-execution --execution-id` にアクセスします。
|検証には問題 ID が必要です | `scoutqa list-issues --execution-id <id>` を実行します。