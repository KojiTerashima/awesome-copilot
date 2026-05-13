# モジュール 7: 高度なテクニック

1. **`@` ファイルメンション** — AI がファイルを見つけてくれることに頼らず、必ず正確なコンテキストを渡す
   - `@src/auth.ts` — 単一ファイル
   - `@src/components/` — ディレクトリ一覧
   - "Fix @src/auth.ts to match @tests/auth.test.ts" — 複数ファイルのコンテキスト

2. **`!` シェルバイパス** — `!git log --oneline -5` は AI のオーバーヘッドなしで即実行される

3. **`/research`** — GitHub 検索と Web ソースを使った詳細な調査を実行する

4. **`/resume` + `--continue`** — CLI 起動をまたいだセッション継続

5. **`/compact`** — コンテキストが大きくなったら履歴を圧縮する（95% で自動）
   - まず `/context` で確認する
   - タスクの自然な区切りで使うのが最適
   - 警告サイン: AI が以前の発言と矛盾する、トークン使用量が 80% 超

6. **`/context`** — どこでトークン予算が消費されているかを可視化する

7. **カスタム指示の優先順位**（高い順）:
   - `CLAUDE.md` / `GEMINI.md` / `AGENTS.md`（git ルート + cwd）
   - `.github/instructions/**/*.instructions.md`（パス単位で適用）
   - `.github/copilot-instructions.md`
   - `~/.copilot/copilot-instructions.md`
   - `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`（環境変数による追加ディレクトリ）

8. **パス単位の指示:**
   - `applyTo: "src/api/**"` を持つ `.github/instructions/backend.instructions.md`
   - コードベースの部位ごとに異なるコーディング標準を適用できる

9. **LSP 設定** — `~/.copilot/lsp-config.json` または `.github/lsp.json`

10. **`/review`** — ターミナルを離れずにコードレビューを受ける

11. **`--allow-all` / `--yolo`** — 完全信頼モード（責任を持って使うこと）

12. **`Ctrl+T`** — AI の思考過程を観察する（推論パターン学習に役立つ）
