# CodeQL Build Modes for Compiled Languages

CodeQL におけるコンパイル言語解析の build mode、autobuild 挙動、runner 要件、ハードウェア要件のリファレンスです。

## Build Modes Overview

| Mode | Description | When to Use |
|---|---|---|
| `none` | ビルドせずにソース解析。依存関係はヒューリスティック推定。 | default setup、高速スキャン |
| `autobuild` | ビルドシステムを自動検出して実行。 | `none` の精度不足、Kotlin を含む場合 |
| `manual` | ユーザーが明示 build コマンドを指定。 | 複雑ビルド、autobuild 失敗時 |

## C/C++
- Supported: `none`, `autobuild`, `manual`
- Default setup: `none`

## C#
- Supported: `none`, `autobuild`, `manual`
- Default setup: `none`
- CodeQL tracer は `/p:EmitCompilerGeneratedFiles=true` などを注入します。レガシープロジェクトで競合する場合があります。

## Go
- Supported: `autobuild`, `manual`（`none` なし）
- Default setup: `autobuild`

## Java/Kotlin
- Java: `none`, `autobuild`, `manual`
- Kotlin: `autobuild`, `manual`（`none` なし）
- Kotlin がある場合は `autobuild` を使用

## Rust
- Supported: `none`, `autobuild`, `manual`
- Default setup: `none`

## Swift
- Supported: `autobuild`, `manual`（`none` なし）
- Default setup: `autobuild`
- macOS runner が必要

## Multi-Language Matrix Example

```yaml
strategy:
  fail-fast: false
  matrix:
    include:
      - language: c-cpp
        build-mode: manual
      - language: csharp
        build-mode: autobuild
      - language: java-kotlin
        build-mode: none
```

## Dependency Caching

```yaml
- uses: github/codeql-action/init@v4
  with:
    languages: java-kotlin
    dependency-caching: true
```
