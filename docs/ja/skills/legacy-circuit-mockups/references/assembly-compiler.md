# 6502 SBC アセンブリコンパイル＆ROMビルド仕様

## 概要

本書は、以下の単一基板コンピュータ向けの**6502アセンブリ言語の完全なコンパイル仕様**を定義します：

* **MOS 6502 CPU**
* **MOS 6522 VIA**
* **AS6C62256（32 KB SRAM）**
* **AT28C256（32 KB EEPROM / ROM）**
* **DFRobot FIT0127（HD44780互換16x2 LCD）**

焦点は**ツールチェーンの動作、メモリ配置、ROM構成、ファームウェア規約**にあり、電気配線は対象外です。

---

## 対象システムアーキテクチャ

### メモリマップ（標準）

```
$0000-$00FF  ゼロページ（RAM）
$0100-$01FF  スタック（RAM）
$0200-$7FFF  汎用RAM（AS6C62256）
$8000-$8FFF  6522 VIA I/O空間
$9000-$FFFF  ROM（AT28C256）
```

> アドレスデコードはデバイスのミラーリングがある場合がありますが、アセンブラはこの標準レイアウトを前提とします。

---

## ROM構成（AT28C256）

| アドレス     | 用途                 |
| ------------ | -------------------- |
| $9000-$FFEF  | プログラムコード＋データ |
| $FFF0-$FFF9  | オプションのシステムデータ |
| $FFFA-$FFFB  | NMIベクター          |
| $FFFC-$FFFD  | RESETベクター        |
| $FFFE-$FFFF  | IRQ/BRKベクター      |

ROMイメージサイズ：**32,768バイト**

---

## リセット＆スタートアップ規約

リセット時：

1. CPUは`$FFFC`のRESETベクターを取得
2. コードはスタックポインタを初期化
3. ゼロページ変数を初期化
4. VIAを設定
5. LCDを初期化
6. メインプログラムに入る

---

## アセンブラ要件

アセンブラは**必ず**以下をサポートすること：

* `.org`絶対アドレッシング
* シンボリックラベル
* バイナリ出力（`.bin`）
* リトルエンディアンのワード出力
* ゼロページ最適化

推奨アセンブラ：

* **ca65**（cc65ツールチェーン）
* **vasm6502**
* **64tass**

---

## アセンブリソース構造

```asm
;---------------------------
; リセットベクターエントリポイント
;---------------------------
        .org $9000
RESET:
        sei
        cld
        ldx #$FF
        txs
        jsr init_via
        jsr init_lcd
MAIN:
        jsr lcd_print
        jmp MAIN
```

---

## ベクターテーブル定義

```asm
        .org $FFFA
        .word nmi_handler
        .word RESET
        .word irq_handler
```

---

## 6522 VIA プログラミングモデル

### レジスタマップ（ベース = $8000）

| オフセット | レジスタ |
| --------- | -------- |
| $0        | ORB      |
| $1        | ORA      |
| $2        | DDRB     |
| $3        | DDRA     |
| $4        | T1CL     |
| $5        | T1CH     |
| $6        | T1LL     |
| $7        | T1LH     |
| $8        | T2CL     |
| $9        | T2CH     |
| $B        | ACR      |
| $C        | PCR      |
| $D        | IFR      |
| $E        | IER      |

---

## LCDインターフェース規約

### LCD配線想定

| LCD   | VIA     |
| ----- | ------- |
| D4-D7 | PB4-PB7 |
| RS    | PA0     |
| E     | PA1     |
| R/W   | GND     |

4ビットモードを想定。

---

## LCD初期化シーケンス

```asm
lcd_init:
        lda #$33
        jsr lcd_cmd
        lda #$32
        jsr lcd_cmd
        lda #$28
        jsr lcd_cmd
        lda #$0C
        jsr lcd_cmd
        lda #$06
        jsr lcd_cmd
        lda #$01
        jsr lcd_cmd
        rts
```

---

## LCDコマンド／データインターフェース

| 操作     | RS | データ           |
| -------- | -- | ---------------- |
| コマンド | 0  | 命令             |
| データ   | 1  | ASCII文字        |

---

## ゼロページ使用規約

| アドレス | 用途          |
| -------- | ------------- |
| $00-$0F  | スクラッチ領域 |
| $10-$1F  | LCDルーチン   |
| $20-$2F  | VIA状態       |
| $30-$FF  | ユーザー定義  |

---

## RAM使用（AS6C62256）

* スタックはページ`$01`を使用
* 全RAMは揮発性と想定
* ROMシャドウイングなし

---

## ビルドパイプライン

### ステップ1：アセンブル

```bash
ca65 main.asm -o main.o
```

### ステップ2：リンク

```bash
ld65 -C rom.cfg main.o -o rom.bin
```

### ステップ3：ROMパディング

`rom.bin`が正確に**32768バイト**であることを確認。

---

## EEPROMプログラミング

* 対象デバイス：**AT28C256**
* プログラマ：**MiniPro / T48**
* 書き込み後に検証

---

## エミュレータ要件

エミュレータは以下を満たすこと：

* `$9000-$FFFF`にROMをロード
* VIA I/Oの副作用をエミュレート
* LCD出力をレンダリング
* RESETベクターを尊重

---

## テストチェックリスト

* リセットベクターの実行
* VIAレジスタ書き込み
* LCDが正しいテキストを表示
* スタック操作が正しい
* ROMイメージのマッピングが正確

---

## 参考文献

* [MOS 6502 プログラミングマニュアル](http://archive.6502.org/datasheets/synertek_programming_manual.pdf)
* [MOS 6522 VIA データシート](http://archive.6502.org/datasheets/mos_6522_preliminary_nov_1977.pdf)
* [AT28C256 データシート](https://ww1.microchip.com/downloads/aemDocuments/documents/MPD/ProductDocuments/DataSheets/AT28C256-Industrial-Grade-256-Kbit-Paged-Parallel-EEPROM-Data-Sheet-DS20006386.pdf)
* [HD44780 LCD データシート](https://www.futurlec.com/LED/LCD16X2BLa.shtml)
* [cc65 ツールチェーンドキュメント](https://cc65.github.io/doc/cc65.html)

---

## 注記

本仕様は意図的に**エンドツーエンド**です：アセンブリソースからEEPROMイメージ、実機またはエミュレータでの動作までを対象とします。ROM、エミュレータ、実際のSBCが同一の動作をするための安定した契約を定義しています。
