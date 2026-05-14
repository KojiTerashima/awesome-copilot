---
name: eval-driven-dev
description: >
  Python LLM アプリケーション向けに eval ベースの QA を構築します。アプリを instrument し、golden dataset を作り、eval test を書いて実行し、失敗をもとに改善を繰り返します。ユーザーが QA のセットアップ、test の追加、eval の追加、評価、benchmark、誤動作の修正、品質改善、または LLM モデルを呼び出す任意の Python project の品質保証を求めた場合は、常にこのスキルを使用してください。
license: MIT
compatibility: Python 3.11+
metadata:
  version: 0.6.1
  pixie-qa-version: ">=0.6.1,<0.7.0"
  pixie-qa-source: https://github.com/yiouli/pixie-qa/
---

# Eval-Driven Development for Python LLM Applications

あなたは、Python アプリケーションを end-to-end でテストする **自動 QA パイプライン** を構築しています。実際のユーザーと同じ方法で、実入力を使ってアプリを走らせ、その出力を evaluator で採点し、`pixie test` によって pass/fail 結果を出します。

**テスト対象はアプリそのものです**。つまり、リクエスト処理、context の組み立て (データ収集、prompt 構築、会話状態管理)、routing、応答整形です。アプリは LLM を使うため出力が非決定的であり、そのため `assertEqual` ではなく evaluator (LLM-as-judge、類似度スコアなど) を使います。しかし、テストしているのは LLM ではなくアプリのコードです。

評価中には、アプリ自身のコードが実際に動きます。routing、prompt 組み立て、LLM 呼び出し、応答整形はすべて本物で、mock や stub は使いません。ただし、アプリが外部ソース (database、cache、third-party API、voice stream) から読むデータは、instrumentation によってテスト指定の値へ置き換えます。これにより、各テスト ケースはアプリに見せるデータを完全に制御しつつ、アプリ全体のコード経路を実行できます。

**成果物は、実スコアを伴って動作する `pixie test` 実行結果です**。計画書でもなく、instrumentation だけでもなく、dataset だけでもありません。

この skill は、説明することではなく作業を実行することに関するものです。コードを読み、ファイルを編集し、コマンドを実行し、動くパイプラインを作ってください。

---

## Before you start

**最初に virtual environment を有効化してください**。プロジェクトに正しい virtual environment を特定して有効化します。virtual environment が有効になったら、この skill の resources に含まれる setup.sh を実行してください。
この script は `eval-driven-dev` skill と `pixie-qa` Python package を最新へ更新し、まだ初期化されていなければ pixie working directory を初期化し、ユーザーに更新状況を表示するための Web server を background で起動します。skill または package の更新に失敗しても継続してください。これらの失敗で残りの workflow を止めてはいけません。

---

## The workflow

Step 1〜6 を途中で止まらず順に実行してください。途中ステップごとの確認をユーザーに求めてはいけません。各ステップは自分で検証し、そのまま次へ進みます。

**作業方法 — 何より先にこれを読んでください:**

- **1 ステップずつ進める。** 今のステップの instructions だけを読むこと。Step 1 の作業中に Step 2〜6 を読んではいけません。
- **reference は、そのステップで指示されたときだけ読む。** 各ステップは specific な reference file を示します。そのステップに到達したときに読むのであって、それ以前に読んではいけません。
- **artifact はすぐ作る。** サブステップのためにコードを読んだら、そのサブステップの output file を書いてから次へ進むこと。複数のサブステップをまたいで理解を溜め込んでからまとめて書いてはいけません。
- **検証してから次へ。** 各ステップには checkpoint があります。検証してから次へ進みます。現在のステップを検証している間に未来のステップを計画してはいけません。

**Step 1〜6 を順番に実行すること。** ユーザーの prompt から、前段ステップがすでに終わっていることが明らかな場合 (例: "run the existing tests"、"re-run evals") は、適切なステップに飛んでかまいません。迷うなら Step 1 から始めてください。

---

### Step 1: Understand the app and define eval criteria

**まず、ユーザーの prompt に specific な要件があるか確認してください。** アプリ コードを読む前に、ユーザーが何を求めているかを確認します。

- **Referenced documents or specs**: prompt に従うべきファイルが示されているか? (例: "follow the spec in EVAL_SPEC.md"、"use the methodology in REQUIREMENTS.md") 示されているなら **そのファイルを先に読む** こと。それが dataset、evaluation dimension、pass criteria、methodology を既定値より優先して定めている可能性があります。
- **Specified datasets or data sources**: prompt に specific な data file が挙げられているか? (例: "use questions from eval_inputs/research_questions.json"、"use the scenarios in call_scenarios.json") 示されているなら **それらのファイルを読む** こと。generic な代替を作るのではなく、必ずそれらを eval dataset の土台に使ってください。
- **Specified evaluation dimensions**: 評価すべき quality aspect が明示されているか? (例: "evaluate on factuality, completeness, and bias"、"test identity verification and tool call correctness") 示された各 dimension に対して、**test file に対応する evaluator が必ず存在しなければなりません**。

これらが prompt で指定されている場合は、それが最優先です。先に読み込み、取り込んでから進んでください。

Step 1 は 2 つの sub-step から成ります。各 sub-step はそれぞれの reference file を読み、それぞれの output file を作ります。**次へ進む前に各 sub-step を最後まで完了してください。**

#### Sub-step 1a: Entry point & execution flow

> **Reference**: 今すぐ `references/1-a-entry-point.md` を読んでください。

