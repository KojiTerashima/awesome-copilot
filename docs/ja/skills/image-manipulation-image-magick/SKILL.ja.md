---
name: image-manipulation-image-magick
description: ImageMagickを使用して画像を処理および操作します。リサイズ、フォーマット変換、バッチ処理、画像メタデータの取得をサポートします。画像の操作、サムネイル作成、壁紙のリサイズ、バッチ画像処理を行う際に使用してください。
compatibility: ImageMagickがインストールされ、PATH上で`magick`として利用可能であることが必要です。PowerShell（Windows）およびBash（Linux/macOS）向けのクロスプラットフォーム例を提供しています。
---

# ImageMagickによる画像操作

このスキルは、Windows、Linux、macOSシステムでImageMagickを使用した画像処理および操作タスクを可能にします。

## このスキルを使うタイミング

以下のような場合にこのスキルを使用してください：

- 画像のリサイズ（単一またはバッチ）
- 画像の寸法やメタデータの取得
- 画像フォーマットの変換
- サムネイルの作成
- さまざまな画面サイズ向けの壁紙処理
- 特定条件に基づく複数画像のバッチ処理

## 前提条件

- システムにImageMagickがインストールされていること
- **Windows**: PowerShellで`magick`としてImageMagickが利用可能（または`C:\Program Files\ImageMagick-*\magick.exe`に存在）
- **Linux/macOS**: パッケージマネージャー（`apt`、`brew`など）でインストールされたBash環境

## 主な機能

### 1. 画像情報の取得

- 画像の寸法（幅×高さ）を取得
- 詳細なメタデータ（フォーマット、カラースペースなど）を取得
- 画像フォーマットの識別

### 2. 画像のリサイズ

- 単一画像のリサイズ
- 複数画像のバッチリサイズ
- 指定寸法のサムネイル作成
- アスペクト比の維持

### 3. バッチ処理

- 寸法に基づく画像処理
- 特定ファイルタイプのフィルタリングと処理
- 複数ファイルへの変換適用

## 使用例

### 例0: `magick`実行ファイルの解決

**PowerShell（Windows）:**
```powershell
# PATH上のImageMagickを優先
$magick = (Get-Command magick -ErrorAction SilentlyContinue)?.Source

# フォールバック: Program Files下の一般的なインストールパターン
if (-not $magick) {
    $magick = Get-ChildItem "C:\\Program Files\\ImageMagick-*\\magick.exe" -ErrorAction SilentlyContinue |
        Select-Object -First 1 -ExpandProperty FullName
}

if (-not $magick) {
    throw "ImageMagickが見つかりません。インストールするか、'magick'をPATHに追加してください。"
}
```

**Bash（Linux/macOS）:**
```bash
# PATH上にmagickがあるか確認
if ! command -v magick &> /dev/null; then
    echo "ImageMagickが見つかりません。パッケージマネージャーでインストールしてください:"
    echo "  Ubuntu/Debian: sudo apt install imagemagick"
    echo "  macOS: brew install imagemagick"
    exit 1
fi
```

### 例1: 画像の寸法を取得

**PowerShell（Windows）:**
```powershell
# 単一画像の場合
& $magick identify -format "%wx%h" path/to/image.jpg

# 複数画像の場合
Get-ChildItem "path/to/images/*" | ForEach-Object { 
    $dimensions = & $magick identify -format "%f: %wx%h`n" $_.FullName
    Write-Host $dimensions 
}
```

**Bash（Linux/macOS）:**
```bash
# 単一画像の場合
magick identify -format "%wx%h" path/to/image.jpg

