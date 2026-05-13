# minipro チッププログラミングユーティリティ仕様

## 1. 概要

**minipro** は、**T48**、**TL866II Plus** などの対応する汎用プログラマを使用して、幅広い **EEPROM、フラッシュ、EPROM、SRAM、GAL、ロジックデバイス** の **プログラム、読み取り、消去、検証** を行うオープンソースのコマンドラインユーティリティです。

主に **Linux**、**macOS**、**Windows** 環境で利用されており、特に **レトロコンピューティング**、**ファームウェア開発**、**電子プロトタイピング** に広く使われています。

---

## 2. 対応プログラマ

| プログラマ       | 備考                         |
| ---------------- | ---------------------------- |
| T48              | 完全対応（推奨）             |
| TL866II Plus     | 完全対応                     |
| TL866A / CS      | 制限あり / レガシーサポート  |

---

## 3. 対応デバイスタイプ

### 3.1 メモリデバイス

* 並列EEPROM（例：AT28C256）
* フラッシュメモリ（29xxxシリーズ）
* EPROM（27xxxシリーズ）
* SRAM（読み取り／検証のみ）

### 3.2 ロジックおよびPLD

* GAL16V8 / GAL22V10
* PALデバイス（一部制限あり）

### 3.3 その他のデバイス

* 一部マイクロコントローラ（デバイス依存）
* ロジックICテスト（選択モデル）

---

## 4. インストール

### 4.1 Linux

```bash
sudo apt install minipro
```

またはソースから：

```bash
git clone https://github.com/vdudouyt/minipro.git
make
sudo make install
```

### 4.2 Windows

* MSYS2または事前ビルド済みバイナリからインストール
* libusbドライバ（WinUSB）が必要

---

## 5. 基本コマンド構文

```bash
minipro [options]
```

よく使うオプション：

| オプション       | 説明                     |
| ---------------- | ------------------------ |
| `-p <device>`    | 対象デバイスの選択       |
| `-r <file>`      | デバイスからファイルへ読み取り |
| `-w <file>`      | ファイルからデバイスへ書き込み |
| `-e`             | デバイスの消去           |
| `-v`             | 内容の検証               |
| `-I`             | デバイス情報の表示       |
| `-l`             | 対応デバイス一覧の表示   |

---

## 6. 一般的なプログラミング操作

### 6.1 対応デバイス一覧の表示

```bash
minipro -l
```

### 6.2 デバイスの識別

```bash
minipro -p AT28C256 -I
```

### 6.3 チップの読み取り

```bash
minipro -p AT28C256 -r rom_dump.bin
```

### 6.4 チップへの書き込み

```bash
minipro -p AT28C256 -w rom.bin
```

### 6.5 検証のみ

```bash
minipro -p AT28C256 -v rom.bin
```

---

## 7. EEPROMプログラミング（AT28C256例）

```bash
minipro -p AT28C256 -w monitor.bin
```

* ソフトウェアデータ保護は自動処理
* 書き込みサイクルの遅延は内部管理
* プログラミング後に検証を実施

---

## 8. フラッシュメモリのプログラミング

```bash
minipro -p SST39SF040 -e -w firmware.bin
```

* フラッシュデバイスは消去ステップが必要
* セクター消去は自動処理

---

## 9. EPROM操作

```bash
minipro -p 27C256 -r eprom.bin
```

* 再プログラム前にUV消去が必要
* miniproは書き込み前に空白状態を検証

---

## 10. GALプログラミング

```bash
minipro -p GAL22V10 -w logic.jed
```

* JEDECファイルを使用
* 読み取り、書き込み、検証をサポート
* `-I` でフューズマップを表示可能

---

## 11. エラー処理とメッセージ

| メッセージ             | 意味                         |
| ---------------------- | ---------------------------- |
| `Device not found`     | デバイス選択が誤っている     |
| `Verification failed`  | データ不一致                 |
| `Chip protected`       | 書き込み保護が有効           |
| `Overcurrent detected` | 挿入ミスまたは配線エラー     |

---

## 12. 安全上の注意とベストプラクティス

* ZIFソケット内のデバイス向きを必ず確認する
* 正しいデバイス識別子（`-p`）を使用する
* 動作中のホットインサートは避ける
* PLCC、SOP、TSOPパッケージにはアダプタを使用する

---

## 13. 典型的なレトロコンピューティングのワークフロー

1. ROMイメージを組み立てる
2. minipro + T48でEEPROMをプログラムする
3. 内容を検証する
4. SBCにチップを取り付ける
5. システムの起動をテストする

---

## 14. 制限事項

* すべてのデバイスに対応しているわけではない
* 一部マイクロコントローラは専用ツールが必要
* インサーキットプログラミング（ISP）は非対応

---

## 15. 参考文献

* <https://gitlab.com/DavidGriffith/minipro>
* <https://www.hadex.cz/spec/m545b.pdf>
* <https://github.com/mikeroyal/Firmware-Guide>
* <https://mike42.me/blog/2021-08-a-first-look-at-programmable-logic>
* <https://retrocomputingforum.com/>

---
