---
name: nuget-manager
description: '.NETプロジェクト/ソリューションのNuGetパッケージを管理します。NuGetパッケージの追加、削除、またはバージョン更新時にこのスキルを使用してください。パッケージ管理には`dotnet` CLIの使用を徹底し、バージョン更新時のみファイルの直接編集を厳格に許可します。'
---

# NuGetマネージャー

## 概要

このスキルは、.NETプロジェクト全体でNuGetパッケージの一貫性と安全な管理を保証します。`dotnet` CLIの使用を優先し、プロジェクトの整合性を維持しつつ、バージョン更新時には厳格な検証と復元のワークフローを強制します。

## 前提条件

- .NET SDKがインストールされていること（通常は.NET 8.0 SDK以降、または対象ソリューションに適合するバージョン）。
- `dotnet` CLIが`PATH`に含まれていること。
- バージョン検証に`jq`（JSONプロセッサ）またはPowerShellが使用可能であること（`dotnet package search`コマンド用）。

## 基本ルール

1.  `.csproj`、`.props`、または`Directory.Packages.props`ファイルを直接編集してパッケージを**追加**または**削除**してはなりません。必ず`dotnet add package`および`dotnet remove package`コマンドを使用してください。
2.  **直接編集**が許可されるのは、既存パッケージの**バージョン変更時のみ**です。
3.  **バージョン更新**は以下の必須ワークフローに従ってください：
    - 対象バージョンがNuGetに存在することを確認する。
    - バージョン管理がプロジェクト単位（`.csproj`）か集中管理（`Directory.Packages.props`）かを判別する。
    - 適切なファイルのバージョン文字列を更新する。
    - 直ちに`dotnet restore`を実行し互換性を検証する。

## ワークフロー

### パッケージの追加
`dotnet add [<PROJECT>] package <PACKAGE_NAME> [--version <VERSION>]`を使用します。  
例：`dotnet add src/MyProject/MyProject.csproj package Newtonsoft.Json`

### パッケージの削除
`dotnet remove [<PROJECT>] package <PACKAGE_NAME>`を使用します。  
例：`dotnet remove src/MyProject/MyProject.csproj package Newtonsoft.Json`

### パッケージバージョンの更新
バージョン更新時は以下の手順に従ってください：

1.  **バージョンの存在確認**：  
    `dotnet package search`コマンドで正確なバージョンが存在するかJSON形式で確認します。  
    `jq`使用例：  
    `dotnet package search <PACKAGE_NAME> --exact-match --format json | jq -e '.searchResult[].packages[] | select(.version == "<VERSION>")'`  
    PowerShell使用例：  
    `(dotnet package search <PACKAGE_NAME> --exact-match --format json | ConvertFrom-Json).searchResult.packages | Where-Object { $_.version -eq "<VERSION>" }`

2.  **バージョン管理方法の判別**：  
    - ソリューションルートに`Directory.Packages.props`があれば、バージョンはそこにある`<PackageVersion Include="Package.Name" Version="1.2.3" />`で管理されている可能性が高いです。  
    - なければ、個別の`.csproj`ファイル内の`<PackageReference Include="Package.Name" Version="1.2.3" />`を確認してください。

3.  **変更の適用**：  
    該当ファイルのバージョン文字列を新しいものに書き換えます。

4.  **安定性の検証**：  
    プロジェクトまたはソリューションで`dotnet restore`を実行し、エラーが発生した場合は変更を元に戻し原因を調査してください。

## 例

### ユーザー：「WebApiプロジェクトにSerilogを追加して」
**操作**：`dotnet add src/WebApi/WebApi.csproj package Serilog`を実行。

### ユーザー：「Newtonsoft.Jsonをソリューション全体で13.0.3に更新して」
**操作**：  
1. 13.0.3が存在するか確認：`dotnet package search Newtonsoft.Json --exact-match --format json`（出力を解析し"13.0.3"があることを確認）。  
2. 定義場所を特定（例：`Directory.Packages.props`）。  
3. ファイルを編集しバージョンを更新。  
4. `dotnet restore`を実行。
