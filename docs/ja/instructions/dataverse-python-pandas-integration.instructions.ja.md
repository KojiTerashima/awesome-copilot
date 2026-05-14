# Dataverse SDK for Python - pandas 連携ガイド

## 概要
Dataverse SDK for Python を pandas DataFrame と統合して、データ サイエンスや分析ワークフローで使うためのガイドです。SDK の JSON response 形式は pandas DataFrame に自然に対応しており、データ サイエンティストは慣れたデータ操作ツールで Dataverse データを扱えます。

---

## 1. PandasODataClient の導入

### PandasODataClient とは?
`PandasODataClient` は標準の `DataverseClient` を薄く包む wrapper で、raw JSON dictionary の代わりに pandas DataFrame 形式でデータを返します。これにより、次の用途に適します。
- 表形式データを扱うデータ サイエンティスト
- 分析およびレポート ワークフロー
- データ探索とクリーニング
- 機械学習パイプラインとの統合

### インストール要件
```bash
# core dependency を install
pip install PowerPlatform-Dataverse-Client
pip install azure-identity

# データ操作用に pandas を install
pip install pandas
```

### PandasODataClient を使うべき場面
✅ **次が必要な場合に使う:**
- データ探索と分析
- 表形式データの処理
- 統計/ML library との統合
- 効率的なデータ操作

❌ **次が必要なら DataverseClient を使う:**
- リアルタイム CRUD operation のみ
- ファイル upload 操作
- metadata 操作
- 単一レコード操作

---

## 2. 基本的な DataFrame ワークフロー

### Query 結果を DataFrame に変換する
```python
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient
import pandas as pd

# 認証設定
base_url = "https://<myorg>.crm.dynamics.com"
credential = InteractiveBrowserCredential()
client = DataverseClient(base_url=base_url, credential=credential)

# データ照会
pages = client.get(
    "account",
    select=["accountid", "name", "creditlimit", "telephone1"],
    filter="statecode eq 0",
    orderby=["name"]
)

# 全ページを 1 つの DataFrame に集約
all_records = []
for page in pages:
    all_records.extend(page)

# DataFrame に変換
df = pd.DataFrame(all_records)

# 先頭数行を表示
print(df.head())
print(f"Total records: {len(df)}")
```

### Query パラメーターは DataFrame に対応する
```python
# すべての query parameter が DataFrame の column として返る
df = pd.DataFrame(
    client.get(
        "account",
        select=["accountid", "name", "creditlimit", "telephone1", "createdon"],
        filter="creditlimit > 50000",
        orderby=["creditlimit desc"]
    )
)

# 結果は次の column を持つ DataFrame:
# accountid | name | creditlimit | telephone1 | createdon
```

---

## 3. pandas を使ったデータ探索

### 基本的な探索
```python
import pandas as pd
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

client = DataverseClient("https://<myorg>.crm.dynamics.com", InteractiveBrowserCredential())

# account データを読み込む
records = []
for page in client.get("account", select=["accountid", "name", "creditlimit", "industrycode"]):
    records.extend(page)

df = pd.DataFrame(records)

# データを探索
print(df.shape)           # (1000, 4)
print(df.dtypes)          # データ型
print(df.describe())      # 統計要約
print(df.info())          # column 情報と null 件数
print(df.head(10))        # 最初の 10 行
```

### Filter と Select
```python
# 条件で行を絞る
high_value = df[df['creditlimit'] > 100000]

# 特定 column を選ぶ
names_limits = df[['name', 'creditlimit']]

# 複数条件
filtered = df[(df['creditlimit'] > 50000) & (df['industrycode'] == 1)]

# 値の出現回数
print(df['industrycode'].value_counts())
```

### Sort と Grouping
```python
# column で並び替え
sorted_df = df.sort_values('creditlimit', ascending=False)

# group 化して集計
by_industry = df.groupby('industrycode').agg({
    'creditlimit': ['mean', 'sum', 'count'],
    'name': 'count'
})

# group 統計
print(df.groupby('industrycode')['creditlimit'].describe())
```

### データ クリーニング
```python
# 欠損値を処理
df_clean = df.dropna()                    # NaN を含む行を削除
df_filled = df.fillna(0)                  # NaN を 0 で埋める
df_ffill = df.fillna(method='ffill')      # 前方補完

# 重複確認
duplicates = df[df.duplicated(['name'])]
df_unique = df.drop_duplicates()

# データ型変換
df['creditlimit'] = pd.to_numeric(df['creditlimit'])
df['createdon'] = pd.to_datetime(df['createdon'])
```

---

## 4. データ分析パターン

