# Apollo クライアント - React 18 互換性の詳細

## Apollo 3.8 以降が必要な理由

Apollo Client 3.7 以前では、React 18 の同時レンダリングと互換性のない内部サブスクリプション モデルが使用されています。同時モードでは、React がレンダリングを中断して再実行する可能性があるため、Apollo のストア サブスクリプションが誤ったタイミングで起動され、古いデータや更新の欠落が発生します。

Apollo 3.8 は、`useSyncExternalStore` を採用した最初のバージョンでした。これは、React 18 が同時レンダリングで外部ストアを正しく動作させるために必要です。

## バージョンの概要

|アポロバージョン | React 18 サポート | React 19 サポート |メモ |
|---|---|---|---|
| < 3.7 | ❌ | ❌ |同時モード データ ティアリング |
| 3.7.x | ⚠️ | ⚠️ |従来のルートのみで動作します (ReactDOM.render) |
| **3.8.x** | ✅ | ✅ |最初の完全互換バージョン |
| 3.9+ | ✅ | ✅ |おすすめ |
| 3.11+ | ✅ | ✅ (確認済み) |明示的な React 19 テストの追加 |

## レガシー ルートを使用して Apollo 3.7 を使用している場合

アプリがまだ `ReactDOM.render` (レガシー ルート) を使用しており、まだ `createRoot` に移行していない場合、Apollo 3.7 は技術的には動作しますが、これは React 18 の同時機能 (自動バッチ処理を含む) を利用できないことを意味します。これは部分的なアップグレードのみです。

`createRoot` を使用したらすぐに、Apollo を 3.8 以降にアップグレードしてください。

## テストの MockedProvider - React 18

Apollo の `MockedProvider` は React 18 で動作しますが、非同期の動作が変更されました。```jsx
// 古いパターン - setTimeout でフラッシュ:
await 新しい Promise(resolve => setTimeout(resolve, 0));
ラッパー.update();

// React 18 パターン - waitFor または findBy を使用します:
await waitFor(() => {
  Expect(screen.getByText('Alice')).toBeInTheDocument();
});
// または:
Expect(await screen.findByText('Alice')).toBeInTheDocument();
「」## Apollo のアップグレード「」バッシュ
npm install @apollo/client@latestgraphql@latest
「」graphql ピア dep が他のパッケージと競合する場合:「」バッシュ
npm lsgraphql # 使用されているバージョンを確認する
npm info @apollo/clientpeerDependency # apollo が必要とするものを確認する
「」Apollo 3.8 以降は、`graphql@15` と `graphql@16` の両方をサポートします。

## InMemoryCache - 変更は必要ありません

`InMemoryCache` 設定は React 18 アップグレードの影響を受けません。以下の場合は移行は必要ありません。

- @@コード3@@
- @@コード4@@
- `possibleTypes`
- カスタムフィールドポリシー

## useQuery / useMutation / useSubscription - 変更なし

Apollo フックの API は変更されていません。このアップグレードは、Apollo が React のレンダリング モデルと統合される方法の完全に内部的なものです。