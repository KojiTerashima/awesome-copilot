---
name: winmd-api-search
description: 'Find and explore Windows desktop APIs. Use when building features that need platform capabilities — camera, file access, notifications, UI controls, AI/ML, sensors, networking, etc. Discovers the right API for a task and retrieves full type details (methods, properties, events, enumeration values).'
license: Complete terms in LICENSE.txt
---
# WinMD API 検索

このスキルは、あらゆる機能に適した Windows API を見つけて、その詳細を取得するのに役立ちます。すべての WinMD メタデータのローカル キャッシュを次の場所から検索します。

- **Windows プラットフォーム SDK** — すべての `Windows.*` WinRT API (常に利用可能、復元は必要ありません)
- **WinAppSDK / WinUI** — キャッシュ ジェネレーターのベースラインとしてバンドルされています (常に利用可能、復元は必要ありません)
- **NuGet パッケージ** — `.winmd` ファイルを含む、復元されたプロジェクト内の追加パッケージ
- **プロジェクト出力 WinMD** — `.winmd` をビルド出力として生成するクラス ライブラリ (C++/WinRT、C#)

復元やビルドを行わない新しいクローンでも、Platform SDK + WinAppSDK を完全にカバーできます。

## このスキルを使用する場合

- ユーザーが機能を構築したいと考えているため、その機能を提供する API を見つける必要があります。
- ユーザーが「X をどうすればいいですか?」と尋ねます。ここで、X はプラットフォーム機能 (カメラ、ファイル、通知、センサー、AI など) に関係します。
- コードを記述する前に、型の正確なメソッド、プロパティ、イベント、または列挙値が必要です
- UI またはシステム タスクにどのコントロール、クラス、またはインターフェイスを使用すればよいかわからない

## 前提条件

- **.NET SDK 8.0 以降** — キャッシュ ジェネレーターの構築に必要です。利用できない場合は、[dotnet.microsoft.com](https://dotnet.microsoft.com/download) からインストールします。

## キャッシュのセットアップ (初めて使用する前に必要)

すべてのクエリおよび検索コマンドはローカル JSON キャッシュから読み取られます。 **クエリを実行する前にキャッシュを生成する必要があります。**```powershell
# All projects in the repo (recommended for first run)
.\.github\skills\winmd-api-search\scripts\Update-WinMdCache.ps1

# Single project
.\.github\skills\winmd-api-search\scripts\Update-WinMdCache.ps1 -ProjectDir <project-folder>
```ベースライン カバレッジ (プラットフォーム SDK + WinAppSDK) のためにプロジェクトの復元やビルドは必要ありません。追加の NuGet パッケージの場合、プロジェクトには `dotnet restore` (`project.assets.json` を生成します) または `packages.config` ファイルが必要です。

キャッシュは `Generated Files\winmd-cache\` に保存され、パッケージ + バージョンごとに重複が排除されます。

### インデックスに登録されるもの

|出典 |利用可能な場合 |
|------|----------------|
| Windows プラットフォーム SDK |常に (ローカル SDK インストールから読み取ります) |
| WinAppSDK (最新) |常に (キャッシュ ジェネレーターのベースラインとしてバンドルされています) |
| WinAppSDK ランタイム |システムにインストールされている場合 (`Get-AppxPackage` によって検出される) |
|プロジェクト NuGet パッケージ | `dotnet restore` の後、または `packages.config` とともに |
|プロジェクト出力 `.winmd` |プロジェクトのビルド後 (WinMD を生成するクラス ライブラリ) |

> **注意:** このキャッシュ ディレクトリは `.gitignore` にある必要があります。これはソースではなく生成されます。

## 使用方法

状況に応じてパスを選択してください。

---

### Discover — 「どの API を使用すればよいかわからない」

ユーザーは自分の言葉で機能を説明します。適切な API を見つける必要があります。

**0。キャッシュが存在することを確認してください**

キャッシュがまだ生成されていない場合は、最初に `Update-WinMdCache.ps1` を実行します。上記の [キャッシュのセットアップ](#cache-setup-required-before-first-use) を参照してください。

**1.ユーザーの言語を翻訳 → キーワードを検索**

ユーザーの日常言語をプログラミング用語にマッピングします。複数のバリエーションを試してください。

|ユーザーの発言 |試してみたい検索キーワード（順番） |
|----------|-------------------------------------|
| 「写真を撮ります」 | `camera`、`capture`、`photo`、`MediaCapture` |
| "ディスクからロード" | `file open`、`picker`、`FileOpen`、`StorageFile` |
| 「何が入っているか説明してください」 | `image description`、`Vision`、`Recognition` |
| "ポップアップを表示" | `dialog`、`flyout`、`popup`、`ContentDialog` |
| 「ドラッグアンドドロップ」 | `drag`、`drop`、`DragDrop` |
| "設定を保存" | `settings`、`ApplicationData`、`LocalSettings` |

まずは簡単な日常の言葉から始めましょう。結果が弱いか無関係な場合は、より技術的なバリエーションを試してください。

**2.検索の実行**```powershell
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action search -Query "<keyword>"
```これにより、上位に一致するタイプと **JSON ファイル パス**を含むランク付けされた名前空間が返されます。

結果のスコアが **低い (60 未満) か無関係である ** 場合は、オンライン ドキュメントの検索に戻ります。

1. Web 検索を使用して、Microsoft Learn で適切な API を見つけます。次に例を示します。
   - `site:learn.microsoft.com/uwp/api <capability keywords>` `Windows.*` API の場合
   - `site:learn.microsoft.com/windows/windows-app-sdk/api/winrt <capability keywords>` `Microsoft.*` WinAppSDK API の場合
2. ドキュメント ページを読んで、ユーザーの要件に一致するタイプを特定します。
3. 型名がわかったら、戻って `-Action members` または `-Action enums` を使用して、正確なローカル署名を取得します。

**3. JSON を読んで適切な API を選択してください**

上位の結果からのパスにあるファイルを読み取ります。 JSON には、その名前空間内のすべての型 (完全なメンバー、署名、パラメーター、戻り値の型、列挙値) が含まれます。

読んで、ユーザーの要件に適合する型とメンバーを決定します。

**4.コンテキストについては公式ドキュメントを参照してください**

キャッシュには署名のみが含まれており、説明や使用方法のガイダンスは含まれていません。説明、例、および注釈については、Microsoft Learn でタイプを検索してください。

|名前空間プレフィックス |ドキュメントのベース URL |
|-----------------|---------------------|
| `Windows.*` | `https://learn.microsoft.com/uwp/api/{fully.qualified.typename}` |
| `Microsoft.*` (WinAppSDK) | `https://learn.microsoft.com/windows/windows-app-sdk/api/winrt/{fully.qualified.typename}` |

たとえば、`Microsoft.UI.Xaml.Controls.NavigationView` は次のようにマッピングされます。
`https://learn.microsoft.com/windows/windows-app-sdk/api/winrt/microsoft.ui.xaml.controls.navigationview`

**5. API の知識を使用して回答またはコードを作成します**

---

### ルックアップ — 「API は知っています。詳細を教えてください」

型または名前空間の名前はすでにわかっています (または疑わしい)。直接移動:```powershell
# Get all members of a known type
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action members -TypeName "Microsoft.UI.Xaml.Controls.NavigationView"

# Get enum values
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action enums -TypeName "Microsoft.UI.Xaml.Visibility"

# List all types in a namespace
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action types -Namespace "Microsoft.UI.Xaml.Controls"

# Browse namespaces
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action namespaces -Filter "Microsoft.UI"
````-Action members` に示されている内容を超える詳細が必要な場合は、`-Action search` を使用して JSON ファイル パスを取得し、JSON ファイルを直接読み取ります。

---

### その他のコマンド```powershell
# List cached projects
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action projects

# List packages for a project
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action packages

# Show stats
.\.github\skills\winmd-api-search\scripts\Invoke-WinMdQuery.ps1 -Action stats
```> キャッシュされているプロジェクトが 1 つだけの場合、`-Project` が自動選択されます。
> 複数のプロジェクトが存在する場合は、`-Project <name>` を追加します (利用可能な名前を確認するには `-Action projects` を使用します)。
> スキャン モードでは、衝突を避けるためにマニフェスト名に短いハッシュ サフィックスが含まれます。明確な場合は、接尾辞なしでベース プロジェクト名を渡すことができます。

## 検索スコアリング

検索では、クエリに対して型名とメンバー名がランク付けされます。

|スコア |一致タイプ |例 |
|------|-----------|----------|
| 100 |正確な名前 | `Button` → `Button` |
| 80 | | で始まる`Navigation` → `NavigationView` |
| 60 |含まれています | `Dialog` → `ContentDialog` |
| 50 | PascalCaseのイニシャル | `ASB` → `AutoSuggestBox` |
| 40 |複数のキーワード AND | `navigation item` → `NavigationViewItem` |
| 20 |あいまい文字一致 | `NavVw` → `NavigationView` |

結果は名前空間ごとにグループ化されます。スコアの高い名前空間が最初に表示されます。

## トラブルシューティング

|問題 |修正 |
|------|-----|
| "キャッシュが見つかりません" | `Update-WinMdCache.ps1` を実行します。
| "複数のプロジェクトがキャッシュされました" | `-Project <name>` を追加 |
| "名前空間が見つかりません" |使用可能なものをリストするには、`-Action namespaces` を使用します。
| "タイプが見つかりません" |完全修飾名を使用します (例: `Microsoft.UI.Xaml.Controls.Button`)。
| NuGet 更新後に古くなります |再実行 `Update-WinMdCache.ps1` |
| git 履歴のキャッシュ | `Generated Files/` を `.gitignore` に追加 |

## 参考文献

- [Windows プラットフォーム SDK API リファレンス](https://learn.microsoft.com/uwp/api/) — `Windows.*` 名前空間のドキュメント
- [Windows App SDK API リファレンス](https://learn.microsoft.com/windows/windows-app-sdk/api/winrt/) — `Microsoft.*` WinAppSDK 名前空間のドキュメント