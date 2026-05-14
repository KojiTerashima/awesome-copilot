---
description: "GNU Make Makefile を記述するためのベストプラクティス"
applyTo: "**/Makefile, **/makefile, **/*.mk, **/GNUmakefile"
---

# Makefile 開発指示

[GNU Make manual](https://www.gnu.org/software/make/manual/) に基づき、クリーンで保守しやすく、移植性のある GNU Make Makefile を書くための指示です。

## 一般原則

- GNU Make の規約に従った、明確で保守しやすい makefile を書く
- target 名は目的が明確に分かる説明的なものにする
- default goal（最初の target）は、最も一般的な build 操作にする
- rule や recipe では、簡潔さよりも可読性を優先する
- 複雑な rule、variable、分かりづらい挙動には comment を追加する

## 命名規約

- makefile 名は `Makefile`（見つけやすいため推奨）または `makefile` にする
- 他の make 実装と互換性のない GNU Make 固有機能が必要な場合にのみ `GNUmakefile` を使う
- object file list には標準的な variable 名 `objects`, `OBJECTS`, `objs`, `OBJS`, `obj`, `OBJ` を使う
- 組み込み variable 名には大文字を使う（例: `CC`, `CFLAGS`, `LDFLAGS`）
- target 名は動作を反映した説明的なものにする（例: `clean`, `install`, `test`）

## ファイル構造

- makefile の最初の rule に default goal（主要 build target）を置く
- 関連する target は論理的にまとめる
- variable は rule より前、makefile の先頭に定義する
- file を表さない target は `.PHONY` で宣言する
- makefile は、variable、rule、phony target の順に構成する

```makefile
# Variables
CC = gcc
CFLAGS = -Wall -g
objects = main.o utils.o

# Default goal
all: program

# Rules
program: $(objects)
	$(CC) -o program $(objects)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Phony targets
.PHONY: clean all
clean:
	rm -f program $(objects)
```

## Variable と置換

- 重複を避け保守性を高めるために variable を使う
- 即時評価には `:=`（simple expansion）、再帰評価には `=` を使う
- 上書き可能な default 値には `?=` を使う
- 既存 variable への追加には `+=` を使う
- variable 参照は `$VARIABLE` ではなく `$(VARIABLE)` を使う（1 文字変数を除く）
- rule を汎用化するため、recipe では automatic variable（`$@`, `$<`, `$^`, `$?`, `$*`）を使う

```makefile
# Simple expansion (evaluates immediately)
CC := gcc

# Recursive expansion (evaluates when used)
CFLAGS = -Wall $(EXTRA_FLAGS)

# Conditional assignment
PREFIX ?= /usr/local

# Append to variable
CFLAGS += -g
```

## Rule と prerequisite

- target、prerequisite、recipe を明確に分ける
- 標準的なコンパイル（例: `.c` から `.o`）には implicit rule を使う
- prerequisite は論理的な順序で並べる（通常 prerequisite を先、order-only prerequisite を後）
- directory や、timestamp 変更で再 build を起こしたくない依存には order-only prerequisite（`|` の後）を使う
- 正しい再 build を保証するため、実際の依存関係はすべて含める
- target 間の循環依存は避ける
- order-only prerequisite は `$^` のような automatic variable には含まれないため、必要なら明示的に参照する

以下の例では、`obj/` directory に object をコンパイルする pattern rule を示しています。directory 自体は order-only prerequisite として扱うため、コンパイル前に作成されますが、timestamp 変更だけで再コンパイルは発生しません。

```makefile
# Normal prerequisites
program: main.o utils.o
	$(CC) -o $@ $^

# Order-only prerequisites (directory creation)
obj/%.o: %.c | obj
	$(CC) $(CFLAGS) -c $< -o $@

obj:
	mkdir -p obj
```

## Recipe と command

- `.RECIPEPREFIX` を変更していない限り、recipe の各行は必ず **タブ文字** で始める
- 必要に応じて `@` プレフィックスで command の echo を抑制する
- 特定 command の error を無視したいときのみ `-` プレフィックスを使う（多用しない）
- まとめて実行すべき command は同じ行で `&&` や `;` でつなぐ
- recipe は読みやすく保ち、長い command はバックスラッシュ継続で複数行に分ける
- 必要なら recipe 内で shell の conditional や loop を使う

```makefile
# Silent command
clean:
	@echo "Cleaning up..."
	@rm -f $(objects)

# Ignore errors
.PHONY: clean-all
clean-all:
	-rm -rf build/
	-rm -rf dist/

# Multi-line recipe with proper continuation
install: program
	install -d $(PREFIX)/bin && \
		install -m 755 program $(PREFIX)/bin
```

## Phony Target

- 同名 file との衝突を避けるため、phony target は必ず `.PHONY` で宣言する
- `clean`, `install`, `test`, `all` などの action には phony target を使う
- phony target の宣言は、その rule の近くか makefile の末尾に置く

```makefile
.PHONY: all clean test install

all: program

clean:
	rm -f program $(objects)

test: program
	./run-tests.sh

install: program
	install -m 755 program $(PREFIX)/bin
```

## Pattern Rule と Implicit Rule

- 一般的な変換には pattern rule（`%.o: %.c`）を使う
- 適切な場合は組み込み implicit rule を活用する（GNU Make は `.c` から `.o` のコンパイル方法を知っている）
- rule を書き直すより、`CC`, `CFLAGS` などの implicit rule variable を上書きする
- 組み込み rule で足りない場合にだけ custom pattern rule を定義する

```makefile
# Use built-in implicit rules by setting variables
CC = gcc
CFLAGS = -Wall -O2

# Custom pattern rule for special cases
%.pdf: %.md
	pandoc $< -o $@
```

## 長い行の分割

- 可読性のために、backslash-newline（`\`）で長い行を分割する
- recipe 以外では backslash-newline が単一スペースに変換されることに注意する
- recipe 内では backslash-newline により shell に対する継続行が保たれる
- バックスラッシュの後ろに余計な空白を置かない

### 空白を追加せずに分割する

空白を追加せずに行を分割したい場合は、特殊な方法が使えます。`$ `（dollar-space）の後に backslash-newline を置きます。`$ ` はスペース 1 文字の名前を持つ variable を参照しますが、その variable は存在しないため空文字に展開され、結果的に空白を挿入せず行を連結できます。

```makefile
# Concatenate strings without adding whitespace
# The following creates the value "oneword"
var := one$ \
       word

# This is equivalent to:
# var := oneword
```

```makefile
# Variable definition split across lines
sources = main.c \
          utils.c \
          parser.c \
          handler.c

# Recipe with long command
build: $(objects)
	$(CC) -o program $(objects) \
	      $(LDFLAGS) \
	      -lm -lpthread
```

## 他の Makefile の読み込み

- 共通定義を共有するには `include` directive を使う
- error なしで optional makefile を取り込むには `-include`（または `sinclude`）を使う
- include 先に影響する variable 定義の後で `include` directive を置く
- 共有 variable、pattern rule、共通 target の取り込みに `include` を使う

```makefile
# Include common settings
include config.mk

# Include optional local configuration
-include local.mk
```

## Conditional Directive

- platform ごとや configuration ごとの rule には conditional directive（`ifeq`, `ifneq`, `ifdef`, `ifndef`）を使う
- conditional は recipe の中ではなく makefile レベルに置く（recipe 内では shell の conditional を使う）
- conditional はシンプルかつ十分に文書化する

```makefile
# Platform-specific settings
ifeq ($(OS),Windows_NT)
    EXE_EXT = .exe
else
    EXE_EXT =
endif

program: main.o
	$(CC) -o program$(EXE_EXT) main.o
```

## Automatic Prerequisite

- header dependency は手動で管理せず、自動生成する
- `.d` file を生成するために `-MMD` や `-MP` のような compiler flag を使う
- まだ存在しない場合でも error にしないよう、`-include $(deps)` で dependency file を取り込む

```makefile
objects = main.o utils.o
deps = $(objects:.o=.d)

# Include dependency files
-include $(deps)

# Compile with automatic dependency generation
%.o: %.c
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@
```

## Error Handling と Debugging

- build 時の診断には `$(error text)` または `$(warning text)` function を使う
- command を実行せず確認するには `make -n`（dry run）で makefile をテストする
- rule と variable のデータベースを確認するには `make -p` を使う
- 必須 variable と tool は makefile 冒頭で検証する

```makefile
# Check for required tools
ifeq ($(shell which gcc),)
    $(error "gcc is not installed or not in PATH")
endif

# Validate required variables
ifndef VERSION
    $(error VERSION is not defined)
endif
```

## Clean Target

- 生成 file を削除する `clean` target を必ず用意する
- `clean` は phony として宣言し、同名 file との衝突を避ける
- file がなくても error にしないよう、`rm` command には `-` プレフィックスを使う
- object だけを消す `clean` と、すべての生成物を消す `distclean` を分けることも検討する

```makefile
.PHONY: clean distclean

clean:
	-rm -f $(objects)
	-rm -f $(deps)

distclean: clean
	-rm -f program config.mk
```

## 移植性の考慮

- 他の make 実装への移植が必要なら、GNU Make 固有機能は避ける
- shell command は標準的なものを使う（POSIX shell 構文を優先）
- すべての target を強制再 build する `make -B` でもテストする
- platform 固有要件や GNU Make 拡張は文書化する

## パフォーマンス最適化

- 再帰評価が不要な variable には `:=` を使う（高速）
- subprocess を起こす `$(shell ...)` の不要な使用を避ける
- prerequisite は効率よく並べる（最も頻繁に変わる file を後ろに置く）
- target 間が競合しないようにして parallel build（`make -j`）を安全に使う

## ドキュメントと comment

- makefile の目的を説明する header comment を追加する
- 分かりにくい variable 設定とその影響を文書化する
- usage example や target を comment に含める
- 複雑な rule や platform 固有の回避策には inline comment を追加する

```makefile
# Makefile for building the example application
#
# Usage:
#   make          - Build the program
#   make clean    - Remove generated files
#   make install  - Install to $(PREFIX)
#
# Variables:
#   CC       - C compiler (default: gcc)
#   PREFIX   - Installation prefix (default: /usr/local)

# Compiler and flags
CC ?= gcc
CFLAGS = -Wall -Wextra -O2

# Installation directory
PREFIX ?= /usr/local
```

## Special Target

- 非 file target には `.PHONY` を使う
- 中間 file を保持するには `.PRECIOUS` を使う
- 中間 file として自動削除したいものは `.INTERMEDIATE` で指定する
- 中間 file を削除したくない場合は `.SECONDARY` を使う
- recipe が失敗したときに target を削除するには `.DELETE_ON_ERROR` を使う
- すべての recipe の echo を抑制する `.SILENT` は、必要最小限で使う

```makefile
# Don't delete intermediate files
.SECONDARY:

# Delete targets if recipe fails
.DELETE_ON_ERROR:

# Preserve specific files
.PRECIOUS: %.o
```

## よくあるパターン

### 標準的な project structure

```makefile
CC = gcc
CFLAGS = -Wall -O2
objects = main.o utils.o parser.o

.PHONY: all clean install

all: program

program: $(objects)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	-rm -f program $(objects)

install: program
	install -d $(PREFIX)/bin
	install -m 755 program $(PREFIX)/bin
```

### 複数 program の管理

```makefile
programs = prog1 prog2 prog3

.PHONY: all clean

all: $(programs)

prog1: prog1.o common.o
	$(CC) -o $@ $^

prog2: prog2.o common.o
	$(CC) -o $@ $^

prog3: prog3.o
	$(CC) -o $@ $^

clean:
	-rm -f $(programs) *.o
```

## 避けるべきアンチパターン

- recipe 行をタブではなくスペースで始めない
- wildcard や function で生成できる file list を hardcode しない
- file list を得るために `$(shell ls ...)` を使わない（代わりに `$(wildcard ...)` を使う）
- recipe に複雑な shell script を書き込まない（別 script file に切り出す）
- phony target の `.PHONY` 宣言を忘れない
- target 間の循環依存を避ける
- どうしても必要な場合を除き recursive make（`$(MAKE) -C subdir`）を使わない
