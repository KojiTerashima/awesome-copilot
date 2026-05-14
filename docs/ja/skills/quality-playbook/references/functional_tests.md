# 機能テストの作成

これが最も重要な成果物です。 Markdown ファイルはドキュメントです。機能テスト ファイルは自動化されたセーフティ ネットです。プロジェクトの規則に従って名前を付けます: `test_functional.py` (Python/pytest)、`FunctionalSpec.scala` (Scala/ScalaTest)、`FunctionalTest.java` (Java/JUnit)、`functional.test.ts` (TypeScript/Jest)、`functional_test.go` (Go) など。

## 構造: 3 つのテスト グループ

テスト フレームワークが提供する構造 (クラス (Python/Java)、記述ブロック (TypeScript/Jest)、特性 (Scala)、またはサブテスト (Go)) を使用して、テストを 3 つの論理グループに編成します。「」
スペック要件
    — テスト可能な仕様セクションごとに 1 つのテスト
    — 各テストのドキュメントには仕様要件が記載されています

フィットネスのシナリオ
    — QUALITY.md シナリオごとに 1 つのテスト (1:1 マッピング)
    — 一致する名前: test_scenario_N_memorable_name (または同等の規則)

境界とエッジケース
    — ステップ 5 の防御パターンごとに 1 つのテスト
    — null ガード、トライ/キャッチ、正規化、フォールバックをターゲットとします。
「」## テスト数ヒューリスティック

**ターゲット = (テスト可能な仕様セクション) + (QUALITY.md シナリオ) + (ステップ 5 の防御パターン)**

例: スペックセクション 12 個 + シナリオ 10 個 + 防御パターン 15 個 = 37 個のテストを目標とします。

中規模のプロジェクト (ソース ファイル 5 ～ 15 個) の場合、通常は 35 ～ 50 個の機能テストが行​​われます。要件が欠落していたり​​、検討が浅かったりすることを示唆するものは大幅に少なくなりました。数値を入力するためにパディングしないでください。すべてのテストで実際のプロジェクト コードを実行し、意味のあるプロパティを検証する必要があります。

## インポート パターン: 既存のテストと一致する

テスト コードを記述する前に、2 ～ 3 の既存のテスト ファイルを読み取り、プロジェクト モジュールがどのようにインポートされるかを特定します。これは重要です。プロジェクトによってインポートの処理方法が異なり、それを誤ると、すべてのテストが解決エラーで失敗することになります。

言語ごとの一般的なパターン:

**パイソン:**
- `sys.path.insert(0, "src/")` をそのままインポート (`from module import func`)
- パッケージのインポート (`from myproject.module import func`)
- conftest.py パス操作による相対インポート

**Java:**
- `import com.example.project.Module;` パッケージ構造と一致する
- テスト ソース ルートはメイン ソース ルートをミラーリングする必要があります

**スカラ:**
- `import com.example.project._` または `import com.example.project.{ClassA, ClassB}`
- SBT プロジェクト レイアウト: `src/test/scala/` ミラー `src/main/scala/`

**TypeScript/JavaScript:**
- `import { func } from '../src/module'` と相対パス
- `tsconfig.json` からのパス エイリアス (例: `@/module`)

**行く:**
- 同じパッケージ: `package mypackage` と同じディレクトリ内のテスト ファイル
- ブラックボックス テスト: `package mypackage_test` と明示的なインポート
- 内部パッケージには特定のインポート パスが必要な場合があります

**錆び:**
- `use crate::module::function;` 同じクレート内の単体テスト用
- `use myproject::module::function;` `tests/` での統合テスト用

既存のテストがどのようなパターンを使用していても、それを正確にコピーしてください。推測したり、別のパターンを考え出したりしないでください。

## テストを作成する前にテスト セットアップを作成する

すべてのテスト フレームワークには、共有セットアップのメカニズムがあります。テストで共有フィクスチャまたはテスト データを使用する場合は、テストを作成する前にセットアップ ファイルを作成する必要があります。テスト フレームワークは、他のディレクトリからフィクスチャを自動検出しません。

