---
name: geofeed-tuner
description: >
  Use this skill whenever the user mentions IP geolocation feeds, RFC 8805, geofeeds, or wants help creating, tuning, validating, or publishing a
  self-published IP geolocation feed in CSV format. Intended user audience is a network
  operator, ISP, mobile carrier, cloud provider, hosting company, IXP, or satellite provider
  asking about IP geolocation accuracy, or geofeed authoring best practices.
  Helps create, refine, and improve CSV-format IP geolocation feeds with opinionated
  recommendations beyond RFC 8805 compliance. Do NOT use for private or internal IP address
  management — applies only to publicly routable IP addresses.
license: Apache-2.0
metadata:
  author: Sid Mathur <support@getfastah.com>
  version: "0.0.9"
compatibility: Requires Python 3
---
# Geofeed Tuner – より良い IP 地理位置情報フィードを作成する

このスキルは、次の方法で CSV 形式で IP 地理位置情報フィードを作成および改善するのに役立ちます。
- CSV が適切な形式で一貫性があることを確認する
- [RFC 8805](references/rfc8805.txt) (業界標準) との整合性のチェック
- 現実世界の展開から学んだ**独自のベスト プラクティス**を適用する
- 正確さ、完全性、プライバシーの改善を提案する

## このスキルを使用する場合

- このスキルは、ユーザーが CSV 形式の IP 地理位置情報フィード ファイルの**作成、改善、または公開**について支援を求めた場合に使用します。
- **CSV 地理位置情報フィードの調整とトラブルシューティング**に使用して、エラーを検出し、改善を提案し、RFC 準拠を超えた実際の使いやすさを確保します。
- **対象読者:**
  - パブリックにルーティング可能な IP アドレス空間を担当するネットワーク オペレータ、管理者、およびエンジニア
  - ISP、携帯通信会社、クラウドプロバイダー、ホスティングおよびコロケーション会社、インターネットエクスチェンジオペレーター、衛星インターネットプロバイダーなどの組織
- **このスキルをプライベートまたは内部 IP アドレスの管理には使用しないでください**。 **パブリックにルーティング可能な IP アドレスのみ**に適用されます。

## 前提条件

- **Python 3** が必要です。

## ディレクトリ構造とファイル管理

このスキルでは、**配布ファイル** (読み取り専用) と **作業ファイル** (実行時に生成される) を明確に分離します。

### 読み取り専用ディレクトリ (変更しないでください)

次のディレクトリには、静的配布アセットが含まれています。 **次のディレクトリ内のファイルを作成、変更、または削除しないでください:**

|ディレクトリ |目的 |
|----------------|------------------------------------------------------------|
| `assets/` |静的データ ファイル (ISO コード、例) |
| `references/` |参考用の RFC 仕様とコード スニペット |
| `scripts/` |レポート用の実行可能コードおよび HTML テンプレート ファイル |

### 作業ディレクトリ (生成されたコンテンツ)

生成されたすべてのファイル、一時ファイル、および出力ファイルは、次のディレクトリに保存されます。

|ディレクトリ |目的 |
|-----------------|------------------------------------------------------|
| `run/` |エージェントが生成したすべてのコンテンツの作業ディレクトリ |
| `run/data/` |リモート URL からダウンロードされた CSV ファイル |
| `run/report/` |生成された HTML チューニング レポート |

### ファイル管理ルール1. **`assets/`、`references/`、または `scripts/`** には決して書き込まないでください。これらはスキル配布の一部であり、変更しないでください。
2. **ダウンロードされたすべての入力ファイル** (リモート URL から) は `./run/data/` に保存する必要があります。
3. **生成されたすべての HTML レポート**は `./run/report/` に保存する必要があります。
4. **生成されたすべての Python スクリプト**は `./run/` に保存する必要があります。
5. `run/` ディレクトリはセッション間でクリアされる場合があります。そこに永続的なデータを保存しないでください。
6. **実行用の作業ディレクトリ:** `./run/` で生成されたすべてのスクリプトは、`assets/iso3166-1.json` や `./run/data/report-data.json` などの相対パスが正しく解決されるように、**スキル ルート ディレクトリ** (`SKILL.md` を含むディレクトリ) を現在の作業ディレクトリとして使用して実行する必要があります。スクリプトを実行する前に `cd` を `./run/` に入力しないでください。


## 処理パイプライン: 順次フェーズの実行

すべてのフェーズは、フェーズ 1 からフェーズ 6 まで**順番に**実行する必要があります。各フェーズは、前のフェーズが正常に完了するかどうかによって異なります。たとえば、**品質分析**を実行する前に**構造チェック**を完了する必要があります。

フェーズを以下にまとめます。エージェントは、各フェーズのセクションで詳しく説明されている詳細な手順に従う必要があります。

|フェーズ |名前 |説明 |
|------|----------------------------|----------------------------------------------------------------------------|
| 1 |標準を理解する |自己公開 IP 地理位置情報フィードに関する RFC 8805 の主要な要件を確認する |
| 2 |インプットを集める |ローカル ファイルまたはリモート URL から IP サブネット データを収集する |
| 3 |チェックと提案 | CSV 構造を検証し、IP プレフィックスを分析し、データ品質をチェックする |
| 4 |チューニングデータのルックアップ | Fastah の MCP ツールを使用して、位置情報の精度を向上させるための調整データを取得します。
| 5 |チューニングレポートの生成 |分析と提案を要約した HTML レポートを作成する |
| 6 |最終レビュー |レポート データの一貫性と完全性を検証する |

**フェーズをスキップしないでください。** 各フェーズでは、後続のステージで必要な重要なチェックやデータ変換が行われます。


### 実行計画のルール

各フェーズを実行する前に、エージェントは目に見える TODO チェックリストを生成しなければなりません (MUST)。

計画は次のことを行う必要があります。
- フェーズの最初に登場します
- すべてのステップを順番にリストします
- チェックボックス形式を使用する
- ステップが完了するとライブで更新されます


### フェーズ 1: 標準を理解する

このスキルが適用する RFC 8805 の主要な要件を以下にまとめます。 **この概要は作業用の参考資料として使用してください。** [RFC 8805 テキスト] (references/rfc8805.txt) の全文は、特殊なケース、あいまいな状況、またはユーザーがここで取り上げられていない標準的な質問をした場合にのみ参照してください。

#### RFC 8805 の重要な事実**目的:** 自己公開 IP 地理位置情報フィードを使用すると、ネットワーク オペレーターは IP アドレス空間の信頼できる位置データをシンプルな CSV 形式で公開できるため、地理位置情報プロバイダーはオペレーター提供の修正を組み込むことができます。

