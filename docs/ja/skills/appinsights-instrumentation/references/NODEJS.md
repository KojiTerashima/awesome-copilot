## コードを変更する

アプリに必要な変更を加えます。

- クライアントライブラリをインストールする
```
npm install @azure/monitor-opentelemetry
```

- アプリが Azure Monitor を使用するように設定する  
Node.js アプリには通常、package.json の "main" プロパティに記載されたエントリファイルがあります。このファイルを見つけて、次の変更を適用してください。
  - 先頭でクライアントライブラリを require します。`const { useAzureMonitor } = require("@azure/monitor-opentelemetry");`
  - セットアップメソッドを呼び出します。`useAzureMonitor();`

> 注: セットアップメソッドはできるだけ早いタイミングで呼び出す必要がありますが、環境変数の設定後でなければなりません。これは、環境変数から App Insights の接続文字列を取得する必要があるためです。たとえば、アプリが dotenv を使って環境変数を読み込んでいる場合は、その後、かつ他の処理より前にセットアップメソッドを呼び出してください。  
> 注: アプリのコードを変更したため、反映するにはデプロイが必要です。

## App Insights の接続文字列を設定する

App Insights リソースには接続文字列があります。実行中のアプリの環境変数として、この接続文字列を追加してください。Azure CLI を使って App Insights リソースの接続文字列を取得できます。接続文字列を取得するために実行する Azure CLI コマンドは、[scripts/appinsights.ps1] を参照してください。

接続文字列を取得したら、次の環境変数にその値を設定します。

```
"APPLICATIONINSIGHTS_CONNECTION_STRING={your_application_insights_connection_string}"
```

アプリに、クラウドインスタンスを表す Bicep や terraform ファイルなどの IaC テンプレートがある場合は、この環境変数を IaC テンプレートに追加し、各デプロイで適用されるようにしてください。IaC テンプレートがない場合は、Azure CLI を使用してアプリのクラウドインスタンスにこの環境変数を手動で適用してください。この環境変数を設定するために実行する Azure CLI コマンドを確認してください。

