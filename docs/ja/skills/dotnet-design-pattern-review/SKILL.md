---
name: dotnet-design-pattern-review
description: 'C#/.NET コードのデザインパターン実装をレビューし、改善点を提案します。'
---

# .NET/C# Design Pattern Review

${selection} にある C#/.NET コードのデザインパターン実装をレビューし、ソリューション/プロジェクトに対する改善案を提案してください。コードは変更せず、レビューのみを提供してください。

## Required Design Patterns

- **Command Pattern**: Generic base classes (`CommandHandler<TOptions>`), `ICommandHandler<TOptions>` interface, `CommandHandlerOptions` inheritance, static `SetupCommand(IHost host)` methods
- **Factory Pattern**: Complex object creation service provider integration
- **Dependency Injection**: Primary constructor syntax, `ArgumentNullException` null checks, interface abstractions, proper service lifetimes
- **Repository Pattern**: Async data access interfaces provider abstractions for connections
- **Provider Pattern**: External service abstractions (database, AI), clear contracts, configuration handling
- **Resource Pattern**: ResourceManager for localized messages, separate .resx files (LogMessages, ErrorMessages)

## Review Checklist

- **Design Patterns**: 使用されているパターンを特定する。Command Handler、Factory、Provider、Repository パターンは正しく実装されているか。有益な未導入パターンはあるか。
- **Architecture**: namespace 規約 (`{Core|Console|App|Service}.{Feature}`) に従っているか。Core/Console プロジェクト間の分離は適切か。モジュール性と可読性はあるか。
- **.NET Best Practices**: primary constructor、Task を返す async/await、ResourceManager の利用、構造化ロギング、強く型付けされた構成を使っているか。
- **GoF Patterns**: Command、Factory、Template Method、Strategy パターンは正しく実装されているか。
- **SOLID Principles**: Single Responsibility、Open/Closed、Liskov Substitution、Interface Segregation、Dependency Inversion に違反していないか。
- **Performance**: 適切な async/await、リソース解放、ConfigureAwait(false)、並列化できる箇所はあるか。
- **Maintainability**: 関心の分離が明確か。一貫したエラーハンドリングか。構成の利用は適切か。
- **Testability**: 依存関係は interface で抽象化されているか。モック可能か。async テストしやすいか。AAA パターンと両立するか。
- **Security**: 入力検証、安全な資格情報の扱い、パラメーター化クエリ、安全な例外処理はできているか。
- **Documentation**: 公開 API に XML ドキュメントがあるか。parameter/return の説明があるか。resource file は整理されているか。
- **Code Clarity**: ドメイン概念を反映した意味のある名前か。パターンにより意図が明確か。構造は自己説明的か。
- **Clean Code**: スタイルは一貫しているか。method/class のサイズは適切か。複雑さは最小限か。重複は排除されているか。

## Improvement Focus Areas

- **Command Handlers**: base class でのバリデーション、一貫したエラーハンドリング、適切なリソース管理
- **Factories**: 依存関係の構成、service provider 連携、破棄パターン
- **Providers**: 接続管理、async パターン、例外処理とロギング
- **Configuration**: data annotations、validation attributes、機密値の安全な取り扱い
- **AI/ML Integration**: Semantic Kernel パターン、structured output の扱い、model 構成

プロジェクトのアーキテクチャと .NET のベスト プラクティスに沿って、具体的で実行可能な改善提案を提示してください。
