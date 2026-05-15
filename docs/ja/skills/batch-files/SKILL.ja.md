---
name: batch-files
description: 'CMD スクリプトの作成、デバッグ、および保守のためのエキスパート レベルの Windows バッチ ファイル (.bat/.cmd) スキル。 「バッチ ファイルの作成」、「.bat スクリプトの作成」、「Windows タスクの自動化」、「CMD スクリプト」、「バッチ自動化」、「スケジュールされたタスク スクリプト」、「Windows シェル スクリプト」を要求された場合、またはワークスペースで .bat/.cmd ファイルを操作する場合に使用します。 cmd.exe 構文、環境変数、制御フロー、文字列処理、エラー処理、システム ツールとの統合について説明します。'
---

# バッチファイル

cmd.exe を使用して Windows バッチ ファイル (.bat/.cmd) を作成、編集、デバッグ、および保守するための包括的なスキル。 CLI ツール開発、システム管理自動化、スケジュールされたタスク、ファイル操作スクリプト、および PATH ベースの実行可能スクリプトに適用されます。

## このスキルをいつ使用するか

- `.bat` または `.cmd` ファイルの作成または編集
- Windows タスクの自動化 (ファイル操作、展開、バックアップ)
- PATH 上の `bin/` フォルダー用の CLI ツールを構築する
- スケジュールされたタスク スクリプトの作成 (SCHTASKS、タスク スケジューラ)
- バッチ スクリプトの問題のデバッグ (変数の展開、エラー レベル、引用符)
- バッチ スクリプトと外部ツール (curl、git、Node.js、Python) の統合
- 構造化されたテンプレートを使用して新しいバッチベースのプロジェクトを構築する

## 前提条件

- Windows NTベースのOS（Windows 7以降）
- cmd.exe (組み込み)
- オプション: スクリプトをコマンドとして配布するための PATH 上の `bin/` ディレクトリ
- オプション: `.BAT;.CMD` を含めるように構成された PATHEXT (Windows のデフォルト)

## コマンドの解釈

cmd.exe は、各行を 4 つの段階で順番に処理します。

1. **変数置換** — `%VAR%` トークンは環境変数値に置き換えられます。 `%0`–`%9` 参照バッチ引数。 `%*` はすべての引数に展開されます。
2. **引用符とエスケープ** — キャレット `^` は特殊文字 (`& | < > ^`) をエスケープします。引用符は、囲まれた特殊文字の解釈を妨げます。バッチ ファイルでは、`%%` はリテラルの `%` を生成します。
3. **構文解析** — 行はパイプライン (`|`)、複合コマンド (`&`、`&&`、`||`)、および括弧で囲まれたグループ `( )` に分割されます。
4. **リダイレクト** — `>` は上書き、`>>` は追加、`<` は入力を読み取り、`2>` は stderr をリダイレクトし、`2>&1` は stderr を stdout にマージし、`>NUL` は出力を破棄します。

## 変数

### 環境変数
```bat
set _MY_VAR=Hello World
echo %_MY_VAR%
set _MY_VAR=
```

- 引数なしの `set` はすべての変数をリストします
- `set _PREFIX` は、`_PREFIX` で始まる変数をリストします。
- `=` の周りにスペースを入れない — `set name = val` は変数 `"name "` を `" val"` に設定します

### 特殊な変数

|変数 |値 |
|----------|----------|
| `%CD%` |現在のディレクトリ |
| `%DATE%` |システム日付 (ロケールに依存) |
| `%TIME%` |システム時刻 HH:MM:SS.mm |
| `%RANDOM%` |擬似乱数 0 ～ 32767 |
| `%ERRORLEVEL%` |最後のコマンドの終了コード |
| `%USERNAME%` |現在のユーザー名 |
| `%USERPROFILE%` |現在のユーザー プロファイル パス |
| `%TEMP%` / `%TMP%` |一時ファイルディレクトリ |
| `%PATHEXT%` |実行可能拡張子のリスト |
| `%COMSPEC%` | cmd.exe へのパス |

### SETLOCAL / ENDLOCAL によるスコープ指定
```bat
setlocal
set _LOCAL_VAR=scoped value
endlocal
REM _LOCAL_VAR is no longer defined here
```

スコープ付きブロックから値を返すには:
```bat
endlocal & set _RESULT=%_LOCAL_VAR%
```

### 遅延した拡張

括弧で囲まれたブロック内の変数は解析時に展開されます。実行時評価に遅延拡張を使用します。
```bat
setlocal EnableDelayedExpansion
set _COUNT=0
for /l %%i in (1,1,5) do (
    set /a _COUNT+=1
    echo !_COUNT!
)
endlocal
```

