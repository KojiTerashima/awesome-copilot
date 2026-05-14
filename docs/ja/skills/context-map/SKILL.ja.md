---
name: context-map
description: '変更を加える前に、タスクに関連するすべてのファイルのマップを生成します'
---

# Context Map

変更を実装する前に、コードベースを分析してコンテキストマップを作成してください。

## Task

{{task_description}}

## Instructions

1. このタスクに関連するファイルをコードベースから検索する
2. 直接依存関係（imports/exports）を特定する
3. 関連する test を見つける
4. 既存コード内で似た pattern を探す

## Output Format

```markdown
## Context Map

### Files to Modify
| File | Purpose | Changes Needed |
|------|---------|----------------|
| path/to/file | description | what changes |

### Dependencies (may need updates)
| File | Relationship |
|------|--------------|
| path/to/dep | imports X from modified file |

### Test Files
| Test | Coverage |
|------|----------|
| path/to/test | tests affected functionality |

### Reference Patterns
| File | Pattern |
|------|---------|
| path/to/similar | example to follow |

### Risk Assessment
- [ ] Breaking changes to public API
- [ ] Database migrations needed
- [ ] Configuration changes required
```

このマップがレビューされるまで、実装に進まないでください。
