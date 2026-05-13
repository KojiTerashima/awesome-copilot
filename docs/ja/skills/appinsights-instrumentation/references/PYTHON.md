## コードを変更する

アプリに必要な変更を加えます。

- クライアント ライブラリをインストールする
```
pip install azure-monitor-opentelemetry
```

- Azure Monitor を使用するようにアプリを構成する  
Python アプリケーションは、Python 標準ライブラリの logger クラスを介してテレメトリを送信します。テレメトリを送信できる logger を構成して作成するモジュールを作成してください。

```python
import logging
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor(
    logger_name="<your_logger_namespace>"
)
logger = logging.getLogger("<your_logger_namespace>")
```

> 注: アプリのコードを変更したため、反映するにはデプロイが必要です。

## App Insights の接続文字列を構成する

App Insights リソースには接続文字列があります。実行中のアプリの環境変数として接続文字列を追加してください。Azure CLI を使用して App Insights リソースの接続文字列を照会できます。接続文字列を照会するために実行する Azure CLI コマンドは [scripts/appinsights.ps1] を参照してください。

接続文字列を取得したら、その値で次の環境変数を設定します。

```
"APPLICATIONINSIGHTS_CONNECTION_STRING={your_application_insights_connection_string}"
```

アプリに、そのクラウド インスタンスを表す Bicep や terraform ファイルなどの IaC テンプレートがある場合は、この環境変数を IaC テンプレートに追加し、各デプロイで適用されるようにする必要があります。そうでない場合は、Azure CLI を使用してアプリのクラウド インスタンスにこの環境変数を手動で適用してください。この環境変数を設定するために実行する Azure CLI コマンドを確認してください。

## データを送信する

テレメトリを送信するように構成された logger を作成します。
```python
logger = logging.getLogger("<your_logger_namespace>")
logger.setLevel(logging.INFO)
```

次に、その logging メソッドを呼び出してテレメトリ イベントを送信します。
```python
logger.info("info log")
```

