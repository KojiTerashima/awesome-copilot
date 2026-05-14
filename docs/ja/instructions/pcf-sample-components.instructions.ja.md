---
description: 'PowerApps-Samples repository の PCF sample component の使用方法と実行方法'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# Sample Components の使い方

この section に掲載されているすべての sample component は、[github.com/microsoft/PowerApps-Samples/tree/master/component-framework](https://github.com/microsoft/PowerApps-Samples/tree/master/component-framework) からダウンロードできるので、model-driven app または canvas app で試すことができます。

この section 内の各 sample component topic では、sample component の概要、見た目、完全版 sample component へのリンクが提供されます。

## Sample Components を試す前に

sample component を試すには、まず次を行う必要があります。

- この repository [github.com/microsoft/PowerApps-Samples](https://github.com/microsoft/PowerApps-Samples) を [Download](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) または [clone](https://docs.github.com/repositories/creating-and-managing-repositories/cloning-a-repository) する。
- [Install Power Platform CLI for Windows](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction#install-power-platform-cli-for-windows) をインストールする。

## Sample Components を試す

control を含む solution を生成し、sample component を model-driven app または canvas app にインポートして試せるようにするには、[README.md](https://github.com/microsoft/PowerApps-Samples/blob/master/component-framework/README.md) の手順に従ってください。

## Sample Components の実行方法

次の手順を使って、sample component を model-driven app または canvas app にインポートして試します。

### ステップごとの手順

1. **repository をダウンロードまたは clone する**
   - [github.com/microsoft/PowerApps-Samples](https://github.com/microsoft/PowerApps-Samples) を [Download](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) または [clone](https://docs.github.com/repositories/creating-and-managing-repositories/cloning-a-repository) します。

2. **Developer Command Prompt を開く**
   - [Developer Command Prompt for Visual Studio](https://learn.microsoft.com/visualstudio/ide/reference/command-prompt-powershell) を開き、`component-framework` folder に移動します。
   - Windows では、Start に `developer command prompt` と入力して developer command prompt を開けます。

3. **依存関係をインストールする**
   - 試したい component (例: `IncrementControl`) に移動し、次を実行します。
   ```bash
   npm install
   ```

4. **project を復元する**
   - command の完了後、次を実行します。
   ```bash
   msbuild /t:restore
   ```

5. **solution folder を作成する**
   - sample component folder 内に新しい folder を作成します。
   ```bash
   mkdir IncrementControlSolution
   ```

6. **solution folder に移動する**
   ```bash
   cd IncrementControlSolution
   ```

7. **solution を初期化する**
   - 作成した folder 内で、`pac solution init` command を実行します。
   ```bash
   pac solution init --publisher-name powerapps_samples --publisher-prefix sample
   ```
   > **注**: この command は、folder 内に `IncrementControlSolution.cdsproj` という新しい file を作成します。

8. **component reference を追加する**
   - `.pcfproj` file の場所を `path` に指定して、`pac solution add-reference` command を実行します。
   ```bash
   pac solution add-reference --path ../../IncrementControl
   ```
   または
   ```bash
   pac solution add-reference --path ../../IncrementControl/IncrementControl.pcfproj
   ```
   > **重要**: 追加したい control の `.pcfproj` file を含む folder を参照してください。

9. **solution をビルドする**
   - solution project から zip file を生成するには、次の 3 つの command を実行します。
   ```bash
   msbuild /t:restore
   msbuild /t:rebuild /restore /p:Configuration=Release
   msbuild
   ```
   - 生成された solution zip file は `IncrementControlSolution\bin\debug` folder に作成されます。

10. **solution をインポートする**
    - zip file ができたら、2 つの方法があります。
      - [make.powerapps.com](https://make.powerapps.com/) を使って environment に手動で [solution をインポート](https://learn.microsoft.com/powerapps/maker/data-platform/import-update-export-solutions) する。
      - あるいは、Power Apps CLI command で solution をインポートする場合は、[Connecting to your environment](https://learn.microsoft.com/powerapps/developer/component-framework/import-custom-controls#connecting-to-your-environment) と [Deployment](https://learn.microsoft.com/powerapps/developer/component-framework/import-custom-controls#deploying-code-components) を参照する。

11. **app に component を追加する**
    - 最後に、model-driven app と canvas app にコード コンポーネントを追加するには、次を参照します。
      - [model-driven apps に component を追加する](https://learn.microsoft.com/powerapps/developer/component-framework/add-custom-controls-to-a-field-or-entity)
      - [canvas app に component を追加する](https://learn.microsoft.com/powerapps/developer/component-framework/component-framework-for-canvas-apps#add-components-to-a-canvas-app)

## 利用可能な Sample Components

この repository には、次のような多数の sample component が含まれています。

- AngularJSFlipControl
- CanvasGridControl
- ChoicesPickerControl
- ChoicesPickerReactControl
- CodeInterpreterControl
- ControlStateAPI
- DataSetGrid
- DeviceApiControl
- FacepileReactControl
- FluentThemingAPIControl
- FormattingAPIControl
- IFrameControl
- ImageUploadControl
- IncrementControl
- LinearInputControl
- LocalizationAPIControl
- LookupSimpleControl
- MapControl
- ModelDrivenGridControl
- MultiSelectOptionSetControl
- NavigationAPIControl
- ObjectOutputControl
- PowerAppsGridCustomizerControl
- PropertySetTableControl
- ReactStandardControl
- TableControl
- TableGrid
- WebAPIControl

各 sample は Power Apps component framework の異なる側面を示しており、自分自身の component を作る際の学習リソースや出発点として利用できます。