**言語別:**

**Python (pytest):** すべてのフィクスチャを定義する `quality/conftest.py` を作成します。 `tests/conftest.py` のフィクスチャは `quality/test_functional.py` では使用できません。推奨: `tmp_path` を使用してデータをインラインで作成するテストを作成し、conftest の依存関係を排除します。

**Java (JUnit):** テスト クラスで `@BeforeEach`/`@BeforeAll` メソッドを使用するか、同じパッケージ内に共有 `TestFixtures` ユーティリティ クラスを作成します。

**Scala (ScalaTest):** 特性を `before`/`after` ブロックと組み合わせるか、インライン データ ビルダーを使用します。 SBT を使用する場合は、テスト ファイルが正しいソース ツリーに存在することを確認してください。

**TypeScript (Jest):** テスト ファイルで `beforeAll`/`beforeEach` を使用するか、ファクトリ関数を使用して `quality/testUtils.ts` を作成します。

**Go (テスト):** `t.Helper()` と同​​じ `_test.go` ファイル内のヘルパー関数。一時ディレクトリには `t.TempDir()` を使用します。 Go の規約では、インライン セットアップが強く推奨されており、テスト状態の共有は避けられます。

**Rust (貨物テスト):** `#[cfg(test)] mod tests` ブロックまたは `test_utils.rs` モジュールのヘルパー関数。テスト データの構築にはビルダー パターンを使用します。統合テストの場合は、ファイルを `tests/` に配置します。**ルール: 参照されるすべてのフィクスチャまたはテスト ヘルパーを定義する必要があります。** テストが存在しない共有セットアップに依存している場合、テストはセットアップ中にエラーになります (アサーション中に失敗するわけではありません)。つまり、合格したように見える壊れたテストが生成されます。

**すべての言語で推奨されるアプローチ:** 独自のデータをインラインで作成するテストを作成します。これにより、ファイル間の依存関係が排除されます。「」パイソン
# パイソン
def test_config_validation(tmp_path):
    config = {"パイプライン": {"名前": "テスト", "ステップ": [...]}}
「」

```ジャワ
// Java
@テスト
void testConfigValidation(@TempDir パス tempDir) {
    var config = Map.of("パイプライン", Map.of("名前", "テスト"));
}
「」

```タイプスクリプト
// TypeScript
test('構成の検証', () => {
    const config = { パイプライン: { 名前: 'テスト'、ステップ: [] } };
});
「」

「行く」
// 行く
func TestConfigValidation(t *testing.T) {
    tmpDir := t.TempDir()
    config := Config{パイプライン: パイプライン{名前: "テスト"}}
}
「」

「錆びる」
// 錆びる
#[テスト]
fn test_config_validation() {
    let config = Config { パイプライン: パイプライン { 名前: "テスト".into() } };
}
「」**すべてのテストを作成した後、テスト スイートを実行してセットアップ エラーを確認します。** セットアップ エラー (フィクスチャが見つからない、インポートの失敗) は、フレームワークによる分類方法に関係なく、壊れたテストとしてカウントされます。

## プレースホルダー テストはありません

すべてのテストでは、実際のプロジェクト コードをインポートして呼び出す必要があります。テスト本体が `pass` である場合、またはその唯一のアサーションが `assert isinstance(errors, list)` である場合、または `assert hasattr(cls, 'validate')` のような簡単なプロパティをチェックする場合は、それを削除して実際のテストを作成するか、完全に削除します。プロジェクト コードを実行しないテストは、テストを行わないよりも悪く、カウントが膨らみ、誤った信頼性が生じます。

防御パターンに対して意味のあるテストを本当に作成できない場合 (たとえば、実行中のサーバーまたは外部サービスが必要な場合)、プレースホルダーを記述するのではなく、コメントでテスト不可能であることをメモしてください。

## 書く前にお読みください: 関数呼び出しマップ

単一のテストを作成する前に、関数呼び出しマップを作成します。テストする予定の関数ごとに次のことを行います。

