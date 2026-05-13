# CodeQL SARIF Output Reference

CodeQL 解析で生成される SARIF v2.1.0 出力のリファレンスです。CodeQL スキャン結果の解釈や処理時に使用します。

## About SARIF

SARIF（Static Analysis Results Interchange Format）は静的解析ツール出力を表現する標準 JSON 形式です。CodeQL は SARIF v2.1.0（`sarifv2.1.0`）を出力します。

- Specification: [OASIS SARIF v2.1.0](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html)
- Schema: [sarif-schema-2.1.0.json](https://docs.oasis-open.org/sarif/sarif/v2.1.0/errata01/os/schemas/sarif-schema-2.1.0.json)
- Format type: `sarifv2.1.0`

## Top-Level Structure

### `sarifLog` Object
| Property | Always Generated | Description |
|---|:---:|---|
| `$schema` | ✅ | SARIF schema へのリンク |
| `version` | ✅ | SARIF 仕様バージョン（`"2.1.0"`） |
| `runs` | ✅ | 言語ごとの `run` を含む配列 |

### `run` Object
| Property | Always Generated | Description |
|---|:---:|---|
| `tool` | ✅ | ツール情報 |
| `artifacts` | ✅ | 結果で参照されたファイルの配列 |
| `results` | ✅ | `result` 配列 |
| `newLineSequences` | ✅ | 改行文字列 |
| `columnKind` | ✅ | 列カウント方法 |
| `properties` | ✅ | `semmle.formatSpecifier` を含む |

## Results

デフォルトでは、結果はメッセージ形式と主 location の組み合わせでグループ化されます。`--ungroup-results` で無効化できます。

## Code Flows

`@kind path-problem` クエリでは `codeFlow` が含まれ、Source から Sink までの経路を示します。

## Key CLI Flags for SARIF

| Flag | Effect |
|---|---|
| `--format=sarif-latest` | SARIF v2.1.0 を出力 |
| `--sarif-category=<cat>` | `automationDetails.id` を設定 |
| `--sarif-add-file-contents` | `artifact.contents` にソース内容を含める |
| `--ungroup-results` | 結果のグルーピングを無効化 |
| `--output=<file>` | 出力先ファイル指定 |

## Upload Limits

- ファイルサイズ上限: **10 MB**（gzip 圧縮後）
- 超過時はクエリ範囲縮小、`--sarif-add-file-contents` の削除、複数アップロード分割を検討
