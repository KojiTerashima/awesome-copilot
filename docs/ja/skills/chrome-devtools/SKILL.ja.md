---
name: chrome-devtools
description: 'Chrome DevTools MCP を使ったエキスパートレベルのブラウザー自動化、デバッグ、パフォーマンス分析。Web ページとの対話、スクリーンショット取得、ネットワークトラフィック分析、パフォーマンスプロファイリングに使用します。'
license: MIT
---

# Chrome DevTools Agent

## 概要

実行中の Chrome ブラウザーを操作し調査するための専用 skill です。この skill は `chrome-devtools` MCP server を活用し、単純な navigation から高度な performance profiling まで、幅広いブラウザー関連タスクを実行します。

## 使用する場面

次のような場合にこの skill を使います。

- **Browser Automation**: ページ移動、要素のクリック、フォーム入力、ダイアログ対応。
- **Visual Inspection**: Web ページのスクリーンショットやテキストスナップショットの取得。
- **Debugging**: console message の確認、ページコンテキストでの JavaScript 評価、network request の分析。
- **Performance Analysis**: performance trace を記録・分析し、ボトルネックや Core Web Vitals の問題を特定。
- **Emulation**: viewport のサイズ変更や network/CPU 条件のエミュレーション。

## ツールカテゴリ

### 1. Navigation と Page Management

- `new_page`: 新しい tab/page を開く。
- `navigate_page`: 特定の URL へ移動、reload、または履歴移動を行う。
- `select_page`: 開いている page 間でコンテキストを切り替える。
- `list_pages`: 開いているすべての page とその ID を確認する。
- `close_page`: 特定の page を閉じる。
- `wait_for`: page に特定の text が現れるまで待つ。

### 2. Input と Interaction

- `click`: 要素をクリックする（snapshot の `uid` を使用）。
- `fill` / `fill_form`: input に text を入力する、または複数 field をまとめて埋める。
- `hover`: 要素の上にマウスを移動する。
- `press_key`: keyboard shortcut や特殊キーを送る（例: "Enter", "Control+C"）。
- `drag`: 要素を drag and drop する。
- `handle_dialog`: browser alert/prompt を受け入れる、または閉じる。
- `upload_file`: file input を通してファイルをアップロードする。

### 3. Debugging と Inspection

- `take_snapshot`: text ベースの accessibility tree を取得する（要素特定に最適）。
- `take_screenshot`: ページ全体または特定要素の見た目をキャプチャする。
- `list_console_messages` / `get_console_message`: page の console 出力を確認する。
- `evaluate_script`: page コンテキストで任意の JavaScript を実行する。
- `list_network_requests` / `get_network_request`: network traffic と request 詳細を分析する。

### 4. Emulation と Performance

- `resize_page`: viewport の寸法を変更する。
- `emulate`: CPU/Network の throttling や geolocation の emulation を行う。
- `performance_start_trace`: performance profile の記録を開始する。
- `performance_stop_trace`: 記録を停止して trace を保存する。
- `performance_analyze_insight`: 記録した performance data の詳細分析を取得する。

## ワークフローパターン

### Pattern A: 要素の特定（Snapshot-First）

要素を探すときは `take_screenshot` より `take_snapshot` を優先します。snapshot は interaction tool で必要になる `uid` を提供します。

```markdown
1. `take_snapshot` で現在の page 構造を取得する。
2. 対象要素の `uid` を見つける。
3. `click(uid=...)` または `fill(uid=..., value=...)` を使う。
```

### Pattern B: エラーのトラブルシューティング

page が失敗している場合は、console log と network request の両方を確認します。

```markdown
1. `list_console_messages` で JavaScript error を確認する。
2. `list_network_requests` で失敗した resource（4xx/5xx）を特定する。
3. `evaluate_script` で特定の DOM 要素や global variable の値を確認する。
```

### Pattern C: Performance Profiling

なぜ page が遅いのかを特定します。

```markdown
1. `performance_start_trace(reload=true, autoStop=true)`
2. page の読み込みまたは trace 完了を待つ。
3. `performance_analyze_insight` で LCP の問題や layout shift を調べる。
```

## ベストプラクティス

- **Context Awareness**: どの tab が現在 active かわからない場合は、必ず `list_pages` と `select_page` を実行する。
- **Snapshots**: 大きな navigation や DOM 変更の後は新しい snapshot を取る。`uid` の値が変わることがあるため。
- **Timeouts**: 読み込みが遅い要素で待ち続けないよう、`wait_for` には妥当な timeout を使う。
- **Screenshots**: 見た目の確認には `take_screenshot` を必要最小限で使い、ロジック判断には `take_snapshot` を使う。