1. **関数/メソッドのシグネチャを読みます** - 名前だけでなく、すべてのパラメータ、その型、デフォルト値も読みます。 Python では、`def` 行を読み、ヒントを入力します。 Java では、メソッド シグネチャとジェネリックを読み取ります。 Scala では、メソッド定義と暗黙的なパラメーターを読み取ります。 TypeScript で、型の注釈を読み取ります。
2. **ドキュメントを読んでください** - docstrings、Javadoc、TSDoc、ScalaDoc。多くの場合、戻り値の型、例外、およびエッジケースの動作が指定されます。
3. **それを呼び出す既存のテストを 1 つ読みます** — 既存のテストは、正確な呼び出し規約、フィクスチャの形状、およびアサーション パターンを示します。
4. **実際のデータ ファイルを読み取る** — 関数が構成、スキーマ、またはデータ ファイルを処理する場合は、プロジェクトから実際のファイルを読み取ります。テスト フィクスチャはこの形状と正確に一致する必要があります。

**一般的な失敗パターン:** エージェントはアーキテクチャを調査し、関数の動作を概念的に理解してから、推測されたパラメーターを使用してテスト呼び出しを作成します。実際の関数が `(items, seed, strategy)` ではなく `(config, items_data, limit)` を取るため、テストは失敗します。実際の署名の読み取りには 5 秒かかりますが、これは完全に防止されます。

**ライブラリのバージョンの認識:** プロジェクトの依存関係マニフェスト (`requirements.txt`、`build.sbt`、`package.json`、`pom.xml`、`build.gradle`、`Cargo.toml`) をチェックして、利用可能なものを確認してください。オプションの依存関係にはテスト フレームワークのスキップ メカニズムを使用します。Python `pytest.importorskip()`、JUnit `Assumptions.assumeTrue()`、ScalaTest `assume()`、Jest 条件付き `describe.skip`、Go `t.Skip()`、Rust `#[ignore]` を前提条件を説明するコメントとともに使用します。

## 仕様から派生したテストの作成

各仕様ドキュメントをセクションごとに説明します。各セクションについて、「これにはどのようなテスト可能な要件が記載されていますか?」と尋ねます。次にテストを書きます。

各テストでは次のことを行う必要があります。
1. **セットアップ** — フィクスチャをロードし、テスト データを作成し、システムを構成します
2. **実行** — 関数を呼び出し、パイプラインを実行し、リクエストを実行します。
3. 仕様に必要な **特定のプロパティをアサート**「」パイソン
# Python (pytest)
クラス TestSpecRequirements:
    def test_requirement_from_spec_section_N(self, fixture):
        """[要件: 正式な設計文書 §N] X は Y を生成する必要があります。"""
        結果 = プロセス(フィクスチャ)
        アサート result.property == Expected_value
「」

```ジャワ
// Java (JUnit 5)
クラス SpecRequirementsTest {
    @テスト
    @DisplayName("[要件: 正式な設計文書 §N] X は Y を生成する必要があります")
    void testRequirementFromSpecSectionN() {
        var result = プロセス(フィクスチャ);
        assertEquals(expectedValue, result.getProperty());
    }
}
「」

「スカラ」
// スカラ (ScalaTest)
class SpecRequirements は Matchers を使用して FlatSpec を拡張します {
  // [要求: 正式 — 設計文書 §N] X は Y を生成する必要があります
  「セクション N の要件」は、{ で「X から Y を生成」する必要があります。
    val 結果 = プロセス(フィクスチャ)
    result.property は (expectedValue) と等しくなる必要があります
  }
}
「」

```タイプスクリプト
// TypeScript (Jest)
description('仕様要件', () => {
  test('[Req: 正式な設計文書 §N] X は Y を生成する必要があります', () => {
    const result = プロセス(フィクスチャ);
    Expect(result.property).toBe(expectedValue);
  });
});
「」

