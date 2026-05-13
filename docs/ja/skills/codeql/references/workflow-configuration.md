# CodeQL Workflow Configuration Reference

GitHub Actions ワークフローで CodeQL 解析を設定するための詳細リファレンスです。

## Trigger Configuration

### Push Trigger
```yaml
on:
  push:
    branches: [main, protected]
```

### Pull Request Trigger
```yaml
on:
  pull_request:
    branches: [main]
```

### Schedule Trigger
```yaml
on:
  schedule:
    - cron: '20 14 * * 1'
```

### Merge Group Trigger
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  merge_group:
```

### Path Filtering
```yaml
on:
  pull_request:
    paths-ignore:
      - '**/*.md'
      - '**/*.txt'
      - 'docs/**'
```

## Runner and OS Configuration

```yaml
jobs:
  analyze:
    runs-on: ubuntu-latest
```

- `ubuntu-latest` は大半の言語で推奨
- Swift は `macos-latest` 必須
- 一部 C/C++ / C# では `windows-latest` が必要

## Language and Build Mode Matrix

```yaml
strategy:
  fail-fast: false
  matrix:
    include:
      - language: javascript-typescript
        build-mode: none
      - language: python
        build-mode: none
      - language: c-cpp
        build-mode: autobuild
```

## Query Suites and Packs

```yaml
- uses: github/codeql-action/init@v4
  with:
    queries: security-extended
```

```yaml
- uses: github/codeql-action/init@v4
  with:
    packs: |
      codeql/javascript-queries:AlertSuppression.ql
      my-org/my-custom-pack@1.2.3
```

## Analysis Category

```yaml
- uses: github/codeql-action/analyze@v4
  with:
    category: "/language:${{ matrix.language }}"
```

## Dependency Caching

```yaml
- uses: github/codeql-action/init@v4
  with:
    dependency-caching: true
```
