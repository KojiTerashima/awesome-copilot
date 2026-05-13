---
description: 'アプリケーション code の変更により documentation 更新が必要になったとき、README.md と documentation file を自動更新する'
applyTo: '**/*.{md,js,mjs,cjs,ts,tsx,jsx,py,java,cs,go,rb,php,rs,cpp,c,h,hpp}'
---

# コード変更時に documentation を更新する

## 概要

README.md、API documentation、configuration guide、その他の documentation file が、code
の変更に基づいて更新を必要としているかを自動判定し、documentation が code 変更と同期した状態を保つこと。

## Instruction Section と設定

この section のうち、`Instruction Sections and Configurable Instruction Sections`
および `Instruction Configuration` にある内容は、この instruction file 自体にのみ関係し、
Copilot instruction の適用方法を簡単に切り替えるための仕組みとして意図されている。要するに、
この document の一部または section を on / off し、特定 section をいつどのように適用するかを
条件付きで変えられるようにするものである。

### Instruction Section と Configurable Instruction Section

この document には複数の instruction section がある。instruction section の開始は、
level two header で示される。これを **INSTRUCTION SECTION** と呼ぶ。一部の instruction
section は configurable であり、一部はそうではなく常に使われる。

configurable な instruction section は必須ではなく、追加 context や条件の影響を受ける。
これらを **CONFIGURABLE INSTRUCTION SECTIONS** と呼ぶ。

**configurable instruction sections** には、section の設定 property が level two header に
backtick 付きで追記される (例: `apply-this`)。これを **CONFIGURABLE PROPERTY** と呼ぶ。

**configurable property** は、この section の **Instruction Configuration** 部分で宣言および定義される。
型は boolean である。`true` なら、その section の instruction を適用・利用・遵守する。

各 **configurable instruction section** には、level two header の直後に、その section の設定 detail を
説明する 1 文がある。これを **CONFIGURATION DETAIL** と呼ぶ。

**configuration detail** は、configurable instruction section を補足するルール群の一部である。
これにより、特定の **configurable instruction section** を最終的にどう適用するかを決めるための、
個別ケースや条件をチェックできる。

**configurable instruction section** をどう適用するか決める前に、
**configurable property** にネストされた、または対応する `apply-condition` がないかを確認し、
最終判断時にはその `apply-condition` を用いること。各 **configurable property** の既定値として
`apply-condition` は未設定だが、設定された例は次のようなものになる:

    - **apply-condition** :
      ` this.parent.property = (git.branch == "master") ? this.parent.property = true : this.parent.property = false; `

すべての **constant instruction sections** と **configurable instruction sections** の合計が、
従うべき完全な instruction を決定する。これを **COMPILED INSTRUCTIONS** と呼ぶ。

**compiled instructions** は設定に依存する。**compiled instructions** に含まれる各 instruction
section は、この instruction file 全体とは独立した別 instruction として解釈および利用される。
これを **FINAL PROCEDURE** と呼ぶ。

### Instruction Configuration

- **apply-doc-file-structure** : true
  - **apply-condition** : unset
- **apply-doc-verification** : true
  - **apply-condition** : unset
- **apply-doc-quality-standard** : true
  - **apply-condition** : unset
- **apply-automation-tooling** : true
  - **apply-condition** : unset
- **apply-doc-patterns** : true
  - **apply-condition** : unset
- **apply-best-practices** : true
  - **apply-condition** : unset
- **apply-validation-commands** : true
  - **apply-condition** : unset
- **apply-maintenance-schedule** : true
  - **apply-condition** : unset
- **apply-git-integration** : false
  - **apply-condition** : unset

