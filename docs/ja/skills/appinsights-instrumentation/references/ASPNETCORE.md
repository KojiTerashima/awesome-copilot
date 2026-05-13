## コードを修正する

アプリに必要な変更を加えます。

- クライアント ライブラリをインストールする
```
dotnet add package Azure.Monitor.OpenTelemetry.AspNetCore
```

- Azure Monitor を使用するようにアプリを構成する  
ASP.NET Core アプリには通常、アプリを「ビルド」する Program.cs ファイルがあります。このファイルを見つけて、次の変更を適用してください。
  - 先頭に `using Azure.Monitor.OpenTelemetry.AspNetCore;` を追加する
  - `builder.Build()` を呼び出す前に、この 1 行 `builder.Services.AddOpenTelemetry().UseAzureMonitor();` を追加する。

> 注: アプリのコードを変更したため、反映するにはアプリをデプロイする必要があります。

## App Insights の接続文字列を構成する

App Insights リソースには接続文字列があります。実行中のアプリの環境変数として、この接続文字列を追加してください。Azure CLI を使用して App Insights リソースの接続文字列を取得できます。接続文字列を取得するために実行する Azure CLI コマンドは、[scripts/appinsights.ps1](scripts/appinsights.ps1) を参照してください。

接続文字列を取得したら、その値で次の環境変数を設定してください。

```
"APPLICATIONINSIGHTS_CONNECTION_STRING={your_application_insights_connection_string}"
```

アプリのクラウド インスタンスを表す Bicep や terraform ファイルなどの IaC テンプレートがある場合は、この環境変数を IaC テンプレートに追加し、各デプロイ時に適用されるようにしてください。そうでない場合は、Azure CLI を使用してアプリのクラウド インスタンスに手動で環境変数を適用してください。この環境変数を設定するために実行する Azure CLI コマンドは、[scripts/appinsights.ps1](scripts/appinsights.ps1) を参照してください。

> 重要: appsettings.json は変更しないでください。これは App Insights を構成するための非推奨の方法です。環境変数が新しい推奨方法です。

