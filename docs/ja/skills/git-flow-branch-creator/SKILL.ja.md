---
name: git-flow-branch-creator
description: 'git status/diff を解析し、nvie Git Flow ブランチモデルに従って適切なブランチを作成するインテリジェントな Git Flow ブランチ作成ツール。'
---

### 手順

```xml
<instructions>
	<title>Git Flow ブランチ作成ツール</title>
	<description>このプロンプトは、git status と git diff（または git diff --cached）を使って現在の git 変更を解析し、Git Flow ブランチモデルに従って適切なブランチ種別をインテリジェントに判断し、意味のあるブランチ名を作成します。</description>
	<note>
		このプロンプトを実行するだけで、Copilot が変更を解析し、適切な Git Flow ブランチを作成します。
	</note>
</instructions>
```

### ワークフロー

**以下の手順に従ってください:**

1. `git status` を実行して、現在のリポジトリ状態と変更ファイルを確認します。
2. `git diff`（ステージされていない変更）または `git diff --cached`（ステージ済み変更）を実行して、変更の性質を分析します。
3. 以下の Git Flow ブランチ分析フレームワークを使って変更を分析します。
4. 分析結果に基づいて適切なブランチ種別を決定します。
5. Git Flow の規約に従って意味のあるブランチ名を生成します。
6. ブランチを作成し、自動でそのブランチに切り替えます。
7. 分析結果の要約と次のステップを提示します。

### Git Flow ブランチ分析フレームワーク

```xml
<analysis-framework>
	<branch-types>
		<feature>
			<purpose>新機能、機能強化、重大ではない改善</purpose>
			<branch-from>develop</branch-from>
			<merge-to>develop</merge-to>
			<naming>feature/descriptive-name または feature/ticket-number-description</naming>
			<indicators>
				<indicator>新しい機能が追加されている</indicator>
				<indicator>UI/UX の改善</indicator>
				<indicator>新しい API エンドポイントまたはメソッド</indicator>
				<indicator>データベーススキーマの追加（互換性を壊さない）</indicator>
				<indicator>新しい設定オプション</indicator>
				<indicator>パフォーマンス改善（重大ではない）</indicator>
			</indicators>
		</feature>

		<release>
			<purpose>リリース準備、バージョン更新、最終テスト</purpose>
			<branch-from>develop</branch-from>
			<merge-to>develop AND master</merge-to>
			<naming>release-X.Y.Z</naming>
			<indicators>
				<indicator>バージョン番号の変更</indicator>
				<indicator>ビルド設定の更新</indicator>
				<indicator>ドキュメントの最終化</indicator>
				<indicator>リリース前の軽微なバグ修正</indicator>
				<indicator>リリースノートの更新</indicator>
				<indicator>依存関係のバージョン固定</indicator>
			</indicators>
		</release>

		<hotfix>
			<purpose>即時デプロイが必要な本番の重大バグ修正</purpose>
			<branch-from>master</branch-from>
			<merge-to>develop AND master</merge-to>
			<naming>hotfix-X.Y.Z または hotfix/critical-issue-description</naming>
			<indicators>
				<indicator>セキュリティ脆弱性の修正</indicator>
				<indicator>本番環境の重大バグ</indicator>
				<indicator>データ破損の修正</indicator>
				<indicator>サービス停止の復旧</indicator>
				<indicator>緊急の設定変更</indicator>
			</indicators>
		</hotfix>
	</branch-types>
</analysis-framework>
```

### ブランチ命名規約

```xml
<naming-conventions>
	<feature-branches>
		<format>feature/[ticket-number-]descriptive-name</format>
		<examples>
			<example>feature/user-authentication</example>
			<example>feature/PROJ-123-shopping-cart</example>
			<example>feature/api-rate-limiting</example>
			<example>feature/dashboard-redesign</example>
		</examples>
	</feature-branches>

	<release-branches>
		<format>release-X.Y.Z</format>
		<examples>
			<example>release-1.2.0</example>
			<example>release-2.1.0</example>
			<example>release-1.0.0</example>
		</examples>
	</release-branches>

	<hotfix-branches>
		<format>hotfix-X.Y.Z OR hotfix/critical-description</format>
		<examples>
			<example>hotfix-1.2.1</example>
			<example>hotfix/security-patch</example>
			<example>hotfix/payment-gateway-fix</example>
			<example>hotfix-2.1.1</example>
		</examples>
	</hotfix-branches>
</naming-conventions>
```

### 分析プロセス

```xml
<analysis-process>
	<step-1>
		<title>変更内容の性質分析</title>
		<description>変更されたファイルの種類と変更の性質を確認する</description>
		<criteria>
			<files-modified>拡張子、ディレクトリ構造、用途を確認する</files-modified>
			<change-scope>変更が追加・修正・準備のどれに当たるかを判断する</change-scope>
			<urgency-level>変更が重大な問題への対応か、開発的な変更かを評価する</urgency-level>
		</criteria>
	</step-1>

	<step-2>
		<title>Git Flow 分類</title>
		<description>変更を適切な Git Flow ブランチ種別にマッピングする</description>
		<decision-tree>
			<question>これは本番の問題に対する重大な修正ですか？</question>
			<if-yes>hotfix ブランチを検討する</if-yes>
			<if-no>
				<question>これはリリース準備の変更ですか（バージョン更新、最終調整など）？</question>
				<if-yes>release ブランチを検討する</if-yes>
				<if-no>デフォルトで feature ブランチにする</if-no>
			</if-no>
		</decision-tree>
	</step-2>

	<step-3>
		<title>ブランチ名の生成</title>
		<description>意味が明確で説明的なブランチ名を作成する</description>
		<guidelines>
			<use-kebab-case>小文字とハイフンを使用する</use-kebab-case>
			<be-descriptive>目的が明確にわかる名前にする</be-descriptive>
			<include-context>利用可能ならチケット番号やプロジェクト文脈を含める</include-context>
			<keep-concise>長すぎる名前は避ける</keep-concise>
		</guidelines>
	</step-3>
</analysis-process>
```

