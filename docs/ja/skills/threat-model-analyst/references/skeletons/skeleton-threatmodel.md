# スケルトン: 1-threatmodel.md

> **⛔ テンプレートの内容を VERBATIM の下にコピーします (外側のコード フェンスを除く)。 `[FILL]` プレースホルダーを置き換えます。 `.md` と `.mmd` の図は同一である必要があります。**
> **⛔ データ フロー テーブルの列: `ID | Source | Target | Protocol | Description`。 `Target` の名前を `Destination` に変更しないでください。列の順序を変更しないでください。**
> **⛔ 信頼境界テーブルの列: `Boundary | Description | Contains` (3 列)。 `Name` 列を追加したり、`Contains` を `Components Inside` に名前変更したりしないでください。**

---````markdown
# Threat Model

## Data Flow Diagram

```人魚
[FILL: 1.1-threatmodel.mmd から正確なコンテンツをコピー]```

## Element Table

| Element | Type | TMT Category | Description | Trust Boundary |
|---------|------|--------------|-------------|----------------|
[CONDITIONAL: For K8s apps with sidecars, add a `Co-located Sidecars` column after Trust Boundary]
[REPEAT: one row per element]
| [FILL] | [FILL: Process / External Interactor / Data Store] | [FILL: SE.P.TMCore.* / SE.EI.TMCore.* / SE.DS.TMCore.*] | [FILL] | [FILL] |
[END-REPEAT]

## Data Flow Table

| ID | Source | Target | Protocol | Description |
|----|--------|--------|----------|-------------|
[REPEAT: one row per data flow]
| [FILL: DF##] | [FILL] | [FILL] | [FILL] | [FILL] |
[END-REPEAT]

## Trust Boundary Table

| Boundary | Description | Contains |
|----------|-------------|----------|
[REPEAT: one row per trust boundary]
| [FILL] | [FILL] | [FILL: comma-separated component list] |
[END-REPEAT]

[CONDITIONAL: Include ONLY if summary diagram was generated (elements > 15 OR boundaries > 4)]

## Summary View

```人魚
[FILL: 1.2-threatmodel-summary.mmd から正確な内容をコピー]```

## Summary to Detailed Mapping

| Summary Element | Contains | Summary Flows | Maps to Detailed Flows |
|-----------------|----------|---------------|------------------------|
[REPEAT]
| [FILL] | [FILL] | [FILL: SDF##] | [FILL: DF##, DF##] |
[END-REPEAT]

[END-CONDITIONAL]
````

**修正されたルール:**
- 詳細なフローには `DF01`、`DF02` を使用します。 `SDF01`、`SDF02` (概要フロー)
- 要素タイプ: 正確に `Process`、`External Interactor`、または `Data Store`
- TMT カテゴリ: tmt-element-taxonomy.md の特定の ID である必要があります (例: `SE.P.TMCore.WebSvc`)