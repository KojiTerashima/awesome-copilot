# スキーマタイプのマッピング (ステップ 5b)

プロジェクトにスキーマ検証レイヤーがある場合は、境界テストを作成する前に、各フィールドが何を受け入れるかを理解する必要があります。言語別の一般的な検証レイヤー: Pydantic モデル (Python)、JSON スキーマ (任意)、TypeScript インターフェイス/Zod スキーマ (TypeScript)、Bean Validation アノテーション (Java)、ケース クラス コーデック/Circe デコーダー (Scala)、serde 属性 (Rust)。このマッピングがないと、テスト対象のコードに到達する前にスキーマが拒否する変更を作成することになり、意味のある境界テストではなく検証エラーが発生します。

## なぜこれが重要なのか

次のよくある間違いについて考えてみましょう。```タイプスクリプト
// TypeScript — 誤り: 要件ではなく、検証メカニズムをテストします
test('不正な値が拒否されました', () => {
    fixture.field = '無効';  // Zod スキーマは処理前にこれを拒否します。
    Expect(() => プロセス(フィクスチャ)).toThrow(ZodError);
    // 出力については何も伝えません
});

// TypeScript — 右: スキーマ有効な突然変異を使用して要件をテストします
test('出力に不正な値がありません', () => {
    fixture.field = 未定義;  // スキーマはオプションのフィールドに未定義を受け入れます
    const 出力 = プロセス (フィクスチャ);
    Expect(output).not.toContain(badProperty);  // 不正なデータが存在しない
    Expect(出力).toContain(expectedType);      // 残りはまだ機能します
});
「」

「」パイソン
# Python — 誤り: 要件ではなく検証メカニズムをテストします
def test_bad_value_rejected(フィクスチャ):
    fixture.field = "invalid" # Pydantic は処理前にこれを拒否します。
    pytest.raises(ValidationError) を使用:
        プロセス（治具）
    # 出力については何も伝えません

# Python — 右: スキーマ有効な突然変異を使用して要件をテストします
def test_bad_value_not_in_output(フィクスチャ):
    fixture.field = None # スキーマはオプションのフィールドとして None を受け入れます
    出力 = プロセス(フィクスチャ)
    assert field_property が出力にありません # 不正なデータが存在しません
    出力で Expected_type をアサート # Rest は引き続き動作します
「」

```ジャワ
// Java — 誤り: Bean Validation をテストしますが、要件ではありません
@テスト
void testBadValueRejected() {
    fixture.setField("無効");  // @NotNull/@Pattern はこれを拒否します。
    assertThrows(ConstraintViolationException.class, () -> process(fixture));
}

// Java — 右: スキーマ有効な突然変異を使用して要件をテストします
@テスト
void testBadValueNotInOutput() {
    fixture.setField(null);  // null 可能 String フィールドは null を受け入れます
    var 出力 = プロセス (フィクスチャ);
    assertFalse(output.contains(badProperty));
    assertTrue(output.contains(expectedType));
}
「」

「スカラ」
// Scala — 誤り: 要件ではなくデコーダをテストします
「不正な値」は { で「拒否」する必要があります
    val input = fixture.copy(field = "invalid") // キルケ デコーダが失敗します。
    プロセス(入力)によって[DecodingFailure]がスローされる必要があります
}

// Scala — 右: スキーマ有効な突然変異を使用して要件をテストします
{ で「オプションのフィールドが欠落しています」と「不正な出力が生成されることはない」はずです。
    val input = fixture.copy(field = None) // Option[String] は None を受け入れます
    val 出力 = プロセス (入力)
    出力には badProperty が含まれないようにする必要があります
}
「」

「行く」
// Go — 誤り: 要件ではなく検証をテストします
func TestBadValueRejected(t *testing.T) {
    fixture.Field = "invalid" // 構造体タグバリデーターはこれを拒否します。
    _, err := プロセス(フィクスチャ)
    if err == nil { t.Fatal("予期される検証エラー") }
    // 出力については何も伝えません
}