「行く」
// 実行（テスト）
func TestSpecRequirement_SectionN_XProducesY(t *testing.T) {
    // [要求: 正式 — 設計文書 §N] X は Y を生成する必要があります
    結果 := プロセス(フィクスチャ)
    if result.Property != ExpectValue {
        t.Errorf("expected %v, got %v", ExpectedValue, result.Property)
    }
}
「」

「錆びる」
// Rust (貨物テスト)
#[テスト]
fn test_spec_requirement_section_n_x_Produces_y() {
    // [要求: 正式 — 設計文書 §N] X は Y を生成する必要があります
    let result = process(&fixture);
    assert_eq!(result.property, Expected_value);
}
「」## 優れた機能テストの条件

- **追跡可能** — テスト名、表示名、またはドキュメントのコメントは、どの仕様要件を検証するかを示します。
- **特定** — 「何かが起こった」だけではなく、特定のプロパティをチェックします。
- **堅牢** — 合成データではなく、実際のデータ (実際のシステムのフィクスチャ) を使用します。
- **クロスバリアント** — プロジェクトが複数の入力タイプを処理する場合は、それらすべてをテストします
- **適切なレイヤーでのテスト** — 関心のある *動作* をテストします。 「無効なデータが間違った出力を生成しない」という要件がある場合は、スキーマ検証ツールが入力を拒否することをテストするだけではなく、パイプライン出力をテストします。

## クロスバリアント テスト戦略

プロジェクトが複数の入力タイプを処理する場合、クロスバリアント カバレッジにはサイレント バグが隠れています。すべてのバリアントを実行するテストの約 30% を目指します。正確な割合は、すべてのバリアントにわたって横断的なプロパティを確実にテストすることよりも重要です。

フレームワークのパラメータ化メカニズムを使用します。「」パイソン
# Python (pytest)
@pytest.mark.parametrize("バリアント", [バリアント_a, バリアント_b, バリアント_c])
def test_feature_works(バリアント):
    出力 = プロセス(バリアント.入力)
    Output.has_expected_property をアサートする
「」

```ジャワ
// Java (JUnit 5)
@ParameterizedTest
@MethodSource("variantProvider")
void testFeatureWorks(Variant バリアント) {
    var 出力 = プロセス(variant.getInput());
    assertTrue(output.hasExpectedProperty());
}
「」

「スカラ」
// スカラ (ScalaTest)
Seq(バリアント A, バリアント B, バリアント C).foreach { バリアント =>
  { では「${variant.name} で動作する」はずです
    val 出力 = プロセス(variant.input)
    出力には ('expectedProperty (true)) が含まれている必要があります
  }
}
「」

```タイプスクリプト
// TypeScript (Jest)
test.each([バリアントA, バリアントB, バリアントC])(
  'この機能は %s で動作します', (バリアント) => {
    const 出力 = プロセス(variant.input);
    Expect(output).toHaveProperty('expectedProperty');
});
「」

「行く」
// Go (テスト) — テーブル駆動テスト
func TestFeatureWorksAcrossVariants(t *testing.T) {
    バリアント := []バリアント{バリアント A、バリアント B、バリアント C}
    for _, v := 範囲のバリアント {
        t.Run(v.Name, func(t *testing.T) {
            出力 := プロセス(v.入力)
            if !output.HasExpectedProperty() {
                t.Errorf("バリアント %s: 予期されたプロパティがありません", v.Name)
            }
        })
    }
}
「」

「錆びる」
// Rust (カーゴテスト) — ケースを反復処理します
#[テスト]
fn test_feature_works_across_variants() {
    バリアント = [variant_a()、variant_b()、variant_c()]; にします。
    for v in &variants {
        let 出力 = process(&v.input);
        アサート!(output.has_expected_property(),
            "バリアント {}: 予期されたプロパティがありません", v.name);
    }
}
「」パラメータ化が適合しない場合は、単一のテスト内で明示的にループします。

**どのテストがクロスバリアントである必要がありますか?** エンティティ ID、構造プロパティ、必要なリンク、時間フィールド、ドメイン固有のセマンティクスなど、入力タイプに関係なく保持 * すべき* プロパティを検証するテスト。

