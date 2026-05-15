---
name: javax-to-jakarta-migration
description: "Java コードを javax.* から jakarta.* 名前空間に移行します。 Tomcat 11、Jakarta EE 10 にアップグレードする場合、またはコードベースで javax インポートが検出された場合に使用します。"
argument-hint: "File, package, or module to migrate"
---

# javax → jakarta 移行スキル

## いつ使用するか
- Tomcat 11 / Jakarta EE 10+ へのアップグレード
- コード レビューで `javax.*` インポートが検出されました
- 既存のプロジェクトを jakarta 名前空間に移行する

## 手順

### ステップ 1 — javax の使用状況をスキャンする
コードベースで移行が必要なすべての `javax.*` インポートを検索します。```
javax.servlet.*      → jakarta.servlet.*
javax.persistence.*  → jakarta.persistence.*
javax.validation.*   → jakarta.validation.*
javax.annotation.*   → jakarta.annotation.*
javax.inject.*       → jakarta.inject.*
javax.enterprise.*   → jakarta.enterprise.*
javax.faces.*        → jakarta.faces.*
javax.ws.rs.*        → jakarta.ws.rs.*
javax.el.*           → jakarta.el.*
javax.json.*         → jakarta.json.*
javax.mail.*         → jakarta.mail.*
javax.websocket.*    → jakarta.websocket.*
```

**移行しないでください** (これらは `javax.*` に残ります):
- `javax.sql.*` — JDK の一部
- `javax.naming.*` — JDK (JNDI) の一部
- `javax.crypto.*` — JDK の一部
- `javax.net.*` — JDK の一部
- `javax.security.auth.*` — JDK の一部
- `javax.swing.*`、`javax.xml.parsers.*` — JDK パッケージ

### ステップ 2 — pom.xml を更新する
依存関係の座標を置き換えます。

|古い |新しい |
|-----|-----|
| `javax.servlet:javax.servlet-api` | `jakarta.servlet:jakarta.servlet-api:6.0.0` |
| `javax.persistence:javax.persistence-api` | `jakarta.persistence:jakarta.persistence-api:3.1.0` |
| `javax.validation:validation-api` | `jakarta.validation:jakarta.validation-api:3.0.2` |
| `javax.annotation:javax.annotation-api` | `jakarta.annotation:jakarta.annotation-api:2.1.1` |

### ステップ 3 — web.xml を更新します (存在する場合)```xml
<!-- Old namespace -->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="4.0">

<!-- New namespace -->
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee" version="6.0">
```

### ステップ4 — Javaソースファイルを更新する
すべての `javax.` インポートを `.java` ファイル内の `jakarta.` に相当するものに置き換えます。

### ステップ 5 — 確認する
1. `mvn clean compile` または `gradlew build` を実行します - コンパイル エラーを修正します
2. `mvn test` または `gradlew test` を実行 — すべてのテストが合格することを確認します
3. 残りの `javax.*` インポートを検索します (JDK パッケージを除く)

### 出力
変更されたすべてのファイル、置き換えられたインポート、および必要な手動手順をリストした移行の概要を提供します。