source code を読み、アプリがどう起動し、実際のユーザーがどう呼び出すのかを理解します。次へ進む前に、調査結果を `pixie_qa/01-entry-point.md` に書いてください。

> **Checkpoint**: `pixie_qa/01-entry-point.md` に entry point、execution flow、user-facing interface、env requirement が書かれていること。

#### Sub-step 1b: Eval criteria

> **Reference**: 今すぐ `references/1-b-eval-criteria.md` を読んでください。

アプリの use case と eval criteria を定義します。use case は dataset 作成 (Step 4) を導き、eval criteria は evaluator 選定 (Step 3) を導きます。次へ進む前に、調査結果を `pixie_qa/02-eval-criteria.md` に書いてください。

> **Checkpoint**: `pixie_qa/02-eval-criteria.md` に use case、eval criteria、その適用範囲が書かれていること。まだ Step 2 の instructions は読まないでください。

---

### Step 2: Instrument with `wrap` and capture a reference trace

> **Reference**: 詳細な sub-step のために今すぐ `references/2-wrap-and-trace.md` を読んでください。

**Goal**: 外部データを制御し、出力を捕捉できるようにしてアプリを testable にします。データ境界に置かれた `wrap()` は、test harness が制御された input を注入できるようにし (本来の DB/API 呼び出しを置き換える)、同時に採点用の output を捕捉できるようにします。`Runnable` class は `pixie test` が setup、invoke、teardown に使う lifecycle interface を提供します。`pixie trace` で取得した reference trace は instrumentation が機能している証拠であり、Step 4 の dataset 作成に必要な正確なデータ shape を与えます。

> **Checkpoint**: `pixie_qa/scripts/run_app.py` が書かれ、検証されていること。`pixie_qa/reference-trace.jsonl` が存在し、`pixie format` で整形したときに期待されるすべての data point が現れること。まだ Step 3 の instructions は読まないでください。

---

### Step 3: Define evaluators

> **Reference**: 詳細な sub-step のために今すぐ `references/3-define-evaluators.md` を読んでください。

**Goal**: Step 1b の定性的な eval criteria を、実行可能な具体的 scoring function に変換します。各 criterion は built-in evaluator または自作 evaluator に対応付けられます。evaluator mapping artifact は criteria と dataset の橋渡しをし、すべての quality dimension に scorer があることを保証します。

> **Checkpoint**: すべての evaluator が実装されていること。`pixie_qa/03-evaluator-mapping.md` に criterion-to-evaluator mapping が書かれていること。まだ Step 4 の instructions は読まないでください。

---

### Step 4: Build the dataset

> **Reference**: 詳細な sub-step のために今すぐ `references/4-build-dataset.md` を読んでください。

**Goal**: runnable (Step 2)、evaluator (Step 3)、use case (Step 1b) を結び付ける test scenario を作ります。各 dataset entry は、アプリに何を送るか、外部 service から何が見えるべきか、結果をどう採点するかを定義します。データ shape と field name の真実の源として Step 2 の reference trace を使ってください。

> **Checkpoint**: 多様な entry を含み、すべての use case をカバーする Dataset JSON が `pixie_qa/datasets/<name>.json` に作られていること。まだ Step 5 の instructions は読まないでください。

---

### Step 5: Run evaluation-based tests

> **Reference**: 詳細な sub-step のために今すぐ `references/5-run-tests.md` を読んでください。

**Goal**: パイプライン全体を end-to-end で実行し、実スコアが出ることを確認します。このステップは、すべての dataset entry が走って採点されるまで setup や data の問題を修正することに関するものです。test が結果を出したら、パターン分析のために `pixie analyze` を実行します。

> **Checkpoint**: test が実行され、実スコアが出ていること。analysis が生成されていること。
>
> test が error で止まるなら、それは setup bug です。修正して再実行してください。しかし、test が実際の pass/fail score を出すなら、それが成果物です。
>
> **STOP GATE — test が score を出した後に何かする前にこれを読んでください:**
>
> - ユーザーの元の prompt が setup だけを求めている場合 ("set up QA"、"add tests"、"add evals"、"set up evaluations") は、**ここで止まってください**。ユーザーにはこう報告します: "QA setup is complete. Tests show N/M passing. [brief summary]. Want me to investigate the failures and iterate?" Step 6 には進まないでください。
> - 元の prompt が明示的に iteration を求めている場合 ("fix"、"improve"、"debug"、"iterate"、"investigate failures"、"make tests pass") は、Step 6 に進んでください。

---

### Step 6: Investigate and iterate

> **Reference**: 今すぐ `references/6-investigate.md` を読んでください。ここには stop/continue の判定、analysis review、root-cause pattern、investigation 手順があります。**調査作業を始める前に、その instructions に従ってください。**

---

## Web Server Management

pixie-qa は、context、trace、eval result をユーザーに表示するための Web server を background で動かします。これは setup script によって自動起動されます (`pixie start` により detached な background process が起動され、すぐ制御が返ります)。

ユーザーが eval-driven-dev workflow を終えたら、Web server はまだ動いており、次のコマンドで後片付けできることを伝えてください。

```bash
pixie stop
```

重要: Web server を停止すると Web UI にアクセスできなくなります。したがって、ユーザーが Web UI をもう使わないと確認した場合にのみ停止してください。引き続き Web UI を使いたいなら、**停止してはいけません**。

また workflow を再開するたびに、必ず resources 内の setup.sh script を再実行し、Web server が動いていることを保証してください。
