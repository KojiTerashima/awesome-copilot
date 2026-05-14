---
name: dotnet-best-practices
description: '.NET/C# コードがソリューション/プロジェクトのベストプラクティスを満たすようにします。'
---

# .NET/C# Best Practices

あなたのタスクは、${selection} にある .NET/C# コードがこのソリューション/プロジェクト固有のベスト プラクティスを満たすようにすることです。これには次が含まれます。

## Documentation & Structure

- すべての public class、interface、method、property に対して包括的な XML documentation comment を作成する
- XML comment に parameter の説明と return value の説明を含める
- 既存の namespace 構造に従う: {Core|Console|App|Service}.{Feature}

## Design Patterns & Architecture

- 依存性注入には primary constructor 構文を使う (例: `public class MyClass(IDependency dependency)`)
- generic base class を使った Command Handler パターンを実装する (例: `CommandHandler<TOptions>`)
- 明確な命名規約で interface segregation を使う (interface 名は `I` で始める)
- 複雑なオブジェクト生成には Factory パターンに従う。

## Dependency Injection & Services

- constructor dependency injection を使い、ArgumentNullException による null check を行う
- 適切な lifetime (Singleton、Scoped、Transient) で service を登録する
- Microsoft.Extensions.DependencyInjection のパターンを使う
- テスト容易性のために service interface を実装する

## Resource Management & Localization

- ローカライズされたメッセージやエラー文字列には ResourceManager を使う
- LogMessages と ErrorMessages の resource file を分離する
- `_resourceManager.GetString("MessageKey")` を通じて resource にアクセスする

## Async/Await Patterns

- すべての I/O 操作と長時間実行タスクに async/await を使う
- async method は Task または Task<T> を返す
- 適切な箇所で ConfigureAwait(false) を使う
- async 例外を適切に処理する

## Testing Standards

- assertion には FluentAssertions を併用した MSTest framework を使う
- AAA pattern (Arrange, Act, Assert) に従う
- 依存関係のモックには Moq を使う
- 成功系と失敗系の両方をテストする
- null parameter validation test を含める

## Configuration & Settings

- data annotations を持つ strongly-typed configuration class を使う
- validation attribute (Required, NotEmptyOrWhitespace) を実装する
- 設定には IConfiguration binding を使う
- appsettings.json 構成ファイルをサポートする

## Semantic Kernel & AI Integration

- AI 操作には Microsoft.SemanticKernel を使う
- 適切な kernel 構成と service 登録を実装する
- AI model の設定 (ChatCompletion, Embedding など) を扱う
- 信頼できる AI 応答のために structured output パターンを使う

## Error Handling & Logging

- Microsoft.Extensions.Logging による構造化ロギングを使う
- 意味のある文脈を含む scoped logging を行う
- 説明的なメッセージ付きで具体的な例外を投げる
- 想定される失敗シナリオには try-catch block を使う

## Performance & Security

- 該当する場合は C# 12+ の機能と .NET 8 の最適化を使う
- 適切な入力検証とサニタイズを実装する
- データベース操作には parameterized query を使う
- AI/ML 操作では安全なコーディング プラクティスに従う

## Code Quality

- SOLID 原則への準拠を確保する
- base class や utility によりコード重複を避ける
- ドメイン概念を反映した意味のある名前を使う
- method は焦点を絞り、凝集度を高く保つ
- リソースには適切な disposal パターンを実装する