# 複数画像の場合
for img in path/to/images/*; do
    magick identify -format "%f: %wx%h\n" "$img"
done
```

### 例2: 画像のリサイズ

**PowerShell（Windows）:**
```powershell
# 単一画像のリサイズ
& $magick input.jpg -resize 427x240 output.jpg

# 複数画像のバッチリサイズ
Get-ChildItem "path/to/images/*" | ForEach-Object { 
    & $magick $_.FullName -resize 427x240 "path/to/output/thumb_$($_.Name)"
}
```

**Bash（Linux/macOS）:**
```bash
# 単一画像のリサイズ
magick input.jpg -resize 427x240 output.jpg

# 複数画像のバッチリサイズ
for img in path/to/images/*; do
    filename=$(basename "$img")
    magick "$img" -resize 427x240 "path/to/output/thumb_$filename"
done
```

### 例3: 詳細な画像情報を取得

**PowerShell（Windows）:**
```powershell
# 画像の詳細情報を取得
& $magick identify -verbose path/to/image.jpg
```

**Bash（Linux/macOS）:**
```bash
# 画像の詳細情報を取得
magick identify -verbose path/to/image.jpg
```

### 例4: 寸法に基づく画像処理

**PowerShell（Windows）:**
```powershell
Get-ChildItem "path/to/images/*" | ForEach-Object { 
    $dimensions = & $magick identify -format "%w,%h" $_.FullName
    if ($dimensions) {
        $width,$height = $dimensions -split ','
        if ([int]$width -eq 2560 -or [int]$height -eq 1440) {
            Write-Host "Processing $($_.Name)"
            & $magick $_.FullName -resize 427x240 "path/to/output/thumb_$($_.Name)"
        }
    }
}
```

**Bash（Linux/macOS）:**
```bash
for img in path/to/images/*; do
    dimensions=$(magick identify -format "%w,%h" "$img")
    if [[ -n "$dimensions" ]]; then
        width=$(echo "$dimensions" | cut -d',' -f1)
        height=$(echo "$dimensions" | cut -d',' -f2)
        if [[ "$width" -eq 2560 || "$height" -eq 1440 ]]; then
            filename=$(basename "$img")
            echo "Processing $filename"
            magick "$img" -resize 427x240 "path/to/output/thumb_$filename"
        fi
    fi
done
```

## ガイドライン

1. **ファイルパスは常に引用符で囲む** - スペースを含む可能性のあるパスは必ず引用符で囲んでください
2. **PowerShellでは`&`演算子を使用** - PowerShellで`magick`実行ファイルを呼び出す際は`&`を使います
3. **パスは変数に格納（PowerShell）** - ImageMagickのパスは`$magick`に格納してコードをすっきりさせます
4. **ループで処理を包む** - 複数ファイル処理時は`ForEach-Object`（PowerShell）や`for`ループ（Bash）を使います
5. **処理前に寸法を確認** - 不要な処理を避けるため、画像の寸法を先にチェックします
6. **適切なリサイズフラグを使う** - 正確な寸法を強制する場合は`!`、最小寸法を指定する場合は`^`を検討してください

## よく使うパターン

### PowerShellパターン

#### パターン: ImageMagickパスの取得

```powershell
$magick = (Get-Command magick).Source
```

#### パターン: 寸法を変数に取得

```powershell
$dimensions = & $magick identify -format "%w,%h" $_.FullName
$width,$height = $dimensions -split ','
```

#### パターン: 条件付き処理

```powershell
if ([int]$width -gt 1920) {
    & $magick $_.FullName -resize 1920x1080 $outputPath
}
```

#### パターン: サムネイル作成

```powershell
& $magick $_.FullName -resize 427x240 "thumbnails/thumb_$($_.Name)"
```

### Bashパターン

#### パターン: ImageMagickインストール確認

```bash
command -v magick &> /dev/null || { echo "ImageMagickが必要です"; exit 1; }
```

#### パターン: 寸法を変数に取得

```bash
dimensions=$(magick identify -format "%w,%h" "$img")
width=$(echo "$dimensions" | cut -d',' -f1)
height=$(echo "$dimensions" | cut -d',' -f2)
```

#### パターン: 条件付き処理

```bash
if [[ "$width" -gt 1920 ]]; then
    magick "$img" -resize 1920x1080 "$outputPath"
fi
```

#### パターン: サムネイル作成

```bash
filename=$(basename "$img")
magick "$img" -resize 427x240 "thumbnails/thumb_$filename"
```

## 制限事項

- 大規模なバッチ処理はメモリを多く消費する可能性があります
- 複雑な操作には追加のImageMagickデリゲートが必要な場合があります
- 古いLinuxシステムでは`magick`の代わりに`convert`を使用してください（ImageMagick 6.xと7.xの違い）
