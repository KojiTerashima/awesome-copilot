---
name: update-avm-modules-in-bicep
description: 'Update Azure Verified Modules (AVM) to latest versions in Bicep files.'
---
# Bicep ファイル内の Azure 検証済みモジュールを更新する

最新の Azure Verified Module (AVM) バージョンを使用するように Bicep ファイル `${file}` を更新します。進行状況の更新を重大な変更以外の変更に限定します。最終出力表とサマリー以外の情報は出力しないでください。

## プロセス

1. **スキャン**: AVM モジュールと現在のバージョンを `${file}` から抽出します。
1. **識別**: `#search` ツールを使用して `avm/res/{service}/{resource}` を照合し、使用されているすべての一意の AVM モジュールをリストします。
1. **確認**: `#fetch` ツールを使用して、MCR から各 AVM モジュールの最新バージョンを取得します: `https://mcr.microsoft.com/v2/bicep/avm/res/{service}/{resource}/tags/list`
1. **比較**: セマンティック バージョンを解析して、更新が必要な AVM モジュールを特定します
1. **確認**: 重大な変更については、`#fetch` ツールを使用して、`https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/{service}/{resource}` からドキュメントを入手してください。
1. **更新**: `#editFiles` ツールを使用して、バージョンの更新とパラメーターの変更を適用します。
1. **検証**: `#runCommands` ツールを使用して `bicep lint` と `bicep build` を実行し、準拠していることを確認します。
1. **出力**: 以下の更新の概要を含む表形式で変更を要約します。

## ツールの使用法

利用可能な場合は、常にツール `#search`、`#searchResults`、`#fetch`、`#editFiles`、`#runCommands`、`#todos` を使用してください。タスクを実行するコードを記述することは避けてください。

## 重大な変更ポリシー

⚠️ 更新に以下が含まれる場合は **承認のために一時停止します**。

- 互換性のないパラメータ変更
- セキュリティ/コンプライアンスの変更
- 行動の変化

## 出力フォーマット

アイコン付きの表に結果のみを表示します。```markdown
| Module | Current | Latest | Status | Action | Docs |
|--------|---------|--------|--------|--------|------|
| avm/res/compute/vm | 0.1.0 | 0.2.0 | 🔄 | Updated | [📖](link) |
| avm/res/storage/account | 0.3.0 | 0.3.0 | ✅ | Current | [📖](link) |

### Summary of Updates

Describe updates made, any manual reviews needed or issues encountered.
```## アイコン

- 🔄更新されました
- ✅ 現在
- ⚠️手動レビューが必要です
- ❌ 失敗しました
- 📖 ドキュメント

## 要件

- MCR タグ API はバージョン検出のみに使用します
- JSON タグ配列を解析し、セマンティック バージョニングによって並べ替えます
- Bicep ファイルの有効性とリンティング コンプライアンスを維持する