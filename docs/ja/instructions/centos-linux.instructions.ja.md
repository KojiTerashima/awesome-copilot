---
description: 'CentOS 管理、RHEL 互換ツール、SELinux 対応操作に関するガイダンス。'
applyTo: '**'
---

# CentOS 管理ガイドライン

CentOS 環境用のガイダンス、スクリプト、またはドキュメントを作成する場合は、次の手順を使用してください。

## プラットフォームの調整

- CentOS のバージョン (ストリームとレガシー) を特定し、コマンドを調整します。
- Stream/8+ の場合は `dnf` を、CentOS 7 の場合は `yum` を優先します。
- RHEL 互換の用語とパスを使用してください。

## パッケージ管理

- GPG チェックを有効にしてリポジトリを検証します。
- パッケージの詳細については、`dnf info`/`yum info` および `dnf repoquery` を使用してください。
- 必要に応じて、安定性を高めるために `dnf versionlock` または `yum versionlock` を使用します。
- EPEL の依存関係と、それらを安全に有効/無効にする方法を説明します。

## 構成とサービス

- 必要に応じてサービス環境ファイルを `/etc/sysconfig/` に配置します。
- オーバーライドには systemd ドロップインを使用し、制御には `systemctl` を使用します。
- `iptables`/`nftables` を明示的に使用しない限り、`firewalld` (`firewall-cmd`) を優先します。

## 安全

- 可能な限り、SELinux を強制モードにしてください。
- ポリシーの調整には `semanage`、`restorecon`、および `setsebool` を使用します。
- 拒否については `/var/log/audit/audit.log` を参照してください。

## 成果物

- コピー＆ペースト可能なブロックにコマンドを入力します。
- 変更後の検証手順を含めます。
- 危険な操作に対してロールバック手順を提供します。
