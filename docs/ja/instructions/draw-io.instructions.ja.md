---
description: ".drawio、.drawio.svg、.drawio.png ファイル内の draw.io 図と mxGraph XML を作成、編集、レビューするときに使用します。"
applyTo: "**/*.drawio,**/*.drawio.svg,**/*.drawio.png"
---

# draw.io 図の標準

> **Skill**: 任意の `.drawio` ファイルを生成または編集する前に、完全なワークフロー、XML レシピ、トラブルシューティングのために `.github/skills/draw-io/SKILL.md` を読み込んでください。

---

## 必須ワークフロー

draw.io のすべてのタスクで、次の手順に従ってください。

1. **特定する**: 図の種類 (flowchart / architecture / sequence / ER / UML / network / BPMN)
2. **選択する**: `.github/skills/draw-io/templates/` から対応するテンプレートを選んで調整するか、最小スケルトンから開始する
3. **計画する**: XML を書く前に紙でレイアウトを考える。最初に層、アクター、またはエンティティを定義する
4. **生成する**: 以下のルールに従って有効な mxGraph XML を生成する
5. **検証する**: `python .github/skills/draw-io/scripts/validate-drawio.py <file>` を使う
6. **確認する**: draw.io 拡張 (`hediet.vscode-drawio`) を使って VS Code で開き、ファイルが描画されることを確認する

---

## XML 構造ルール (交渉不可)

```xml
<!-- Set modified to the current ISO 8601 timestamp when generating a new file -->
<mxfile host="Electron" modified="" version="26.0.0">
  <diagram id="unique-id" name="Page Name">
    <mxGraphModel ...>
      <root>
        <mxCell id="0" />                          <!-- REQUIRED: always first -->
        <mxCell id="1" parent="0" />               <!-- REQUIRED: always second -->
        <!-- all other cells go here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

- `id="0"` と `id="1"` は **必須** であり、最初の 2 つの cell でなければなりません。例外はありません
- すべての cell `id` は、図の中で **一意** でなければなりません
- すべての vertex (`vertex="1"`) は、子の `<mxGeometry x y width height as="geometry">` を持つ必要があります
- すべての edge (`edge="1"`) は、既存の vertex id を指す `source`/`target` を持つ必要があります。ただし **例外** として、floating edge (sequence diagram の lifeline) は `source`/`target` 属性の代わりに、`<mxGeometry>` の中で `<mxPoint as="sourcePoint">` と `<mxPoint as="targetPoint">` を使います
- id=0 を除くすべての cell は、既存 id を指す `parent` を持つ必要があります
- コンテナー (swimlane) の子は、キャンバスではなく **親に対する相対座標** を使います

---

## 必須スタイル規約

### セマンティック カラーパレット — プロジェクト全体で一貫して使う

| 役割 | fillColor | strokeColor |
|---|---|---|
| Primary / Info (既定) | `#dae8fc` | `#6c8ebf` |
| Success / Start / Positive | `#d5e8d4` | `#82b366` |
| Warning / Decision | `#fff2cc` | `#d6b656` |
| Error / End / Danger | `#f8cecc` | `#b85450` |
| Neutral / Interface | `#f5f5f5` | `#666666` |
| External / Partner | `#e1d5e7` | `#9673a6` |

### vertex 形状には常に次を含める

```
whiteSpace=wrap;html=1;
```

### ラベルに HTML タグ (`<b>`, `<i>`, `<br>`) が含まれる場合は、必ず `html=1` を使う

### 標準コネクター

```
edgeStyle=orthogonalEdgeStyle;html=1;
```

---

## 図種別クイック リファレンス

| 種類 | コンテナー | 主な形状 | コネクター スタイル |
|---|---|---|---|
| Flowchart | なし | `ellipse` (開始/終了), `rounded=1` (処理), `rhombus` (判断) | `orthogonalEdgeStyle` |
| Architecture | 層ごとの `swimlane` | `rounded=1` のサービス、cloud/DB 形状 | ラベル付き `orthogonalEdgeStyle` |
| Sequence | なし | `mxgraph.uml.actor`, 破線 lifeline edge | `endArrow=block` (sync), `endArrow=open;dashed=1` (return) |
| ER Diagram | `shape=table;childLayout=tableLayout` | `shape=tableRow`, `shape=partialRectangle` | `entityRelationEdgeStyle;endArrow=ERmany;startArrow=ERone` |
| UML Class | クラスごとの `swimlane` | 属性/メソッド用のテキスト行 | `endArrow=block;endFill=0` (継承), `dashed=1` (realize) |

---

## レイアウトのベストプラクティス

- すべての座標を **10 px グリッド** に合わせる (10 で割り切れる値)
- **横方向**: 同じ行の図形間は 40–60 px の間隔
- **縦方向**: 層の行間は 80–120 px の間隔
- 標準図形サイズ: `120 × 60` px (処理), `200 × 100` px (判断のひし形)
- 既定キャンバス: A4 横 `1169 × 827` px
- 1 ページあたり **最大 40 cell**。それ以上は複数ページに分割する
- 各ページ上部に **タイトル text cell** を追加する:
  ```
  style="text;strokeColor=none;fillColor=none;fontSize=18;fontStyle=1;align=center;"
  ```

---

## ファイルと命名規則

- 拡張子: バージョン管理する図には `.drawio`、Markdown に埋め込むファイルには `.drawio.svg`
- 命名: `kebab-case`。例: `order-flow.drawio`, `database-schema.drawio`
- 配置場所: `docs/` または `architecture/`。対象コードの近くに置く
- 複数ページ: 同じ `<mxfile>` 内で、論理ビューごとに 1 つの `<diagram>` 要素を使う

---

## 検証チェックリスト (毎回コミット前に実行)

- [ ] `<mxCell id="0" />` と `<mxCell id="1" parent="0" />` が最初の 2 つの cell になっている
- [ ] すべての cell id が図の中で一意である
- [ ] すべての edge の `source`/`target` id が既存の vertex を参照している
- [ ] すべての vertex cell が `<mxGeometry as="geometry">` を持っている
- [ ] すべての cell (id=0 を除く) が有効な `parent` を持っている
- [ ] XML が整形式である。未閉じタグがなく、属性値に裸の `&`, `<`, `>` がない
- [ ] セマンティック カラーパレットが一貫して使われている
- [ ] すべてのページにタイトル cell がある

```bash
# 自動検証を実行
python .github/skills/draw-io/scripts/validate-drawio.py <file.drawio>
```

---

## 参照ファイル

| ファイル | 用途 |
|---|---|
| `.github/skills/draw-io/SKILL.md` | 完全なエージェント ワークフロー、レシピ、トラブルシューティング |
| `.github/skills/draw-io/references/drawio-xml-schema.md` | 完全な mxCell 属性リファレンス |
| `.github/skills/draw-io/references/style-reference.md` | すべての style key、shape 名、edge 種別 |
| `.github/skills/draw-io/references/shape-libraries.md` | style string 付き shape library カタログ |
| `.github/skills/draw-io/templates/` | 図種別ごとのすぐ使える `.drawio` テンプレート |
| `.github/skills/draw-io/scripts/validate-drawio.py` | XML 構造バリデーター |
| `.github/skills/draw-io/scripts/add-shape.py` | CLI: 既存の図に shape を追加 |
