---
name: dotnet-timezone
description: '.NET timezone handling guidance for C# applications. TimeZoneInfo、DateTimeOffset、NodaTime、UTC 変換、夏時間、複数タイムゾーンをまたぐスケジューリング、Windows/IANA のクロスプラットフォーム timezone ID、または都市・住所・地域・国に対応する timezone とそのまま使える C# コードが必要な .NET ユーザー向けです。'
---

# .NET Timezone

本番で安全に使えるガイダンスと、そのまま貼り付けて使えるスニペットで、.NET と C# の timezone に関する質問を解決してください。

## Start With The Right Path

まず依頼の種類を特定してください。

- Address or location lookup
- Timezone ID lookup
- UTC/local conversion
- Cross-platform timezone compatibility
- Scheduling or DST handling
- API or persistence design

利用ライブラリが不明な場合は、クロスプラットフォーム用途では `TimeZoneConverter` を既定とします。定期スケジュールや厳密な DST ルールを扱う場合は `NodaTime` を優先してください。

## Resolve Addresses And Locations

ユーザーが住所、都市、地域、国、または地名を含む文書を提供した場合:

1. 入力から各 location を抽出する。
2. `references/timezone-index.md` を読んで一般的な Windows と IANA の対応を確認する。
3. 正確な location が載っていない場合は、地理情報から正しい IANA zone を推定し、それを Windows ID に対応付ける。
4. 両方の ID と、そのまま使える C# の例を返す。

解決した各 location について、次を提示してください。

```text
Location: <resolved place>
Windows ID: <windows id>
IANA ID: <iana id>
UTC offset: <standard offset and DST offset when relevant>
DST: <yes/no>
```

その後、次のようなクロスプラットフォーム スニペットを含めてください。

```csharp
using TimeZoneConverter;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime local = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

複数の location がある場合は location ごとに 1 ブロックずつ示し、そのあとに複数 timezone をまとめて扱うスニペットを含めてください。

location が曖昧な場合は、考えられる timezone の候補を列挙し、ユーザーに正しいものを選んでもらってください。

## Look Up Timezone IDs

Windows から IANA への対応には `references/timezone-index.md` を使ってください。

常に両方の形式を提示してください。

- Windows で `TimeZoneInfo.FindSystemTimeZoneById()` を使うための Windows ID
- Linux、container、`NodaTime`、`TimeZoneConverter` 向けの IANA ID

## Generate Code

`references/code-patterns.md` を使い、要件に合う最小のパターンを選んでください。

- Pattern 1: Windows 専用コード向けの `TimeZoneInfo`
- Pattern 2: クロスプラットフォーム変換向けの `TimeZoneConverter`
- Pattern 3: 厳密な timezone 演算や DST に敏感なスケジューリング向けの `NodaTime`
- Pattern 4: API とデータ転送向けの `DateTimeOffset`
- Pattern 5: ASP.NET Core の永続化と表示
- Pattern 6: 定期ジョブと scheduler
- Pattern 7: 曖昧または無効な DST timestamp

サードパーティ ライブラリを推奨する場合は、必ず package の案内も含めてください。

## Warn About Common Pitfalls

該当する場合は、関連する注意点を述べてください。

- `TimeZoneInfo.FindSystemTimeZoneById()` は timezone ID の扱いがプラットフォーム依存です。
- データベースに `DateTime.Now` を保存せず、UTC を保存してください。
- `DateTimeKind.Unspecified` は、意図的な入力でない限りバグ要因として扱ってください。
- DST の切り替え時にはローカル時刻が飛ぶ、または重複する場合があります。
- Azure Windows と Azure Linux 環境では、期待される timezone ID 形式が異なる場合があります。

## Response Shape

住所や location の依頼では:

1. 各 location について解決した timezone block を返す。
2. 推奨する実装を 1 文で述べる。
3. そのまま使える C# スニペットを含める。

コードやアーキテクチャの依頼では:

1. 推奨アプローチを 1 文で述べる。
2. relevant であれば timezone ID を提示する。
3. 最小限の動作するコード スニペットを含める。
4. 必要なら package 要件を記載する。
5. 重要なら pitfall warning を 1 つ追加する。

回答は簡潔にし、コード優先で提示してください。

## References

- `references/timezone-index.md`: 一般的な Windows と IANA timezone の対応
- `references/code-patterns.md`: すぐ使える .NET timezone パターン