**CSV 列の順序 (セクション 2.1.1.1 ～ 2.1.1.5):**

|コラム |フィールド |必須 |メモ |
|----------|------|----------|--------------------------------------------------------|
| 1 | `ip_prefix` |はい | CIDR 表記。 IPv4 または IPv6。ネットワークアドレスでなければなりません |
| 2 | `alpha2code` |いいえ | ISO 3166-1 alpha-2 国コード。空または "ZZ" = 地理位置情報を使用しない |
| 3 | `region` |いいえ | ISO 3166-2 サブディビジョン コード (例: `US-CA`) |
| 4 | `city` |いいえ |自由記述の都市名。権威ある検証セットがありません |
| 5 | `postal_code` |いいえ | **非推奨** — 空にするか、存在しないままにしておく必要があります。

**構造ルール:**
- ファイルには、`#` で始まるコメント行が含まれる場合があります (存在する場合はヘッダーも含みます)。
- ヘッダー行はオプションです。存在する場合、`#` で始まるコメントとして扱われます。
- ファイルは UTF-8 でエンコードされている必要があります。
- サブネット ホスト ビットを設定してはなりません (つまり、`192.168.1.1/24` は無効です。`192.168.1.0/24` を使用してください)。
- **グローバルにルーティング可能な**ユニキャスト アドレスにのみ適用され、プライベート、ループバック、リンクローカル、またはマルチキャスト スペースには適用されません。

**地理位置情報を使用しない:** 空の `alpha2code` または大文字と小文字を区別しない `ZZ` (地域/都市の値に関係なく) を含むエントリは、オペレーターがそのプレフィックスに地理位置情報を適用しないことを明示的に示します。

**郵便番号は非推奨になりました (セクション 2.1.1.5):** 5 列目に郵便番号を含めることはできません。 IP 範囲のマッピングには粒度が細かすぎるため、プライバシー上の懸念が生じます。


### フェーズ 2: 意見を収集する

- ユーザーが IP サブネットまたは範囲 (`inetnum` または `inet6num` と呼ばれることもあります) のリストをまだ提供していない場合は、それを提供するように求めます。受け入れられる入力形式:
  - チャットに貼り付けられたテキスト
  - ローカルの CSV ファイル
  - CSV ファイルを指すリモート URL

- 入力が **リモート URL** の場合:
  - 処理する前に、CSV ファイルを `./run/data/` にダウンロードしてみます。
  - HTTP エラー (4xx、5xx、タイムアウト、またはリダイレクト ループ) が発生した場合は、**ただちに停止**し、ユーザーに報告します。
    `Feed URL is not reachable: HTTP {status_code}. Please verify the URL is publicly accessible.`
  - 不完全なダウンロードまたは空のダウンロードでフェーズ 3 に進まないでください。

- 入力が **ローカル ファイル** の場合は、ダウンロードせずに直接処理します。

- **エンコーディングの検出と正規化:**
  1. まずファイルを UTF-8 として読み取ってみます。
  2. `UnicodeDecodeError` が発生した場合は、`utf-8-sig` (BOM 付き UTF-8)、次に `latin-1` を試してください。
  3. 正常にデコードされたら、再エンコードして作業コピーを UTF-8 として書き込みます。
  4. エンコードが成功しない場合は、停止して報告します: `Unable to decode input file. Please save it as UTF-8 and try again.`


### フェーズ 3: チェックと提案#### 実行ルール
- このフェーズの **スクリプト** を生成します。
- このフェーズを他のフェーズと組み合わせないでください。
- 将来のフェーズのデータ​​を事前計算しないでください。
- 出力を JSON ファイルとして次の場所に保存します: [`./run/data/report-data.json`](./run/data/report-data.json)

#### スキーマ定義

以下の JSON 構造は、フェーズ 3 では **IMMUTABLE** です。フェーズ 4 では、後で `TunedEntry` オブジェクトを `Entries` の各オブジェクトに追加します。これは唯一許可されるスキーマ拡張であり、別のフェーズで行われます。

JSON キーは、`{{.CountryCode}}`、`{{.HasError}}` などのテンプレート プレースホルダーに直接マッピングされます。```json
{
  "InputFile": "",
  "Timestamp": 0,

  "TotalEntries": 0,
  "IpV4Entries": 0,
  "IpV6Entries": 0,
  "InvalidEntries": 0,

  "Errors": 0,
  "Warnings": 0,
  "OK": 0,
  "Suggestions": 0,

  "CityLevelAccuracy": 0,
  "RegionLevelAccuracy": 0,
  "CountryLevelAccuracy": 0,
  "DoNotGeolocate": 0,

  "Entries": [
    {
      "Line": 0,
      "IPPrefix": "",
      "CountryCode": "",
      "RegionCode": "",
      "City": "",

      "Status": "",
      "IPVersion": "",

      "Messages": [
        {
          "ID": "",
          "Type": "",
          "Text": "",
          "Checked": false
        }
      ],

      "HasError": false,
      "HasWarning": false,
      "HasSuggestion": false,
      "DoNotGeolocate": false,
      "GeocodingHint": "",
      "Tunable": false
    }
  ]
}
```フィールドの定義:

**トップレベルのメタデータ:**
- `InputFile`: 元の入力ソース (ローカル ファイル名またはリモート URL)。
- `Timestamp`: チューニングが実行されたときの Unix エポックからのミリ秒。
- `TotalEntries`: 処理されたデータ行の総数(コメントと空行を除く)。
- `IpV4Entries`: IPv4 サブネットであるエントリの数。
- `IpV6Entries`: IPv6 サブネットであるエントリの数。
- `InvalidEntries`: IP プレフィックス解析と CSV 解析に失敗したエントリの数。
- `Errors`: `Status` が `ERROR` であるエントリの合計。
- `Warnings`: `Status` が `WARNING` であるエントリの合計。
- `OK`: `Status` が `OK` であるエントリの合計。
- `Suggestions`: `Status` が `SUGGESTION` であるエントリの合計。
- `CityLevelAccuracy`: `City` が空ではない有効なエントリの数。
- `RegionLevelAccuracy`: `RegionCode` が空ではなく、`City` が空である有効なエントリの数。
- `CountryLevelAccuracy`: `CountryCode` が空ではない、`RegionCode` が空、および `City` が空である有効なエントリの数。
- `DoNotGeolocate` (メタデータ): `CountryCode`、`RegionCode`、および `City` がすべて空である有効なエントリの数。