**すべてのテストを作成した後、クロスバリアント監査を実行します。** クロスバリアント テストの数を合計で割ります。 30% 未満の場合は、さらに変換します。

## 避けるべきアンチパターン

これらのパターンはテストのように見えますが、実際のバグは検出されません。

- **存在のみのチェック** — 1 つの正しい結果が見つかっても、すべてが正しいとは限りません。また、カウントをチェックしたり、総合的に検証したりします。
- **存在のみのアサーション** — 値が存在することをアサートすることは、正確性ではなく存在を証明するだけです。実際の値をアサートします。
- **単一バリアント テスト** — 1 つの入力タイプをテストし、他の入力タイプが機能することを期待します。パラメータ化を使用します。
- **ポジティブのみのテスト** — 無効な入力によって不正な出力が生成されないことをテストする必要があります。
- **不完全な否定的なアサーション** — 拒否をテストするときは、1 つだけではなく、すべての結果が存在しないことをアサートします。
- **出力をチェックする代わりに例外をキャッチする** — コードが特定の方法でクラッシュすることをテストすることは、入力が正しく処理されることをテストすることにはなりません。出力をテストします。

### 例外キャッチのアンチパターンの詳細```ジャワ
// Java — 誤り: 検証メカニズムをテストします
@テスト
void testBadValueRejected() {
    fixture.setField("無効");  // スキーマはこれを拒否します。
    assertThrows(ValidationException.class, () -> process(fixture));
    // 出力については何も伝えません
}

// Java — 右: 要件をテストします
@テスト
void testBadValueNotInOutput() {
    fixture.setField(null);  // スキーマはオプションとして null を受け入れます
    var 出力 = プロセス (フィクスチャ);
    assertFalse(output.contains(badProperty));  // 不正なデータが存在しない
    assertTrue(output.contains(expectedType));   // 残りはまだ機能します
}
「」

「スカラ」
// Scala — 誤り: 要件ではなくデコーダをテストします
「不正な値」は { で「拒否」する必要があります
  val input = fixture.copy(field = "invalid") // キルケ デコーダが失敗します。
  プロセス(入力)によって[DecodingFailure]がスローされる必要があります
  // 出力については何も伝えません
}

// Scala — 右: 要件をテストします
{ で「オプションのフィールドが欠落しています」と「不正な出力が生成されることはない」はずです。
  val input = fixture.copy(field = None) // Option[String] は None を受け入れます
  val 出力 = プロセス (入力)
  出力には badProperty を含めないでください // 不正なデータが存在しません
  出力には (expectedType) が含まれている必要があります // Rest は引き続き機能します
}
「」