<!--
| Configuration Property         | Default | Description                                                                 | When to Enable/Disable                                      |
|-------------------------------|---------|-----------------------------------------------------------------------------|-------------------------------------------------------------|
| apply-doc-file-structure      | true    | Ensures documentation follows a consistent file structure.                  | Disable if you want to allow free-form doc organization.    |
| apply-doc-verification        | true    | Verifies that documentation matches code changes.                           | Disable if verification is handled elsewhere.               |
| apply-doc-quality-standard    | true    | Enforces documentation quality standards.                                   | Disable if quality standards are not required.              |
| apply-automation-tooling      | true    | Uses automation tools to update documentation.                              | Disable if you prefer manual documentation updates.         |
| apply-doc-patterns            | true    | Applies common documentation patterns and templates.                        | Disable for custom or unconventional documentation styles.  |
| apply-best-practices          | true    | Enforces best practices in documentation.                                   | Disable if best practices are not a priority.               |
| apply-validation-commands     | true    | Runs validation commands to check documentation correctness.                 | Disable if validation is not needed.                        |
| apply-maintenance-schedule    | true    | Schedules regular documentation maintenance.                                | Disable if maintenance is managed differently.              |
| apply-git-integration         | false   | Integrates documentation updates with Git workflows.                        | Enable if you want automatic Git integration.               |
-->
## documentation を更新するタイミング

### Trigger 条件

次のとき、documentation 更新が必要か自動的に確認する:

- 新しい feature や機能が追加されたとき
- API endpoint、method、interface が変更されたとき
- breaking change が導入されたとき
- dependency や要件が変わったとき
- configuration option や environment variable が変更されたとき
- install や setup 手順が変わったとき
- command-line interface や script が更新されたとき
- documentation 内の code example が古くなったとき

## Documentation 更新ルール

### README.md の更新

**次の場合は README.md を常に更新する:**

- 新しい feature や capability を追加したとき
  - "Features" section に feature の説明を追加する
  - 必要なら usage example を含める
  - 存在する場合は table of contents を更新する

- install や setup process を変更したとき
  - "Installation" または "Getting Started" section を更新する
  - dependency requirement を見直す
  - prerequisite list を更新する

- 新しい CLI command や option を追加したとき
  - command syntax と example を文書化する
  - option の説明と default value を含める
  - usage example を追加する

- configuration option を変更したとき
  - configuration example を更新する
  - 新しい environment variable を文書化する
  - config file template を更新する

### API Documentation の更新

**次の場合は API documentation を同期する:**

- 新しい endpoint を追加したとき
  - HTTP method、path、parameter を文書化する
  - request / response example を含める
  - OpenAPI / Swagger spec を更新する

- endpoint signature が変わったとき
  - parameter list を更新する
  - response schema を見直す
  - breaking change を文書化する

- authentication または authorization が変わったとき
  - authentication example を更新する
  - security requirement を見直す
  - API key / token documentation を更新する

### Code Example の同期

**次の場合は code example を確認・更新する:**

- function signature が変わったとき
  - その function を使うすべての code snippet を更新する
  - example が依然として compile / run できるか確認する
  - 必要なら import statement を更新する

- API interface が変わったとき
  - example request と response を更新する
  - client code example を見直す
  - SDK usage example を更新する

- best practice が進化したとき
  - 古くなった pattern を example で置き換える
  - 現在推奨される approach を使うよう更新する
  - 古い pattern には deprecation notice を追加する

### Configuration Documentation

**次の場合は configuration docs を更新する:**

- 新しい environment variable を追加したとき
  - .env.example file に追加する
  - README.md または docs/configuration.md で文書化する
  - default value と説明を含める

- config file structure が変わったとき
  - example config file を更新する
  - 新しい option を文書化する
  - 非推奨 option を明記する

- deployment configuration が変わったとき
  - Docker / Kubernetes config を更新する
  - deployment guide を見直す
  - infrastructure-as-code example を更新する

### 移行と breaking change

**次の場合は migration guide を作成する:**

- breaking API change が起きたとき
  - 何が変わったかを文書化する
  - before / after example を示す
  - step-by-step の migration 手順を含める

- major version update のとき
  - すべての breaking change を列挙する
  - upgrade checklist を用意する
  - よくある migration issue と解決策を含める

- feature を非推奨化するとき
  - 非推奨 feature を明確に示す
  - 代替 approach を提案する
  - 削除までの timeline を含める

## Documentation File Structure `apply-doc-file-structure`

If `apply-doc-file-structure == true`, then apply the following configurable instruction section.

### 標準 documentation file

次の documentation file を維持し、必要に応じて更新する:

- **README.md**: project overview、quick start、基本 usage
- **CHANGELOG.md**: version history と user-facing change
- **docs/**: 詳細 documentation
  - `installation.md`: setup と install guide
  - `configuration.md`: configuration option と example
  - `api.md`: API reference documentation
  - `contributing.md`: contribution guideline
  - `migration-guides/`: version migration guide
- **examples/**: 動作する code example と tutorial

### Changelog 管理

**次の内容について changelog entry を追加する:**

- 新しい feature ("Added" section)
- bug fix ("Fixed" section)
- breaking change ("Changed" section に **BREAKING** prefix を付ける)
- 非推奨 feature ("Deprecated" section)
- 削除された feature ("Removed" section)
- security fix ("Security" section)

**Changelog format:**

    ```markdown
    ## [Version] - YYYY-MM-DD

    ### Added
    - PR / issue 参照付きの新機能説明

    ### Changed
    - **BREAKING**: breaking change の説明
    - その他の変更

    ### Fixed
    - bug fix の説明
    ```

## Documentation Verification `apply-doc-verification`

If `apply-doc-verification == true`, then apply the following configurable instruction section.

### 変更適用前

**documentation の完全性を確認する:**

1. 新しい public API がすべて文書化されている
2. code example が compile / run できる
3. documentation 内の link が有効である
4. configuration example が正確である
5. install 手順が最新である
6. README.md が現在の状態を反映している

### Documentation Test

**documentation validation を含める:**

#### Example Task

- docs 内の code example が compile / run できることを確認する
- internal / external の broken link を確認する
- configuration example を schema に対して検証する
- API example が現在の実装と一致することを確認する

    ```bash
    # Example validation commands
    npm run docs:check         # docs build を確認
    npm run docs:test-examples # code example をテスト
    npm run docs:lint         # 問題を確認
    ```

## Documentation Quality Standards `apply-doc-quality-standard`

If `apply-doc-quality-standard == true`, then apply the following configurable instruction section.

### Writing Guideline

- 明確で簡潔な言葉を使う
- 動作する code example を含める
- 基本例と高度な例の両方を提供する
- 用語を一貫させる
- error handling の例を含める
- edge case と limitation を文書化する

### Code Example Format

    ```markdown
    ### Example: [その example が何を示すかの明確な説明]

    \`\`\`language
    // 必要な import / setup を含める
    import { function } from 'package';

    // 完全に実行可能な example
    const result = function(parameter);
    console.log(result);
    \`\`\`

    **Output:**
    \`\`\`
    expected output
    \`\`\`
    ```

### API Documentation Format

    ```markdown
    ### `functionName(param1, param2)`

    function が何をするかの簡潔な説明。

    **Parameters:**
    - `param1` (type): parameter の説明
    - `param2` (type, optional): default value を含む説明

    **Returns:**
    - `type`: return value の説明

    **Example:**
    \`\`\`language
    const result = functionName('value', 42);
    \`\`\`

    **Throws:**
    - `ErrorType`: どのような場合に、なぜ error が投げられるか
    ```

## Automation and Tooling `apply-automation-tooling`

If `apply-automation-tooling == true`, then apply the following configurable instruction section.

### Documentation 生成

**利用可能なら自動 tool を使う:**

#### 自動 tool の例

- JavaScript / TypeScript には JSDoc / TSDoc
- Python には Sphinx / pdoc
- Java には Javadoc
- C# には xmldoc
- Go には godoc
- Rust には rustdoc

### Documentation Linting

**documentation は次で検証する:**

- Markdown linter (markdownlint)
- Link checker (markdown-link-check)
- Spell checker (cspell)
- Code example validator

### Pre-update Hook

**pre-commit check には次を追加する:**

- documentation build が成功する
- broken link がない
- code example が有効である
- 変更に対する changelog entry が存在する

## Common Documentation Patterns `apply-doc-patterns`

If `apply-doc-patterns == true`, then apply the following configurable instruction section.

### Feature Documentation Template

    ```markdown
    ## Feature Name

    feature の簡潔な説明。

    ### Usage

    code snippet を含む基本 usage example。

    ### Configuration

    example 付き configuration option。

    ### Advanced Usage

    複雑な scenario と edge case。

    ### Troubleshooting

    よくある問題と解決策。
    ```

### API Endpoint Documentation Template

    ```markdown
    ### `HTTP_METHOD /api/endpoint`

    endpoint が何をするかの説明。

    **Request:**
    \`\`\`json
    {
      "param": "value"
    }
    \`\`\`

    **Response:**
    \`\`\`json
    {
      "result": "value"
    }
    \`\`\`

    **Status Codes:**
    - 200: 成功
    - 400: 不正な request
    - 401: 認証されていない
    ```

## Best Practices `apply-best-practices`

If `apply-best-practices == true`, then apply the following configurable instruction section.

### Do

- ✅ code 変更と同じ commit で documentation を更新する
- ✅ 適用前 review 用に、変更の before / after example を含める
- ✅ commit 前に code example をテストする
- ✅ formatting と用語を一貫させる
- ✅ limitation と edge case を文書化する
- ✅ breaking change には migration path を提供する
- ✅ documentation は DRY に保つ (重複ではなく link を使う)

### Don't

- ❌ documentation を更新せずに code 変更を commit する
- ❌ 古い example を documentation に残す
- ❌ まだ存在しない feature を文書化する
- ❌ 曖昧で不明確な言葉を使う
- ❌ changelog 更新を忘れる
- ❌ broken link や失敗する example を無視する
- ❌ user が不要な実装 detail を文書化する

## Validation Example Commands `apply-validation-commands`

If `apply-validation-commands == true`, then apply the following configurable instruction section.

documentation validation 用に project へ適用する script 例:

```json
{
  "scripts": {
    "docs:build": "Build documentation",
    "docs:test": "Test code examples in docs",
    "docs:lint": "Lint documentation files",
    "docs:links": "Check for broken links",
    "docs:spell": "Spell check documentation",
    "docs:validate": "Run all documentation checks"
  }
}
```

## Maintenance Schedule `apply-maintenance-schedule`

If `apply-maintenance-schedule == true`, then apply the following configurable instruction section.

### 定期 review

- **Monthly**: documentation の正確性を見直す
- **Per release**: version number と example を更新する
- **Quarterly**: 古い pattern や非推奨 feature がないか確認する
- **Annually**: 包括的な documentation audit を行う

### 非推奨化プロセス

feature を非推奨化するとき:

1. documentation に deprecation notice を追加する
2. example を推奨代替に更新する
3. migration guide を作成する
4. changelog に非推奨 notice を追加する
5. 削除予定の timeline を設定する
6. 次の major version で非推奨 feature と docs を削除する

## Git Integration `apply-git-integration`

If `apply-git-integration == true`, then apply the following configurable instruction section.

### Pull Request 要件

**documentation は code 変更と同じ PR で更新しなければならない:**

- feature PR 内で新機能を文書化する
- code 変更時に example を更新する
- code 変更と一緒に changelog entry を追加する
- interface 変更時に API docs を更新する

### Documentation Review

**code review 時には次を確認する:**

- documentation が変更内容を正確に説明している
- example が明確で完全である
- 文書化されていない breaking change がない
- changelog entry が適切である
- 必要なら migration guide が提供されている

## Review Checklist

documentation 完了と **final procedure** の確定前に、次を確認する:

- [ ] **Compiled instructions** が **constant instruction sections** と
**configurable instruction sections** の合計に基づいている
- [ ] README.md が現在の project 状態を反映している
- [ ] すべての新機能が文書化されている
- [ ] code example がテストされ、動作する
- [ ] API documentation が完全かつ正確である
- [ ] configuration example が最新である
- [ ] breaking change が migration guide とともに文書化されている
- [ ] CHANGELOG.md が更新されている
- [ ] link が有効で壊れていない
- [ ] install 手順が最新である
- [ ] environment variable が文書化されている

## コード変更時に documentation を更新する GOAL

- 可能なら documentation を code の近くに置く
- API reference には documentation generator を使う
- code とともに進化する living documentation を維持する
- documentation を feature 完成度の一部として扱う
- code review で documentation を確認する
- documentation を見つけやすく、辿りやすくする
