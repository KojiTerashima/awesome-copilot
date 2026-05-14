---
name: pytest-coverage
description: 'Run pytest tests with coverage, discover lines missing coverage, and increase coverage to 100%.'
---
目標は、テストでコードのすべての行をカバーすることです。

以下を使用してカバレッジ レポートを生成します。

pytest --cov --cov-report=annotate:cov_annotate

特定のモジュールのカバレッジを確認する場合は、次のように指定できます。

pytest --cov=your_module_name --cov-report=annotate:cov_annotate

実行する特定のテストを指定することもできます。次に例を示します。

pytest テスト/test_your_module.py --cov=your_module_name --cov-report=annotate:cov_annotate

cov_annotate ディレクトリを開いて、注釈付きのソース コードを表示します。
ソース ファイルごとに 1 つのファイルが存在します。ファイルのソース カバレッジが 100% である場合は、すべての行がテストでカバーされていることを意味するため、ファイルを開く必要はありません。

テスト カバレッジが 100% 未満の各ファイルについて、cov_annotate で一致するファイルを見つけて、そのファイルを確認します。

行が ! で始まる場合(感嘆符) は、その行がテストの対象外であることを意味します。
欠落している行をカバーするテストを追加します。

すべての行がカバーされるまでテストを実行し、カバレッジを改善し続けます。