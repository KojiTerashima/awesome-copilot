---
applyTo: '**/*.Tests.ps1'
description: 'Pester v5 規約に基づく PowerShell Pester テストのベスト プラクティス'
---

# PowerShell Pester v5 テスト ガイドライン

このガイドは、PowerShell Pester v5 module を使って自動テストを作成するための PowerShell 固有 instruction を提供します。一般的な PowerShell scripting の best practice については、[powershell.instructions.md](./powershell.instructions.md) の PowerShell cmdlet 開発ガイドラインに従ってください。

## ファイル名と構造

- **ファイル規約:** `*.Tests.ps1` 命名パターンを使う
- **配置:** テスト file は対象 code の隣、または専用の test directory に置く
- **Import パターン:** テスト対象関数の import には `BeforeAll { . $PSScriptRoot/FunctionName.ps1 }` を使う
- **直接 code を置かない:** すべての code は Pester block (`BeforeAll`、`Describe`、`Context`、`It` など) の中に置く

## テスト構造の階層

```powershell
BeforeAll { # Import tested functions }
Describe 'FunctionName' {
    Context 'When condition' {
        BeforeAll { # Setup for context }
        It 'Should behavior' { # Individual test }
        AfterAll { # Cleanup for context }
    }
}
```

## コア キーワード

- **`Describe`**: 最上位の grouping。通常は対象関数名にする
- **`Context`**: 特定シナリオ向けの `Describe` 内 sub-grouping
- **`It`**: 個別のテスト ケース。説明的な名前を使う
- **`Should`**: テスト検証用の assertion keyword
- **`BeforeAll/AfterAll`**: block ごとに 1 回だけ行う setup / teardown
- **`BeforeEach/AfterEach`**: 各テストの前後に行う setup / teardown

## セットアップとクリーンアップ

- **`BeforeAll`**: 含まれる block の先頭で 1 回実行される。高コスト操作に使う
- **`BeforeEach`**: block 内の各 `It` の前に実行される。テスト固有の setup に使う
- **`AfterEach`**: 各 `It` の後に実行され、テスト失敗時でも保証される
- **`AfterAll`**: block の最後で 1 回実行される。cleanup に使う
- **変数スコープ:** `BeforeAll` の変数は子 block から参照可能 (読み取り専用)。`BeforeEach` / `It` / `AfterEach` は同じスコープを共有する

## Assertion (Should)

- **基本比較**: `-Be`、`-BeExactly`、`-Not -Be`
- **Collection**: `-Contain`、`-BeIn`、`-HaveCount`
- **数値**: `-BeGreaterThan`、`-BeLessThan`、`-BeGreaterOrEqual`
- **文字列**: `-Match`、`-Like`、`-BeNullOrEmpty`
- **型**: `-BeOfType`、`-BeTrue`、`-BeFalse`
- **File**: `-Exist`、`-FileContentMatch`
- **例外**: `-Throw`、`-Not -Throw`

## Mocking

- **`Mock CommandName { ScriptBlock }`**: command の挙動を置き換える
- **`-ParameterFilter`**: parameter が条件に一致したときだけ mock する
- **`-Verifiable`**: 検証必須の mock として印を付ける
- **`Should -Invoke`**: mock が指定回数呼ばれたことを検証する
- **`Should -InvokeVerifiable`**: verifiable mock がすべて呼ばれたことを検証する
- **Scope**: mock の既定 scope は含まれる block

```powershell
Mock Get-Service { @{ Status = 'Running' } } -ParameterFilter { $Name -eq 'TestService' }
Should -Invoke Get-Service -Exactly 1 -ParameterFilter { $Name -eq 'TestService' }
```

## Test Cases (データ駆動テスト)

parameter 化テストには `-TestCases` または `-ForEach` を使います。

```powershell
It 'Should return <Expected> for <Input>' -TestCases @(
    @{ Input = 'value1'; Expected = 'result1' }
    @{ Input = 'value2'; Expected = 'result2' }
) {
    Get-Function $Input | Should -Be $Expected
}
```

## データ駆動テスト