- `!VAR!` は実行時に展開されます (遅延)
- `%VAR%` は解析時に展開されます (即時)

## 制御フロー

### 条件付き実行
```bat
if exist "output.txt" echo File found
if not defined _MY_VAR echo Variable not set
if "%_STATUS%"=="ready" (echo Go) else (echo Wait)
if %ERRORLEVEL% neq 0 echo Command failed
```

比較演算子: `equ`、`neq`、`lss`、`leq`、`gtr`、`geq`。大文字と小文字を区別しない文字列比較には `/i` を使用します。

### 複合コマンド
```bat
command1 & command2        & REM Always run both
command1 && command2       & REM Run command2 only if command1 succeeds
command1 || command2       & REM Run command2 only if command1 fails
```

### FOR ループ
```bat
REM Iterate over a set of values
for %%i in (alpha beta gamma) do echo %%i

REM Numeric range: start, step, end
for /l %%i in (1,1,10) do echo %%i

REM Files in a directory
for %%f in (*.txt) do echo %%f

REM Recursive file search
for /r %%f in (*.log) do echo %%f

REM Directories only
for /d %%d in (*) do echo %%d

REM Parse command output
for /f "tokens=1,2 delims=:" %%a in ('ipconfig ^| findstr "IPv4"') do echo %%b

REM Parse file lines
for /f "usebackq tokens=*" %%a in ("data.txt") do echo %%a
```

### GOTOとラベル
```bat
goto :main_logic
:usage
echo Usage: %~nx0 [options]
exit /b 1

:main_logic
echo Running main logic...
goto :eof
```

`goto :eof` は、現在のバッチまたはサブルーチンを終了します。ラベルは `:` で始まります。

## コマンドライン引数

|構文 |値 |
|------|------|
| `%0` |呼び出されたスクリプト名 |
| `%1`–`%9` |位置引数 |
| `%*` |すべての引数 (SHIFT の影響を受けない) |
| `%~1` |引用符を削除した引数 1 |
| `%~f1` |引数 1 のフルパス |
| `%~d1` |引数 1 のドライブ文字 |
| `%~p1` |引数 1 のパス (ドライブなし) |
| `%~n1` |引数1のファイル名(拡張子なし) |
| `%~x1` |引数 1 の拡張 |
| `%~dp0` |バッチ ファイル自体のドライブとパス |
| `%~nx0` |バッチファイルの拡張子が付いたファイル名 |
| `%~z1` |引数1のファイルサイズ |
| `%~$PATH:1` |引数 1 の PATH を検索 |

### 引数解析パターン
```bat
:parse_args
if "%~1"=="" goto :args_done
if /i "%~1"=="--help" goto :usage
if /i "%~1"=="--output" (
    set "_OUTPUT_DIR=%~2"
    shift
)
shift
goto :parse_args
:args_done
```

## 文字列処理

### 部分文字列
```bat
set _STR=Hello World
echo %_STR:~0,5%       & REM "Hello"
echo %_STR:~6%         & REM "World"
echo %_STR:~-5%        & REM "World"
echo %_STR:~0,-6%      & REM "Hello"
```

### 検索と置換
```bat
set _STR=Hello World
echo %_STR:World=Earth%       & REM "Hello Earth"
echo %_STR:Hello=%            & REM " World" (remove "Hello")
```

### 部分文字列の包含テスト
```bat
if not "%_STR:World=%"=="%_STR%" echo Contains "World"
```

## 機能

関数はラベル、CALL、SETLOCAL/ENDLOCAL を使用します。
```bat
@echo off
call :greet "Jane Doe"
echo Result: %_GREETING%
exit /b 0

:greet
setlocal
set "_MSG=Hello, %~1"
endlocal & set "_GREETING=%_MSG%"
exit /b 0
```

- `call :label args` は関数を呼び出します
- `exit /b` は (スクリプトではなく) 関数から戻ります
- `endlocal & set` トリックを使用して、スコープ付きブロックから値を渡します

## 算術

`set /a` は 32 ビットの符号付き整数演算を実行します。
```bat
set /a _RESULT=10 * 5 + 3
set /a _COUNTER+=1
set /a _REMAINDER=14 %% 3       & REM Use %% for modulo in batch files
set /a _BITS="255 & 0x0F"       & REM Bitwise AND
```

サポートされている演算子: `+ - * / %% ( )` およびビットごとの `& | ^ ~ << >>`。