### エッジケースとバリデーション

```xml
<edge-cases>
	<mixed-changes>
		<scenario>変更に機能追加とバグ修正の両方が含まれる</scenario>
		<resolution>最も重要な変更種別を優先するか、複数ブランチへの分割を提案する</resolution>
	</mixed-changes>

	<no-changes>
		<scenario>git status/diff で変更が検出されない</scenario>
		<resolution>ユーザーに通知し、git status の確認または先に変更を行うことを提案する</resolution>
	</no-changes>

	<existing-branch>
		<scenario>すでに feature/hotfix/release ブランチ上にいる</scenario>
		<resolution>新しいブランチが必要か、現在のブランチが適切かを分析する</resolution>
	</existing-branch>

	<conflicting-names>
		<scenario>提案したブランチ名がすでに存在する</scenario>
		<resolution>連番サフィックスを付けるか、代替名を提案する</resolution>
	</conflicting-names>
</edge-cases>
```

### 例

```xml
<examples>
	<example-1>
		<scenario>新しいユーザー登録 API エンドポイントを追加した</scenario>
		<analysis>新機能、追加的な変更、重大ではない</analysis>
		<branch-type>feature</branch-type>
		<branch-name>feature/user-registration-api</branch-name>
		<command>git checkout -b feature/user-registration-api develop</command>
	</example-1>

	<example-2>
		<scenario>認証の重大なセキュリティ脆弱性を修正した</scenario>
		<analysis>セキュリティ修正、本番に対して重大、即時デプロイが必要</analysis>
		<branch-type>hotfix</branch-type>
		<branch-name>hotfix/auth-security-patch</branch-name>
		<command>git checkout -b hotfix/auth-security-patch master</command>
	</example-2>

	<example-3>
		<scenario>バージョンを 2.1.0 に更新し、リリースノートを最終化した</scenario>
		<analysis>リリース準備、バージョン更新、ドキュメント作業</analysis>
		<branch-type>release</branch-type>
		<branch-name>release-2.1.0</branch-name>
		<command>git checkout -b release-2.1.0 develop</command>
	</example-3>

	<example-4>
		<scenario>データベースクエリ性能を改善し、キャッシュを更新した</scenario>
		<analysis>パフォーマンス改善、重大ではない機能強化</analysis>
		<branch-type>feature</branch-type>
		<branch-name>feature/database-performance-optimization</branch-name>
		<command>git checkout -b feature/database-performance-optimization develop</command>
	</example-4>
</examples>
```

### バリデーションチェックリスト

```xml
<validation>
	<pre-analysis>
		<check>リポジトリがクリーンな状態であること（競合する未コミット変更がない）</check>
		<check>現在のブランチが適切な開始点であること（feature/release は develop、hotfix は master）</check>
		<check>リモートリポジトリが最新であること</check>
	</pre-analysis>

	<analysis-quality>
		<check>変更分析がすべての変更ファイルをカバーしている</check>
		<check>ブランチ種別の選択が Git Flow の原則に従っている</check>
		<check>ブランチ名が意味的で規約に従っている</check>
		<check>エッジケースが考慮され、対処されている</check>
	</analysis-quality>

	<execution-safety>
		<check>対象ブランチ（develop/master）が存在し、アクセス可能である</check>
		<check>提案ブランチ名が既存ブランチと衝突しない</check>
		<check>ユーザーにブランチ作成の適切な権限がある</check>
	</execution-safety>
</validation>
```

### 最終実行

```xml
<execution-protocol>
	<analysis-summary>
		<git-status>git status コマンドの出力</git-status>
		<git-diff>git diff 出力の関連部分</git-diff>
		<change-analysis>変更が何を意味するかの詳細分析</change-analysis>
		<branch-decision>特定のブランチ種別を選んだ理由の説明</branch-decision>
	</analysis-summary>

	<branch-creation>
		<command>git checkout -b [branch-name] [source-branch]</command>
		<confirmation>ブランチ作成と現在のブランチ状態を確認する</confirmation>
		<next-steps>次のアクション（変更のコミット、ブランチの push など）を案内する</next-steps>
	</branch-creation>

	<fallback-options>
		<alternative-names>主提案が適さない場合に、代替ブランチ名を 2〜3 個提案する</alternative-names>
		<manual-override>分析結果が不正確に見える場合、ユーザーが別のブランチ種別を指定できるようにする</manual-override>
	</fallback-options>
</execution-protocol>
```

### Git Flow リファレンス

```xml
<gitflow-reference>
	<main-branches>
		<master>本番準備済みコード。すべてのコミットがリリース</master>
		<develop>機能統合用ブランチ。最新の開発変更を保持</develop>
	</main-branches>

	<supporting-branches>
		<feature>develop から分岐し、develop にマージ</feature>
		<release>develop から分岐し、develop と master の両方にマージ</release>
		<hotfix>master から分岐し、develop と master の両方にマージ</hotfix>
	</supporting-branches>

	<merge-strategy>
		<flag>ブランチ履歴を保持するため、常に --no-ff フラグを使用する</flag>
		<tagging>master ブランチでリリースにタグを付ける</tagging>
		<cleanup>マージ成功後にブランチを削除する</cleanup>
	</merge-strategy>
</gitflow-reference>
```

