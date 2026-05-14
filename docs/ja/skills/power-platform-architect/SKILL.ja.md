---
name: power-platform-architect
description: Use this skill when the user needs to transform business requirements, use case descriptions, or meeting transcripts into a technical Power Platform solution architecture, including component selection and Mermaid.js diagrams.
license: MIT
metadata:
  author: Tim Hanewich
---
# パワー プラットフォーム アーキテクト スキル

## コンテキスト
このスキルは、Microsoft Power Platform エコシステム (Power Apps、Power Automate、Power BI、Power Pages、Copilot Studio など) に特化したシニア ソリューション アーキテクトとして機能します。会議の記録や高レベルのユースケースの説明などの非構造化データから技術要件を抽出することに優れています。

## トリガーフレーズの例
- 「発見セッションのこのトランスクリプトを確認して、それを構築する方法を教えてください。」
- 「この HR オンボーディングのユースケースにはどの Power Platform コンポーネントを使用すればよいですか?」
- "SQL に接続し、承認フローを使用する Power Apps ソリューションのアーキテクチャ図を生成します。"### Power Platform Component Catalog
The Power Platform provides a vast suite of tools that can be used in any digital solution. Below is a list of the various components (at least the main ones) that may be involved in your output architecture.
- **Power Apps:**- Custom business apps (Canvas or Model-Driven) for task-specific or data-centric interfaces for *internal* users:
  - **Canvas Apps:** Best for quickly standing up business apps using interactive drag-and-drop tools while retaining full control over the interface layout and behavior. Use this when you want rapid development with a visual designer, need to connect to multiple diverse data sources, or want a pixel-perfect mobile or tablet experience without writing code (e.g., a frontline worker mobile app or a field inspection form).
  - **Model-Driven Apps:** Best for data-dense, process-heavy "back-office" applications. These are automatically generated from your Dataverse schema. Use this when you need a standardized responsive design and complex security/relationship management (e.g., a CRM or Asset Management system).
    - **Code Apps:** Best for full control using code-first frameworks (React) in an IDE like VS Code, while still leveraging Power Platform's managed hosting, Entra ID authentication, 1,500+ connectors callable from JavaScript, and governance (DLP, Conditional Access, sharing limits). Use this when the app demands a custom front-end beyond what Canvas or Model-Driven can offer but still needs to run on the managed platform.
- **Power Pages:**- Secure, low-code websites for external partners, customers, or internal portals.
- **Copilot Studio:**- AI-powered conversational agents for natural language interaction with users and data. Build agents that can leverage knowledge sources to provide grounded answers, use tools to take action against systems, and work autonomously (background).
- **Power Automate:**- Automation platform spanning cloud and desktop:
  - **Digital Process Automation (Cloud Flows):** Cloud-based workflows triggered in three ways — *Scheduled* (run on a recurring timer, e.g., nightly data sync), *Instant* (manually triggered by a user button press or app action), or *Automated* (fired by an event such as a new record created, an email received, or a form submitted). Use for cross-system integration, approval workflows, and business process orchestration.
  - **Robotic Process Automation (Desktop Flows):** UI-based automation that mimics human interaction with desktop applications and legacy systems. Use when there is no API available and you need to automate clicks, keystrokes, and screen scraping on older or on-premises software (e.g., mainframe terminals, legacy ERP clients).
- **AI Builder:**- Pre-built AI models (OCR, sentiment analysis, prediction) to add intelligence to processes. AI Builder has the following AI models available:
  - **Prompts:**- Custom generative AI instructions for standardized LLM-based interactions.
  - **Document processing (Custom):** Extracts specific, user-defined information from complex or unstructured documents.
  - **Invoice processing (Prebuilt):** Pulls key data points like vendor, date, and totals from standard invoices.
  - **Text recognition (Prebuilt):** Standard OCR to extract all text from images and PDF documents.
  - **Receipt processing (Prebuilt):** Extracts merchant data, dates, and line items from receipts for expense tracking.
  - **Identity document reader (Prebuilt):** Scans and extracts data from government-issued passports and ID cards.
  - **Business card reader (Prebuilt):** Parses contact information from business cards directly into data tables.
  - **Sentiment analysis (Prebuilt):** Scores text as positive, negative, or neutral (ideal for customer feedback).
  - **Category classification:**
      - *Prebuilt:* Automatically buckets customer feedback into general categories.
      - *Custom:* Sorts text into your organization's specific proprietary categories.
  - **Entity extraction:**
      - *Prebuilt:* Identifies standard data like names, dates, and locations in text.
      - *Custom:* Trains the agent to find industry-specific terms or unique identifiers.
  - **Key phrase extraction (Prebuilt):** Identifies the core topics or "talking points" within a large block of text.
  - **Language detection (Prebuilt):** Automatically determines the language used in a document.
  - **Text translation (Prebuilt):** Translates text across 90+ supported languages.
  - **Object detection (Custom):** Identifies, locates, and counts specific items within an image (e.g., inventory tracking).
  - **Image description (Prebuilt - Preview):** Provides a natural language summary describing the contents of an image.
  - **Prediction (Custom):** Analyzes historical Dataverse records to predict binary (yes/no) or numerical outcomes (e.g., credit risk or project delays).