### 集計と要約
```python
# 要約レポートを作成
summary = df.groupby('industrycode').agg({
    'accountid': 'count',
    'creditlimit': ['mean', 'min', 'max', 'sum'],
    'name': lambda x: ', '.join(x.head(3))  # サンプル名
}).round(2)

print(summary)
```

### 時系列分析
```python
# datetime に変換
df['createdon'] = pd.to_datetime(df['createdon'])

# 月次へ resample
monthly = df.set_index('createdon').resample('M').size()

# 日付 component を抽出
df['year'] = df['createdon'].dt.year
df['month'] = df['createdon'].dt.month
df['day_of_week'] = df['createdon'].dt.day_name()
```

### Join と Merge 操作
```python
# 関連する 2 つの table を読み込む
accounts = pd.DataFrame(client.get("account", select=["accountid", "name"]))
contacts = pd.DataFrame(client.get("contact", select=["contactid", "parentcustomerid", "fullname"]))

# リレーションで結合
merged = accounts.merge(
    contacts,
    left_on='accountid',
    right_on='parentcustomerid',
    how='left'
)

print(merged.head())
```

### 統計分析
```python
# 相関行列
correlation = df[['creditlimit', 'industrycode']].corr()

# 分布分析
print(df['creditlimit'].describe())
print(df['creditlimit'].skew())
print(df['creditlimit'].kurtosis())

# パーセンタイル
print(df['creditlimit'].quantile([0.25, 0.5, 0.75]))
```

---

## 5. ピボット テーブルとレポート

### ピボット テーブルを作る
```python
# 業界別・状態別の pivot table
pivot = pd.pivot_table(
    df,
    values='creditlimit',
    index='industrycode',
    columns='statecode',
    aggfunc=['sum', 'mean', 'count']
)

print(pivot)
```

### レポート生成
```python
# 業界別の sales report
industry_report = df.groupby('industrycode').agg({
    'accountid': 'count',
    'creditlimit': 'sum',
    'name': 'first'
}).rename(columns={
    'accountid': 'Account Count',
    'creditlimit': 'Total Credit Limit',
    'name': 'Sample Account'
})

# CSV へ export
industry_report.to_csv('industry_report.csv')

# Excel へ export
industry_report.to_excel('industry_report.xlsx')
```

---

## 6. データ可視化

### Matplotlib 連携
```python
import matplotlib.pyplot as plt

# 可視化を作成
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# ヒストグラム
df['creditlimit'].hist(bins=30, ax=axes[0, 0])
axes[0, 0].set_title('Credit Limit Distribution')

# 棒グラフ
df['industrycode'].value_counts().plot(kind='bar', ax=axes[0, 1])
axes[0, 1].set_title('Accounts by Industry')

# 箱ひげ図
df.boxplot(column='creditlimit', by='industrycode', ax=axes[1, 0])
axes[1, 0].set_title('Credit Limit by Industry')

# 散布図
df.plot.scatter(x='creditlimit', y='industrycode', ax=axes[1, 1])
axes[1, 1].set_title('Credit Limit vs Industry')

plt.tight_layout()
plt.show()
```

### Seaborn 連携
```python
import seaborn as sns

# 相関ヒートマップ
plt.figure(figsize=(8, 6))
sns.heatmap(df[['creditlimit', 'industrycode']].corr(), annot=True)
plt.title('Correlation Matrix')
plt.show()

# 分布プロット
sns.distplot(df['creditlimit'], kde=True)
plt.title('Credit Limit Distribution')
plt.show()
```

---

## 7. 機械学習との統合

### ML 用データ準備
```python
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

# データを読み込んで準備
records = []
for page in client.get("account", select=["accountid", "creditlimit", "industrycode", "statecode"]):
    records.extend(page)

df = pd.DataFrame(records)

# 特徴量エンジニアリング
df['log_creditlimit'] = np.log1p(df['creditlimit'])
df['industry_cat'] = pd.Categorical(df['industrycode']).codes

# 特徴量と目的変数に分割
X = df[['industrycode', 'log_creditlimit']]
y = df['statecode']

# train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

print(f"Training set: {len(X_train)}, Test set: {len(X_test)}")
```

### 分類モデルの構築
```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# モデル学習
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# 評価
y_pred = model.predict(X_test)
print(classification_report(y_test, y_pred))

# 特徴量重要度
importances = pd.Series(
    model.feature_importances_,
    index=X.columns
).sort_values(ascending=False)

print(importances)
```

---

## 8. 高度な DataFrame 操作

### カスタム関数
```python
# 関数を column に適用
df['name_length'] = df['name'].apply(len)

# 関数を row に適用
df['category'] = df.apply(
    lambda row: 'High' if row['creditlimit'] > 100000 else 'Low',
    axis=1
)

# 条件付き操作
df['adjusted_limit'] = df['creditlimit'].where(
    df['statecode'] == 0,
    df['creditlimit'] * 0.5
)
```