16 進数 (`0xFF`) および 8 進数 (`077`) のリテラルがサポートされています。

## エラー処理

### エラーレベルの表記規則

- `0` = 成功
- ゼロ以外 = 失敗 (通常は `1`)
```bat
mycommand.exe
if %ERRORLEVEL% neq 0 (
    echo ERROR: mycommand failed with code %ERRORLEVEL%
    exit /b %ERRORLEVEL%
)
```

### フェイルファストパターン
```bat
command1 || (echo command1 failed & exit /b 1)
command2 || (echo command2 failed & exit /b 1)
```

### 終了コードの設定
```bat
exit /b 0        & REM Return success from a batch/function
exit /b 1        & REM Return failure
cmd /c "exit /b 42"   & REM Set ERRORLEVEL to 42 inline
```

## 重要なコマンドのリファレンス

### ファイル操作

|コマンド |目的 |
|----------|----------|
| `DIR` |ディレクトリの内容をリストする |
| `COPY` |ファイルをコピーする |
| `XCOPY` |サブディレクトリを含む拡張コピー (レガシー) |
| `ROBOCOPY` |再試行、ミラーリング、ロギングを備えた堅牢なコピー |
| `MOVE` |ファイルを移動または名前変更する |
| `DEL` |ファイルを削除する |
| `REN` |ファイルの名前を変更する |
| `MD` / `MKDIR` |ディレクトリを作成する |
| `RD` / `RMDIR` |ディレクトリを削除する |
| `MKLINK` |シンボリック リンクまたはハード リンクを作成する |
| `ATTRIB` |ファイル属性を表示または設定する |
| `TYPE` |ファイルの内容を印刷する |
| `MORE` |ページ分割されたファイルの表示 |
| `TREE` |ディレクトリ構造を表示 |
| `REPLACE` |宛先のファイルをソースで置き換える |
| `COMPACT` | NTFS 圧縮を表示または設定する |
| `EXPAND` | .cab ファイルから抽出 |
| `MAKECAB` | .cab アーカイブを作成する |
| `TAR` | tar アーカイブを作成または抽出する |

### テキストの検索と処理

|コマンド |目的 |
|----------|----------|
| `FIND` |リテラル文字列を検索する |
| `FINDSTR` |限定された正規表現で検索する |
| `SORT` |行をアルファベット順に並べ替える |
| `CLIP` |パイプされた入力をクリップボードにコピー |
| `FC` | 2 つのファイルを比較する |
| `COMP` |バイナリ ファイルの比較 |
| `CERTUTIL` | Base64 のエンコード/デコード、ハッシュの計算 |

### システム情報

|コマンド |目的 |
|----------|----------|
| `SYSTEMINFO` |完全なシステム構成 |
| `HOSTNAME` |コンピュータ名を表示 |
| `VER` | Windows版 |
| `WHOAMI` |現在のユーザーとグループの情報 |
| `TASKLIST` |実行中のプロセスをリストする |
| `TASKKILL` |プロセスを終了する |
| `WMIC` | WMI クエリ (ドライブ、OS、メモリ) |
| `SC` |サービス制御（クエリ、開始、停止） |
| `DRIVERQUERY` |インストールされているドライバーを一覧表示する |
| `REG` |レジストリ操作 (クエリ、追加、削除) |
| `SETX` |永続的な環境変数を設定する |

### ネットワーク

|コマンド |目的 |
|----------|----------|
| `PING` |ネットワーク接続をテストする |
| `IPCONFIG` | IP 構成 |
| `NSLOOKUP` | DNS ルックアップ |
| `NETSTAT` |ネットワーク接続とポート |
| `TRACERT` |ホストへのルートをトレース |
| `NET USE` |ネットワークドライブのマッピング/切断 |
| `NET USER` |ユーザーアカウントを管理する |
| `NETSH` |ネットワーク設定ユーティリティ |
| `ARP` | ARP キャッシュ管理 |
| `ROUTE` |ルーティングテーブル管理 |
| `CURL` | HTTP リクエスト (Windows 10 以降) |
| `SSH` |セキュア シェル (Windows 10 以降) |

### スケジューリングと自動化

|コマンド |目的 |
|----------|----------|
| `SCHTASKS` |スケジュールされたタスクの作成と管理 |
| `TIMEOUT` | N 秒待ちます (Vista+) |
| `START` |プログラムを非同期的に起動する |
| `RUNAS` |別のユーザーとして実行 |
| `SHUTDOWN` |シャットダウンまたは再起動 |
| `FORFILES` |日付でファイルを検索し、コマンドを実行します。