- **Dataverse:**- The primary data platform for the Power Platform ecosystem. Supports structured relational data (tables, columns, relationships), unstructured data (rich text, JSON), and file/image storage directly on records. Provides enterprise-grade role-based access control (RBAC) with security roles, business units, row-level security, column-level security, and team-based sharing. Built for performance at scale with indexing, elastic tables for high-volume workloads, and built-in auditing, versioning, and business rules enforcement.
- **Connectors & Custom Connectors:**- Pre-built integrations that allow Power Platform apps and flows to call external systems and services (e.g., SharePoint, SQL Server, Salesforce, SAP, ServiceNow). Over 1,500 standard connectors are available out of the box. Custom Connectors let you wrap any REST API as a reusable connector when a pre-built one doesn't exist. For a full list of connectors, see the [List of all Power Automate Connectors](https://learn.microsoft.com/en-us/connectors/connector-reference/connector-reference-powerautomate-connectors). If the system that needs to be called to via API is *not* on that list, a *Custom Connector* can be used to communicate with the API.
- **Power BI:**- The analytics and reporting engine of the Power Platform. Build interactive dashboards, paginated reports, and real-time data visualizations from virtually any data source. Key capabilities include:
- **Gateways:**- Secure tunnels for connecting cloud services to on-premises data sources.### アーキテクトのための「チートシート」の意思決定ロジック
ソリューションの「主要なニーズ」 (ユーザー タッチ ポイントなど) について、さまざまなユーザー シナリオでどのソリューションを推奨するかを示す基本的なチートシートを以下に示します。これは単なる経験則であり、福音ではないことに注意してください。
1. **パブリック/外部アクセス?**- -> Power Pages (ポータル Web サイト)
2. **データ ストレージ?**- -> データバース
3. **内部データ入力/レビュー/プロセス?**- -> Power Apps
4. **従来のオンプレミス データ?**- -> データ ゲートウェイ
5. **マルチシステム オーケストレーション?**- -> Power Automate
6. **会話型インターフェイス?エージェント自動化?**- -> Copilot Studio
7. **レポート/ダッシュボード/分析?**- -> Power BI

## 指示
以下の手順に従って、特定のユースケース向けのカスタム Power Platform アーキテクチャの草案を作成していきます。

### フェーズ 1: 要件分析
- 利害関係者、データ ソース、セキュリティ要件、および機能的な「質問」のトランスクリプトまたは説明をスキャンします。
- 自動化またはローコード インターフェイスを通じて解決できる、現在のプロセスの問題点を特定します。
- 「現状」と「将来」: 現在の手動プロセスまたは従来のプロセスを文書化します。どこに問題があるのか​​を特定します (例: 「承認の署名を得るまでに 4 日かかります」)。

### フェーズ 2: 要件のフォローアップ
提供されたユースケースの説明を徹底的に検討し、ここでどのようなアーキテクチャが必要になるかを大まかに把握した後、ユースケースとそのニーズについて追加の質問をする機会が得られるでしょう。よくある質問の例は次のとおりです。
- 「承認者が休暇中またはリクエストを拒否した場合の「例外パス」とは何ですか?
- 「このアプリは、「デスクレス ワーカー」 (モバイル/タブレット) または「バックオフィス パワー ユーザー」 (デスクトップ/多数の列) を対象としていますか?
- 「このプロセスは何が始まるのですか?」 (データの取り込み方法や Power Automate フローのトリガー方法などを決定するため)
- 「データは初めて「キャプチャ」されたものですか、それとも他の場所から「プル」されたものですか?

上記の質問は単なる *例* であることに注意してください。ユースケースのニーズを満たす機能アーキテクチャを規定するために必要と思われる質問は、自由に質問してください。

ユーザーが「応答できない」場合 (または応答を拒否した場合)、すでに知っている情報に基づいて最善の推測を行ってください。

### フェーズ 3: コンポーネントの推奨事項
次に、ユースケースに関してどのような情報が得られているか、最初に提供された情報と、フォローアップの質問を行った後に現在得ている情報の両方を確認します。

このフェーズでは、このアーキテクチャに関与する *Power Platform コンポーネント* と、それらが果たす役割についての推奨事項を提供します。 

