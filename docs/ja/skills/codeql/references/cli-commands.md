# CodeQL CLI Command Reference

CodeQL CLI の詳細リファレンス（インストール、database 作成、解析、SARIF アップロード、CI 統合）。

## Installation

### Download the CodeQL Bundle
常に以下から CodeQL bundle（CLI + precompiled queries）を取得してください:  
**https://github.com/github/codeql-action/releases**

```bash
# Extract the bundle
tar xf codeql-bundle-linux64.tar.zst

# Add to PATH
export PATH="$HOME/codeql:$PATH"

# Verify installation
codeql resolve packs
codeql resolve languages
```

## Core Commands

### `codeql database create`
```bash
codeql database create <output-dir> \
  --language=<language> \
  --source-root=<source-dir>
```

```bash
codeql database create <output-dir> \
  --language=java-kotlin \
  --command='./gradlew build' \
  --source-root=.
```

```bash
codeql database create <output-dir> \
  --db-cluster \
  --language=java,python,javascript-typescript \
  --command='./build.sh' \
  --source-root=.
```

### `codeql database analyze`
```bash
codeql database analyze <database-dir> \
  <query-suite-or-pack> \
  --format=sarif-latest \
  --sarif-category=<category> \
  --output=<output-file>
```

### `codeql github upload-results`
```bash
codeql github upload-results \
  --repository=<owner/repo> \
  --ref=<git-ref> \
  --commit=<commit-sha> \
  --sarif=<sarif-file>
```

認証には `security-events: write` 権限付き `GITHUB_TOKEN` が必要です。

### `codeql resolve packs`
```bash
codeql resolve packs
```

### `codeql resolve languages`
```bash
codeql resolve languages
```

### `codeql database bundle`
```bash
codeql database bundle <database-dir> \
  --output=<archive-file>
```

## CLI Server Mode

### `codeql execute cli-server`
```bash
codeql execute cli-server [options]
```

## CI Integration Pattern

```bash
#!/bin/bash
set -euo pipefail

REPO="my-org/my-repo"
REF="refs/heads/main"
COMMIT=$(git rev-parse HEAD)
LANGUAGES=("javascript-typescript" "python")

codeql database create codeql-dbs \
  --db-cluster \
  --source-root=. \
  --language=$(IFS=,; echo "${LANGUAGES[*]}")

for lang in "${LANGUAGES[@]}"; do
  codeql database analyze "codeql-dbs/$lang" \
    "${lang}-security-extended.qls" \
    --format=sarif-latest \
    --sarif-category="$lang" \
    --output="${lang}-results.sarif" \
    --threads=0

  codeql github upload-results \
    --repository="$REPO" \
    --ref="$REF" \
    --commit="$COMMIT" \
    --sarif="${lang}-results.sarif"
done
```
