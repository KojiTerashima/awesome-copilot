# スタック検出リファレンス

技術スタックが曖昧な場合にこのファイルを読み込んでください。たとえば、複数のマニフェストファイルが存在する、見慣れない拡張子がある、または明確な `package.json` / `go.mod` がない場合です。

---

## マニフェストファイル → エコシステム

| File | Ecosystem | Key fields to read |
|------|-----------|--------------------|
| `package.json` | Node.js / JavaScript / TypeScript | `dependencies`, `devDependencies`, `scripts`, `main`, `type`, `engines` |
| `go.mod` | Go | モジュールパス、Go バージョン、`require` ブロック |
| `requirements.txt` | Python (pip) | バージョン固定付きパッケージ一覧 |
| `Pipfile` | Python (pipenv) | `[packages]`, `[dev-packages]`, `[requires]` の python バージョン |
| `pyproject.toml` | Python (poetry / uv / hatch) | `[tool.poetry.dependencies]`, `[project]`, `[build-system]` |
| `setup.py` / `setup.cfg` | Python (setuptools, legacy) | `install_requires`, `python_requires` |
| `Cargo.toml` | Rust | `[dependencies]`, `[[bin]]`, `[lib]` |
| `pom.xml` | Java / Kotlin (Maven) | `<dependencies>`, `<artifactId>`, `<groupId>`, `<java.version>` |
| `build.gradle` / `build.gradle.kts` | Java / Kotlin (Gradle) | `dependencies {}`, `sourceCompatibility` |
| `composer.json` | PHP | `require`, `require-dev` |
| `Gemfile` | Ruby | `gem` 宣言、`ruby` バージョン制約 |
| `mix.exs` | Elixir | `deps/0`, `elixir: "~> X.Y"` |
| `pubspec.yaml` | Dart / Flutter | `dependencies`, `dev_dependencies`, `environment.sdk` |
| `*.csproj` | .NET / C# | `<PackageReference>`, `<TargetFramework>` |
| `*.sln` | .NET solution | 複数の `.csproj` プロジェクトを参照 |
| `deno.json` / `deno.jsonc` | Deno (TypeScript runtime) | `imports`, `tasks` |
| `bun.lockb` | Bun (JavaScript runtime) | バイナリ lockfile。依存関係は `package.json` を確認 |

---

## 言語ランタイムのバージョン検出

| Language | Where to find the version |
|----------|--------------------------|
| Node.js | `.nvmrc`, `.node-version`, `package.json` の `engines.node`, Docker の `FROM node:X` |
| Python | `.python-version`, `pyproject.toml [requires-python]`, Docker の `FROM python:X` |
| Go | `go.mod` の1行目（`go 1.21`） |
| Java | `pom.xml` の `<java.version>`、`build.gradle` の `sourceCompatibility`、Docker の `FROM eclipse-temurin:X` |
| Ruby | `.ruby-version`, `Gemfile` の `ruby 'X.Y.Z'` |
| Rust | `rust-toolchain.toml`, `rust-toolchain` ファイル |
| .NET | `.csproj` の `<TargetFramework>`（例: `net8.0`） |

---

## フレームワーク検出（Node.js / TypeScript）

| Dependency in `package.json` | Framework |
|-----------------------------|-----------|
| `express` | Express.js（最小構成の HTTP サーバー） |
| `fastify` | Fastify（高性能 HTTP サーバー） |
| `next` | Next.js（SSR/SSG React。`pages/` または `app/` ディレクトリを確認） |
| `nuxt` | Nuxt.js（SSR/SSG Vue） |
| `@nestjs/core` | NestJS（DI を備えた規約重視の Node.js フレームワーク） |
| `koa` | Koa（ミドルウェア重視、組み込みルーターなし） |
| `@hapi/hapi` | Hapi |
| `@trpc/server` | tRPC（REST/GraphQL スキーマなしの型安全 API） |
| `routing-controllers` | routing-controllers（デコレーター方式の Express ラッパー） |
| `typeorm` | TypeORM（デコレーター対応の SQL ORM） |
| `prisma` | Prisma（型安全 ORM。`prisma/schema.prisma` を確認） |
| `mongoose` | Mongoose（MongoDB ODM） |
| `sequelize` | Sequelize（SQL ORM） |
| `drizzle-orm` | Drizzle（軽量 SQL ORM） |
| `next` なしの `react` | 素の React SPA（`react-router-dom` を確認） |
| `nuxt` なしの `vue` | 素の Vue SPA |

---

## フレームワーク検出（Python）

| Package | Framework |
|---------|-----------|
| `fastapi` | FastAPI（非同期 REST、自動 OpenAPI ドキュメント） |
| `flask` | Flask（最小構成の WSGI Web フレームワーク） |
| `django` | Django（機能全部入り。`settings.py` を確認） |
| `starlette` | Starlette（ASGI。FastAPI のベースとしてよく使われる） |
| `aiohttp` | aiohttp（非同期 HTTP クライアント兼サーバー） |
| `sqlalchemy` | SQLAlchemy（SQL ORM。`alembic` マイグレーションを確認） |
| `alembic` | Alembic（SQLAlchemy のマイグレーションツール） |
| `pydantic` | Pydantic（データバリデーション。FastAPI の中核） |
| `celery` | Celery（分散タスクキュー） |

---

## モノレポ検出

次のシグナルを順に確認してください。

1. `pnpm-workspace.yaml` — pnpm workspaces
2. `lerna.json` — Lerna monorepo
3. `nx.json` — Nx monorepo（`workspace.json` も確認）
4. `turbo.json` — Turborepo
5. `rush.json` — Rush（Microsoft のモノレポマネージャー）
6. `moon.yml` — Moon
7. `"workspaces": [...]` を持つ `package.json` — npm/yarn workspaces
8. 独自の `package.json` を持つ `packages/`, `apps/`, `libs/`, `services/` ディレクトリの存在

モノレポが検出された場合: 各ワークスペースは**独立した**依存関係や規約を持つ可能性があります。`STACK.md` では各サブパッケージを個別に整理し、`STRUCTURE.md` にはモノレポ構造を記載してください。

---

## TypeScript パスエイリアス検出

`tsconfig.json` に `paths` キーがある場合、非相対プレフィックスの import はエイリアスです。構造を記載する前に対応関係をマッピングしてください。

```json
// tsconfig.json example
"paths": {
  "@/*": ["./src/*"],
  "@components/*": ["./src/components/*"],
  "@utils/*": ["./src/utils/*"]
}
```

`import { foo } from '@/utils/bar'` のような import は `src/utils/bar` に解決されます。`@/utils/bar` ではなく `src/utils/bar` として記載してください。

---

## Docker ベースイメージ → ランタイム

マニフェストファイルがなく `Dockerfile` が存在する場合、`FROM` 行でランタイムを判別できます。

| FROM line pattern | Runtime |
|------------------|---------|
| `FROM node:X` | Node.js X |
| `FROM python:X` | Python X |
| `FROM golang:X` | Go X |
| `FROM eclipse-temurin:X` | Java X（Eclipse Temurin JDK） |
| `FROM mcr.microsoft.com/dotnet/aspnet:X` | .NET X |
| `FROM ruby:X` | Ruby X |
| `FROM rust:X` | Rust X |
| `FROM alpine`（単独） | `RUN apk add` で何がインストールされているかを確認 |