### シェルユーティリティ

|コマンド |目的 |
|----------|----------|
| `WHERE` | PATH | で実行可能ファイルを見つけます。
| `DOSKEY` |コマンドマクロを作成する |
| `CHOICE` |単一キー入力を求めるプロンプト |
| `MODE` |コンソールのサイズとポートを構成する |
| `SUBST` |フォルダーをドライブ文字にマップする |
| `CHCP` |コンソール コード ページを取得または設定する |
| `COLOR` |コンソールの色を設定する |
| `TITLE` |コンソール ウィンドウのタイトルを設定する |
| `ASSOC` / `FTYPE` |ファイルタイプの関連付け |

## シェルの構文と式

### グループ化のためのかっこ

かっこは、複合コマンドをリダイレクトまたは条件付き実行の単一ユニットに変換します。
```bat
(echo Line 1 & echo Line 2) > output.txt
if exist "data.csv" (
    echo Processing...
    call :process "data.csv"
) else (
    echo No data found.
)
```

### エスケープ文字

キャレット `^` は次の文字をエスケープします。
```bat
echo Total ^& Summary          & REM Outputs: Total & Summary
echo 100%% complete            & REM Outputs: 100% complete (in batch)
echo Line one^
Line two                       & REM Caret escapes the newline
```

パイプの後にはトリプルキャレットが必要です: `echo x ^^^& y | findstr x`

### ワイルドカード

- `*` は任意の文字シーケンスに一致します
- `?` は単一の文字 (またはピリオドのないセグメントの末尾のゼロ) に一致します
```bat
dir *.txt           & REM All .txt files
ren *.jpeg *.jpg    & REM Bulk rename
```

### リダイレクトの概要
```bat
command > file.txt          & REM Overwrite stdout to file
command >> file.txt         & REM Append stdout to file
command 2> errors.log       & REM Redirect stderr
command > all.log 2>&1      & REM Merge stderr into stdout
command < input.txt         & REM Read stdin from file
command > NUL 2>&1          & REM Discard all output
```

## 本番品質のバッチ ファイルの作成

### 標準スクリプト構造
```bat
@echo off
setlocal EnableDelayedExpansion

REM ============================================================
REM  Script: example.bat
REM  Purpose: Describe what this script does
REM ============================================================

call :main %*
exit /b %ERRORLEVEL%

:main
    call :parse_args %*
    if not defined _TARGET (
        echo ERROR: --target is required. 1>&2
        call :usage
        exit /b 1
    )
    echo Processing: %_TARGET%
    exit /b 0

:parse_args
    if "%~1"=="" exit /b 0
    if /i "%~1"=="--target" set "_TARGET=%~2" & shift
    if /i "%~1"=="--help"   call :usage & exit /b 0
    shift
    goto :parse_args

:usage
    echo Usage: %~nx0 --target ^<path^> [--help]
    echo.
    echo Options:
    echo   --target   Path to process (required)
    echo   --help     Show this help message
    exit /b 0
```

### ベストプラクティス

1. **常に `@echo off` および `setlocal`** で開始する — ノイズの多い出力と呼び出し元への変数の漏洩を防ぎます。
2. **処理前に入力を検証** — 必要な引数とファイルの存在を早期にチェックします。 `if not defined` と `if not exist` を使用します。
3. **パスと変数の引用** — `"%~1"` および `"%_MY_PATH%"` を使用して、スペースと特殊文字を安全に処理します。
4. **`exit` の代わりに `exit /b` を使用します** — 親コンソール ウィンドウを閉じることを回避します。
5. **意味のある終了コードを返す** — 成功の場合は `exit /b 0`、特定の失敗の場合は 0 以外。
6. **スクリプトの相対パスには `%~dp0` を使用します** — 呼び出し元の作業ディレクトリに関係なくスクリプトが確実に動作するようにします。
7. **`XCOPY`** よりも `ROBOCOPY` を優先 — より信頼性が高く、再試行、ミラーリング、ロギングをサポートします。
8. **ループまたは括弧で囲まれたブロック内の変数を変更する場合は、`EnableDelayedExpansion` を使用します。**
9. **エラーを標準エラー出力に書き込む** — `echo ERROR: message 1>&2` は、パイピング用に標準出力をクリーンに保ちます。
10. **コメントには `REM` を使用してください** — `::` は、`FOR` ループ本体内で問題を引き起こす可能性があります。

### セキュリティに関する考慮事項

