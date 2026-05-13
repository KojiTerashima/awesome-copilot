# アプリを自動インストルメント化する

Azure App Service でホストされている Web アプリを、コードを一切変更せずに App Insights 向けに自動インストルメント化するには、Azure Portal を使用します。自動インストルメント化できるのは、次の種類のアプリのみです。[サポートされている環境と言語、およびリソース プロバイダー](https://learn.microsoft.com/azure/azure-monitor/app/codeless-overview#supported-environments-languages-and-resource-providers) を参照してください。

- Azure App Service でホストされている ASP.NET Core アプリ
- Azure App Service でホストされている Node.js アプリ

ユーザーを Azure Portal の App Service アプリ用 Application Insights ブレードに移動させる URL を組み立てます。
```
https://portal.azure.com/#resource/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Web/sites/{app_service_name}/monitoringSettings
```

コンテキストを利用するか、ユーザーに確認して、Web アプリをホストしている subscription_id、resource_group_name、app_service_name を取得してください。

