---
applyTo: '**/*.ps1,**/*.psm1'
description: 'Microsoft ガイドラインに基づく PowerShell cmdlet と scripting のベスト プラクティス'
---

# PowerShell Cmdlet 開発ガイドライン

このガイドは、GitHub Copilot が慣用的で安全かつ保守しやすい script を生成できるようにするための、PowerShell 固有 instruction を提供します。Microsoft の PowerShell cmdlet 開発ガイドラインに準拠しています。

## 命名規則

- **Verb-Noun 形式:**
  - 承認済みの PowerShell verb を使う (Get-Verb)
  - noun は単数形を使う
  - verb と noun の両方に PascalCase を使う
  - 特殊文字と空白を避ける

- **Parameter 名:**
  - PascalCase を使う
  - 明確で説明的な名前を選ぶ
  - 常に複数でない限り単数形を使う
  - PowerShell の標準名に従う

- **Variable 名:**
  - public variable には PascalCase を使う
  - private variable には camelCase を使う
  - 略語を避ける
  - 意味のある名前を使う

- **Alias の回避:**
  - 完全な cmdlet 名を使う
  - script 内で alias を使わない (例: `gci` ではなく `Get-ChildItem` を使う)
  - custom alias は文書化する
  - 完全な parameter 名を使う

### 例 - 命名規則

```powershell
function Get-UserProfile {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Username,

        [Parameter()]
        [ValidateSet('Basic', 'Detailed')]
        [string]$ProfileType = 'Basic'
    )

    process {
        $outputString = "Searching for: '$($Username)'"
        Write-Verbose -Message $outputString
        Write-Verbose -Message "Profile type: $ProfileType"
        # Logic here
    }
}
```

## Parameter 設計

- **標準 Parameter:**
  - 一般的な parameter 名 (`Path`、`Name`、`Force`) を使う
  - 組み込み cmdlet の規約に従う
  - 特化した用語には alias を使う
  - parameter の目的を文書化する

- **Parameter 名:**
  - 常に複数でない限り単数形を使う
  - 明確で説明的な名前を選ぶ
  - PowerShell 規約に従う
  - PascalCase 形式を使う

- **型の選択:**
  - 一般的な .NET type を使う
  - 適切な validation を実装する
  - 選択肢が限られる場合は ValidateSet を検討する
  - 可能なら tab completion を有効にする

- **Switch Parameter:**
  - boolean flag には **常に** `[switch]` を使い、`[bool]` は使わない
  - **絶対に** `[bool]$Parameter` を使ったり default 値を割り当てたりしない
  - switch parameter は省略時に `$false` が既定になる
  - 明確で action 指向の名前を使う
  - 存在確認には `.IsPresent` を使う
  - parameter attribute 内で `$true` / `$false` を使うこと (例: `Mandatory = $true`) は問題ない

### 例 - Parameter 設計

```powershell
function Set-ResourceConfiguration {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Name,

        [Parameter()]
        [ValidateSet('Dev', 'Test', 'Prod')]
        [string]$Environment = 'Dev',

        # ✔️ CORRECT: Use `[switch]` with no default value
        [Parameter()]
        [switch]$Force,

         # ❌ WRONG: Shows incorrect default assignment, however this is correct syntax (requires `[switch]` cast).
        [Parameter()]
        [switch]$Quiet = [switch]$true,

        [Parameter()]
        [ValidateNotNullOrEmpty()]
        [string[]]$Tags
    )

    process {
        # Use .IsPresent to check switch state
        if ($Quiet.IsPresent) {
            Write-Verbose "Quiet mode enabled"
        }
    }
}
```

## Pipeline と Output

- **Pipeline Input:**
  - 直接 object を受け取るには `ValueFromPipeline` を使う
  - property 名による mapping には `ValueFromPipelineByPropertyName` を使う
  - pipeline 処理には Begin / Process / End block を実装する
  - pipeline input の要件を文書化する

- **Output Object:**
  - 整形済み text ではなく、リッチな object を返す
  - structured data には PSCustomObject を使う
  - データ出力に Write-Host を使わない
  - 下流 cmdlet で処理できるようにする

- **Pipeline Streaming:**
  - 一度に 1 object ずつ出力する
  - streaming には process block を使う
  - 大きな array を集めない
  - 即時処理を可能にする

- **PassThru パターン:**
  - action cmdlet では既定で output しない
  - object を返すための `-PassThru` switch を実装する
  - `-PassThru` がある場合は変更済み / 作成済み object を返す
  - 状態更新には verbose / warning を使う

### 例 - Pipeline と Output

```powershell
function Update-ResourceStatus {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
        [string]$Name,

        [Parameter(Mandatory)]
        [ValidateSet('Active', 'Inactive', 'Maintenance')]
        [string]$Status,

        [Parameter()]
        [switch]$PassThru
    )

    begin {
        Write-Verbose 'Starting resource status update process'
        $timestamp = Get-Date
    }

    process {
        # Process each resource individually
        Write-Verbose "Processing resource: $Name"

        $resource = [PSCustomObject]@{
            Name        = $Name
            Status      = $Status
            LastUpdated = $timestamp
            UpdatedBy   = "$($env:USERNAME)"
        }

        # Only output if PassThru is specified
        if ($PassThru.IsPresent) {
            Write-Output $resource
        }
    }

    end {
        Write-Verbose 'Resource status update process completed'
    }
}
```

## エラー処理と安全性

- **ShouldProcess 実装:**
  - `[CmdletBinding(SupportsShouldProcess = $true)]` を使う
  - 適切な `ConfirmImpact` level を設定する
  - `$PSCmdlet.ShouldProcess()` は変更アクションのできるだけ直前で呼ぶ
  - 追加確認には `$PSCmdlet.ShouldContinue()` を使う

