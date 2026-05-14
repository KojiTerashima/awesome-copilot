---
name: appinsights-instrumentation
description: 'Web アプリを計装して、有用なテレメトリ データを Azure App Insights に送信します'
---

# AppInsights の計装

このスキルを使うと、Web アプリのテレメトリ データを Azure App Insights に送信でき、アプリの健全性に関する可観測性を向上できます。

## このスキルを使うタイミング

ユーザーが自分の Web アプリでテレメトリを有効化したい場合に、このスキルを使用します。

## 前提条件

ワークスペース内のアプリは、次のいずれかの種類である必要があります。

- Azure でホストされている ASP.NET Core アプリ
- Azure でホストされている Node.js アプリ

## ガイドライン

### コンテキスト情報を収集する

ユーザーがテレメトリ対応を追加しようとしているアプリについて、（プログラミング言語、アプリケーション フレームワーク、ホスティング）の組み合わせを把握してください。これによって、アプリをどのように計装できるかが決まります。ソースコードを読み、根拠のある推測を行ってください。不明な点はユーザーに確認してください。アプリがどこでホストされているか（例: 個人のコンピューター上、Azure App Service のコードとして、Azure App Service のコンテナーとして、Azure Container App 上 など）は、必ずユーザーに確認する必要があります。 

### 可能であれば自動計装を優先する

アプリが Azure App Service でホストされている C# ASP.NET Core アプリの場合は、[AUTO guide](references/AUTO.md) を使って、ユーザーがアプリを自動計装できるよう支援してください。

### 手動で計装する

AppInsights リソースを作成し、アプリのコードを更新して手動で計装します。 

#### AppInsights リソースを作成する

環境に適した次のいずれかの方法を使用します。

- 既存の Bicep テンプレートに AppInsights を追加する。追加内容は [examples/appinsights.bicep](examples/appinsights.bicep) を参照してください。ワークスペースに既存の Bicep テンプレート ファイルがある場合、これが最適な方法です。
- Azure CLI を使う。App Insights リソース作成時に実行する Azure CLI コマンドは [scripts/appinsights.ps1](scripts/appinsights.ps1) を参照してください。

どの方法を選ぶ場合でも、リソース管理をしやすくするため、意味のあるリソース グループに App Insights リソースを作成するようユーザーに推奨してください。Azure 上でホストされているアプリのリソースを含む同じリソース グループが、有力な候補です。

#### アプリケーション コードを変更する

- アプリが ASP.NET Core アプリの場合、C# コードの変更方法は [ASPNETCORE guide](references/ASPNETCORE.md) を参照してください。
- アプリが Node.js アプリの場合、JavaScript/TypeScript コードの変更方法は [NODEJS guide](references/NODEJS.md) を参照してください。
- アプリが Python アプリの場合、Python コードの変更方法は [PYTHON guide](references/PYTHON.md) を参照してください。