**入力フィールド:**
- `Entries`: オブジェクトの配列。データ行ごとに 1 つあり、次のエントリごとのフィールドがあります。
  - `Line`: 元の CSV の 1 から始まる行番号 (コメントと空白を含むすべての行をカウントします)。
  - `IPPrefix`: CIDR スラッシュ表記の正規化された IP プレフィックス。
  - `CountryCode`: ISO 3166-1 alpha-2 国コード、または空の文字列。
  - `RegionCode`: ISO 3166-2 地域コード (例: `US-CA`)、または空の文字列。
  - `City`: 都市名、または空の文字列。
  - `Status`: 割り当てられた最高の重大度: `ERROR` > `WARNING` > `SUGGESTION` > `OK`。
  - `IPVersion`: 解析された IP プレフィックスに基づく `"IPv4"` または `"IPv6"`。
  - `Messages`: メッセージ オブジェクトの配列。それぞれ次の内容を持ちます。
    - `ID`: 以下の **検証ルール リファレンス** 表の文字列識別子 (例: `"1101"`、`"3301"`)。
    - `Type`: 重大度の種類: `"ERROR"`、`"WARNING"`、または `"SUGGESTION"`。
    - `Text`: 人間が読める検証メッセージ文字列。
    - `Checked`: 検証ルールが自動調整可能な場合は `true` (参照テ​​ーブルの `Tunable: true`)、そうでない場合は `false`。レポートのチェックボックスが `checked` または `disabled` のどちらであるかを制御します。
  - `HasError`: `true` (メッセージに `Type` `"ERROR"` が含まれる場合)。
  - `HasWarning`: `Type` `"WARNING"` が含まれるメッセージがある場合は `true`。
  - `HasSuggestion`: `Type` `"SUGGESTION"` が含まれるメッセージがある場合は `true`。
  - `DoNotGeolocate` (エントリ): `CountryCode` が空の場合、または `"ZZ"` の場合は `true` — エントリは明示的な do-not-geolocate シグナルです。
  - `GeocodingHint`: フェーズ 3 では常に空の文字列 `""`。将来の使用のために予約されています。
  - `Tunable`: エントリ内の**いずれか**のメッセージに `Checked: true` が含まれている場合は `true`。すべてのメッセージの `Checked` 値の論理 OR として計算されます。このフラグは、レポート内の「調整」ボタンの表示を制御します。

#### 検証ルールのリファレンスエントリにメッセージを追加する場合は、この表の `ID`、`Type`、`Text`、および `Checked` の値を使用します。| ID |タイプ |テキスト |チェック済み |状態のリファレンス |
|----------|--------------|--------------------------------------------------------------------------------------------|----------------------|--------------------------------------|
| `1101` | `ERROR` | IP プレフィックスが空です | `false` | IP プレフィックス分析: 空 |
| `1102` | `ERROR` |無効な IP プレフィックス: IPv4 または IPv6 ネットワークとして解析できません | `false` | IP プレフィックス分析: 無効な構文 |
| `1103` | `ERROR` |非パブリック IP 範囲は RFC 8805 フィードでは許可されません。 `false` | IP プレフィックス分析: 非公開 |
| `3101` | `SUGGESTION` | IPv4 プレフィックスが異常に大きいため、タイプミスを示している可能性があります。 `false` | IP プレフィックス分析: IPv4 < /22 |
| `3102` | `SUGGESTION` | IPv6 プレフィックスが異常に大きいため、タイプミスを示している可能性があります。 `false` | IP プレフィックス分析: IPv6 < /64 |
| `1201` | `ERROR` |無効な国コード: 有効な ISO 3166-1 alpha-2 値ではありません。 `true` |国コード分析: 無効 |
| `1301` | `ERROR` |無効な領域形式です。予想される国-サブディビジョン (例: 米国-カナダ) | `true` |リージョンコード分析: 不正なフォーマット |
| `1302` | `ERROR` |無効な地域コード: 有効な ISO 3166-2 サブディビジョンではありません | `true` |地域コード分析: 不明なコード |
| `1303` | `ERROR` |地域コードが指定された国コードと一致しません。 `true` |地域コード分析: 不一致 |
| `1401` | `ERROR` |無効な都市名: プレースホルダー値は許可されません | `false` |都市名分析: プレースホルダー |
| `1402` | `ERROR` |無効な都市名: 省略形またはコードベースの値が検出されました。 `true` |都市名分析: 略語 |
| `2401` | `WARNING` |都市名の形式が一貫していません。値を正規化することを検討してください。 `true` |都市名の分析: 書式設定 |
| `1501` | `ERROR` |郵便番号は RFC 8805 によって非推奨になっており、プライバシー上の理由から削除する必要があります。 `true` |郵便番号チェック |
| `3301` | `SUGGESTION` |通常、領域が小さい場合はリージョンは不要です。リージョン値を削除することを検討してください。 `true` |チューニング: 小さな領域領域 |
| `3402` | `SUGGESTION` |都市レベルの細分性は通常、小規模な地域では不要です。都市の値を削除することを検討してください。 `true` |チューニング: 小さな領土の都市 |
| `3303` | `SUGGESTION` |都市を指定する場合は地域コードの使用をお勧めします。ドロップダウンから地域を選択します | `true` |チューニング: 都市を含む地域が欠落しています |
| `3104` | `SUGGESTION` |このサブネットが意図的に地理位置情報なしとしてマークされているか、位置データが欠落しているかどうかを確認します。 `true` |調整: 未指定の地理位置情報 |#### メッセージの入力

検証チェックが一致した場合、参照テーブルの値を使用して、エントリの `Messages` 配列にメッセージを追加します。```python
entry["Messages"].append({
    "ID": "1201",      # From the table
    "Type": "ERROR",   # From the table
    "Text": "Invalid country code: not a valid ISO 3166-1 alpha-2 value",  # From the table
    "Checked": True    # From the table (True = tunable)
})
```エントリのすべてのメッセージを入力した後、エントリレベルのフラグを取得します。```python
entry["HasError"] = any(m["Type"] == "ERROR" for m in entry["Messages"])
entry["HasWarning"] = any(m["Type"] == "WARNING" for m in entry["Messages"])
entry["HasSuggestion"] = any(m["Type"] == "SUGGESTION" for m in entry["Messages"])
entry["Tunable"] = any(m["Checked"] for m in entry["Messages"])
```#### 精度レベルのカウント規則

精度レベルは**相互に排他的**です。最も詳細な空でない地理フィールドに基づいて、各有効な (エラーでも無効でもない) エントリを 1 つのバケットに割り当てます。

|状態 |バケツ |
|--------------------------------------------------------------|----------------------------|
| `City` は空ではありません | `CityLevelAccuracy` |
| `RegionCode` が空ではない、かつ `City` が空である | `RegionLevelAccuracy` |
| `CountryCode` は空ではない、`RegionCode` および `City` は空 | `CountryLevelAccuracy` |
| `DoNotGeolocate` (エントリ) は `true` | `DoNotGeolocate` (メタデータ) |

