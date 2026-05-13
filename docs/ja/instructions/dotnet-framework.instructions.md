---
description: '.NET Framework プロジェクトで作業するためのガイダンス。プロジェクト構造、C# 言語バージョン、NuGet 管理、ベストプラクティスを含みます。'
applyTo: '**/*.csproj, **/*.cs'
---

# .NET Framework 開発

## ビルドとコンパイル要件
- ソリューションまたはプロジェクトのビルドには、`dotnet build` ではなく常に `msbuild /t:rebuild` を使用します

## プロジェクト ファイル管理

### 非 SDK スタイルのプロジェクト構造
.NET Framework プロジェクトは旧来のプロジェクト形式を使用しており、現代的な SDK スタイルのプロジェクトとは大きく異なります。

- **明示的なファイル追加**: 新しいソース ファイルは、`<Compile>` 要素を使って必ずプロジェクト ファイル (`.csproj`) に明示的に追加する必要があります
  - .NET Framework プロジェクトは、SDK スタイルのプロジェクトのようにディレクトリ内のファイルを自動的には取り込みません
  - 例: `<Compile Include="Path\To\NewFile.cs" />`

- **暗黙の import はない**: SDK スタイルのプロジェクトと異なり、.NET Framework プロジェクトは一般的な名前空間やアセンブリを自動 import しません

- **ビルド構成**: Debug/Release 構成向けの明示的な `<PropertyGroup>` セクションを含みます

- **出力パス**: 明示的な `<OutputPath>` と `<IntermediateOutputPath>` の定義があります

- **ターゲット フレームワーク**: `<TargetFramework>` ではなく `<TargetFrameworkVersion>` を使用します
  - 例: `<TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>`

## NuGet パッケージ管理
- .NET Framework プロジェクトでの NuGet パッケージのインストールや更新は、複数ファイルにまたがる調整が必要な複雑な作業です。そのため、このプロジェクトでは **NuGet パッケージのインストールや更新を試みないでください**。
- 代わりに、NuGet 参照の変更が必要な場合は、Visual Studio の NuGet Package Manager または Visual Studio package manager console を使用して、ユーザー自身にインストールまたは更新してもらうよう依頼してください。
- NuGet パッケージを推奨する際は、.NET Framework または .NET Standard 2.0 と互換性があることを確認してください (.NET Core または .NET 5+ 専用のものは不可)。

## C# 言語バージョンは 7.3
- このプロジェクトでは C# 7.3 の機能のみ使用できます。次の機能は使わないでください。

### C# 8.0+ の機能 (未サポート):
  - using 宣言 (`using var stream = ...`)
  - await using ステートメント (`await using var resource = ...`)
  - switch 式 (`variable switch { ... }`)
  - null 合体代入 (`??=`)
  - Range と Index 演算子 (`array[1..^1]`, `array[^1]`)
  - 既定インターフェイス メソッド
  - 構造体内の readonly メンバー
  - static ローカル関数
  - null 許容参照型 (`string?`, `#nullable enable`)

### C# 9.0+ の機能 (未サポート):
  - records (`public record Person(string Name)`)
  - init 専用プロパティ (`{ get; init; }`)
  - トップレベル プログラム (Main メソッドを持たない program)
  - 拡張されたパターン マッチング
  - ターゲット型推論 new 式 (`List<string> list = new()`)

### C# 10+ の機能 (未サポート):
  - グローバル using ステートメント
  - ファイル スコープ名前空間
  - record struct
  - required メンバー

### 代わりに使うもの (C# 7.3 互換):
  - 波かっこ付きの従来の using ステートメント
  - switch 式の代わりに switch ステートメント
  - null 合体代入の代わりに明示的な null チェック
  - 手動インデックス指定による配列スライス
  - 既定インターフェイス メソッドの代わりに抽象クラスまたはインターフェイス

## 環境上の考慮事項 (Windows 環境)
- Windows スタイルのパス区切り (`C:\path\to\file.cs` など) を使用します
- ターミナル操作を提案する場合は Windows に適したコマンドを使用します
- ファイル システム操作では Windows 固有の挙動を考慮します

## よくある .NET Framework の落とし穴とベストプラクティス

### Async/Await パターン
- **ConfigureAwait(false)**: デッドロックを避けるため、ライブラリ コードでは常に `ConfigureAwait(false)` を使用します
  ```csharp
  var result = await SomeAsyncMethod().ConfigureAwait(false);
  ```
- **sync-over-async を避ける**: `.Result`、`.Wait()`、`.GetAwaiter().GetResult()` は使わないでください。これらはデッドロックや性能低下の原因になります。非同期呼び出しでは常に `await` を使用します。

### DateTime の扱い
- **タイムスタンプには DateTimeOffset を使う**: 絶対時刻には `DateTime` より `DateTimeOffset` を優先します
- **DateTimeKind を明示する**: `DateTime` を使う場合は常に `DateTimeKind.Utc` または `DateTimeKind.Local` を指定します
- **カルチャを意識した書式化**: シリアライズ/パースには `CultureInfo.InvariantCulture` を使用します

### 文字列操作
- **連結には StringBuilder**: 複数の文字列連結には `StringBuilder` を使用します
- **StringComparison**: 文字列操作では必ず `StringComparison` を指定します
  ```csharp
  string.Equals(other, StringComparison.OrdinalIgnoreCase)
  ```

### メモリ管理
- **Dispose パターン**: アンマネージド リソースには `IDisposable` を正しく実装します
- **using ステートメント**: `IDisposable` オブジェクトは常に using ステートメントで囲みます
- **Large Object Heap を避ける**: LOH 割り当てを避けるため、オブジェクトは 85KB 未満に保ちます

### 構成
- **ConfigurationManager を使う**: アプリ設定は `ConfigurationManager.AppSettings` から取得します
- **接続文字列**: `<appSettings>` ではなく `<connectionStrings>` セクションに保存します
- **変換**: 環境別設定には web.config/app.config 変換を使います

### 例外処理
- **具体的な例外**: 汎用的な `Exception` ではなく、具体的な例外型を捕捉します
- **例外を握りつぶさない**: 常に例外をログ出力するか、適切に再スローします
- **破棄が必要なリソースには using を使う**: 例外発生時でも適切にクリーンアップされます

### パフォーマンス上の考慮
- **ボックス化を避ける**: 値型とジェネリックでの boxing/unboxing に注意します
- **文字列 intern**: 頻繁に使う文字列には `string.Intern()` を慎重に使用します
- **遅延初期化**: コストの高いオブジェクト生成には `Lazy<T>` を使います
- **ホットパスで reflection を避ける**: 必要な場合は `MethodInfo`、`PropertyInfo` をキャッシュします
