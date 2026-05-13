---
name: code-exemplars-blueprint-generator
description: 'コードベースを走査して高品質な code exemplar を特定するための、技術非依存のカスタマイズ可能な AI prompt generator。複数のプログラミング言語（.NET、Java、JavaScript、TypeScript、React、Angular、Python）に対応し、analysis depth、分類方法、documentation format を設定して、開発チーム全体の coding standard と一貫性の維持を支援します。'
---

# Code Exemplars Blueprint Generator

## Configuration Variables
${PROJECT_TYPE="Auto-detect|.NET|Java|JavaScript|TypeScript|React|Angular|Python|Other"} <!-- 主要技術 -->
${SCAN_DEPTH="Basic|Standard|Comprehensive"} <!-- コードベースをどの深さまで分析するか -->
${INCLUDE_CODE_SNIPPETS=true|false} <!-- ファイル参照に加えて実際のコードスニペットも含める -->
${CATEGORIZATION="Pattern Type|Architecture Layer|File Type"} <!-- exemplar の整理方法 -->
${MAX_EXAMPLES_PER_CATEGORY=3} <!-- 各カテゴリでの最大例数 -->
${INCLUDE_COMMENTS=true|false} <!-- 各 exemplar に説明コメントを含める -->

## Generated Prompt

"このコードベースを走査し、高品質で代表的なコード例を特定する exemplars.md ファイルを生成してください。exemplar は私たちの coding standard と pattern を示し、一貫性の維持に役立つ必要があります。次の方針で進めてください。

### 1. Codebase Analysis Phase
- ${PROJECT_TYPE == "Auto-detect" ? "ファイル拡張子と設定ファイルを走査して、主要なプログラミング言語とフレームワークを自動検出する" : `Focus on ${PROJECT_TYPE} code files`}
- 実装品質が高く、documentation が充実し、構造が明確なファイルを特定する
- よく使われている pattern、architecture component、よく整理された実装を探す
- 現在の技術スタックにおける best practice を示すファイルを優先する
- 実在するファイルだけを参照し、仮想の例は含めない

### 2. Exemplar Identification Criteria
- 明確な命名規則を備えた、構造が良く読みやすいコード
- 充実したコメントと documentation
- 適切な error handling と validation
- design pattern と architectural principle への準拠
- 関心の分離と単一責任原則
- code smell のない効率的な実装
- 標準的なアプローチを代表する内容

### 3. Core Pattern Categories

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" ? `#### .NET Exemplars（検出された場合）
- **Domain Models**: 適切な encapsulation と domain logic を実装している entity を見つける
- **Repository Implementations**: データアクセス方針を示す例
- **Service Layer Components**: 構造の良い business logic 実装
- **Controller Patterns**: 適切な validation と response を備えたクリーンな API controller
- **Dependency Injection Usage**: DI の設定と利用の良い例
- **Middleware Components**: custom middleware の実装
- **Unit Test Patterns**: 配置と assertion が適切な、構造の良い test` : ""}

${(PROJECT_TYPE == "JavaScript" || PROJECT_TYPE == "TypeScript" || PROJECT_TYPE == "React" || PROJECT_TYPE == "Angular" || PROJECT_TYPE == "Auto-detect") ? `#### Frontend Exemplars（検出された場合）
- **Component Structure**: クリーンで構造の良い component
- **State Management**: state 処理の良い例
- **API Integration**: 適切に実装された service call と data handling
- **Form Handling**: validation と submission の pattern
- **Routing Implementation**: navigation と route 設定
- **UI Components**: 再利用しやすく構造の良い UI 要素
- **Unit Test Examples**: component と service の test` : ""}

${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Auto-detect" ? `#### Java Exemplars（検出された場合）
- **Entity Classes**: よく設計された JPA entity または domain model
- **Service Implementations**: クリーンな service layer component
- **Repository Patterns**: data access 実装
- **Controller/Resource Classes**: API endpoint 実装
- **Configuration Classes**: application 設定
- **Unit Tests**: 構造の良い JUnit test` : ""}

${PROJECT_TYPE == "Python" || PROJECT_TYPE == "Auto-detect" ? `#### Python Exemplars（検出された場合）
- **Class Definitions**: 適切な documentation を備えた、構造の良い class
- **API Routes/Views**: クリーンな API 実装
- **Data Models**: ORM model 定義
- **Service Functions**: business logic 実装
- **Utility Modules**: helper と utility function
- **Test Cases**: 構造の良い unit test` : ""}

### 4. Architecture Layer Exemplars

- **Presentation Layer**:
  - user interface component
  - controller/API endpoint
  - view model/DTO

- **Business Logic Layer**:
  - service 実装
  - business logic component
  - workflow orchestration

- **Data Access Layer**:
  - repository 実装
  - data model
  - query pattern

- **Cross-Cutting Concerns**:
  - logging 実装
  - error handling
  - authentication/authorization
  - validation

### 5. Exemplar Documentation Format

特定した各 exemplar について、次を記録してください。
- ファイルパス（repository root からの相対パス）
- そのファイルが exemplar と言える理由の簡潔な説明
- それが表す pattern または component type
${INCLUDE_COMMENTS ? "- 実装上の重要点と、示されている coding principle" : ""}
${INCLUDE_CODE_SNIPPETS ? "- 小さく代表的なコードスニペット（該当する場合）" : ""}

${SCAN_DEPTH == "Comprehensive" ? `### 6. Additional Documentation

- **Consistency Patterns**: コードベース全体で一貫して見られる pattern を記録する
- **Architecture Observations**: コードから読み取れる architecture pattern を文書化する
- **Implementation Conventions**: 命名や構造に関する慣例を特定する
- **Anti-patterns to Avoid**: codebase が best practice から外れている箇所があれば記録する` : ""}

### ${SCAN_DEPTH == "Comprehensive" ? "7" : "6"}. Output Format

次の構成で exemplars.md を作成してください。
1. 文書の目的を説明する introduction
2. カテゴリへのリンク付き table of contents
3. ${CATEGORIZATION} に基づいて整理した section
4. 各カテゴリにつき最大 ${MAX_EXAMPLES_PER_CATEGORY} 件の exemplar
5. code quality を維持するための提言を含む conclusion

この文書は、既存 pattern と一貫した新機能実装の指針を必要とする開発者にとって、実行可能な内容である必要があります。

重要: 実際に存在するコードベース内のファイルだけを含めてください。すべての file path が存在することを確認し、placeholder や仮想の例は含めないでください。
"

## Expected Output
この prompt を実行すると、GitHub Copilot はコードベースを走査し、選択した parameter に従って整理された、repository 内の高品質なコード例への実在参照を含む exemplars.md ファイルを生成します。