**精度バケット内の `HasError: true` のエントリまたは `InvalidEntries` のエントリはカウントしないでください**。

エージェントは次のことを行ってはなりません:
- フィールドの名前を変更する
- フィールドの追加または削除
- データ型を変更する
- キーを並べ替える
- ネストを変更する
- オブジェクトを包みます
- 複数のファイルに分割

値が不明な場合は、**値を空のままにしてください**。決してデータを作成しないでください。

#### 構造と形式のチェック

このフェーズでは、フィードが適切な形式で解析可能であることを確認します。 **重大な構造エラー**は、チューナーが地理位置情報の品質を分析する前に解決する必要があります。

##### CSV 構造

このサブセクションでは、IP 地理位置情報フィードに使用される **CSV 形式の入力ファイル**のルールを定義します。
目標は、ファイルを確実に解析し、**一貫した内部表現**に正規化できるようにすることです。

- **CSV 構造チェック**
  - `pandas` が利用可能な場合は、CSV の解析に使用します。
  - それ以外の場合は、Python の組み込み `csv` モジュールにフォールバックします。

  - CSV に **正確に 4 つまたは 5 つの論理列**が含まれていることを確認します。
  - コメント行を使用できます。
  - ヘッダー行は **存在する場合と存在しない場合があります**。
  - ヘッダー行が存在しない場合は、暗黙的な列順序が想定されます。```
    ip_prefix, alpha2code, region, city, postal code (deprecated)
    ```- 入力ファイルの例を参照してください。
    [`assets/example/01-user-input-rfc8805-feed.csv`](assets/example/01-user-input-rfc8805-feed.csv)

- **CSV のクレンジングと正規化**
  - 次の操作と同等の Python ロジックを使用して、CSV をクリーンアップおよび正規化します。
    - **最初の 5 列** のみを選択し、5 番目以降の列は削除します。
    - **UTF-8 BOM** を使用して出力ファイルを書き込みます。

  - **コメント**
    - **最初の列が `#`** で始まるコメント行を削除します。
    - `#` で始まるヘッダー行も削除されます。
    - **1 から始まる行番号**をキーとして、元の行全体を値として使用して、コメントのマップを作成します。空行も格納します。
    - このマップを JSON ファイルに保存します: [`./run/data/comments.json`](./run/data/comments.json)
    - 例: `{ "4": "# It's OK for small city states to leave state ISO2 code unspecified" }`

- **注意事項**
  - 両方の実装パス (`pandas` と組み込み `csv`) は、次を使用して出力を書き込む必要があります。
    **UTF-8 BOM** が存在することを確認するための `utf-8-sig` エンコード。

#### IP プレフィックス分析
  - 各エントリの `IPPrefix` フィールドが存在し、空でないことを確認します。
  - エントリ間で重複する `IPPrefix` 値をチェックします。
  - 重複が見つかった場合は、スキルを停止し、メッセージ `Duplicate IP prefix detected: {ip_prefix_value} appears on lines {line_numbers}` でユーザーに報告します。
  - 重複が見つからない場合は、分析を続行します。

  - **チェック**
    - 各サブネットは、`references/` フォルダー内のコード スニペットを使用して **IPv4 または IPv6 ネットワーク**として適切に解析する必要があります。
    - サブネットは正規化され、**CIDR スラッシュ表記**で表示される必要があります。
      - 単一ホストの IPv4 サブネットは **`/32`** として表す必要があります。
      - 単一ホストの IPv6 サブネットは **`/128`** として表す必要があります。

  - **エラー**
    - 次の状態を **エラー** として報告します。

    - **無効なサブネット構文**
      - メッセージID: `1102`

    - **非パブリック アドレス空間**
      - **プライベート、ループバック、リンクローカル、マルチキャスト、またはその他の非パブリック**のサブネットに適用されます。
        - Python では、`is_private` と、`./references` に示す関連するアドレス プロパティを使用して非公開範囲を検出します。
      - メッセージID: `1103`

  - **提案**
    - 次の状況を **提案** として報告します。

    - **大きすぎるIPv6サブネット**
      - `/64` より短いプレフィックス
      - メッセージID: `3102`

    - **大きすぎるIPv4サブネット**
      - `/22` より短いプレフィックス
      - メッセージID: `3101`

#### 地理位置情報の品質チェック

地理位置情報データの**精度と一貫性**を分析します。
  - 国コード
  - 地域コード
  - 都市名
  - 非推奨のフィールド

このフェーズは、構造チェックに合格した後に実行されます。##### 国コード分析
  - ローカルで利用可能なデータテーブル [`ISO3166-1`](assets/iso3166-1.json) を使用して確認します。
    - ISO コードを含む国と地域の JSON 配列
    - 各オブジェクトには以下が含まれます:
      - `alpha_2`: 2 文字の国コード
      - `name`: 短い国名
      - `flag`: 旗の絵文字
    - このファイルは、RFC 8805 CSV の **有効な `CountryCode` 値のスーパーセット** を表します。
  - エントリの `CountryCode` (RFC 8805 セクション 2.1.1.2、列 `alpha2code`) を `alpha_2` 属性に対してチェックします。
  - サンプルコードは `references/` ディレクトリにあります。

  - [`assets/small-territories.json`](assets/small-territories.json) で国が見つかった場合は、そのエントリを内部的に小規模な地域としてマークします。このフラグは後のチェックと提案で使用されますが、**出力 JSON には保存されません** (一時的な検証状態です)。

  - **注意:** `small-territories.json` には、`iso3166-1.json` には存在しない歴史的/議論のあるコード (`AN`、`CS`、`XK`) が含まれています。これらのいずれかを `CountryCode` として使用するエントリは、たとえ小さな地域として一致したとしても、国コードの検証 (エラー) に失敗します。国コード ERROR が優先されます。小規模地域の国旗に基づいて国コードを抑制しないでください。

  - **エラー**
    - 次の状態を **エラー** として報告します。
    - **無効な国コード**
      - 条件: `CountryCode` は存在しますが、`alpha_2` セット内に見つかりません
      - メッセージID: `1201`

  - **提案**
    - 次の状況を **提案** として報告します。

    - **サブネットの地理位置情報が指定されていません**
      - 条件: サブネットのすべての地理フィールド (`CountryCode`、`RegionCode`、`City`) が空です。
      - アクション: 
        - エントリに `DoNotGeolocate = true` を設定します。
        - エントリの `CountryCode` を `ZZ` に設定します。
      - メッセージID: `3104`


