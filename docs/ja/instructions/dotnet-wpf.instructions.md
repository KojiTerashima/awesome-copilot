---
description: '.NET WPF コンポーネントおよびアプリケーション パターン'
applyTo: '**/*.xaml, **/*.cs'
---

## 概要

これらの instruction は、MVVM パターンを用いた高品質で保守しやすく、高性能な WPF アプリケーションの構築を GitHub Copilot が支援できるよう導きます。XAML、データ バインディング、UI 応答性、.NET パフォーマンスに関するベストプラクティスを含みます。

## 理想的なプロジェクト種別

- C# と WPF を使用するデスクトップ アプリケーション
- MVVM (Model-View-ViewModel) デザイン パターンに従うアプリケーション
- .NET 8.0 以降を使用するプロジェクト
- XAML で構築された UI コンポーネント
- パフォーマンスと応答性を重視するソリューション

## 目標

- `INotifyPropertyChanged` と `RelayCommand` のボイラープレートを生成する
- ViewModel と View ロジックの明確な分離を提案する
- `ObservableCollection<T>`、`ICommand`、適切なバインディングの使用を促す
- パフォーマンス改善のヒント (例: 仮想化、非同期読み込み) を推奨する
- code-behind ロジックの密結合を避ける
- テストしやすい ViewModel を生成する

## プロンプト挙動の例

### ✅ 良い提案
- "ユーザー名とパスワードのプロパティ、および LoginCommand を持つログイン画面用の ViewModel を生成して"
- "UI 仮想化を使い、ObservableCollection にバインドする ListView の XAML スニペットを書いて"
- "この code-behind の click handler を ViewModel の RelayCommand にリファクタリングして"
- "WPF で非同期にデータを取得している間、ローディング スピナーを追加して"

### ❌ 避けること
- code-behind にビジネス ロジックを提案すること
- 文脈なしで static event handler を使うこと
- バインディングのない密結合な XAML を生成すること
- WinForms や UWP のアプローチを提案すること

## 優先する技術
- .NET 8.0+ の C#
- MVVM 構造を持つ XAML
- `CommunityToolkit.Mvvm` または独自の `RelayCommand` 実装
- UI をブロックしないための async/await
- `ObservableCollection`、`ICommand`、`INotifyPropertyChanged`

## 従うべき一般的なパターン
- ViewModel ファーストのバインディング
- .NET またはサードパーティー コンテナー (例: Autofac, SimpleInjector) を使った依存性注入
- XAML の命名規則 (コントロールは PascalCase、バインディングは camelCase)
- バインディングでマジック文字列を避ける (`nameof` を使用)

## Copilot が利用できるサンプル instruction スニペット

```csharp
public class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string userName;

    [ObservableProperty]
    private string password;

    [RelayCommand]
    private void Login()
    {
        // ここにログイン ロジックを追加
    }
}
```

```xml
<StackPanel>
    <TextBox Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged}" />
    <PasswordBox x:Name="PasswordBox" />
    <Button Content="Login" Command="{Binding LoginCommand}" />
</StackPanel>
```
