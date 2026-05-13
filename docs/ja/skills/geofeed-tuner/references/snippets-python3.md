# Python 3 用コード例

- Python 3 標準搭載の [`ipaddress` package](https://docs.python.org/3/library/ipaddress.html) を使い、利用できる場合はコンストラクタに `strict=True` を渡します。
- IPv4 と IPv6 のアドレス解析は意識して区別してください。専門のネットワークエンジニアにとって同じものではありません。利用できる中で最も適切な型/クラスを使います。
- サブネットは単一ホストを含められることを覚えておきます。IPv6 では `/128`、IPv4 では `/32` を使います。

## IP アドレスとサブネットの解析

- [`ipaddress` の簡易ファクトリー関数](https://docs.python.org/3/library/ipaddress.html#convenience-factory-functions) を使います。

    以下の `ipaddress.ip_address(textAddress)` の例は、テキストをそれぞれ `IPv4Address` と `IPv6Address` オブジェクトに解析します。

    ```python
    ipaddress.ip_address('192.168.0.1')
    ipaddress.ip_address('2001:db8::')
    ```

    以下の `ipaddress.ip_network(address, strict=True)` の例は、サブネット文字列を解析し、無効な入力では失敗しながら `IPv4Network` または `IPv6Network` オブジェクトを返します。

    ```python
    ipaddress.ip_network('192.168.0.0/28', strict=True)
    ```

    以下の strict モード呼び出しは、`ValueError: 192.168.0.1/30 has host bits set` で正しく失敗します。この種のエラーはユーザーに修正してもらい、補正を推測してはいけません。

    ```python
    ipaddress.ip_network('192.168.0.1/30', strict=True)
    ```

- 厳格形式のパーサー [`ipaddress.ip_network(address, strict=True)`](https://docs.python.org/3/library/ipaddress.html#ipaddress.ip_network) を使います。

## IP サブネットの辞書

Python の辞書を使って、サブネットと関連するジオロケーション属性を追跡します。`IPv4Network`、`IPv6Network`、`IPv4Address`、`IPv6Address` はすべて hashable なので辞書キーに使えます。

## 公開されていない IP 範囲の検出

SKILL.md では非公開範囲の検出に `is_private` を参照しています。ネットワークのプロパティを使います。

```python
import ipaddress

def is_non_public(network):
    """Check if a network is non-public (private, loopback, link-local, multicast, or reserved).
    
    Note: In Python < 3.11, is_private may incorrectly flag some ranges
    (e.g., 100.64.0.0/10 CGNAT space). Use is_global as the primary check
    when available, with fallbacks for edge cases.
    """
    addr = network.network_address
    return (
        network.is_private
        or network.is_loopback
        or network.is_link_local
        or network.is_multicast
        or network.is_reserved
        or not network.is_global  # catches most non-routable space
    )
```

**Python < 3.11 での `is_private` に関する注意:** `100.64.0.0/10`（Carrier-Grade NAT）範囲は古い Python では `is_private=True` かつ `is_global=False` を返します。CGNAT 空間はグローバルにルーティング可能ではないため、RFC 8805 の観点では非公開として扱うのが正しいです。

## ISO 3166-1 国コードの検証

[assets/iso3166-1.json](../assets/iso3166-1.json) から有効な ISO 2 文字国コードを読み込みます。具体的には `alpha_2` 属性を使います。

```python
import json

with open('assets/iso3166-1.json') as f:
    data = json.load(f)
    valid_countries = {c['alpha_2'] for c in data['3166-1']}
```

## ISO 3166-2 地域コードの検証

[assets/iso3166-2.json](../assets/iso3166-2.json) から有効な地域コードを読み込みます。具体的には `code` 属性を使います。トップレベルキーは `3166-2` です（iso3166-1 のパターンと一致）。

```python
import json

with open('assets/iso3166-2.json') as f:
    data = json.load(f)
    valid_regions = {r['code'] for r in data['3166-2']}
```