##### 地域コード分析
  - ローカルで利用可能なデータテーブル [`ISO3166-2`](assets/iso3166-2.json) を使用して確認します。
    - ISO が割り当てたコードを含む国の下位区分の JSON 配列
    - 各オブジェクトには以下が含まれます:
      - `code`: 国コードが接頭辞として付けられた細分コード (例: `US-CA`)
      - `name`: 短いサブディビジョン名
    - このファイルは、RFC 8805 CSV の **有効な `RegionCode` 値のスーパーセット** を表します。
  - `RegionCode` 値が指定されている場合 (RFC 8805 セクション 2.1.1.3):
    - 形式が `{COUNTRY}-{SUBDIVISION}` (例: `US-CA`、`AU-NSW`) と一致することを確認します。
    - `code` 属性 (すでに国コードが接頭辞として付けられている) と値を比較して確認します。

  - **小規模地域の例外:** エントリが小規模な地域であり、**かつ** `RegionCode` 値がエントリの `CountryCode` と等しい場合 (例: `SG` がシンガポールの国と地域の両方である)、その地域を許容可能なものとして扱います。このエントリのすべての地域検証チェックをスキップします。小さな領土は事実上都市国家であり、意味のある ISO 3166-2 行政区画はありません。- **エラー**
    - 次の状態を **エラー** として報告します。
    - **無効な地域形式**
      - 条件: `RegionCode` は `{COUNTRY}-{SUBDIVISION}` と一致せず **また** 小規模地域の例外は適用されません
      - メッセージID: `1301`
    - **不明な地域コード**
      - 条件: `RegionCode` 値が `code` セットに見つからず、** 小規模地域の例外は適用されません。
      - メッセージID: `1302`
    - **国と地域の不一致**
      - 条件: `RegionCode` の国部分が `CountryCode` と一致しません
      - メッセージID: `1303`

##### 都市名分析

  - 都市名は **ヒューリスティック チェックのみ**を使用して検証されます。
  - 現在、都市名の検証に利用できる**信頼できるデータセット**はありません。

  - **エラー**
    - 次の状態を **エラー** として報告します。
    - **プレースホルダーまたは意味のない値**
      - 条件: プレースホルダーまたは意味のない値。以下を含みますが、これらに限定されません。
        - `undefined`
        - `Please select`
        - `null`
        - `N/A`
        - `TBD`
        - `unknown`
      - メッセージID: `1401`

    - **短縮された名前、略語、または空港コード**
      - 条件: 有効な都市名を表さない切り捨てられた名前、略語、または空港コード:
        - `LA`
        - `Frft`
        - `sin01`
        - `LHR`
        - `SIN`
        - `MAA`
      - メッセージID: `1402`

  - **警告**
    - 次の状況を **警告** として報告します。
    - **大文字と小文字または形式が一貫していない**
      - 条件: データ品質を低下させる可能性がある、大文字小文字、間隔、または書式設定が一貫していない都市名。例:
        - `HongKong` 対 `Hong Kong`
        - 大文字と小文字の混合または予期しないスクリプトの使用
      - メッセージID: `2401`

##### 郵便番号の確認
  - RFC 8805 セクション 2.1.1.5 では、**郵便番号を明示的に非推奨**としています。
  - 郵便番号は非常に少数の母集団を表す可能性があり、本質的に統計的な IP アドレス範囲のマッピングでは **プライバシーに安全であるとみなされません**。

  - **エラー**
    - 次の状態を **エラー** として報告します。
    - **郵便番号が存在します**
      - 条件: 郵便番号フィールドに空ではない値が存在します。
      - メッセージID: `1501`

#### チューニングと推奨事項

このフェーズでは、実際のジオフィード展開から学んだ、RFC 8805 を超える**独自の推奨事項**を適用し、精度と使いやすさを向上させます。

- **提案**
  - 次の状況を **提案** として報告します。

  - **小規模な地域に指定された地域または都市**
    - 状態:
      - 立ち入りは狭い領域です
      - `RegionCode` は空ではない **OR**
      - `City` は空ではありません。
    - メッセージ ID: `3301` (地域の場合)、`3402` (都市の場合)

  - **都市が指定されている場合に地域コードがありません**
    - 状態:
      - `City` は空ではありません
      - `RegionCode` は空です
      - 参入は**小さな領域ではありません**
    - メッセージID: `3303`

### フェーズ 4: データ ルックアップの調整#### 目的
Fastah の `rfc8805-row-place-search` ツールを使用して、すべての `Entries` を検索します。

#### 実行ルール
- ペイロード生成のみのための新しい **スクリプト** を生成します (データセットを読み取り、1 つ以上のペイロード JSON ファイルを書き込みます。このスクリプトから MCP を呼び出さないでください)。
- サーバーはリクエストごとに 1000 エントリのみを受け入れるため、1000 を超えるエントリがある場合は複数のリクエストに分割します。
- エージェントは、生成されたペイロード ファイルを読み取り、そこからリクエストを構築し、それらのリクエストをそれぞれ最大 1000 エントリのバッチで MCP サーバーに送信する必要があります。
- **MCP 障害時:** MCP サーバーに到達できない場合、エラーが返された場合、またはどのバッチに対しても結果が返されなかった場合は、警告をログに記録し、フェーズ 5 に進みます。影響を受けるすべてのエントリに `TunedEntry: {}` を設定します。レポートの生成をブロックしないでください。ユーザーに明確に通知します: `Tuning data lookup unavailable; the report will show validation results only.`
- 提案は **アドバイスのみ** — **決して自動入力しないでください**。

#### ステップ 1: 重複排除を使用してルックアップ ペイロードを構築する

データセットを次から読み込みます: [./run/data/report-data.json](./run/data/report-data.json)
- `Entries` 配列を読み取ります。各エントリは、MCP ルックアップ ペイロードの構築に使用されます。

同一のエントリの重複を排除することで、サーバー リクエストを削減します。
- `Entries` の各エントリについて、コンテンツ ハッシュ (`CountryCode` + `RegionCode` + `City` のハッシュ) を計算します。
- 重複排除マップを作成します: `{ contentHash -> { rowKey, payload, entryIndices: [] } }`。 rowKey は、応答を照合するために MCP サーバーに送信される UUID です。
- エントリのハッシュがすでに存在する場合は、`Entries` の **0 から始まる配列インデックス ** をその重複排除エントリの `entryIndices` 配列に追加します。
- ハッシュが新しい場合は、**UUID (rowKey)** を生成し、新しい重複排除エントリを作成します。

