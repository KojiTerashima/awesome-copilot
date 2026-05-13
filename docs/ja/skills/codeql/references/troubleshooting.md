# CodeQL Troubleshooting Reference

CodeQL 解析エラー、SARIF アップロード問題、一般的な設定不備を診断・解決するためのガイドです。

## Build and Analysis Errors

### "No source code was seen during the build"
**原因**: database 作成時に extractor がソースを検出できなかった。  
**対処**:
- `--source-root` が正しいか確認
- コンパイル言語では build コマンドが実際にコンパイルしているか確認
- `autobuild` の検出結果を確認
- `manual` build mode へ切替
- 言語指定が実コードと一致しているか確認

### Automatic Build Failed
**原因**: `autobuild` がビルドシステムを検出/実行できない。  
**対処**:
- `build-mode: manual` に切替
- runner に必要依存を導入
- C/C++: `gcc`, `make`, `cmake`, `msbuild` を確認
- C#: `.NET SDK` または `MSBuild` を確認
- Java: `gradle` または `maven` を確認

### C# Compiler Unexpectedly Failing
**原因**: 注入フラグ `/p:EmitCompilerGeneratedFiles=true` などが競合。  
**対処**:
- 必要に応じて `<EmitCompilerGeneratedFiles>false</EmitCompilerGeneratedFiles>` を指定
- 許容できるなら `build-mode: none` を使用
- 問題プロジェクトを解析対象外にする

## Permission and Access Errors

### Error: 403 "Resource not accessible by integration"
**原因**: `GITHUB_TOKEN` 権限不足。  
**対処**:
```yaml
permissions:
  security-events: write
  contents: read
  actions: read
```

## SARIF Upload Errors

### SARIF File Too Large
- 上限: 10 MB（gzip）
- クエリ範囲を絞る
- `--sarif-add-file-contents` を外す
- 複数ジョブ/複数 SARIF に分割

### SARIF File Invalid
- [Microsoft SARIF validator](https://sarifweb.azurewebsites.net/) で検証
- `version`、`$schema`、`runs` など必須項目を確認