### 文字列操作
```python
# 文字列メソッド
df['name_upper'] = df['name'].str.upper()
df['name_starts'] = df['name'].str.startswith('A')
df['name_contains'] = df['name'].str.contains('Inc')
df['name_split'] = df['name'].str.split(',').str[0]

# 置換とマッピング
df['industry'] = df['industrycode'].map({
    1: 'Retail',
    2: 'Manufacturing',
    3: 'Technology'
})
```

### データ形状の変換
```python
# 転置
transposed = df.set_index('name').T

# Stack/Unstack
stacked = df.set_index(['name', 'industrycode'])['creditlimit'].unstack()

# Melt で long format へ
melted = pd.melt(df, id_vars=['name'], var_name='metric', value_name='value')
```

---

## 9. パフォーマンス最適化

### 効率的なデータ読み込み
```python
# 大規模データセットを chunk 単位で読み込む
all_records = []
chunk_size = 1000

for page in client.get(
    "account",
    select=["accountid", "name", "creditlimit"],
    top=10000,        # 総件数を制限
    page_size=chunk_size
):
    all_records.extend(page)
    if len(all_records) % 5000 == 0:
        print(f"Loaded {len(all_records)} records")

df = pd.DataFrame(all_records)
print(f"Total: {len(df)} records")
```

### メモリ最適化
```python
# メモリ使用量を削減
# 繰り返し値には categorical を使う
df['industrycode'] = df['industrycode'].astype('category')

# 適切な数値型を使う
df['creditlimit'] = pd.to_numeric(df['creditlimit'], downcast='float')

# 不要な column を削除
df = df.drop(columns=['unused_col1', 'unused_col2'])

# メモリ使用量を確認
print(df.memory_usage(deep=True).sum() / 1024**2, "MB")
```

### Query 最適化
```python
# filter は client 側ではなく server 側に適用
# ✅ GOOD: server 側 filter
accounts = client.get(
    "account",
    filter="creditlimit > 50000",  # Server-side filter
    select=["accountid", "name", "creditlimit"]
)

# ❌ BAD: 全件読み込んでローカルで filter
all_accounts = client.get("account")  # 全件読み込み
filtered = [a for a in all_accounts if a['creditlimit'] > 50000]  # Client-side
```

---

## 10. 完全な例: Sales Analytics

```python
import pandas as pd
import numpy as np
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

# Setup
client = DataverseClient(
    "https://<myorg>.crm.dynamics.com",
    InteractiveBrowserCredential()
)

# データ読み込み
print("Loading account data...")
records = []
for page in client.get(
    "account",
    select=["accountid", "name", "creditlimit", "industrycode", "statecode", "createdon"],
    orderby=["createdon"]
):
    records.extend(page)

df = pd.DataFrame(records)
df['createdon'] = pd.to_datetime(df['createdon'])

# データ クリーニング
df = df.dropna()

# 特徴量エンジニアリング
df['year'] = df['createdon'].dt.year
df['month'] = df['createdon'].dt.month
df['year_month'] = df['createdon'].dt.to_period('M')

# 分析
print("\n=== ACCOUNT OVERVIEW ===")
print(f"Total accounts: {len(df)}")
print(f"Total credit limit: ${df['creditlimit'].sum():,.2f}")
print(f"Average credit limit: ${df['creditlimit'].mean():,.2f}")

print("\n=== BY INDUSTRY ===")
industry_summary = df.groupby('industrycode').agg({
    'accountid': 'count',
    'creditlimit': ['sum', 'mean']
}).round(2)
print(industry_summary)

print("\n=== BY STATUS ===")
status_summary = df.groupby('statecode').agg({
    'accountid': 'count',
    'creditlimit': 'sum'
})
print(status_summary)

# レポート出力
print("\n=== EXPORTING REPORT ===")
industry_summary.to_csv('industry_analysis.csv')
print("Report saved to industry_analysis.csv")
```

---

## 11. 既知の制限

- `PandasODataClient` は現在、query 結果から手動で DataFrame を作る必要があります
- 非常に大きい DataFrame (数百万行) ではメモリ制約に当たる可能性があります
- pandas 操作は client-side です。大規模データでは server-side 集計の方が効率的です
- ファイル操作には pandas wrapper ではなく標準 `DataverseClient` が必要です

---

## 12. 関連リソース

- [Pandas Documentation](https://pandas.pydata.org/docs/)
- [Official Example: quickstart_pandas.py](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/blob/main/examples/quickstart_pandas.py)
- [SDK for Python README](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/blob/main/README.md)
- [Microsoft Learn: Working with data](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/sdk-python/work-data)
