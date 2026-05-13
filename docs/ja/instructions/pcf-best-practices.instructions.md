---
description: 'PCF コード コンポーネント開発のベスト プラクティスとガイダンス'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj,css,html}'
---

# コード コンポーネントのベスト プラクティスとガイダンス

コード コンポーネントの開発、デプロイ、保守には、複数領域にまたがる知識の組み合わせが必要です。この記事では、コード コンポーネントを開発する専門家向けに、確立された best practice とガイダンスを示します。

## Power Apps Component Framework

### 開発ビルドを Dataverse にデプロイしない

コード コンポーネントは [production mode または development mode](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/code-components-alm#building-pcfproj-code-component-projects) で build できます。development build を Dataverse にデプロイすると、パフォーマンスへ悪影響を及ぼし、サイズの大きさによってはデプロイ自体がブロックされる場合もあるため避けてください。後から release build をデプロイする予定でも、自動化された release pipeline がないと再デプロイを忘れやすくなります。詳細情報: [Debugging custom controls](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls)

### サポートされない Framework Method を使わない

これには、`ComponentFramework.Context` 上に存在する未公開の internal method の利用が含まれます。これらの method は動作するかもしれませんが、サポート対象ではないため、将来の version で動かなくなる可能性があります。host application の HTML Document Object Model (DOM) に access する control script の利用もサポートされていません。コード コンポーネント境界の外側にある host application DOM の部分は、予告なく変更される可能性があります。

### `init` Method を使って network 必須 resource を要求する

hosting context がコード コンポーネントを読み込むと、最初に `init` method が呼び出されます。`updateView` method を待つのではなく、この method を使って metadata などの network resource を要求してください。要求が戻る前に `updateView` method が呼ばれた場合に備え、コード コンポーネントはこの状態を処理し、視覚的な loading indicator を提供する必要があります。

### `destroy` Method 内で resource を cleanup する

hosting context は、コード コンポーネントが browser DOM から削除されたときに `destroy` method を呼び出します。`destroy` method を使って `WebSockets` を閉じ、container element の外側に追加した event handler を削除してください。React を使っている場合は、`destroy` method 内で `ReactDOM.unmountComponentAtNode` を使います。このように resource を cleanup することで、特定の browser session 内でコード コンポーネントが読み込まれたり解除されたりする際に起こるパフォーマンス問題を防げます。

### Dataset Property の Refresh 呼び出しを不要に行わない

コード コンポーネントが dataset type の場合、bound dataset property は hosting context にデータ再読み込みをさせる `refresh` method を公開します。この method を不必要に呼ぶと、コード コンポーネントのパフォーマンスに悪影響を与えます。

### `notifyOutputChanged` の呼び出しを最小限にする

状況によっては、UI control の更新 (key press や mouse move event など) ごとに `notifyOutputChanged` を呼ぶのは望ましくありません。呼び出しが増えると、必要以上に多くの event が親 context に伝播するためです。代わりに、control が focus を失ったときや、ユーザーの touch / mouse event が完了したときの event を使うことを検討してください。

### API の利用可否を確認する

異なる host (model-driven apps、canvas apps、portals) 向けにコード コンポーネントを開発する場合は、使っている API がそれらの platform でサポートされているかを常に確認してください。たとえば `context.webAPI` は canvas apps では利用できません。個別の API 利用可否については、[Power Apps component framework API reference](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/) を参照してください。

### `updateView` に渡される一時的に Null な Property 値を扱う

データの準備ができていないとき、null 値が `updateView` method に渡されます。component はこの状況を考慮し、データが null になり得ること、そして後続の `updateView` cycle で更新済み値が渡され得ることを前提に作る必要があります。`updateView` は [standard](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/control/updateview) component と [React](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/react-control/updateview) component の両方で利用可能です。

## Model-Driven Apps

### `formContext` と直接やり取りしない

client API を扱った経験がある場合、attribute や control に access したり、`save`、`refresh`、`setNotification` のような API method を呼ぶために `formContext` とやり取りすることに慣れているかもしれません。コード コンポーネントは model-driven apps、canvas apps、dashboard などさまざまな product で動作することが期待されるため、`formContext` に依存してはいけません。

回避策として、コード コンポーネントを column に bind し、その column に `OnChange` event handler を追加する方法があります。コード コンポーネントは column 値を更新でき、`OnChange` event handler は `formContext` に access できます。将来は custom event のサポートが追加され、column 構成を増やさずに control の外へ変更を伝えられるようになります。

### `WebApi` への呼び出しのサイズと頻度を制限する

`context.WebApi` method を使う場合は、呼び出し回数とデータ量の両方を制限してください。`WebApi` を呼び出すたびに、ユーザーの API entitlement と service protection limit にカウントされます。record に対する CRUD 操作を行う場合は payload のサイズも考慮してください。一般に、request payload が大きいほどコード コンポーネントは遅くなります。

## Canvas Apps

### 画面上の Component 数を最小限にする

canvas app に component を 1 つ追加するたびに、render に一定の時間がかかります。render 時間は component を追加するごとに増えます。Developer Performance tool を使って、screen に追加するコード コンポーネントの数が増える中でのパフォーマンスを慎重に測定してください。

現在、各コード コンポーネントは Fluent UI や React などの shared library を自前で bundle しています。同じ library の複数 instance を読み込んでも、それらの library が何度も読み込まれるわけではありません。ただし、異なる複数のコード コンポーネントを読み込むと、browser はこれらの library の複数の bundle version を読み込みます。将来的には、これらの library はコード コンポーネント間で読み込んで共有できるようになります。

### Maker がコード コンポーネントを style できるようにする

app maker が canvas app 内からコード コンポーネントを利用するとき、他の app と一致する style を使いたいと考えます。色やサイズなどの theme 要素向けに、input property を使って customization option を提供してください。Microsoft Fluent UI を使う場合は、これらの property を library が提供する theme 要素にマッピングします。将来的には、このプロセスを容易にするためコード コンポーネントに theming support が追加されます。

### Canvas Apps のパフォーマンス Best Practice に従う

canvas apps には、app 内や solution checker から利用できる広範な best practice があります。コード コンポーネントを追加する前に、app がこれらの推奨に従っていることを確認してください。詳細は次を参照してください。

- [Tips to improve canvas app performance](https://learn.microsoft.com/en-us/powerapps/maker/canvas-apps/performance-tips)
- [Considerations for optimized performance in Power Apps](https://powerapps.microsoft.com/blog/considerations-for-optimized-performance-in-power-apps/)

## TypeScript と JavaScript

### ES5 vs ES6

既定では、コード コンポーネントは古い browser をサポートするため ES5 を target にしています。これらの古い browser をサポートしたくない場合は、`pcfproj` folder 内の `tsconfig.json` で target を ES6 に変更できます。詳細情報: [ES5 vs ES6](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls#es5-vs-es6)

### Module Imports

`SCRIPT` tag で読み込む必要がある script を使うのではなく、コード コンポーネントに必要な module は常に bundle してください。たとえば、sample では page に `<script type="text/javascript" src="somechartlibrary.js></script>` を追加して使うような非 Microsoft の charting API を使いたい場合でも、これはコード コンポーネント内ではサポートされません。必要な module をすべて bundle することで、コード コンポーネントは他の library から隔離され、offline mode でも実行できます。

> **注**: 現時点では、component manifest の library node を使った component 間の shared library はサポートされていません。

### Linting

linting とは、tool が code を走査して潜在的な問題を見つけることです。`pac pcf init` が使う template は `eslint` module を project にインストールし、`.eslintrc.json` file を追加して設定します。

設定するには、command-line で次を使います。

```bash
npx eslint --init
```

その後、prompt されたら次のように回答します。

- **How would you like to use ESLint?** Answer: To check syntax, find problems, and enforce code style
- **What type of modules does your project use?** Answer: JavaScript modules (import/export)
- **Which framework does your project use?** Answer: React
- **Does your project use TypeScript?** Answer: Yes
- **Where does your code run?** Answer: Browser
- **How would you like to define a style for your project?** Answer: Answer questions about your style
- **What format do you want your config file to be in?** Answer: JSON
- **What style of indentation do you use?** Answer: Spaces
- **What quotes do you use for strings?** Answer: Single
- **What line endings do you use?** Answer: Windows
- **Do you require semicolons?** Answer: Yes

`eslint` を使う前に、`package.json` にいくつか script を追加する必要があります。

```json
"scripts": {
   ...
   "lint": "eslint MY_CONTROL_NAME --ext .ts,.tsx",
   "lint:fix": "npm run lint -- --fix"
}
```

その後、command-line から次を使えます。

```bash
npm run lint:fix
```

さらに、`.eslintrc.json` に追加して無視する file を設定できます。

```json
"ignorePatterns": ["**/generated/*.ts"]
```

## HTML Browser UI 開発

### Microsoft Fluent UI React を使う

[Fluent UI React](https://developer.microsoft.com/fluentui#/get-started/web) は、広範な Microsoft 製品にシームレスに適合する体験を構築するために設計された、公式の [open source](https://github.com/microsoft/fluentui) React front-end framework です。Power Apps 自身も Fluent UI を使っているため、アプリ全体と一貫した UI を作成できます。

#### Bundle Size を減らすため Fluent の Path-Based Import を使う

現在、`pac pcf init` で使われるコード コンポーネント template は、`webpack` が使われていない import 済み module を検出して取り除く tree-shaking を使いません。次の command で Fluent UI から import すると、library 全体を import して bundle してしまいます。

```typescript
import { Button } from '@fluentui/react'
```

library 全体の import / bundle を避けるには、明示的な path で特定 component を import する path-based import を使えます。

```typescript
import { Button } from '@fluentui/react/lib/Button';
```

特定 path を使うことで、development build と release build の両方で bundle size を減らせます。

#### React Rendering を最適化する

React を使う場合は、component render を最小限にする React 固有の best practice に従ってください。

- bound property または framework aspect の変化により UI を反映させる必要がある場合にのみ、`updateView` method 内で `ReactDOM.render` を呼び出す。何が変わったかは `updatedProperties` を使って判断できます。
- 不要な再 render を避けるため、可能な限り `PureComponent` (class component) または `React.memo` (function component) を使う。
- 大きな React component では、パフォーマンス向上のため UI を小さな component に分割する。
- render 関数内での arrow function や function binding は、各 render ごとに新しい callback closure を作るため避ける。

### アクセシビリティを確認する

コード コンポーネントは keyboard のみのユーザーや screen reader ユーザーが使えるよう、accessible にしてください。

- mouse / touch event に対する keyboard navigation の代替手段を提供する
- screen reader がコード コンポーネント interface を正確に読み上げられるよう、`alt` および [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) (Accessible Rich Internet Applications) attribute を設定する
- 現代の browser 開発ツールには、アクセシビリティを確認する便利な手段がある

詳細情報: [Create accessible canvas apps in Power Apps](https://learn.microsoft.com/en-us/powerapps/maker/canvas-apps/accessible-apps)

### Network Call では常に非同期を使う

network call を行うときは、同期ブロッキング request を決して使わないでください。アプリが応答しなくなり、パフォーマンス低下の原因になります。詳細情報: [Interact with HTTP and HTTPS resources asynchronously](https://learn.microsoft.com/en-us/powerapps/developer/model-driven-apps/best-practices/business-logic/interact-http-https-resources-asynchronously)

### 複数 Browser を前提に code を書く

model-driven apps、canvas apps、portals はいずれも複数 browser をサポートしています。現代的な browser すべてでサポートされる手法だけを使い、対象ユーザーに対して代表的な browser 群でテストしてください。

- [Limits and configurations](https://learn.microsoft.com/en-us/powerapps/maker/canvas-apps/limits-and-config)
- [Supported web browsers](https://learn.microsoft.com/en-us/power-platform/admin/supported-web-browsers-and-mobile-devices)
- [Browsers used by office](https://learn.microsoft.com/en-us/office/dev/add-ins/concepts/browsers-used-by-office-web-add-ins)

### コード コンポーネントは複数 Client と Screen Format の対応を計画するべき

コード コンポーネントは、複数 client (model-driven apps、canvas apps、portals) と screen format (mobile、tablet、web) で render される可能性があります。

- `trackContainerResize` を使うと、利用可能な幅と高さの変化にコード コンポーネントが応答できる
- `allocatedHeight` と `allocatedWidth` は `getFormFactor` と組み合わせることで、mobile、tablet、web のどれで動作しているかを判断できる
- `setFullScreen` を実装すると、space が限られる場面でユーザーが使用可能な全画面へ拡張できる
- 与えられた container size で意味のある体験を提供できない場合は、適切に機能を無効化し、ユーザーにフィードバックを与えるべき

### CSS Rule は常に Scoped にする

CSS を使ってコード コンポーネントに style を実装するときは、その CSS が component の container `DIV` element に自動生成される CSS class を使って、自身の component に scope されるようにしてください。CSS が global に scope されていると、コード コンポーネントが render される form や screen の既存 style を壊す可能性があります。

たとえば namespace が `SampleNamespace` で、コード コンポーネント名が `LinearInputComponent` の場合、次のように custom CSS rule を追加します。

```css
.SampleNamespace\.LinearInputComponent rule-name
```

### Web Storage Object の使用を避ける

コード コンポーネントでは、`window.localStorage` や `window.sessionStorage` のような HTML web storage object を使ってデータを保存すべきではありません。ユーザーの browser や mobile client にローカル保存されたデータは安全ではなく、安定して利用できる保証もありません。

## ALM / Azure DevOps / GitHub

ALM / Azure DevOps / GitHub とコード コンポーネントに関する best practice については、[Code component application lifecycle management (ALM)](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/code-components-alm) の記事を参照してください。

## 関連記事

- [コード コンポーネントとは何か](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/custom-controls-overview)
- [canvas apps 向けコード コンポーネント](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/component-framework-for-canvas-apps)
- [コード コンポーネントを作成してビルドする](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/create-custom-controls-using-pcf)
- [Power Apps component framework を学ぶ](https://learn.microsoft.com/en-us/training/paths/use-power-apps-component-framework)
- [Power Pages でコード コンポーネントを使う](https://learn.microsoft.com/en-us/power-apps/maker/portals/component-framework)
