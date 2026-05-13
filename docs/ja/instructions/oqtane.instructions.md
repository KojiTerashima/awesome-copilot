---
description: 'Oqtane Module パターン'
applyTo: '**/*.razor, **/*.razor.cs, **/*.razor.css'
---

## Blazor のコードスタイルと構造

- 慣用的で効率のよい Blazor と C# のコードを書く。
- .NET と Blazor の規約に従う。
- component ベースの UI 開発には Razor Component を適切に使う。
- component ベースの UI 開発には Blazor Component を適切に使う。
- 小さな component では inline function を優先するが、複雑な logic は code-behind や service class に分離する。
- UI をブロックしないため、適用できる箇所では async/await を使う。


## 命名規約

- component 名、method 名、public member には PascalCase を使う。
- private field と local variable には camelCase を使う。
- interface 名には "I" を接頭辞として付ける（例: IUserService）。

## Blazor と .NET 固有のガイドライン

- component lifecycle（例: OnInitializedAsync、OnParametersSetAsync）には Blazor の組み込み機能を活用する。
- `@bind` による data binding を効果的に使う。
- Blazor の service には Dependency Injection を活用する。
- Blazor component と service は Separation of Concerns に従って構成する。
- 常に最新の C#、現在なら C# 13 の record type、pattern matching、global usings などを使う。

## Oqtane 固有のガイドライン
- base class と pattern は [Main Oqtane repo](https://github.com/oqtane/oqtane.framework) を参照する。
- module 開発では client / server pattern に従う。
- Client project には modules folder 内にさまざまな module がある。
- client module の各 action は別々の razor file とし、`index.razor` は default action として ModuleBase を継承する。
- data 取得のような複雑な client 処理では、ServiceBase を継承した service class を作成し services folder に配置する。module ごとに service class は 1 つにする。
- Client service は ServiceBase method を使って server endpoint を呼び出す。
- Server project には MVC Controller があり、module ごとに 1 つずつ作成して client service call と一致させる。各 controller は DI で管理される server-side service または repository を呼び出す。
- Server project では module ごとに 1 つの repository class を持つ repository pattern を使い、controller に対応させる。

## Error Handling と Validation

- Blazor page と API call には適切な error handling を実装する。
- base class から利用できる Oqtane の組み込み logging method を使う。
- backend の error tracking には logging を使い、UI レベルの error は ErrorBoundary などで捉えることを検討する。
- form の validation には FluentValidation または DataAnnotations を使う。

## Blazor API とパフォーマンス最適化

- project の要件に応じて、Blazor server-side または WebAssembly を最適に使い分ける。
- API call や UI action で main thread をブロックし得る箇所では、非同期 method（async/await）を使う。
- Razor component では不要な render を減らし、`StateHasChanged()` を効率よく使って最適化する。
- 必要がない限り再 render しないようにして component render tree を最小化し、必要なら `ShouldRender()` を使う。
- user interaction の処理には EventCallback を使い、発火時に渡す data は最小限にする。

## Caching Strategy

- 頻繁に使う data、特に Blazor Server app の data には in-memory caching を実装する。軽量な caching には IMemoryCache を使う。
- Blazor WebAssembly では、user session をまたいだ application state の保持に localStorage または sessionStorage を活用する。
- 複数 user / client 間で共有 state が必要な大規模 app では、Distributed Cache（Redis や SQL Server Cache など）を検討する。
- API response を保存して不要な再呼び出しを避けることで、user experience を改善する。

## State Management Library

- component 間で基本的な state 共有を行うには、Blazor 組み込みの Cascading Parameter と EventCallback を使う。
- 適切な場面では、PageState や SiteState のような base class の Oqtane 組み込み state management を使う。
- app が複雑になっても、Fluxor や BlazorState のような追加 dependency は避ける。
- Blazor WebAssembly における client-side state persistence には、page reload をまたいで state を維持するために Blazored.LocalStorage または Blazored.SessionStorage を検討する。
- server-side Blazor では、Scoped Service と StateContainer pattern を使って user session 内の state を管理しつつ、不要な再 render を抑える。

## API 設計と統合

- 外部 API や server project backend との通信には service base method を使う。
- API call の error は try-catch で処理し、UI には適切な user feedback を返す。

## Visual Studio でのテストとデバッグ

- unit testing と integration testing はすべて Visual Studio Enterprise で行う。
- Blazor component と service の test には xUnit、NUnit、または MSTest を使う。
- test 時の dependency の mock には Moq または NSubstitute を使う。
- Blazor UI の問題は browser developer tool で、backend / server-side の問題は Visual Studio の debugging tool で調査する。
- パフォーマンス profiling と最適化には Visual Studio の diagnostics tool を使う。

## セキュリティと認証

- Authentication と Authorization は、User.Roles のような Oqtane base class member を使って実装する。
- すべての web communication で HTTPS を使い、適切な CORS policy を実装する。