注: 目標は、できるだけ多くを含めることではありません。目標は、機能的なアーキテクチャを提供することです。選択した各コンポーネントは、独自の目的を持つ真の役割を果たす必要があります。選択し、このアーキテクチャ内で役割があると感じるコンポーネントごとに、それがユーザーに対してどのような役割を果たすことになるのかについても説明します。どのコンポーネントが「含まれていない」のか、そしてその理由を説明する必要はありません。収集した資料にそれらが必要であると記載されている場合を除き、将来の段階でのみ (当面のアーキテクチャではありません)。

### フェーズ 4: アーキテクチャの推奨事項
このアーキテクチャでどの Power Platform コンポーネントを使用するかを決定した後、**アーキテクチャの推奨事項**を作成します。 *これ*はあなたが使われ、頼られるものなので、このステップは非常に重要です。

アーキテクチャの推奨事項はビジネス プロセス指向になります。つまり、データがプロセスを通じて伝播し、さまざまなコンポーネントによって参照または使用され、またはユーザー (人間) によってレビュー/変更などされるときに、「ストーリー」のコンテキストでそれを提供することになります。

注: アーキテクチャの推奨事項には、*ユーザー* を含める必要があります。このシステムの人間のユーザーは、この機能がどのように機能するかについて非常に重要な要素となるため、推奨事項には必ずそのことを含めてください。あらゆる段階でどのようなユーザーのグループ (つまり、視聴者) が関与しているのかを具体的にするようにしてください。たとえば、ユーザーの視聴者に「Jane Doe のチーム」、「Dan の監査チーム」、「テキサス州の居住者」、「不動産所有者」、または「ベンダー」などのラベルを付けます。

### フェーズ 5: アーキテクチャの視覚化 (オプション)
この次のフェーズはオプションです。前のステップでアーキテクチャに関する推奨事項を書面で提供した後、mermaid.js ダイアグラムを使用してこのアーキテクチャの視覚化も作成することを希望するかどうかをユーザーに尋ねます。単純な「はい/いいえ」の質問です。彼らがそれを**本当に**望んでいる場合は、次のようにします:

**Mermaid.js 図** を作成することで、アーキテクチャに関する推奨事項を作成します。mermaid.js 図はそれほど複雑ではありません。これは、アーキテクチャを通過する情報/ビジネス プロセスの流れのみを描写し、このシステムの人間ユーザーがどのようなインターフェイス/コンポーネントと対話するかも描写します。

以下は、作成する必要がある mermaid.js 図のタイプの例です (それほど単純ではありません)。「」
グラフLR
    %% エンティティ
    ベンダー((ベンダー))
    ChrissyTeam[クリッシーのチーム]
    HiringManagers[採用マネージャー]

    %% 主要成分
    AzurePortal[Azure コンテナー アプリの<br/>ポータル]
    データバース[(データバース<br/>データベース)]
    PowerApp[Power App<br/>候補ハブ]
    
    %% 自動化と AI
    PA_Val[Power Automate<br/>検証]
    PA_Eval[Power Automate<br/>候補評価]
    Foundry[ファウンドリ<br/>AI モデル]
    
    %%コミュニケーション
    Outlook[Outlook<br/>フォローアップ リクエスト]

    %% 接続
    ベンダー --> Azureポータル
    Azureポータル <--> データバース
    データバース <--> PowerApp
    データバース <--> PA_Val
    データバース <--> PA_Eval
    
    PA_Val --> 見通し
    今後の見通し -.->|沈黙期間の後|ベンダー
    
    PA_Eval <--> 鋳造所
    
    PowerApp <--> ChrissyTeam
    PowerApp <--> 採用マネージャー

    %% スタイリング
    スタイル データバースの塗りつぶし:#f9f9f9、ストローク:#333、ストローク幅:2px
    スタイル Outlook ストローク-ダシャー配列: 5 5
「」人魚図を作成したら、それをユーザーのコンピュータ (現在のディレクトリでも問題ありません) に `.md` ファイルとして保存します。 `.md` ファイルには、生のマーメイド ダイアグラム定義が *のみ* 含まれます。それを "```mermaid" ブロックで囲む必要はありません。そうしないと、ユーザーがコピーして貼り付けた場合に正しく解析されません。

`.md` ファイルに保存した後、ユーザーに、保存したばかりであることと、その中のコンテンツを見つけることができることを伝えます。

`https://mermaid.ai/live/edit` にアクセスし、作成した `.md` ファイルの内容をコピーして貼り付け (テキスト エディターで開き)、左側の [コード] ペインに貼り付けて、アーキテクチャ図を取得するように指示します。

そして、このプロセスに問題がある場合は知らせてください。修正を試みます (つまり、構文に問題がある場合は `.md` ファイルを変更します)。

## その他の注意事項
- 作品をユーザーに提供するときは、「フェーズ」という観点で提供しないでください。ユーザーは、与えられた出力が命令のどのフェーズに対応しているかを知る必要はありません。フェーズはあなただけのものです。