- **`-ForEach`**: `Describe`、`Context`、`It` で利用でき、データから複数のテストを生成する
- **`-TestCases`**: `It` block における `-ForEach` の alias (後方互換性)
- **Hashtable データ**: 各項目はテストで利用可能な変数を定義する (例: `@{ Name = 'value'; Expected = 'result' }`)
- **Array データ**: 現在の項目には `$_` 変数を使う
- **Template**: テスト名に `<variablename>` を使って動的展開する

```powershell
# Hashtable approach
It 'Returns <Expected> for <Name>' -ForEach @(
    @{ Name = 'test1'; Expected = 'result1' }
    @{ Name = 'test2'; Expected = 'result2' }
) { Get-Function $Name | Should -Be $Expected }

# Array approach
It 'Contains <_>' -ForEach 'item1', 'item2' { Get-Collection | Should -Contain $_ }
```

## Tags

- **利用可能な block**: `Describe`、`Context`、`It`
- **Filtering**: `Invoke-Pester` に対して `-TagFilter` と `-ExcludeTagFilter` を使う
- **Wildcard**: tag は `-like` wildcard をサポートし、柔軟に filter できる

```powershell
Describe 'Function' -Tag 'Unit' {
    It 'Should work' -Tag 'Fast', 'Stable' { }
    It 'Should be slow' -Tag 'Slow', 'Integration' { }
}

# Run only fast unit tests
Invoke-Pester -TagFilter 'Unit' -ExcludeTagFilter 'Slow'
```

## Skip

- **`-Skip`**: `Describe`、`Context`、`It` で利用でき、テストを skip する
- **条件付き**: `-Skip:$condition` で動的に skip する
- **実行時 Skip**: テスト実行中に `Set-ItResult -Skipped` を使う (setup / teardown は引き続き実行される)

```powershell
It 'Should work on Windows' -Skip:(-not $IsWindows) { }
Context 'Integration tests' -Skip { }
```

## エラー処理

- **失敗後も継続**: 複数の失敗を収集したい場合は `Should.ErrorAction = 'Continue'` を使う
- **重要条件で停止**: 前提条件には `-ErrorAction Stop` を使う
- **例外テスト**: 例外検証には `{ Code } | Should -Throw` を使う

## ベスト プラクティス

- **説明的な名前**: 振る舞いが分かる明確なテスト名を使う
- **AAA Pattern**: Arrange (準備)、Act (実行)、Assert (検証)
- **独立したテスト**: 各テストは互いに独立させる
- **Alias を避ける**: 完全な cmdlet 名を使う (`?` ではなく `Where-Object`)
- **単一責任**: 可能な限り 1 テスト 1 assertion にする
- **テスト file の整理**: 関連テストは Context block でグループ化する。Context block はネスト可能

## テスト パターンの例

```powershell
BeforeAll {
    . $PSScriptRoot/Get-UserInfo.ps1
}

Describe 'Get-UserInfo' {
    Context 'When user exists' {
        BeforeAll {
            Mock Get-ADUser { @{ Name = 'TestUser'; Enabled = $true } }
        }

        It 'Should return user object' {
            $result = Get-UserInfo -Username 'TestUser'
            $result | Should -Not -BeNullOrEmpty
            $result.Name | Should -Be 'TestUser'
        }

        It 'Should call Get-ADUser once' {
            Get-UserInfo -Username 'TestUser'
            Should -Invoke Get-ADUser -Exactly 1
        }
    }

    Context 'When user does not exist' {
        BeforeAll {
            Mock Get-ADUser { throw "User not found" }
        }

        It 'Should throw exception' {
            { Get-UserInfo -Username 'NonExistent' } | Should -Throw "*not found*"
        }
    }
}
```

## 設定

設定は **テスト file の外側** で `Invoke-Pester` 呼び出し時に定義し、実行挙動を制御します。

```powershell
# Create configuration (Pester 5.2+)
$config = New-PesterConfiguration
$config.Run.Path = './Tests'
$config.Output.Verbosity = 'Detailed'
$config.TestResult.Enabled = $true
$config.TestResult.OutputFormat = 'NUnitXml'
$config.Should.ErrorAction = 'Continue'
Invoke-Pester -Configuration $config
```

**主な section**: Run (Path、Exit)、Filter (Tag、ExcludeTag)、Output (Verbosity)、TestResult (Enabled、OutputFormat)、CodeCoverage (Enabled、Path)、Should (ErrorAction)、Debug
