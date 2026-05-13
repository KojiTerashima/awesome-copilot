---
name: creating-oracle-to-postgres-master-migration-plan
description: '.NET solution 内のすべての project を検出し、それぞれの Oracle-to-PostgreSQL migration 適格性を分類し、永続的な master migration plan を作成します。複数 project にまたがる Oracle-to-PostgreSQL migration を開始するとき、migration inventory を作成するとき、Oracle dependency を含む .NET project を評価するときに使用します。'
---

# Creating an Oracle-to-PostgreSQL Master Migration Plan

.NET solution を分析し、すべての project について Oracle→PostgreSQL migration の適格性を分類し、後続の agent や skill が parse できる構造化 plan を作成します。

## Workflow

```
Progress:
- [ ] Step 1: Discover projects in the solution
- [ ] Step 2: Classify each project
- [ ] Step 3: Confirm with user
- [ ] Step 4: Write the plan file
```

**Step 1: Discover projects**

workspace root にある Solution File（`.sln` または `.slnx` 拡張子）を見つけます。複数ある場合はユーザーに確認します。これを parse して、すべての `.csproj` project 参照を抽出します。各 project について、name、path、type（class library、web API、console、test など）を記録します。

**Step 2: Classify each project**

test project 以外の各 project を走査し、次の Oracle indicator を確認します。

- NuGet 参照: `Oracle.ManagedDataAccess`、`Oracle.EntityFrameworkCore`（`.csproj` と `packages.config` を確認）
- Config 設定: `appsettings.json`、`web.config`、`app.config` 内の Oracle connection string
- コード使用箇所: `OracleConnection`、`OracleCommand`、`OracleDataReader`
- `.github/oracle-to-postgres-migration/DDL/Oracle/` 配下の DDL cross-reference（存在する場合）

各 project に次のいずれか 1 つの classification を割り当てます。

| Classification | Meaning |
|---|---|
| **MIGRATE** | 変換が必要な Oracle interaction がある |
| **SKIP** | Oracle indicator がない（UI-only、shared utility など） |
| **ALREADY_MIGRATED** | `-postgres` または `.Postgres` の duplicate が存在し、処理済みに見える |
| **TEST_PROJECT** | test project。testing workflow で扱う |

**Step 3: Confirm with user**

分類済み一覧を提示し、確定前にユーザーが classification や migration order を調整できるようにします。

**Step 4: Write the plan file**

保存先: `.github/oracle-to-postgres-migration/Reports/Master Migration Plan.md`

この template を**正確に**使ってください。後続 consumer はこの構造に依存します。

````markdown
# Master Migration Plan

**Solution:** {solution file name}
**Solution Root:** {REPOSITORY_ROOT}
**Created:** {timestamp}
**Last Updated:** {timestamp}

## Solution Summary

| Metric | Count |
|--------|-------|
| Total projects in solution | {n} |
| Projects requiring migration | {n} |
| Projects already migrated | {n} |
| Projects skipped (no Oracle usage) | {n} |
| Test projects (handled separately) | {n} |

## Project Inventory

| # | Project Name | Path | Classification | Notes |
|---|---|---|---|---|
| 1 | {name} | {relative path} | MIGRATE | {notes} |
| 2 | {name} | {relative path} | SKIP | No Oracle dependencies |

## Migration Order

1. **{ProjectName}** — {rationale, e.g., "Core data access library; other projects depend on it."}
2. **{ProjectName}** — {rationale}
````

shared/foundational library をその依存先より先に migration する順番にしてください。