- **Message Stream:**
  - `Write-Verbose` は `-Verbose` 時の運用詳細に使う
  - `Write-Warning` は警告条件に使う
  - `Write-Error` は non-terminating error に使う
  - `throw` は terminating error に使う
  - ユーザー interface 用 text を除き `Write-Host` は避ける

- **エラー処理パターン:**
  - エラー管理には try / catch block を使う
  - 適切な ErrorAction 設定を行う
  - 意味のある error message を返す
  - 必要に応じて ErrorVariable を使う
  - 適切な terminating / non-terminating error 処理を含める
  - `[CmdletBinding()]` を持つ advanced function では、`Write-Error` より `$PSCmdlet.WriteError()` を優先する
  - `[CmdletBinding()]` を持つ advanced function では、`throw` より `$PSCmdlet.ThrowTerminatingError()` を優先する
  - category、target、exception の詳細を持つ適切な ErrorRecord object を構築する

- **非対話設計:**
  - input は parameter 経由で受け取る
  - script で `Read-Host` を使わない
  - automation scenario をサポートする
  - 必須 input はすべて文書化する

### 例 - エラー処理と安全性

```powershell
function Remove-CacheFiles {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param(
        [Parameter(Mandatory)]
        [string]$Path
    )

    try {
        $files = Get-ChildItem -Path $Path -Filter "*.cache" -ErrorAction Stop

        # Demonstrates WhatIf support
        if ($PSCmdlet.ShouldProcess($Path, 'Remove cache files')) {
            $files | Remove-Item -Force -ErrorAction Stop
            Write-Verbose "Removed $($files.Count) cache files from $Path"
        }
    } catch {
        $errorRecord = [System.Management.Automation.ErrorRecord]::new(
            $_.Exception,
            'RemovalFailed',
            [System.Management.Automation.ErrorCategory]::NotSpecified,
            $Path
        )
        $PSCmdlet.WriteError($errorRecord)
    }
}
```

## ドキュメントとスタイル

- **Comment-Based Help:** public 向けの関数や cmdlet には comment-based help を含める。関数内に `<# ... #>` の help comment を追加し、最低でも次を含める:
  - `.SYNOPSIS` 簡潔な説明
  - `.DESCRIPTION` 詳細な説明
  - `.EXAMPLE` 実用的な使用例の section
  - `.PARAMETER` parameter の説明
  - `.OUTPUTS` 返される output の型
  - `.NOTES` 補足情報

- **一貫した Formatting:**
  - 一貫した PowerShell style に従う
  - 適切な indentation を使う (4 space 推奨)
  - 開き brace は statement と同じ行に置く
  - 閉じ brace は新しい行に置く
  - pipeline operator の後で改行する
  - 関数名と parameter 名には PascalCase を使う
  - 不要な空白を避ける

- **Pipeline サポート:**
  - pipeline 関数には Begin / Process / End block を実装する
  - 適切な場面で ValueFromPipeline を使う
  - property 名による pipeline input をサポートする
  - 整形済み text ではなく proper object を返す

- **Alias を避ける:** 完全な cmdlet 名と parameter を使う
  - script 内で alias を使わない (例: `gci` ではなく Get-ChildItem を使う)。alias は対話シェル用途では許容される。
  - `?` や `where` ではなく `Where-Object` を使う
  - `%` ではなく `ForEach-Object` を使う
  - `ls` や `dir` ではなく `Get-ChildItem` を使う

---

## 完全な例: End-to-End Cmdlet パターン

```powershell
function Remove-UserAccount {
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'High')]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [ValidateNotNullOrEmpty()]
        [string]$Username,

        [Parameter()]
        [switch]$Force
    )

    begin {
        Write-Verbose 'Starting user account removal process'
        $currentErrorActionValue = $ErrorActionPreference
        $ErrorActionPreference = 'Stop'
    }

    process {
        try {
            # Validation
            if (-not (Test-UserExists -Username $Username)) {
                $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                    [System.Exception]::new("User account '$Username' not found"),
                    'UserNotFound',
                    [System.Management.Automation.ErrorCategory]::ObjectNotFound,
                    $Username
                )
                $PSCmdlet.WriteError($errorRecord)
                return
            }

            # ShouldProcess enables -WhatIf and -Confirm support
            if ($PSCmdlet.ShouldProcess($Username, "Remove user account")) {
                # ShouldContinue provides an additional confirmation prompt for high-impact operations
                # This prompt is bypassed when -Force is specified
                if ($Force -or $PSCmdlet.ShouldContinue("Are you sure you want to remove '$Username'?", "Confirm Removal")) {
                    Write-Verbose "Removing user account: $Username"

                    # Main operation
                    Remove-ADUser -Identity $Username -ErrorAction Stop
                    Write-Warning "User account '$Username' has been removed"
                }
            }
        } catch [Microsoft.ActiveDirectory.Management.ADException] {
            $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                $_.Exception,
                'ActiveDirectoryError',
                [System.Management.Automation.ErrorCategory]::NotSpecified,
                $Username
            )
            $PSCmdlet.ThrowTerminatingError($errorRecord)
        } catch {
            $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                $_.Exception,
                'UnexpectedError',
                [System.Management.Automation.ErrorCategory]::NotSpecified,
                $Username
            )
            $PSCmdlet.ThrowTerminatingError($errorRecord)
        }
    }

    end {
        Write-Verbose 'User account removal process completed'
        # Set ErrorActionPreference back to the value it had
        $ErrorActionPreference = $currentErrorActionValue
    }
}
```
