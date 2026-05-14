---
description: 'Fedora (Red Hat 系) システム、dnf ワークフロー、SELinux、モダンな systemd 運用に関するガイダンス。'
applyTo: '**'
---

# Fedora 管理ガイドライン

Fedora システム向けのガイダンス、スクリプト、ドキュメントを作成する際は、これらの指示を使用してください。

## プラットフォーム整合性

- 必要に応じて Fedora のリリース番号を明記します。
- モダンなツール (`dnf`, `systemctl`, `firewall-cmd`) を優先します。
- リリースサイクルが速い点に触れ、古いガイダンスとの互換性を確認します。

## パッケージ管理

- インストールと更新には `dnf` を使い、ロールバックには `dnf history` を使います。
- パッケージ確認には `dnf info` と `rpm -qi` を使います。
- COPR リポジトリに触れるのは、サポート上の注意点を明確に示せる場合だけにします。

## 設定とサービス

- systemd の drop-in は `/etc/systemd/system/<unit>.d/` に配置します。
- ログには `journalctl`、サービス状態の確認には `systemctl status` を使います。
- 明示的に `nftables` を使う場合を除き、`firewalld` を優先します。

## セキュリティ

- ユーザーが permissive mode を求めない限り、SELinux は enforcing のままにします。
- ポリシー変更には `semanage`、`setsebool`、`restorecon` を使います。
- 広範な `audit2allow` ルールではなく、対象を絞った修正を推奨します。

## 成果物

- コマンドはそのまま貼り付けて実行できるブロックで示します。
- 変更後の検証手順を含めます。
- リスクのある操作にはロールバック手順も提示します。