- **認証情報をバッチ ファイルに保存しないでください** — 環境変数、認証情報ストア、またはプロンプトを使用します。
- **ユーザー入力の検証** — `&`、`|`、または `>` を含む引用符で囲まれていない変数はコマンドを挿入できます。常に引用符で囲みます: `"%_USER_INPUT%"`。
- **`SETLOCAL`** を使用する — 変数値が親プロセスに漏洩するのを防ぎます。
- **ファイル パスのサニタイズ** — 意図しない削除を防ぐために、`DEL`、`RD`、または `ROBOCOPY` に渡す前にパスを検証します。
- **機密性の高い入力には `SET /P` を使用しないでください** — 入力は表示され、コンソールの履歴に保存されます。可能な場合は、専用の認証情報ツールを使用してください。

## デバッグとトラブルシューティング

|テクニック |どのように |
|----------|-----|
|トレース実行 | `@echo off` を削除するか、`@echo on` を一時的に使用してください。
|ステップスルー |セクション間に `PAUSE` を追加します。
|エラーレベルを確認する |各コマンドの後に `echo Exit code: %ERRORLEVEL%` |
|変数を検査する | `set _MY_` `_MY_` で始まるすべての変数をリストします。
|拡張の遅れに関する問題 | `( )` ブロック内の変数が更新されませんか? `!VAR!` 構文を有効にする |
| FOR ループ `%%` と `%` |バッチ ファイルでは `%%i` を使用し、コマンド ラインでは `%i` を使用します。
| SET 内のスペース | `set name = value` ではなく `set name=value` |
|パイプ内のキャレット | 写真 パイプ内のキャレットパイプの後に `^^^` を使用して特殊文字をエスケープします。
| SET /A | のかっこ`if` ブロック内で `^(` と `^)` を使用してエスケープするか、引用符を使用します |
|モジュロの場合は 2 パーセント |バッチ ファイル内の `set /a r=14 %% 3` |

## クロスプラットフォームおよび拡張ツール

バッチ スクリプトが限界に達すると、次のツールが cmd.exe の機能を拡張します。

|ツール |目的 |
|-----|----------|
| **Cygwin** | Windows 上の完全な POSIX 環境 (grep、sed、awk、ssh) |
| **MSYS2** |軽量の Unix ツールとパッケージ マネージャー (pacman) |
| **WSL** | Linux 用 Windows サブシステム — ネイティブ Linux バイナリを実行する |
| **GnuWin32** |ネイティブ Windows 実行可能ファイルとしての個々の GNU ユーティリティ |
| **PowerShell** | .NET 統合による最新の Windows スクリプト |

高速スタートアップ、単純なファイル操作、PATH ベースの CLI ツール、またはタスク スケジューラの統合が必要な場合は、バッチを使用します。複雑なデータ処理、REST API、またはオブジェクト指向スクリプトについては、PowerShell または WSL を検討してください。

## CMD キーボード ショートカット

|ショートカット |アクション |
|----------|----------|
| `Tab` |ファイル/フォルダー名のオートコンプリート |
| `Up` / `Down` |コマンド履歴をナビゲートする |
| `F7` |コマンド履歴ポップアップを表示 |
| `F3` |最後のコマンドを繰り返します |
| `Esc` |現在の行をクリア |
| `Ctrl+C` |実行中のコマンドをキャンセル |
| `Alt+F7` |コマンド履歴をクリア |

## 参照ファイル

`references/` フォルダーには、詳細なドキュメントが含まれています。

|ファイル |目次 |
|------|----------|
| `tools-and-resources.md` | Windows ツール、ユーティリティ、パッケージ マネージャー、ターミナル |
| `batch-files-and-functions.md` |サンプル スクリプト、テクニック、ベスト プラクティスのリンク |
| `windows-commands.md` |包括的な A-Z Windows コマンド リファレンス |
| `cygwin.md` | Cygwin ユーザーガイドと FAQ |
| `msys2.md` | MSYS2 のインストール、パッケージ、環境 |
| `windows-subsystem-on-linux.md` | WSL のセットアップ、コマンド、およびドキュメント |

## アセットテンプレート

`assets/` フォルダーには、スターター バッチ ファイル テンプレート データがテキスト ファイルとして含まれています。

|テンプレート |目的 |
|----------|----------|
| `executable.txt` |引数解析機能を備えたスタンドアロン CLI ツール |
| `library.txt` | CALL 可能なラベルを持つ再利用可能な関数ライブラリ |
| `task.txt` |スケジュールされたタスク/自動化スクリプト |