ビルドリクエストのバッチ:
- 重複排除された固有のエントリをマップから抽出し、重複排除の順序を維持します。
- それぞれ最大 1000 アイテムのリクエスト バッチを構築します。
- バッチごとに、`[{ rowKey, payload, entryIndices }, ...]` のようなメモリ内構造を保持し、rowKey で応答を照合します。
- MCP ペイロード ファイルを書き込むときは、各ペイロード オブジェクトに `rowKey` フィールドを含めます。```json
[
    {"rowKey": "550e8400-e29b-41d4-a716-446655440000", "countryCode":"CA","regionCode":"CA-ON","cityName":"Toronto"},
    {"rowKey": "6ba7b810-9dad-11d1-80b4-00c04fd430c8", "countryCode":"IN","regionCode":"IN-KA","cityName":"Bangalore"},
    {"rowKey": "6ba7b811-9dad-11d1-80b4-00c04fd430c8", "countryCode":"IN","regionCode":"IN-KA"}
]
```- 応答を読み取るときは、各応答 `rowKey` フィールドを対応する重複排除エントリと照合して、関連付けられたすべての `entryIndices` を取得します。

ルール:
- ペイロードを次の場所に書き込みます: [./run/data/mcp-server-payload.json](./run/data/mcp-server-payload.json)
- ペイロードを書き込んだ後、スクリプトを終了します。

#### ステップ 2: Fastah MCP ツールを起動する

- Fastah MCP サーバーの `mcp.json` スタイル構成の例は次のとおりです。```json
    "fastah-ip-geofeed": {
      "type": "http",
      "url": "https://mcp.fastah.ai/mcp"
    }
```- サーバー: `https://mcp.fastah.ai/mcp`
- ツールとそのスキーマ: 最初の `tools/call` の前に、エージェントは **`rfc8805-row-place-search`** の入力および出力スキーマを読み取る `tools/list` リクエストを送信しなければなりません。
  検出されたスキーマをフィールド名、タイプ、および制約の信頼できるソースとして使用します。