// Go — 右: 有効なゼロ値を使用して要件をテストします
func TestBadValueNotInOutput(t *testing.T) {
    fixture.Field = "" // オプションの文字列フィールドにはゼロ値が有効です
    出力、エラー := プロセス(フィクスチャ)
    if err != nil { t.Fatalf("予期しないエラー: %v", err) }
    // 不良データが存在しないことをアサートしますが、残りはまだ機能します
}
「」

「錆びる」
// Rust — 誤り: 要件ではなく、serde 逆シリアル化をテストします
#[テスト]
fn test_bad_value_rejected() {
    let input = Fixture { フィールド: "invalid".into(), ..default() };
    // serde は処理前に拒否します。
    アサート!(プロセス(&入力).is_err());
}

// Rust — 右: スキーマ有効な突然変異を使用して要件をテストします
#[テスト]
fn test_bad_value_not_in_output() {
    let input = Fixture { フィールド: なし、..default() };  // Option<String> は None を受け入れます
    let Output = process(&input).expect("成功するはずです");
    アサート!(!output.contains(bad_property));
    アサート!(output.contains(expected_type));
}
「」突然変異値がスキーマ有効ではないため、WRONG テストは検証/デコード エラーで失敗します。 RIGHT テストでは、スキーマが受け入れる値 (null、None、nil、ゼロ値、空のオプション) を使用するため、突然変異が実際の処理ロジックに到達します。

## マップの構築方法

ステップ 5 で見つけた守備パターンごとに、次のことを記録します。

|フィールド |スキーマタイプ |受け入れる |拒否 |
|----------|-----------|----------|----------|
| `metadata` |オプションのオブジェクト (`Optional[MetadataObject]` / `MetadataObject?` / `MetadataObject \| null`) |有効なオブジェクト、`null`/`undefined` | `string`、`number`、`array` |
| `count_field` |オプションの整数 (`Optional[int]` / `number?` / `Integer`) |整数、`null` | `string`、`object` |
| `child_list` |オブジェクトの配列 (`List[Child]` / `Child[]` / `Seq[Child]`) |オブジェクトの配列、`[]` | `[null, "invalid"]`、`null` |
| `optional_object` |オプションのオブジェクト | `{"key": value}`、`null` | `"bad"`、`[1,2]` |

## 突然変異値を選択するためのルール

境界テストを作成するときは、常に「Accepts」列の値を使用してください。慣用的な「欠落/空」値は言語によって異なります。

- **オプション/NULL 可能フィールド:** Python `None`、Java `null`、Scala `None` (`Option` の場合)、TypeScript `undefined`/`null`、Go ゼロ値 (`""`、`0`、`nil` ポインターの場合)、Rust `None` (`Option<T>` の場合)
- **数値フィールド:** `0`、負の値、または境界値 - 言語に依存しない
- **配列/リスト:** Python `[]`、Java `List.of()`、Scala `Seq.empty`、TypeScript `[]`、Go `nil` または空のスライス、Rust `Vec::new()`
- **文字列:** `""` (空の文字列) — 言語に依存しない
- **オブジェクト/構造体:** Python `{}`、フィールドが欠落している Java `new Obj()`、Scala `copy()` と `None`、TypeScript `{}`、Go ゼロ値構造体、Rust `Default::default()` またはフィールドが欠落しているビルダー

「拒否」列の値は決して使用しないでください。これらの値は、ビジネス ロジックではなく、スキーマ バリデーターをテストします。

## このステップをスキップする場合

プロジェクトにスキーマ検証レイヤーがない場合 (データは型チェックなしで処理に直接流れます)、マッピングをスキップして、任意の突然変異値を使用できます。ただし、最新のプロジェクトのほとんどには何らかの形で検証が行われているため、最初に確認してください。