```タイプスクリプト
// TypeScript — 誤り: 検証メカニズムをテストします
test('不正な値が拒否されました', () => {
    fixture.field = '無効';  // Zod スキーマはこれを拒否します。
    Expect(() => プロセス(フィクスチャ)).toThrow(ZodError);
    // 出力については何も伝えません
});

// TypeScript — 右: 要件をテストします
test('出力に不正な値がありません', () => {
    fixture.field = 未定義;  // スキーマはオプションとして未定義を受け入れます
    const 出力 = プロセス (フィクスチャ);
    Expect(output).not.toContain(badProperty);  // 不正なデータが存在しない
    Expect(出力).toContain(expectedType);      // 残りはまだ機能します
});
「」

「」パイソン
# Python — 誤り: 検証メカニズムをテストします
def test_bad_value_rejected(フィクスチャ):
    fixture.field = "invalid" # スキーマはこれを拒否します。
    pytest.raises(ValidationError) を使用:
        プロセス（治具）
    # 出力については何も説明しません

# Python — 右: 要件をテストします
def test_bad_value_not_in_output(フィクスチャ):
    fixture.field = None # スキーマはオプションとして None を受け入れます
    出力 = プロセス(フィクスチャ)
    assert field_property が出力にありません # 不正なデータが存在しません
    出力で Expected_type をアサート # Rest は引き続き動作します
「」

「行く」
// Go - 間違っています: 結果ではなくエラーをテストします
func TestBadValueRejected(t *testing.T) {
    fixture.Field = "invalid" // バリデーターはこれを拒否します。
    _, err := プロセス(フィクスチャ)
    if err == nil { t.Fatal("予期されたエラー") }
    // 出力については何も伝えません
}

// Go — 右: 要件をテストします
func TestBadValueNotInOutput(t *testing.T) {
    fixture.Field = "" // ゼロ値は有効です
    出力、エラー := プロセス(フィクスチャ)
    if err != nil { t.Fatalf("予期しないエラー: %v", err) }
    if containsBadProperty(output) { t.Error("不正なデータは存在しないはずです") }
    if !containsExpectedType(output) { t.Error("予期されるデータが存在するはずです") }
}
「」

「錆びる」
// Rust — 誤り: 結果ではなくエラーをテストします
#[テスト]
fn test_bad_value_rejected() {
    let input = Fixture { フィールド: "invalid".into(), ..default() };
    アサート!(プロセス(&入力).is_err());  // 出力については何も伝えません
}

// Rust — 右: 要件をテストします
#[テスト]
fn test_bad_value_not_in_output() {
    let input = Fixture { フィールド: なし、..default() };  // オプションは None を受け入れます
    let Output = process(&input).expect("成功するはずです");
    アサート!(!output.contains(bad_property));  // 不正なデータが存在しない
    アサート!(output.contains(expected_type));   // 残りはまだ機能します
}
「」突然変異値を選択する前に、必ずステップ 5b のスキーマ マップを確認してください。

## 適切なレイヤーでのテスト

「*仕様* では何が起こるべきだと書かれていますか?」と尋ねます。仕様には、「無効なデータは出力に表示されるべきではない」と記載されており、「検証層がそれを拒否すべき」ではありません。実装ではなく仕様をテストします。

**例外:** 仕様で特定のメカニズムが明示的に義務付けられている場合 (たとえば、「スキーマ層でフェイルファストする必要がある」など)、そのメカニズムをテストするのが適切です。しかし、これはまれです。

## 目的に合ったシナリオのテスト

QUALITY.md のシナリオごとにテストを作成します。これは 1:1 マッピングです。「スカラ」
// スカラ (ScalaTest)
class FitnessScenarios は Matchers を使用して FlatSpec を拡張します {
  // [要件: 正式 — QUALITY.md シナリオ 1]
  「シナリオ 1: [名前]」は、{ で「[障害モード] を防ぐ」必要があります。
    val 結果 = プロセス(フィクスチャ)
    result.property は (expectedValue) と等しくなる必要があります
  }
}
「」

「」パイソン
# Python (pytest)
クラス TestFitnessScenarios:
    """QUALITY.md の目的適合性シナリオのテスト"""

    def test_scenario_1_memorable_name(self, fixture):
        """[要求: 正式 — QUALITY.md シナリオ 1] [名前]。
        要件: [コードが実行する必要があること]。
        「」
        結果 = プロセス(フィクスチャ)
        失敗を防ぐ条件をアサートします
「」

```ジャワ
// Java (JUnit 5)
クラス FitnessScenariosTest {
    @テスト
    @DisplayName("[要求: 正式 — QUALITY.md シナリオ 1] [名前]")
    void testScenario1MemorableName() {
        var result = プロセス(フィクスチャ);
        assertTrue(conditionThatPreventsFailure(result));
    }
}
「」

