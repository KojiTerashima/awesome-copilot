---
name: create-llms
description: 'https://llmstxt.org/ の llms.txt specification に従い、repository structure から `llms.txt` file を新規作成します'
---

# Create LLMs.txt File from Repository Structure

official な llms.txt specification https://llmstxt.org/ に従って、repository root に新しい `llms.txt` file をゼロから作成してください。この file は、repository の目的や specification を理解するために関連 content をどこで見つけるべきかについて、大規模言語モデル（LLM）へ高レベルの案内を提供します。

## Primary Directive

LLM が repository を理解し効果的にたどれる入口となる、包括的な `llms.txt` file を作成してください。この file は llms.txt specification に準拠し、LLM 向けに最適化されつつ、人間にも読みやすい必要があります。

## Analysis and Planning Phase

`llms.txt` file を作成する前に、十分な分析を完了しなければなりません。

### Step 1: Review llms.txt Specification

- 完全準拠のため、official specification を https://llmstxt.org/ で確認する
- 必要な format structure と guideline を理解する
- markdown structure に関する具体的 requirement を把握する

### Step 2: Repository Structure Analysis

- 適切な tool を使って repository structure 全体を調査する
- repository の主要な目的と範囲を特定する
- 重要な directory とその目的を一覧化する
- LLM の理解に有用な主要 file を列挙する

### Step 3: Content Discovery

- README file とその場所を特定する
- documentation file（`/docs/`、`/spec/` などの `.md` file）を見つける
- specification file とその目的を把握する
- configuration file とその関連性を見つける
- example file と code sample を見つける
- 既存 documentation structure を特定する

### Step 4: Create Implementation Plan

分析結果に基づき、次を含む構造化 plan を作成する:

- repository の目的と範囲の要約
- LLM が理解するうえで必須となる file の優先順位付き一覧
- 追加文脈を提供する secondary file
- `llms.txt` file の構成案

## Implementation Requirements

### Format Compliance

`llms.txt` file は specification に従って次の厳密な構造に従う必要があります。

1. **H1 Header**: repository/project 名を 1 行で記載（必須）
2. **Blockquote Summary**: blockquote 形式の簡潔な説明（任意だが推奨）
3. **Additional Details**: heading なしの markdown section を 0 個以上置ける
4. **File List Sections**: markdown link list を含む H2 section を 0 個以上置ける

### Content Requirements

#### Required Elements

- **Project Name**: H1 として明確で説明的な title
- **Summary**: repository の目的を説明する簡潔な blockquote
- **Key Files**: category ごとに整理した必須 file

#### File Link Format

各 file link は次の形式に従うこと: `[descriptive-name](relative-url): optional description`

#### Section Organization

file は、たとえば次のような論理的 H2 section に整理すること:

- **Documentation**: 中核となる documentation file
- **Specifications**: technical specification と requirement
- **Examples**: sample code と usage example
- **Configuration**: setup と configuration file
- **Optional**: 補助的な file（特別な意味を持ち、短い context では省略可能）

### Content Guidelines

#### Language and Style

- 簡潔で明確、曖昧さのない言語を使う
- 説明なしの jargon を避ける
- 人間と LLM の両方に向けて書く
- description は具体的で有用にする

#### File Selection Criteria

次の file を含める:
- repository の目的と範囲を説明するもの
- 必須の technical documentation を提供するもの
- usage example や pattern を示すもの
- interface や specification を定義するもの
- setup と configuration instruction を含むもの

次の file は除外する:
- 純粋な実装詳細にすぎないもの
- 重複情報を含むもの
- build artifact や generated content
- project 理解に無関係なもの

## Execution Steps

### Step 1: Repository Analysis

1. repository structure 全体を調査する
2. project を理解するため main README.md を読む
3. documentation directory と file をすべて特定する
4. specification file とその目的を一覧化する
5. example file と configuration file を見つける

### Step 2: Content Planning

1. 主要な purpose statement を定める
2. blockquote 用の簡潔な summary を書く
3. 特定した file を論理 category に分類する
4. LLM 理解にとっての重要度で file に優先順位を付ける
5. 各 file link の description を作成する

### Step 3: File Creation

1. repository root に `llms.txt` file を作成する
2. specification の format に正確に従う
3. 必要な section をすべて含める
4. 適切な markdown formatting を使う
5. すべての link が有効な relative path であることを確認する

### Step 4: Validation
1. https://llmstxt.org/ specification への準拠を確認する
2. すべての link が有効でアクセス可能か確認する
3. file が有効な LLM navigation tool になっていることを確認する
4. file が人間にも machine にも読みやすいことを確認する

## Quality Assurance

### Format Validation

- ✅ project 名の H1 header
- ✅ blockquote summary（含める場合）
- ✅ file list 用の H2 section
- ✅ 適切な markdown link format
- ✅ 壊れた link や無効な link がない
- ✅ 全体で一貫した formatting

### Content Validation

- ✅ 明確で曖昧さのない言語
- ✅ 必須 file を網羅した内容
- ✅ 論理的な content 構成
- ✅ 適切な file description
- ✅ 有効な LLM navigation tool として機能すること

### Specification Compliance

- ✅ https://llmstxt.org/ format に正確に従う
- ✅ 必須の markdown structure を使う
- ✅ optional section を適切に実装する
- ✅ file は repository root (`/llms.txt`) に置く

## Example Structure Template

```txt
# [Repository Name]

> [repository の目的と範囲を簡潔に説明する文]

[heading なしの任意追加 context paragraph]

## Documentation

- [Main README](README.md): 主な project documentation と getting started guide
- [Contributing Guide](CONTRIBUTING.md): project への貢献 guideline
- [Code of Conduct](CODE_OF_CONDUCT.md): community guideline と期待事項

## Specifications

- [Technical Specification](spec/technical-spec.md): 詳細な technical requirement と constraint
- [API Specification](spec/api-spec.md): interface 定義と data contract

## Examples

- [Basic Example](examples/basic-usage.md): 単純な usage の例
- [Advanced Example](examples/advanced-usage.md): 複雑な implementation pattern

## Configuration

- [Setup Guide](docs/setup.md): installation と configuration の instruction
- [Deployment Guide](docs/deployment.md): production deployment の guideline

## Optional

- [Architecture Documentation](docs/architecture.md): 詳細な system architecture
- [Design Decisions](docs/decisions.md): 過去の design decision record
```

## Success Criteria

作成する `llms.txt` file は次を満たす必要があります:
1. LLM が repository の目的をすばやく理解できること
2. 必須 documentation への明確な navigation を提供すること
3. official llms.txt specification に正確に従うこと
4. 包括的でありながら簡潔であること
5. 人間と machine の両方に有用であること
6. project 理解に必要な重要 file をすべて含むこと
7. 全体を通して明確で曖昧さのない言語を使うこと
8. 容易に読めるよう論理的に content を整理すること
