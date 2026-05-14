---
description: 'PCF コード コンポーネント向け application lifecycle management (ALM)'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj,sln}'
---

# コード コンポーネントの Application Lifecycle Management (ALM)

ALM は、開発、保守、ガバナンスを含む software application のライフサイクル管理を説明するために使われる用語です。詳細情報: [Application lifecycle management (ALM) with Microsoft Power Platform](https://learn.microsoft.com/en-us/power-platform/alm/overview-alm)

この記事では、Microsoft Dataverse におけるコード コンポーネントの観点から、ライフサイクル管理の具体的な側面を扱う際の考慮事項と戦略について説明します。

1. 開発とデバッグに関する ALM の考慮事項
2. コード コンポーネント solution の戦略
3. version 管理と update のデプロイ
4. canvas apps における ALM の考慮事項

## 開発とデバッグに関する ALM の考慮事項

コード コンポーネントを開発するときは、通常次の手順に従います。

1. `pac pcf init` を使って template からコード コンポーネント project (`pcfproj`) を作成する。詳細情報: [Create and build a code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf)
2. コード コンポーネントのロジックを実装する。詳細情報: [Component implementation](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/custom-controls-overview#component-implementation)
3. local test harness を使ってコード コンポーネントをデバッグする。詳細情報: [Debug code components](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls)
4. solution project (`cdsproj`) を作成し、コード コンポーネント project を reference として追加する。詳細情報: [Package a code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/import-custom-controls)
5. 配布およびデプロイ用に release mode でコード コンポーネントを build する

### Dataverse への 2 つのデプロイ方法

コード コンポーネントが model-driven app、canvas app、または portal 内でのテスト準備ができたら、次の 2 つの方法があります。

1. **`pac pcf push`**: 一度に 1 つのコード コンポーネントを、`--solution-unique-name` parameter で指定した solution、または solution が未指定の場合は一時的な PowerAppsTools solution にデプロイします。

2. **`pac solution init` と `msbuild` を使う方法**: 1 つ以上のコード コンポーネントを reference に持つ `cdsproj` solution project を build します。各コード コンポーネントは `pac solution add-reference` を使って `cdsproj` に追加します。solution project には複数のコード コンポーネント reference を含められますが、コード コンポーネント project には 1 つのコード コンポーネントしか含められません。

次の図は、`cdsproj` と `pcfproj` project の 1 対多の関係を示しています。

![cdsproj と pcfproj project の 1 対多の関係](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/code-component-projects.png)

詳細情報: [Package a code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/import-custom-controls#package-a-code-component)

## pcfproj コード コンポーネント Project の Build

`pcfproj` project を build するとき、生成される JavaScript は build に使った command と `pcfproj` file 内の `PcfBuildMode` に依存します。

development mode で build したコード コンポーネントを Microsoft Dataverse にデプロイすることは通常ありません。サイズが大きくて import しにくく、実行時パフォーマンスが低下することが多いためです。詳細情報: [Debugging after deploying into Microsoft Dataverse](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls#debugging-after-deploying-into-microsoft-dataverse)

`pac pcf push` で release build にするには、`pcfproj` 内の `OutputPath` element の下に新しい element を追加して `PcfBuildMode` を設定します。

```xml
<PropertyGroup>
   <Name>my-control</Name>
   <ProjectGuid>6aaf0d27-ec8b-471e-9ed4-7b3bbc35bbab</ProjectGuid>
   <OutputPath>$(MSBuildThisFileDirectory)out\controls</OutputPath>
   <PcfBuildMode>production</PcfBuildMode>
</PropertyGroup>
```

### Build コマンド

| Command | Default Behavior | With PcfBuildMode=production |
|---------|-----------------|------------------------------|
| npm start watch | 常に development |   |
| pac pcf push | development build | release build |
| npm run build | development build | `npm run build -- --buildMode production` |

詳細情報: [Package a code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/import-custom-controls#package-a-code-component)

## .cdsproj Solution Project の Build

solution project (`.cdsproj`) を build するとき、managed solution と unmanaged solution のどちらを output にするかを選べます。managed solution は、その solution の開発環境ではない environment へのデプロイに使われます。これには test、UAT、SIT、本番環境が含まれます。詳細情報: [Managed and unmanaged solutions](https://learn.microsoft.com/en-us/power-platform/alm/solution-concepts-alm#managed-and-unmanaged-solutions)

`SolutionPackagerType` は `pac solution init` が作成する `.cdsproj` file に含まれていますが、初期状態ではコメントアウトされています。コメントを外し、Managed、Unmanaged、Both のいずれかに設定してください。

```xml
<!-- Solution Packager overrides, un-comment to use: SolutionPackagerType (Managed, Unmanaged, Both) -->
<PropertyGroup>
   <SolutionPackageType>Managed</SolutionPackageType>
</PropertyGroup>
```

### Build 構成ごとの結果

| Command | SolutionPackageType | Result |
|---------|-------------------|---------|
| msbuild | Managed | Managed Solution 内の development build |
| msbuild /p:configuration=Release | Managed | Managed Solution 内の release build |
| msbuild | Unmanaged | Unmanaged Solution 内の development build |
| msbuild /p:configuration=Release | Unmanaged | Unmanaged Solution 内の release build |

詳細情報: [Package a code component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/import-custom-controls#package-a-code-component)

## コード コンポーネントの Source Code Control

コード コンポーネントを開発する際は、Azure DevOps や GitHub のような source code control provider の利用が推奨されます。git source control で変更を commit する際、`pac pcf init` template が提供する `.gitignore` file により、一部の file は source control に追加されません。これらは `npm` によって復元されるか、build process の一部として生成されるためです。

```
# dependencies
/node_modules

# generated directory
**/generated

# output directory
/out

# msbuild output directories
/bin
/obj
```

`/out` folder は除外されるため、build された `bundle.js` file (および関連 resource) は source control に追加されません。コード コンポーネントを手動または自動 build pipeline の一部として build するとき、`bundle.js` は最新 code を使って build され、すべての変更が含まれることが保証されます。

さらに、solution が build されたとき、関連する solution zip file も source control には commit されません。代わりに、output は binary release artifact として publish されます。

## コード コンポーネントで SolutionPackager を使う

`pcfproj` と `cdsproj` の source control に加えて、[SolutionPackager](https://learn.microsoft.com/en-us/power-platform/alm/solution-packager-tool) を使うと、solution を段階的に unpack して、それぞれの XML file 群として source control に commit できます。これにより、メタデータの完全な姿を human-readable な形式で保持でき、pull request などを通じて変更を追跡しやすくなります。

> **注**: 現時点では、SolutionPackager は `pac solution clone` と異なり、Dataverse solution からの変更を段階的に export する用途に使えます。

### Solution 構造の例

コード コンポーネントを含む solution を `SolutionPackager /action: Extract` で unpack すると、次のような構造になります。

```
.
├── Controls
│   └── prefix_namespace.ControlName
│       ├── bundle.js *
│       └── css
│          └── ControlName.css *
│       ├── ControlManifest.xml *
│       └── ControlManifest.xml.data.xml
├── Entities
│   └── Contact
│       ├── FormXml
│       │   └── main
│       │       └── {3d60f361-84c5-eb11-bacc-000d3a9d0f1d}.xml
│       ├── Entity.xml
│       └── RibbonDiff.xml
└── Other
    ├── Customizations.xml
    └── Solution.xml
```

`Controls` folder の下には、solution に含まれる各コード コンポーネント用の subfolder があります。この folder 構造を source control に commit するときは、上記で asterisk (*) が付いた file を除外します。これらは対応する component の `pcfproj` project が build されたときに出力されるためです。

必要なのは `*.data.xml` file だけです。これには packaging process に必要な resource を説明する metadata が含まれているためです。

詳細情報: [SolutionPackager command-line arguments](https://learn.microsoft.com/en-us/power-platform/alm/solution-packager-tool#solutionpackager-command-line-arguments)

## コード コンポーネント Solution の戦略

コード コンポーネントは Dataverse solution を使って下流 environment にデプロイされます。solution 内でコード コンポーネントをデプロイする戦略は 2 つあります。

### 1. Segmented Solutions

`pac solution init` を使って solution project を作成し、`pac solution add-reference` を使って 1 つ以上のコード コンポーネントを追加します。この solution は下流 environment に export / import でき、他の segmented solution はコード コンポーネント solution に dependency を持つため、先にその environment にデプロイされている必要があります。

**segmented solution アプローチを採用する理由:**

1. **Versioning lifecycle** - コード コンポーネントを、solution の他部分とは別の lifecycle で開発・デプロイ・version 管理したい場合。これは、開発者が作成したコード コンポーネントを app maker が消費する「fusion team」シナリオでよく見られます。

2. **Shared use** - 複数の environment 間でコード コンポーネントを共有したく、他の solution component と結合したくない場合。ISV である場合や、組織内の異なる領域で使うコード コンポーネントを開発する場合などです。

### 2. Single Solution

単一の solution を Dataverse environment 内で作成し、そこに他の solution component (table、model-driven app、canvas app など) と一緒にコード コンポーネントを追加します。これらの component はコード コンポーネントを参照します。この solution は、solution 間 dependency なしで下流 environment へ export / import できます。

### Solution Lifecycle の概要

![Solution Strategies](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/solution-strategies.png)

詳細情報: [Package and distribute extensions using solutions](https://learn.microsoft.com/en-us/powerapps/developer/data-platform/introduction-solutions)

## コード コンポーネントと自動 Build Pipeline

コード コンポーネント solution を手動で build / deploy する代わりに、自動 build pipeline を使って build / package することもできます。

- Azure DevOps を使っている場合は、[Microsoft Power Platform Build Tool for Azure DevOps](https://learn.microsoft.com/en-us/power-platform/alm/devops-build-tools) を利用できます。
- GitHub を使っている場合は、[Power Platform GitHub Actions](https://learn.microsoft.com/en-us/power-platform/alm/devops-github-actions) を利用できます。

### 自動 Build Pipeline の利点

- **時間効率が高い** - 手動作業をなくすことで build と package が速くなる
- **繰り返し可能** - 毎回同じように実行され、特定メンバーに依存しない
- **Versioning の一貫性** - 以前の version に対して自動的に version を進められる
- **保守しやすい** - build に必要なものがすべて source control に含まれる

## Version 管理と Update のデプロイ

コード コンポーネントをデプロイして更新する際は、一貫した versioning 戦略を持つことが重要です。一般的な戦略として [semantic versioning](https://semver.org/) があり、形式は `MAJOR.MINOR.PATCH` です。

### PATCH Version を増やす

`ControlManifest.Input.xml` は control element 内にコード コンポーネント version を保持しています。

```xml
<control namespace="..." constructor="..." version="1.0.0" display-name-key="..." description-key="..." control-type="...">
```

コード コンポーネントの update をデプロイする際は、変更が検出されるよう `ControlManifest.Input.xml` 内の version について、少なくとも PATCH (version の最後の部分) を増やす必要があります。

**version 更新用 command:**

```bash
# Advance the PATCH version by one
pac pcf version --strategy manifest

# Specify an exact PATCH value (e.g., in automated build pipeline)
pac pcf version --patchversion <PATCH VERSION>
```

### MAJOR と MINOR Version を増やすタイミング

コード コンポーネント version の MAJOR と MINOR は、配布される Dataverse solution と同期させることが推奨されます。

[Dataverse solution には 4 つの部分](https://learn.microsoft.com/en-us/powerapps/maker/data-platform/update-solutions#understanding-version-numbers-for-updates) があります: `MAJOR.MINOR.BUILD.REVISION`

| Code Component | Dataverse Solution | Notes |
|----------------|-------------------|--------|
| MAJOR | MAJOR | Pipeline Variable または最後に commit された値で設定 |
| MINOR | MINOR | Pipeline Variable または最後に commit された値で設定 |
| PATCH | BUILD | $(Build.BuildId) |
| --- | REVISION | $(Rev:r) |

## Canvas Apps における ALM の考慮事項

canvas apps でコード コンポーネントを利用する方法は、model-driven apps とは異なります。コード コンポーネントは、Insert panel で **Get more components** を選んで app に明示的に追加する必要があります。コード コンポーネントが canvas app に追加されると、app 定義の中に content として含まれます。

新しい version のコード コンポーネントがデプロイされ (かつ control version が増えて) 以降、その新 version に更新するには、app maker はまず Power Apps Studio で app を開き、Update code components dialog が表示されたら **Update** を選択する必要があります。その後、新しい version をユーザーが使うためには、app を保存して publish しなければなりません。

![コード コンポーネントを更新する](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/upgrade-code-component.png)

app が更新されない、または **Skip** が使われた場合、environment 内では新しい version に上書きされて存在しなくなっていても、app は引き続き古い version のコード コンポーネントを使い続けます。

app にはコード コンポーネントのコピーが含まれるため、単一 environment の異なる canvas app 間で異なる version のコード コンポーネントを並行して実行することは可能です。ただし、同じ app 内で異なる version のコード コンポーネントを並行して実行することはできません。

> **注**: 現時点では、一致するコード コンポーネントがその environment にデプロイされていなくても canvas app を import できますが、app は常にコード コンポーネントの最新 version を使うよう更新し、同じ version を事前または同じ solution の一部としてその environment にデプロイすることを推奨します。

## 関連記事

- [Application lifecycle management (ALM) with Microsoft Power Platform](https://learn.microsoft.com/en-us/power-platform/alm/overview-alm)
- [Power Apps component framework API reference](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/)
- [Create your first component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript)
- [Debug code components](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls)
