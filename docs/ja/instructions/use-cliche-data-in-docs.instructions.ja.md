---
description: 'ドキュメントや例では、ローカルの script、configuration、task file、prompt context 由来の実データや機密データを決して使わず、汎用的でありきたりな placeholder data のみを使うことを徹底する。'
applyTo: '**/*.{md,js,mjs,cjs,ts,tsx,jsx,py,json}'
---

# ドキュメントではありきたりなデータを使う

ツールのドキュメントを更新または作成するときは、**実データを決して含めてはならない**。prompt、ローカル設定、script、task file、その他の implementation-specific な source から提供されたデータは使わないこと。ドキュメントでは、機密情報を露出し得ない、汎用的で一般的によく知られた placeholder data だけを使う。

## なぜ重要か

ツールの source code やローカル設定には、実在の名前、実在の email address、実在の organization 情報、実在の domain 名が含まれていることが多い。これらの値はツールを動作させるためには必要でも、**公開向けドキュメントに載せる場所ではない**。実データを docs に漏らすと、次のような情報を露出する恐れがある:

- 内部の事業名や連絡先
- email address や domain 名
- client や customer の識別子
- account 名や資格情報
- private な運用を推測できる organization 固有の用語

## 中核ルール

> **データが prompt、ローカル file、script、config、または task に由来するなら、それはドキュメントに入れない。**
>
> ドキュメントの例では、よく知られた架空のデータ、または明らかに placeholder とわかるデータだけを使う。

## 実データに当たるもの

次に由来する任意の値:

- **ローカル configuration file** (例: `config.json`、`.env`、account module)
- **script と task file** (例: batch script、shell script、task runner)
- **prompt context** (例: agent にツールの作成や更新を依頼するときにユーザーが渡したデータ)
- **map や filter file** (例: JSON mapping、data extraction rule)
- **Git で無視される file** (例: environment-specific な値を含むためバージョン管理から除外されている file)

## ドキュメントで承認される placeholder data

すべてのドキュメントと例では、次のような generic でありきたりな代替データを使う:

| Category | Approved Placeholder Examples |
| --- | --- |
| **People** | Jane Doe, John Smith, Alice, Bob |
| **Email addresses** | `jane.doe@example.com`, `admin@example.org` |
| **Organizations** | Acme Corp, Contoso, Northwind Traders |
| **Domains** | `example.com`, `example.org`, `example.net` |
| **Addresses** | 123 Main Street, Suite 100, Springfield |
| **Phone numbers** | `(555) 123-4567` |
| **Accounts / usernames** | `demo-user`, `test-account` |
| **File paths** | `accounts/acme.mjs`, `config/reports.json` |
| **Project names** | My Project, Sample App, Demo Tool |

## このルールの適用方法

### 機能を追加するとき

実在する account data を使って機能を追加した場合 (例: 実在 client 名を使った script)、その機能を文書化するときは、架空の account 名に置き換える。

**実装上の実ファイル:** 特定の business 向けに設定された account module

**ドキュメント例:**

```javascript
// accounts/acme.mjs — account 設定の例
export default {
  name: 'Acme Corp',
  email: 'reports@example.com',
  folder: 'INBOX',
};
```

### 設定ドキュメントを更新するとき

config file が実在 domain、実在 path、または実在 credential を参照している場合、ドキュメントに含める前にすべて placeholder に置き換える。

**ドキュメント例:**

```json
{
  "host": "imap.example.com",
  "user": "admin@example.com",
  "folder": "INBOX/Reports",
  "outputDir": "./downloads"
}
```

### script 例を書くとき

script が特定 organization 向けの task を自動化する場合でも、ドキュメント例では generic な organization 名と generic な parameter を使わなければならない。

**ドキュメント例:**

```batch
@echo off
REM 例: Acme Corp 向けの extraction task を実行する
node extractEmail.mjs --account acme --task download
```

## コードとドキュメントの境界

| Context | Real Data Allowed? |
| --- | --- |
| 実行時に使うローカル script と config file | Yes |
| environment-specific な値を持つ Git 無視 file | Yes |
| ツールの構築 / 設定のために提供された prompt data | Yes (code 内のみ) |
| README.md、docs/ folder、example template | **No — placeholder のみを使う** |
| CHANGELOG.md の項目 | **No — 一般化して記述する** |
| commit 済み source file の code comment | **No — 汎用的に保つ** |

## 1 つの例外

実データ由来の語が、例示の文脈ではなく、通常の意味を持つ一般的な英単語として使われる場合に限り、ドキュメントに現れてもよい。たとえば、"development" は実在 organization 名にも含まれていたとしても、"This tool is under active development" のような文では問題ない。

## まとめ

ドキュメントは公開される。実装データは非公開である。両者を分離すること。あらゆる doc file のあらゆる例は、次の単純な問いに合格しなければならない: *見知らぬ人がこれを読んでも、このツールの背後にいる実際の users、clients、organizations について何もわからないか?* もし答えが no なら、そのデータをありきたりな placeholder に置き換える。
