---
name: plantuml-ascii
description: "Generate ASCII art diagrams using PlantUML text mode. Use when user asks to create ASCII diagrams, text-based diagrams, terminal-friendly diagrams, or mentions plantuml ascii, text diagram, ascii art diagram. Supports: Converting PlantUML diagrams to ASCII art, Creating sequence diagrams, class diagrams, flowcharts in ASCII format, Generating Unicode-enhanced ASCII art with -utxt flag"
license: MIT
allowed-tools: Bash, Write, Read
---
# PlantUML ASCII アート図ジェネレーター

## 概要

PlantUML を使用してテキストベースの ASCII アート図を作成します。ターミナル環境でのドキュメント、README ファイル、電子メール、またはグラフィカルな図が適さないシナリオに最適です。

## PlantUML アスキーアートとは何ですか?

PlantUML は、画像ではなくプレーンテキスト (ASCII アート) として図を生成できます。これは次の場合に役立ちます。

- ターミナルベースのワークフロー
- イメージをサポートしない Git コミット/PR
- バージョン管理が必要なドキュメント
- グラフィカルツールが利用できない環境

## インストール「」バッシュ
# macOS
醸造インストールplantuml

# Linux (ディストリビューションによって異なります)
sudo apt-get install plantuml # Ubuntu/Debian
sudo yum install plantuml # RHEL/CentOS

# または JAR を直接ダウンロードする
wget https://github.com/plantuml/plantuml/releases/download/v1.2024.0/plantuml-1.2024.0.jar
「」## 出力形式

|旗 |フォーマット |説明 |
| ------- | ------------- | ------------------------------------ |
| `-txt` |アスキー |純粋な ASCII 文字 |
| `-utxt` |ユニコード ASCII |ボックス描画文字で強化 |

## 基本的なワークフロー

### 1. PlantUML ダイアグラム ファイルを作成する「」プランタムル
@startuml
参加者のボブ
俳優アリス

ボブ -> アリス : こんにちは
アリス→ボブ：いいですか？
@enduml
「」### 2. アスキーアートを生成する「」バッシュ
# 標準 ASCII 出力
plantuml -txt 図.puml

# Unicode で強化された出力 (見栄えが良くなります)
plantuml -utxt 図.puml

# JAR を直接使用する
java -jar plantuml.jar -txt 図.puml
java -jar plantuml.jar -utxt 図.puml
「」### 3. 出力を表示する

出力は `diagram.atxt` (ASCII) または `diagram.utxt` (Unicode) として保存されます。

## サポートされる図の種類

### シーケンス図「」プランタムル
@startuml
アクターユーザー
参加者「Web アプリ」をアプリとして
データベース DBとしての「データベース」

ユーザー -> アプリ : ログイン要求
アプリ -> DB : 資格情報の検証
DB --> アプリ : ユーザーデータ
アプリ --> ユーザー : 認証トークン
@enduml
「」### クラス図「」プランタムル
@startuml
クラス ユーザー {
  +id: 整数
  +名前: 文字列
  +メールアドレス: 文字列
  +login(): ブール値
}

クラスの順序 {
  +id: 整数
  +合計: 浮動小数点
  +アイテム: リスト
  +calculateTotal(): 浮動小数点
}

ユーザー "1" -- "*" 順序 : 位
@enduml
「」### アクティビティ図「」プランタムル
@startuml
始める
:初期化;
if (有効ですか?) then (はい)
  :データを処理します。
  :結果を保存;
それ以外（いいえ）
  :ログエラー;
  停止
エンドイフ
:完了;
停止
@enduml
「」### 状態図「」プランタムル
@startuml
[*] --> アイドル状態
アイドル --> 処理中: 開始
処理中 --> 成功 : 完了
処理中 --> エラー: 失敗しました
成功 --> [*]
エラー --> アイドル状態: 再試行
@enduml
「」### コンポーネント図「」プランタムル
@startuml
[クライアント] クライアントとして
ゲートウェイとして[APIゲートウェイ]
[サービスA] svcAとして
[サービス B] svcB として
[データベース] データベースとして

