---
description: 'Gang of Four (GoF) パターンや SOLID 原則を含む、Object-Oriented Programming (OOP) の design pattern を適用して、クリーンで保守しやすく拡張性の高いコードを実現するためのベストプラクティス。'
applyTo: '**/*.py, **/*.java, **/*.ts, **/*.js, **/*.cs'
---


# クリーンコードのためのオブジェクト指向設計パターン

これらの指示は、GitHub Copilot が code を生成または refactor する際に、Gang of Four (GoF) の Design Pattern、SOLID 原則、クリーンな Object-Oriented Programming (OOP) 実践を優先するよう設定します。

## コアアーキテクチャ哲学

- **実装ではなく interface に対してプログラムする:** 常に具体実装より abstract class や interface を優先してください。具体 instance の提供には dependency injection を使います。
- **class 継承より object composition を優先する:** runtime で動的に振る舞いを組み合わせるには composition を使ってください。深い継承ツリーは避けます。encapsulation を壊さずに振る舞いを再利用するには Delegation を使ってください。
- **変化するものをカプセル化する:** application のうち変化する側面を特定し、変化しない部分から分離してください。Strategy、State、Bridge のような pattern を使って変化点を隔離します。
- **疎結合:** class 間の直接依存を最小化してください。Mediator、Observer、abstract factory を使い、component を疎結合に保ちます。

## Creational Pattern のガイドライン

object creation や instantiation を伴う code を生成する場合は、object がどのように生成されるかから system を分離するため、次の pattern を適用してください。

- **Abstract Factory:** system が複数の関連 product family のいずれか 1 つで構成される必要がある場合に使う（例: cross-platform UI widget）。client が abstract factory と abstract product interface のみを扱うようにする。
- **Factory Method:** class 自身が生成すべき object の class を事前に決められない場合に使う。instantiation を subclass に委譲する。
- **Builder:** 複雑な object を段階的に構築する必要があり、同じ構築手順で異なる表現を作り得る場合に使う。
- **Singleton:** class の instance を 1 つに限定し、global access point を提供する必要が **本当にある場合にのみ** 使う（例: central configuration manager、hardware interface）。可能なら strict Singleton より Dependency Injection を優先する。
- **Prototype:** factory hierarchy を作るのを避けたい場合や、既存 object の clone の方が一から生成するより低コストな場合に使う。

## Structural Pattern のガイドライン

class や object の組み合わせ方を定義して大きな構造を作る code を生成する場合は、次の pattern を適用してください。

- **Adapter:** 互換性のない interface 同士を協調させるために使う。柔軟性のため、multiple inheritance を使う Class Adapter より composition を使う Object Adapter を優先する。
- **Bridge:** 抽象と実装を分離し、それぞれを独立に変化させたい場合に使う（例: 高水準の `Window` 概念と platform 固有 `WindowImpl` の分離）。
- **Composite:** 部分-全体の階層を表現するために使う。共通 `Component` interface を通じて、個別 object と複合 object を client が同様に扱えるようにする。
- **Decorator:** object に責務を動的に追加するために使う。class 爆発を防ぐため、機能拡張には subclass 化より優先する。Decorator は装飾対象 component と **まったく同じ interface** を持たなければならない。
- **Facade:** 複雑な subsystem に対して、単純で統一的な interface を提供するために使う。
- **Flyweight:** 類似 object 間で可能な限り共有し、memory 使用量や計算コストを削減するために使う。
- **Proxy:** access control、lazy loading、remote communication のように、別 object への access を制御するための代理 object として使う。

## Behavioral Pattern のガイドライン

algorithm、control flow、object 間 communication を含む code を生成する場合は、次の pattern を適用してください。

- **Strategy:** algorithm family を定義し、それぞれをカプセル化して交換可能にする。Strategy object へ委譲し、複雑な条件分岐（`switch` / `if-else`）を排除する。
- **Observer:** 1 対多の依存を定義し、1 つの object（Subject）の変化を他の object（Observer）へ自動通知・更新する。subject と observer は疎結合に保つ。
- **Command:** request を object としてカプセル化する。undo / redo、queue、request logging を実装する際に重要。
- **State:** object の振る舞いが内部状態に大きく依存し、runtime で振る舞いを切り替える必要がある場合に使う。各 state を独立した class で表現する。
- **Template Method:** algorithm の骨格を base class に定義し、詳細手順は subclass に委ねる。ただし algorithm 構造自体は変えない。
- **Chain of Responsibility:** request を複数 handler の chain に流し、いずれかが処理するまで渡す。sender を特定 receiver に結び付けないようにする。
- **Mediator:** object 同士が互いを直接参照しないようにしつつ、複雑な communication と control logic を集中管理する。
- **Iterator:** aggregate object の内部表現を公開せずに、要素へ順にアクセスする標準手段を提供する。
- **Visitor:** object structure の class を変更せずに新しい operation を定義する。Abstract Syntax Tree のような安定した composite structure に対する複数の解析に有効。
- **Memento:** encapsulation を壊さずに object の内部状態を取り出して保存し、後で復元できるようにする（複雑な Undo 機構に有用）。

## Copilot 向けコード生成ルール

