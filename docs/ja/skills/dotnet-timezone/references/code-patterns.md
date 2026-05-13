# .NET Timezone Code Patterns

## Pattern 1: Basic TimeZoneInfo

アプリケーションが Windows 専用で、Windows timezone ID を使ってよい場合にのみこれを使用してください。

```csharp
DateTime utcNow = DateTime.UtcNow;
TimeZoneInfo sriLankaTz = TimeZoneInfo.FindSystemTimeZoneById("Sri Lanka Standard Time");
DateTime localTime = TimeZoneInfo.ConvertTimeFromUtc(utcNow, sriLankaTz);

DateTime backToUtc = TimeZoneInfo.ConvertTimeToUtc(localTime, sriLankaTz);

TimeZoneInfo tokyoTz = TimeZoneInfo.FindSystemTimeZoneById("Tokyo Standard Time");
DateTime tokyoTime = TimeZoneInfo.ConvertTime(localTime, sriLankaTz, tokyoTz);
```

Linux、container、または混在環境では、代わりに `TimeZoneConverter` または `NodaTime` を使ってください。

## Pattern 2: Cross-Platform With TimeZoneConverter

Windows と Linux の両方で動作するほとんどの .NET アプリに対する推奨既定です。

```xml
<PackageReference Include="TimeZoneConverter" Version="6.*" />
```

```csharp
using TimeZoneConverter;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime converted = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

これは Windows ID も受け付けます。

```csharp
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Sri Lanka Standard Time");
```

## Pattern 3: NodaTime

厳密な timezone 演算、定期スケジュール、または最小依存より正確さが重要な DST 境界ケースではこれを使ってください。

```xml
<PackageReference Include="NodaTime" Version="3.*" />
```

```csharp
using NodaTime;

DateTimeZone colomboZone = DateTimeZoneProviders.Tzdb["Asia/Colombo"];
Instant now = SystemClock.Instance.GetCurrentInstant();
ZonedDateTime colomboTime = now.InZone(colomboZone);

DateTimeZone tokyoZone = DateTimeZoneProviders.Tzdb["Asia/Tokyo"];
ZonedDateTime tokyoTime = colomboTime.WithZone(tokyoZone);

LocalDateTime localDt = new LocalDateTime(2024, 6, 15, 14, 30, 0);
ZonedDateTime zoned = colomboZone.AtStrictly(localDt);
Instant utcInstant = zoned.ToInstant();
```

## Pattern 4: DateTimeOffset For APIs

サービスやプロセスの境界をまたぐ値には `DateTimeOffset` を優先してください。

```csharp
using TimeZoneConverter;

DateTimeOffset utcNow = DateTimeOffset.UtcNow;
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTimeOffset colomboTime = TimeZoneInfo.ConvertTime(utcNow, tz);
```

## Pattern 5: ASP.NET Core Persistence And Presentation

UTC で保存し、境界で変換してください。

```csharp
using TimeZoneConverter;

entity.CreatedAtUtc = DateTime.UtcNow;

public DateTimeOffset ToUserTime(DateTime utc, string userIanaTimezone)
{
    var tz = TZConvert.GetTimeZoneInfo(userIanaTimezone);
    return TimeZoneInfo.ConvertTimeFromUtc(utc, tz);
}
```

## Pattern 6: Scheduling And Recurring Jobs

ユーザー向けのローカル時刻を、スケジュール前に UTC へ変換してください。

```csharp
using TimeZoneConverter;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime scheduledLocal = new DateTime(2024, 12, 1, 9, 0, 0, DateTimeKind.Unspecified);
DateTime scheduledUtc = TimeZoneInfo.ConvertTimeToUtc(scheduledLocal, tz);
```

With Hangfire:

```csharp
RecurringJob.AddOrUpdate(
    "morning-job",
    () => DoWork(),
    "0 9 * * *",
    new RecurringJobOptions { TimeZone = tz });
```

## Pattern 7: Ambiguous And Invalid DST Times

その timezone が夏時間を採用している場合、ローカル timestamp の重複や欠落がないか確認してください。

```csharp
using TimeZoneConverter;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("America/New_York");
DateTime localTime = new DateTime(2024, 11, 3, 1, 30, 0);

if (tz.IsAmbiguousTime(localTime))
{
    var offsets = tz.GetAmbiguousTimeOffsets(localTime);
    var standardOffset = offsets.Min();
    var dto = new DateTimeOffset(localTime, standardOffset);
}

if (tz.IsInvalidTime(localTime))
{
    localTime = localTime.AddHours(1);
}
```

## Common Mistakes

| Wrong | Better |
| --- | --- |
| `DateTime.Now` in server code | `DateTime.UtcNow` |
| Storing local timestamps in the database | Store UTC and convert for display |
| Hardcoding offsets such as `+05:30` | Use timezone IDs |
| Using `FindSystemTimeZoneById("Asia/Colombo")` on Windows | Use `TZConvert.GetTimeZoneInfo("Asia/Colombo")` |
| Comparing local `DateTime` values from different zones | Compare UTC or use `DateTimeOffset` |
| Creating `DateTime` without intentional kind semantics | Use `Utc`, `Local`, or deliberate `Unspecified` |

## Decision Guide

- Windows 専用コードで Windows ID を使う場合にのみ `TimeZoneInfo` を使う。
- ほとんどのクロスプラットフォーム アプリケーションでは `TimeZoneConverter` を使う。
- DST 演算やカレンダー精度が重要なら `NodaTime` を使う。
- API やシリアライズされる timestamp には `DateTimeOffset` を使う。
