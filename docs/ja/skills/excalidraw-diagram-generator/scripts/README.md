# Excalidraw Library Tools

この directory には、Excalidraw library を扱うための script が入っています。

## split-excalidraw-library.py

Excalidraw library file (`*.excalidrawlib`) を、個別 icon JSON file に分割します。AI assistant が token を効率よく使えるようにするためのものです。

### Prerequisites

- Python 3.6 以上
- 追加 dependency は不要 (standard library のみ使用)

### Usage

```bash
python split-excalidraw-library.py <path-to-library-directory>
```

### Step-by-Step Workflow

1. **library directory を作成**:
   ```bash
   mkdir -p skills/excalidraw-diagram-generator/libraries/aws-architecture-icons
   ```

2. **library file をダウンロードして配置**:
   - https://libraries.excalidraw.com/ にアクセス
   - "AWS Architecture Icons" を検索し、`.excalidrawlib` file をダウンロード
   - directory 名に合わせて `aws-architecture-icons.excalidrawlib` に rename
   - step 1 で作成した directory に配置

3. **script を実行**:
   ```bash
   python skills/excalidraw-diagram-generator/scripts/split-excalidraw-library.py skills/excalidraw-diagram-generator/libraries/aws-architecture-icons/
   ```

### Output Structure

script は library directory に次の構造を作成します。

```
skills/excalidraw-diagram-generator/libraries/aws-architecture-icons/
  aws-architecture-icons.excalidrawlib  # Original file (kept)
  reference.md                          # Generated: Quick reference table
  icons/                                # Generated: Individual icon files
    API-Gateway.json
    CloudFront.json
    EC2.json
    S3.json
    ...
```

### What the Script Does

1. `.excalidrawlib` file を **読み込む**
2. `libraryItems` array から各 icon を **抽出**する
3. icon 名を sanitize して valid filename を作る (space → hyphen、special character を除去)
4. 各 icon を `icons/` directory に個別 JSON file として **保存**する
5. icon 名と filename を対応づけた table を持つ `reference.md` を **生成**する

### Benefits

- **Token Efficiency**: AI は先に軽量な `reference.md` を読んで relevant icon を探し、必要な icon file だけを読み込める
- **Organization**: icon が分かりやすい directory structure で整理される
- **Extensibility**: 複数の library set を並べて追加できる

### Recommended Workflow

1. https://libraries.excalidraw.com/ から必要な Excalidraw library をダウンロード
2. 各 library file に対してこの script を実行
3. 生成された folder を `../libraries/` へ移動
4. AI assistant は `reference.md` を使って、icon を効率的に見つけて利用する

### Library Sources (Examples — verify availability)

- https://libraries.excalidraw.com/ には cloud / service icon set がある場合があります。
- availability は変わるため、使う前に正確な library 名を site で確認してください。
- この script は、提供された任意の valid `.excalidrawlib` file で動作します。

### Troubleshooting

**Error: File not found**
- file path が正しいか確認する
- file extension が `.excalidrawlib` であることを確認する

**Error: Invalid library file format**
- file が valid な Excalidraw library file か確認する
- `libraryItems` array が含まれているか確認する

### License Considerations

third-party icon library を使う場合:
- **AWS Architecture Icons**: AWS Content License の対象
- **GCP Icons**: Google の terms に従う
- **Other libraries**: 各 library の license を確認する

この script は個人 / 組織内利用を想定しています。分割した icon file を再配布する場合は、元 library の license 条件に従ってください。

## add-icon-to-diagram.py

分割済み Excalidraw library から特定 icon を読み出し、既存 `.excalidraw` diagram に追加します。coordinate translation と ID collision 回避を処理し、必要なら icon 下に label も追加できます。

### Prerequisites

- Python 3.6 以上
- diagram file (`.excalidraw`)
- 分割済み icon library directory (`split-excalidraw-library.py` で作成)

### Usage

```bash
python add-icon-to-diagram.py <diagram-path> <icon-name> <x> <y> [OPTIONS]
```

**Options**
- `--library-path PATH` : icon library directory への path (default: `aws-architecture-icons`)
- `--label TEXT` : icon の下に text label を追加
- `--use-edit-suffix` : editor の overwrite issue を避けるため `.excalidraw.edit` 経由で編集 (default で有効。無効化は `--no-use-edit-suffix`)

### Examples

```bash
# Add EC2 icon at position (400, 300)
python add-icon-to-diagram.py diagram.excalidraw EC2 400 300

# Add VPC icon with label
python add-icon-to-diagram.py diagram.excalidraw VPC 200 150 --label "VPC"

# Safe edit mode is enabled by default (avoids editor overwrite issues)
# Use `--no-use-edit-suffix` to disable
python add-icon-to-diagram.py diagram.excalidraw EC2 500 300

# Add icon from another library
python add-icon-to-diagram.py diagram.excalidraw Compute-Engine 500 200 \
   --library-path libraries/gcp-icons --label "API Server"
```

### What the Script Does

1. library の `icons/` directory から icon JSON を **読み込む**
2. icon の bounding box を **計算**する
3. すべての coordinate に target position への **offset** を適用する
4. すべての element と group に対して **unique ID** を生成する
5. 変換済み element を diagram へ **追加**する
6. **(Optional)** icon 下に label を追加する

---

## add-arrow.py

既存 `.excalidraw` diagram に 2 点間の straight arrow を追加します。optional label と line style をサポートします。

### Prerequisites

- Python 3.6 以上
- diagram file (`.excalidraw`)

### Usage

```bash
python add-arrow.py <diagram-path> <from-x> <from-y> <to-x> <to-y> [OPTIONS]
```

**Options**
- `--style {solid|dashed|dotted}` : line style (default: `solid`)
- `--color HEX` : arrow color (default: `#1e1e1e`)
- `--label TEXT` : arrow 上に text label を追加
- `--use-edit-suffix` : editor の overwrite issue を避けるため `.excalidraw.edit` 経由で編集 (default で有効。無効化は `--no-use-edit-suffix`)

### Examples

```bash
# Simple arrow
python add-arrow.py diagram.excalidraw 300 200 500 300

# Arrow with label
python add-arrow.py diagram.excalidraw 300 200 500 300 --label "HTTPS"

# Dashed arrow with custom color
python add-arrow.py diagram.excalidraw 400 350 600 400 --style dashed --color "#7950f2"

# Safe edit mode is enabled by default (avoids editor overwrite issues)
# Use `--no-use-edit-suffix` to disable
python add-arrow.py diagram.excalidraw 300 200 500 300
```

### What the Script Does

1. 与えられた coordinate から arrow element を **作成**する
2. **(Optional)** arrow 中点近くに label を追加する
3. element を diagram に **追加**する
4. 更新後 file を **保存**する