- **Pattern Recognition:** GoF pattern に対応する問題（例: "この操作を元に戻せるようにしたい"、"税金計算方法が複数ある"）を解く prompt が来たら、適用する pattern を comment で明示する。
- **Interface First:** concrete implementation より **先に** interface または abstract base class を生成する。
- **Immutability & Encapsulation:** field はデフォルトで `private` にする。getter / setter は必要な場合にだけ提供する。immutable object を優先する。
- **Naming Convention:** 理解を助ける場合は class 名に pattern 名（例: `TaxCalculationStrategy`, `ButtonDecorator`, `WidgetFactory`）を含めるが、適切なら domain に自然な名前を優先する。
- **God Class を避ける:** 巨大で複雑な class は、Mediator で連携する小さな class や、小さな Strategy object の composition に分割する。
- **Single Responsibility Principle:** 各 class は変更理由を 1 つだけ持つようにする。やりすぎている class は複数 class に分割する。
- **Open/Closed Principle:** class は拡張に対して開き、修正に対して閉じるよう設計する。既存 code を変えずに新しい振る舞いを加えられるよう、abstract class や interface を使う。
- **Liskov Substitution Principle:** subclass が base class の代わりに置き換わっても program の正しさが変わらないようにする。derived class が precondition を強めたり postcondition を弱めたりしないこと。
- **Interface Segregation Principle:** 汎用的な 1 つの interface より、用途特化の複数 interface を優先する。client が使わない interface に依存しないようにする。
- **Dependency Inversion Principle:** 具体ではなく抽象に依存する。high-level module と low-level module の双方が抽象に依存するようにする。
- **Design Pattern は慎重に使う:** codebase で現実の問題を解決するときにだけ pattern を適用する。保守性・柔軟性・可読性に明確な利益がある場合に限って使い、過剰設計を避ける。
- **意図を文書化する:** design pattern を使う場合は、その pattern を選んだ理由と適用方法を説明する comment を入れる。将来の保守者が設計判断を理解しやすくなる。
- **Testability:** 生成 code は test しやすいものにする。unit test を容易にする pattern（例: mocking しやすい Dependency Injection）を使う。使っている pattern の振る舞いを検証する test を書く。
- **段階的な Refactor:** 既存 code に design pattern を適用して refactor するときは、少しずつ進める。bug を入れずに設計を改善する小さな変更から始め、test で振る舞いが保たれていることを確認する。
- **Performance の考慮:** design pattern は追加の抽象レイヤーを導入し、performance に影響することがある。bottleneck の特定には profiling tool を使い、保守性を犠牲にせず必要な最適化を行う。
- **一貫性:** codebase 全体で design pattern を一貫して適用する。ある箇所で特定 pattern を使っているなら、似た状況でも同様の設計言語を保つことを検討する。
- **Review と改善:** design pattern を適用できる箇所や、より OOP 原則に沿う refactor の余地がないか、定期的に codebase を見直す。design quality とこれらの guideline への準拠に焦点を当てた code review を促す。
- **継続的に学ぶ:** OOP design pattern とベストプラクティスの最新動向を追い、codebase 品質を高める新しい知見や技法を取り入れていく。
- **単純さと柔軟性のバランス:** design pattern は強力だが複雑さも増やし得る。将来の変更に適応できる柔軟性を保ちつつ、理解しやすく保守しやすい code を目指す。単純な function で解決できる問題には function を優先し、明確な整理上の利益があるときだけ class や pattern を使う。
- **Repository と型定義を使う:** 複雑な data structure や interaction を伴う code を生成する場合は、data access の抽象化として repository を、型安全性と明確さのために typing definition を検討する。これにより関心分離が保たれ、保守性が向上する。

## Logging と Error Handling

- design pattern を適用する際は、logging と error handling を適切に統合する。
- fail safe、loud、clear、early を徹底する。
- silent failure は避け、debugging と保守に十分な context を含む error log を残す。
- より意味のある error message と、client code 側での細かな error handling のために custom exception を適切に使う。
- exception block は expected error condition の処理に使い、通常フローの制御には使わない。
- logging framework を使って log level と出力先を管理し、環境（development / production など）に応じて制御できるようにする。
- info、debug、warning、error、critical の各 log level を各 class / function で適切に使い、application の振る舞いや潜在問題を明確に把握できるようにする。さらに、一貫した error response と logging を実現するため、global exception handler のような集中型 error handling 機構も検討する。

## ドキュメント

- design pattern を適用する際は、code が十分に文書化されていることを確認する。
- class や method の目的は English の docstring で説明し、複雑な logic や設計判断は comment で補足する。既存 codebase で別の docstring 形式が使われていない限り、parameter と return の docstring には numpy pattern を使う。最初にこの instruction が使われるとき、開発者へどの docstring 形式を望むか確認し、その選択で今後の programming task を統一する。これにより、ほかの開発者が設計意図と使い方を理解しやすくなる。
- Sphinx や JSDoc のような tool を使って codebase から documentation を生成することも検討し、利用可能な class / method とその意図を開発者が把握しやすくする。
- さらに、README や専用 documentation file に高レベルの architecture overview を維持し、各 component や pattern が system 全体の中でどう組み合わさるかを説明する。
- documentation は user 向け（使い方）と developer 向け（仕組みと保守方法）に分け、code の進化に合わせて両方を更新する。
- class や pattern 間の関係を視覚的に表すため、適切な場面では UML などの diagram を使う。
- documentation culture をチーム内で促進し、クリーンで保守しやすい codebase を維持する上での重要性を共有する。
- 同じ内容を含む documentation file を乱立させない。
- 既存の doc file を確認し、同じ style で拡張するか、新しい必要文書を作る。簡潔で明確、かつ最重要事項に集中する。
- 冗長で過度に長い documentation は避け、開発者に必要な情報を埋もれさせない。
