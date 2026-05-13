---
description: 'GitHub Copilot がコメントを少なくしつつ、自己説明的なコードを実現するためのコメントを書くためのガイドラインです。例は JavaScript ですが、コメントがある任意の言語で機能するはずです。'
applyTo: '**'
---

# 自己説明的なコードのためのコメント指示

## 中核原則
**コード自体が語るように書くこと。コメントは WHAT ではなく WHY を説明する必要があるときだけ書くこと。**
ほとんどの場合、コメントは不要です。

## コメントガイドライン

### ❌ こうしたコメントは避ける

**明白なコメント**
```javascript
// 悪い例: 明らかなことを述べている
let counter = 0;  // カウンターを 0 で初期化
counter++;  // カウンターを 1 増やす
```

**冗長なコメント**
```javascript
// 悪い例: コメントがコードを繰り返している
function getUserName() {
    return user.name;  // ユーザー名を返す
}
```

**古くなったコメント**
```javascript
// 悪い例: コメントがコードと一致していない
// 税率 5% で税額を計算
const tax = price * 0.08;  // 実際は 8%
```

### ✅ こうしたコメントを書く

**複雑な業務ロジック**
```javascript
// 良い例: なぜこの計算を行うのかを説明している
// 累進課税区分を適用: 10k までは 10%、それを超える分は 20%
const tax = calculateProgressiveTax(income, [0.10, 0.20], [10000]);
```

**一見してわかりにくいアルゴリズム**
```javascript
// 良い例: アルゴリズム選択の理由を説明している
// すべてのノード間距離が必要なので
// 全点対最短経路に Floyd-Warshall を使用
for (let k = 0; k < vertices; k++) {
    for (let i = 0; i < vertices; i++) {
        for (let j = 0; j < vertices; j++) {
            // ... 実装
        }
    }
}
```

**正規表現パターン**
```javascript
// 良い例: その regex が何に一致するかを説明している
// email 形式に一致: username@domain.extension
const emailPattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
```

**API の制約や落とし穴**
```javascript
// 良い例: 外部制約を説明している
// GitHub API のレート制限: 認証済みユーザーは 1 時間あたり 5000 リクエスト
await rateLimiter.wait();
const response = await fetch(githubApiUrl);
```

## 判断フレームワーク

コメントを書く前に、次を確認する:
1. **コード自体で意図が伝わるか?** → コメント不要
2. **より良い変数名 / 関数名で不要にできるか?** → まずリファクタリング
3. **WHAT ではなく WHY を説明しているか?** → 良いコメント
4. **将来の保守担当者の助けになるか?** → 良いコメント

## コメントの特殊ケース

### 公開 API
```javascript
/**
 * 標準的な式を使って複利を計算する。
 *
 * @param {number} principal - 初期投資額
 * @param {number} rate - 年利率 (小数表記。例: 5% なら 0.05)
 * @param {number} time - 年単位の期間
 * @param {number} compoundFrequency - 1 年あたりの複利計算回数 (既定値: 1)
 * @returns {number} 複利計算後の最終金額
 */
function calculateCompoundInterest(principal, rate, time, compoundFrequency = 1) {
    // ... 実装
}
```

### 設定と定数
```javascript
// 良い例: 出典や理由を説明している
const MAX_RETRIES = 3;  // ネットワーク信頼性の調査結果に基づく
const API_TIMEOUT = 5000;  // AWS Lambda のタイムアウトは 15s。余裕を残す
```

### 注釈
```javascript
// TODO: セキュリティーレビュー後に適切なユーザー認証へ置き換える
// FIXME: 本番環境でメモリーリークが発生 - connection pooling を調査する
// HACK: library v2.1.0 のバグに対する回避策 - upgrade 後に削除する
// NOTE: この実装はすべての計算で UTC timezone を前提とする
// WARNING: この関数はコピーを作らず、元の配列を変更する
// PERF: hot path で頻繁に呼ばれるなら、この結果のキャッシュを検討する
// SECURITY: query で使う前に入力を検証して SQL injection を防ぐ
// BUG: 配列が空のときのエッジケースで失敗する - 要調査
// REFACTOR: 再利用性のため、このロジックを別の utility function へ抽出する
// DEPRECATED: 代わりに newApiFunction() を使う - これは v3.0 で削除予定
```

## 避けるべきアンチパターン

### デッドコードコメント
```javascript
// 悪い例: コードをコメントアウトして残さない
// const oldFunction = () => { ... };
const newFunction = () => { ... };
```

### 変更履歴コメント
```javascript
// 悪い例: コメントで履歴管理しない
// John が 2023-01-15 に変更
// Sarah が 2023-02-03 に報告したバグを修正
function processData() {
    // ... 実装
}
```

### 区切りコメント
```javascript
// 悪い例: 装飾目的のコメントを使わない
//=====================================
// UTILITY FUNCTIONS
//=====================================
```

## 品質チェックリスト

コミット前に、コメントが次を満たしていることを確認する:
- [ ] WHAT ではなく WHY を説明している
- [ ] 文法的に正しく明確である
- [ ] コードが進化しても正確さを保てる
- [ ] コード理解に本当に価値を加えている
- [ ] 適切な位置 (説明対象コードの上) に置かれている
- [ ] 正しい綴りとプロフェッショナルな言葉遣いを使っている

## まとめ

覚えておくこと: **最良のコメントは、コードが自己文書化されているため書く必要のないコメントである。**