```タイプスクリプト
// TypeScript (Jest)
description('フィットネス シナリオ', () => {
  test('[要求: 正式 — QUALITY.md シナリオ 1] [名前]', () => {
    const result = プロセス(フィクスチャ);
    Expect(conditionThatPreventsFailure(result)).toBe(true);
  });
});
「」

「行く」
// 実行（テスト）
func TestScenario1_MemorableName(t *testing.T) {
    // [要求: 正式 — QUALITY.md シナリオ 1] [名前]
    // 要件: [コードが実行する必要があること]
    結果 := プロセス(フィクスチャ)
    if !conditionThatPreventsFailure(result) {
        t.Error("シナリオ 1 が失敗しました: [予期される動作について説明]")
    }
}
「」

「錆びる」
// Rust (貨物テスト)
#[テスト]
fn test_scenario_1_memorable_name() {
    // [要求: 正式 — QUALITY.md シナリオ 1] [名前]
    // 要件: [コードが実行する必要があること]
    let result = process(&fixture);
    アサート!(失敗を防ぐ条件(&結果));
}
「」## 境界テストと否定テスト

ステップ 5 の防御パターンごとに 1 つのテスト:```タイプスクリプト
// TypeScript (Jest)
description('境界とエッジケース', () => {
  test('[Req: 推論 — functionName() ガードから] X をガードします', () => {
    const input = { ...validFixture、フィールド: null };
    const 結果 = プロセス (入力);
    Expect(result).not.toContainBadOutput();
  });
});
「」

「」パイソン
# Python (pytest)
クラス TestBoundariesAndEdgeCases:
    """境界条件、不正な入力、エラー処理をテストします。"""

    def test_defensive_pattern_name(self, fixture):
        """[要求: function_name() ガードから推論] は X を防ぎます。"""
        # 変異して防御コードパスをトリガーする
        # 適切な処理をアサートします
「」

```ジャワ
// Java (JUnit 5)
クラス BoundariesAndEdgeCasesTest {
    @テスト
    @DisplayName("[要求: メソッド名() ガードから推論] X をガードします")
    void testDefensivePatternName() {
        fixture.setField(null);  // 防御コードパスをトリガーする
        var result = プロセス(フィクスチャ);
        アサートノットヌル(結果);  // 適切な処理をアサートします
        assertFalse(result.containsBadData());
    }
}
「」

「スカラ」
// スカラ (ScalaTest)
class BoundariesAndEdgeCases は Matchers を使用して FlatSpec を拡張します {
  // [Req: 推論 — methodName() ガードから]
  「防御パターン:methodName()」は、{ で「X に対して防御」する必要があります。
    val input = fixture.copy(field = None) // 防御コードパスをトリガーします
    val 結果 = プロセス(入力)
    結果は（定義済み）と等しくなる必要があります
    result.get には badData を含めないでください
  }
}
「」

「行く」
// 実行（テスト）
func TestDefensivePattern_FunctionName_GuardsAgainstX(t *testing.T) {
    // [Req: 推論 — FunctionName() ガードから] X をガードします
    入力:=defaultFixture()
    input.Field = nil // 防御コードパスをトリガーします
    結果、エラー := プロセス(入力)
    エラーの場合 != nil {
        t.Fatalf("予期された正常な処理、取得: %v"、エラー)
    }
    // エッジケース入力にもかかわらず結果が有効であることをアサートします
}
「」

「錆びる」
// Rust (貨物テスト)
#[テスト]
fn test_defensive_pattern_function_name_guards_against_x() {
    // [Req: 推論 — function_name() ガードから] X をガードします
    let input = Fixture { フィールド: なし、..default_fixture() };
    let result = process(&input).expect("期待される正常な処理");
    // エッジケース入力にもかかわらず結果が有効であることをアサートします
}
「」突然変異値を選択するときは、ステップ 5b のスキーマ マップを使用します。すべての変更では、スキーマが受け入れる値を使用する必要があります。

体系的なアプローチ:
- **フィールドがありません** — オプションのフィールドがありませんか? null に設定します。
- **タイプが間違っています** — フィールドのタイプが異なりますか?スキーマが有効な代替手段を使用してください。
- **空の値** — 空のリスト?空の文字列?空の辞書?
- **境界値** — ゼロ、負、最大、最初、最後。
- **モジュール間の境界** — モジュール A は異常だが有効な出力を生成します — B はそれを処理しますか?

10 個以上の防御パターンを見つけたものの、境界テストを 4 つしか書いていない場合は、戻ってさらに書いてください。 1:1 の比率を目標にします。