クライアント --> ゲートウェイ
ゲートウェイ --> svcA
ゲートウェイ --> svcB
svcA --> データベース
svcB --> データベース
@enduml
「」### ユースケース図「」プランタムル
@startuml
ユーザーとしてのアクター「ユーザー」
俳優「Admin」を管理者として

四角形「システム」{
  ユーザー -- (ログイン)
  ユーザー -- (プロフィールの表示)
  ユーザー -- (設定の更新)
  admin -- (ユーザーの管理)
  管理者 -- (システムの構成)
}
@enduml
「」### 展開図「」プランタムル
@startuml
ユーザーとしてのアクター「ユーザー」
ノード「ロードバランサー」をポンドとして指定
ノード「Web サーバー 1」を ws1 として指定
ノード「Web サーバー 2」を ws2 として指定
データベース「プライマリ DB」を db1 として指定
データベース「レプリカ DB」（db2）

ユーザー --> ポンド
ポンド --> ws1
ポンド --> ws2
ws1 --> db1
ws2 --> db1
db1 --> db2 : レプリケート
@enduml
「」## コマンドラインオプション「」バッシュ
# 出力ディレクトリを指定
plantuml -txt -o ./出力ダイアグラム.puml

# ディレクトリ内のすべてのファイルを処理します
plantuml -txt ./図/

# ドットファイル（隠しファイル）を含める
plantuml -txt -includeドット図/

# 詳細な出力
plantuml -txt -v 図.puml

# 文字セットを指定する
plantuml -txt -charset UTF-8 図.puml
「」## Ant タスクの統合```xml
<target name="generate-ascii">
  <plantuml dir="./src" format="txt" />
</ターゲット>

<target name="generate-unicode-ascii">
  <plantuml dir="./src" format="utxt" />
</ターゲット>
「」## より良い ASCII 図のためのヒント

1. **シンプルにしてください**: 複雑な図は ASCII ではうまく表示されません
2. **短いラベル**: 長いテキストは ASCII 配置を壊します
3. **Unicode を使用する (`-utxt`)**: ボックス描画文字による視覚的な品質の向上
4. **共有前のテスト**: 固定幅フォントを使用して端末で確認します
5. **代替案を検討します**: 複雑な図の場合は、Mermaid.js またはgraphviz を使用します。

## 出力の比較例

**標準 ASCII (`-txt`)**:「」
     、---。          、---。
     |ボブ|          |アリス|
     `---'          `---'
      |   こんにちは |
      |----------->|
      |              |
      |  大丈夫ですか？   |
      |<-------------|
      |              |
「」**Unicode ASCII (`-utxt`)**:「」
┌─────┐ ┌─────┐
│ ボブ │ │アリス│
━━━━┘ ━━━━┘
  │ こんにちは │
  │─────────>│
  │ │
  │大丈夫ですか？   │
  │<─────────│
  │ │
「」## クイックリファレンス「」バッシュ
# シーケンス図をASCIIで作成
cat > seq.puml << 'EOF'
@startuml
アリス -> ボブ: リクエスト
ボブ --> アリス: 応答
@enduml
終了後

plantuml -txt seq.puml
猫のシーケンスatxt

# Unicode で作成する
plantuml -utxt seq.puml
猫のシーケンスutxt
「」## トラブルシューティング

**問題**: Unicode 文字化け

- **解決策**: 端末が UTF-8 をサポートし、適切なフォントを備えていることを確認してください。

**問題**: 図の位置がずれて見える

- **解決策**: 固定幅フォントを使用します (Courier、Monaco、Consolas)

**問題**: コマンドが見つかりません

- **解決策**: PlantUML をインストールするか、Java JAR を直接使用します

**問題**: 出力ファイルが作成されません

- **解決策**: ファイルのアクセス許可を確認し、PlantUML に書き込みアクセス権があることを確認してください。