- 以下は単なる例です。常に `tools/list` によって返されるスキーマに従います。```json
  [
      {"rowKey": "550e8400-...", "countryCode":"CA", ...},
      {"rowKey": "690e9301-...", "countryCode":"ZZ", ...}
  ]
- Open [./run/data/mcp-server-payload.json](./run/data/mcp-server-payload.json) and send all deduplicated entries with their rowKeys.
- If there are more than 1000 deduplicated entries after deduplication, split into multiple requests of 1000 entries each.
- The server will respond with the same `rowKey` field in each response for mapping back.
- Do NOT use local data.

#### Step 3: Attach Tuned Data to Entries

- Generate a new **script** for attaching tuned data.
- Load both [./run/data/report-data.json](./run/data/report-data.json) and the deduplication map (held in memory from Step 1, or re-derived from the payload file).
- For each response from the MCP server:
  - Extract the `rowKey` from the response.
  - Look up the `entryIndices` array associated with that `rowKey` from the deduplication map.
  - For each index in `entryIndices`, attach the best match to `Entries[index]`.
- Use the **first (best) match** from the response when available.

Create the field on each affected entry if it does not exist. Remap the MCP API response keys to Go struct field names:

```json
"TunedEntry": {
  "名前": "",
  "国コード": "",
  "地域コード": "",
  "場所の種類": "",
  "H3Cell": [],
  「バウンディングボックス」: []
}```

The `TunedEntry` field is a **single object** (not an array). It holds the best match from the MCP server.

**MCP response key → JSON key mapping**:
| MCP API response key | JSON key                   |
|----------------------|----------------------------|
| `placeName`          | `Name`                     |
| `countryCode`        | `CountryCode`              |
| `stateCode`          | `RegionCode`               |
| `placeType`          | `PlaceType`                |
| `h3Cells`            | `H3Cells`                  |
| `boundingBox`        | `BoundingBox`              |

Entries with no UUID match (i.e. the MCP server returned no response for their UUID) must receive an empty `TunedEntry: {}` object — never leave the field absent.

- Write the dataset back to: [./run/data/report-data.json](./run/data/report-data.json)
- Rules:
  - Maintain all existing validation flags.
  - Do NOT create additional intermediate files.


### Phase 5: Generate Tuning Report

Generate a **self-contained HTML report** by rendering the template at `./scripts/templates/index.html` with data from `./run/data/report-data.json` and `./run/data/comments.json`.

Write the completed report to `./run/report/geofeed-report.html`. After generating, attempt to open it in the system's default browser (e.g., `webbrowser.open()`). If running in a headless environment, CI pipeline, or remote container where no browser is available, skip the browser step and instead present the file path to the user so they can open or download it.

**The template uses Go `html/template` syntax** (`{{.Field}}`, `{{range}}`, `{{if eq}}`, etc.). Write a Python script that reads the template, builds a rendering context from the JSON data files, and processes the template placeholders to produce final HTML. Do not modify the template file itself — all processing happens in the Python script at render time.

#### Step 1: Replace Metadata Placeholders

Replace each `{{.Metadata.X}}` placeholder in the template with the corresponding value from `report-data.json`. Since JSON keys match the template placeholder, the mapping is direct — `{{.Metadata.InputFile}}` maps to the `InputFile` JSON key, etc.

| Template placeholder                   | JSON key (`report-data.json`)     |
|----------------------------------------|-----------------------------------|
| `{{.Metadata.InputFile}}`              | `InputFile`                       |
| `{{.Metadata.Timestamp}}`              | `Timestamp`                       |
| `{{.Metadata.TotalEntries}}`           | `TotalEntries`                    |
| `{{.Metadata.IpV4Entries}}`            | `IpV4Entries`                     |
| `{{.Metadata.IpV6Entries}}`            | `IpV6Entries`                     |
| `{{.Metadata.InvalidEntries}}`         | `InvalidEntries`                  |
| `{{.Metadata.Errors}}`                 | `Errors`                          |
| `{{.Metadata.Warnings}}`               | `Warnings`                        |
| `{{.Metadata.Suggestions}}`            | `Suggestions`                     |
| `{{.Metadata.OK}}`                     | `OK`                              |
| `{{.Metadata.CityLevelAccuracy}}`      | `CityLevelAccuracy`               |
| `{{.Metadata.RegionLevelAccuracy}}`    | `RegionLevelAccuracy`             |
| `{{.Metadata.CountryLevelAccuracy}}`   | `CountryLevelAccuracy`            |
| `{{.Metadata.DoNotGeolocate}}`         | `DoNotGeolocate` (metadata)       |

**Note on `{{.Metadata.Timestamp}}`:** This placeholder appears inside a JavaScript `new Date(...)` call. Replace it with the raw integer value (no HTML escaping needed for a numeric literal inside `<script>`). All other metadata values should be HTML-escaped since they appear inside HTML element text.

#### Step 2: Replace the Comment Map Placeholder

Locate this pattern in the template:
```JavaScript
const commentMap = {{.Comments}};```

Replace `{{.Comments}}` with the serialized JSON object from `./run/data/comments.json`. The JSON is embedded directly as a JavaScript object literal (not inside a string), so no extra escaping is needed:

```パイソン
comments_json = json.dumps(コメント)
template = template.replace("{{.Comments}}", comments_json)```

#### Step 3: Expand the Entries Range Block

The template contains a `{{range .Entries}}...{{end}}` block inside `<tbody id="entriesTableBody">`. Process it as follows:

1. **Extract** the range block body using regex. **Critical:** The block contains nested `{{end}}` tags (from `{{if eq .Status ...}}`, `{{if .Checked}}`, and `{{range .Messages}}`). A naive non-greedy match like `\{\{range \.Entries\}\}(.*?)\{\{end\}\}` will match the **first** inner `{{end}}`, truncating the block. Instead, anchor the outer `{{end}}` to the `</tbody>` that follows it:
    ```パイソン
    m = リサーチ(
        r'\{\{range \.Entries\}\}(.*?)\{\{end\}\}\s*</tbody>',
        テンプレート、
        re.ドットール、
    ）
    entry_body = m.group(1) # 1 つのエントリ反復のテンプレート テキスト```
    This ensures you capture the full block body including all three `<tr>` rows and the nested `{{range .Messages}}...{{end}}`.
2. **Iterate** over each entry in `report-data.json`'s `Entries` array.
3. **Expand** the block body for each entry using the processing order below.
4. **Replace** the entire match (from `{{range .Entries}}` through `</tbody>`) with the concatenated expanded HTML followed by `</tbody>`.

**Processing order for each entry** (innermost constructs first to avoid `{{end}}` confusion):
1. Evaluate `{{if eq .Status ...}}...{{end}}` conditionals (status badge class and icon).
2. Evaluate `{{if .Checked}}...{{end}}` conditional (message checkbox).
3. Expand `{{range .Messages}}...{{end}}` inner range.
4. Replace simple `{{.Field}}` placeholders.

##### Entry Field Mapping

Within the range block body, replace these placeholders for each entry. Since JSON keys match the template placeholder, the template placeholder `{{.X}}` maps directly to JSON key `X`:

| Template placeholder           | JSON key (`Entries[]`)       | Notes                                                        |
|--------------------------------|------------------------------|--------------------------------------------------------------|
| `{{.Line}}`                    | `Line`                       | Direct integer value                                         |
| `{{.IPPrefix}}`                | `IPPrefix`                   | HTML-escaped                                                 |
| `{{.CountryCode}}`             | `CountryCode`                | HTML-escaped                                                 |
| `{{.RegionCode}}`              | `RegionCode`                 | HTML-escaped                                                 |
| `{{.City}}`                    | `City`                       | HTML-escaped                                                 |
| `{{.Status}}`                  | `Status`                     | HTML-escaped                                                 |
| `{{.HasError}}`                | `HasError`                   | Lowercase string: `"true"` or `"false"`                      |
| `{{.HasWarning}}`              | `HasWarning`                 | Lowercase string: `"true"` or `"false"`                      |
| `{{.HasSuggestion}}`           | `HasSuggestion`              | Lowercase string: `"true"` or `"false"`                      |
| `{{.GeocodingHint}}`           | `GeocodingHint`              | Empty string `""`                                            |
| `{{.DoNotGeolocate}}`          | `DoNotGeolocate`             | `"true"` or `"false"`                                        |
| `{{.Tunable}}`                 | `Tunable`                    | `"true"` or `"false"`                                        |
| `{{.TunedEntry.CountryCode}}`  | `TunedEntry.CountryCode`     | `""` if `TunedEntry` is empty `{}`                           |
| `{{.TunedEntry.RegionCode}}`   | `TunedEntry.RegionCode`      | `""` if `TunedEntry` is empty `{}`                           |
| `{{.TunedEntry.Name}}`         | `TunedEntry.Name`            | `""` if `TunedEntry` is empty `{}`                           |
| `{{.TunedEntry.H3Cells}}`      | `TunedEntry.H3Cells`         | Bracket-wrapped space-separated; `"[]"` if empty (see format below) |
| `{{.TunedEntry.BoundingBox}}`  | `TunedEntry.BoundingBox`     | Bracket-wrapped space-separated; `"[]"` if empty (see format below) |

**`data-h3-cells` and `data-bounding-box` format:** These are **NOT JSON arrays**. They are bracket-wrapped, space-separated values. Do **not** use JSON serialization (no quotes around string elements, no commas between numbers). Examples:
- `[836752fffffffff 836755fffffffff]` — correct
- `["836752fffffffff","836755fffffffff"]` — **WRONG**, quotes will break parsing
- `[-71.70 10.73 -71.52 10.55]` — correct
- `[]` — correct for empty

##### Evaluating Status Conditionals

**Process these BEFORE replacing simple `{{.Field}}` placeholders** — otherwise the `{{end}}` markers get consumed and the regex won't match.

The template uses `{{if eq .Status "..."}}` conditionals for the status badge CSS class and icon. Evaluate these by checking the entry's `status` value and keeping only the matching branch text.

The status badge line contains **two** `{{if eq .Status ...}}...{{end}}` blocks on a single line — one for the CSS class, one for the icon. Use `re.sub` with a callback to resolve all occurrences:

```パイソン
STATUS_CSS = {"ERROR": "エラー"、"WARNING": "警告"、"SUGGESTION": "提案"、"OK": "OK"}
STATUS_ICON = {
    "エラー": "bi-x-circle-fill",
    "警告": "二重感嘆符-三角形の塗りつぶし",
    "提案": "2 つの電球の充填",
    "OK": "bi-check-circle-fill",
}

defsolve_status_if(match_obj, status):
    """`status` に一致するブランチを {{if eq .Status ...}}...{{end}} ブロックから選択します。"""
    ブロック = match_obj.group(0)
    # 各ブランチを試してみましょう: {{if eq .Status "X"}}val{{else if ...}}val{{else}}val{{end}}
    st、val の場合 [("エラー",), ("警告",), ("提案",)]:
        # 一般的に解析する必要はありません - 既知のパターンからマッピングするだけです
    ...```

A simpler approach: since there are exactly two known patterns, replace them as literal strings:
```パイソン
css_class = STATUS_CSS.get(ステータス, "ok")
icon_class = STATUS_ICON.get(ステータス, "bi-check-circle-fill")
body = body.replace(
    '{{if eq .Status "ERROR"}}エラー{{else if eq .Status "WARNING"}}警告{{else if eq .Status "SUGGESTION"}}提案{{else}}ok{{end}}',
    css_class、
）
body = body.replace(
    '{{if eq .Status "ERROR"}}bi-x-circle-fill{{else if eq .Status "WARNING"}}bi-exclamation-triangle-fill{{else if eq .Status "SUGGESTION"}}bi-lightbulb-fill{{else}}bi-check-circle-fill{{end}}',
    アイコンクラス、
）```
This avoids regex entirely and is safe because these exact strings appear verbatim in the template.

#### Step 4: Expand the Nested Messages Range

The `{{range .Messages}}...{{end}}` block contains a **nested** `{{if .Checked}} checked{{else}} disabled{{end}}` conditional, so its inner `{{end}}` would cause a simple non-greedy regex to match too early. Anchor the regex to `</td>` (the tag immediately after the messages range closing `{{end}}`) to capture the full block body:

```パイソン
msg_match = re.search(
    r'\{\{range \.Messages\}\}(.*?)\{\{end\}\}\s*(?=</td>)',
    本体、re.DOTALL
）```

The lookahead `(?=</td>)` ensures the regex skips past the checkbox conditional's `{{end}}` (which is followed by `>`, not `</td>`) and matches only the range-closing `{{end}}` (which is followed by whitespace then `</td>`).

For each message in the entry's `Messages` array, clone the captured block body and expand it:

1. **Resolve the checkbox conditional** per message (must happen before simple placeholder replacement to remove the nested `{{end}}`):
   ```パイソン
   if msg.get("チェック済み"):
       msg_body = msg_body.replace(
           '{{if .Checked}} チェック済み{{else}} 無効化{{end}}'、' チェック済み'
       ）
   それ以外の場合:
       msg_body = msg_body.replace(
           '{{if .Checked}} チェック済み{{else}} 無効化{{end}}'、' 無効化'
       ）```

2. **Replace message field placeholders**:

   | Template placeholder | Source                            | Notes                          |
   |--------------------------|-----------------------------------|--------------------------------|
   | `{{.ID}}`                | `Messages[i].ID`                  | Direct string value from JSON  |
   | `{{.Text}}`              | `Messages[i].Text`                | HTML-escaped                   |

3. **Concatenate** all expanded message blocks and replace the original `{{range .Messages}}...{{end}}` match (`msg_match.group(0)`) with the result:
   ```パイソン
   body = body[:msg_match.start()] + "".join(expanded_msgs) + body[msg_match.end():]
   「」

`Messages` が空の場合は、一致した領域全体を空の文字列に置き換えます (メッセージ div はなく、問題ヘッダーのみが残ります)。

#### 出力の保証

- レポートは、テンプレートに既に含まれている CDN リンク (`leaflet`、`h3-js`、`bootstrap-icons`、Raleway フォント) を超える追加のネットワーク依存関係を持たずに、最新のブラウザーで読み取ることができる必要があります。
- レンダリングの問題を防ぐために、HTML に埋め込まれたすべての値は **HTML エスケープ** (`<`、`>`、`&`、`"`) する必要があります。
- `commentMap` は直接 JavaScript オブジェクト リテラル (文字列内ではない) として埋め込まれるため、JS 文字列エスケープは必要ありません。有効な JSON を出力するだけです。
- すべての値は、ヒューリスティックに再計算するのではなく、**分析出力のみから**導出する必要があります。


### フェーズ 6: 最終レビュー

ユーザーに結果を提示する前に、具体的なチェック可能なアサーションを使用して最終検証パスを実行します。

**チェック 1 — エントリ数の整合性**
- 元の入力 CSV 内のコメントと空白以外のデータ行をカウントします。
- アサート: `len(entries) in report-data.json == data_row_count`
- 失敗時: `Row count mismatch: input has {N} data rows but report contains {M} entries.`

**チェック 2 — サマリー カウンタの整合性**
- これらのカウンターは、最も重大度の高い `Status` フィールドを反映するブール値フラグに基づく **相互排他** を使用します。 `HasError: true` と `HasWarning: true` の両方を含むエントリは、`Errors` でのみカウントされ、`Warnings` ではカウントされません。これは、エントリの `Status` フィールドによってカウントすることと同じです。
- 次のすべてをアサートします。レポートを生成する前に、失敗したものを修正します。
  - `Errors == sum(1 for e in Entries if e['HasError'])`
  - `Warnings == sum(1 for e in Entries if e['HasWarning'] and not e['HasError'])`
  - `Suggestions == sum(1 for e in Entries if e['HasSuggestion'] and not e['HasError'] and not e['HasWarning'])`
  - `OK == sum(1 for e in Entries if not e['HasError'] and not e['HasWarning'] and not e['HasSuggestion'])`
  - `Errors + Warnings + Suggestions + OK == TotalEntries - InvalidEntries`

**チェック 3 — 精度バケットの整合性**
- アサート: `CityLevelAccuracy + RegionLevelAccuracy + CountryLevelAccuracy + DoNotGeolocate == TotalEntries - InvalidEntries`
- **注意:** フェーズ 3 で定義された精度バケットには「`HasError: true` のエントリをカウントしない」と記載されていますが、上記のチェック 3 の式では `TotalEntries - InvalidEntries` が使用されています (これにはまだ ERROR エントリが含まれています)。これは、エラー エントリ (有効な IP として解析されたが検証に失敗したエントリ) が、地理フィールドの存在によって**精度バケットにカウントされる**ことを意味します。 `InvalidEntries` (解析できない IP プレフィックス) のみが除外されます。権威あるルールとしてチェック 3 の公式に従ってください。
- 失敗した場合は、続行する前にバケット化ロジックをトレースして修正します。

**チェック 4 — 行番号が重複していないこと**
- アサート: `Entries` 内のすべての `Line` 値は一意です。
- 失敗した場合は、重複した行番号をユーザーに報告します。

**チェック 5 — TunedEntry の完全性**
- アサート: `Entries` のすべてのオブジェクトには `TunedEntry` キーがあります (値が `{}` であっても)。
- 失敗した場合は、キーが欠落しているエントリに `"TunedEntry": {}` を追加し、`report-data.json` を再保存します。

**チェック 6 — レポート ファイルが存在し、空ではない**
- `./run/report/geofeed-report.html` が書き込まれ、ファイル サイズが 0 バイトより大きいことを確認します。
- 失敗した場合は、ユーザーに提示する前にレポートを再生成します。