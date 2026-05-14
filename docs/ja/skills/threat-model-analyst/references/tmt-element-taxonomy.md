# TMT 要素分類 — コードから脅威モデルへの DFD リファレンス

ソース コード分析から DFD 要素を特定するための完全なリファレンス。
TM7 との互換性を確保するために、Microsoft Threat Modeling Tool (TMT) 要素タイプと調整します。
これは、すべての TMT タイプ分類に対する **単一の権限のあるファイル**です。

**図のスタイル設定とレンダリング ルール** は次の場所にあります: [diagram-conventions.md](./diagram-conventions.md)
**このファイルの内容は次のとおりです。** コード内で何を探すか、コードを分類する方法、コードに名前を付ける方法。

---

## 1. 要素の種類

**注:** TMT ID (例: `SE.P.TMCore.OSProcess`) は分類の参照のみを目的としています。 **TMT ID を Mermaid ノード ID として使用しないでください。** 簡潔で読みやすい PascalCase ID (例: `WebServer`、`SqlDatabase`) を使用してください。

### 1.1 プロセスの種類

| TMT ID |名前 |識別するコード パターン |
|----------|------|--------------------------|
| `SE.P.TMCore.OSProcess` | OSプロセス |ネイティブ実行可能ファイル、システム プロセス、生成されたプロセス |
| `SE.P.TMCore.Thread` |スレッド |スレッド プール、`Task`、`pthread`、ワーカー スレッド |
| `SE.P.TMCore.WinApp` |ネイティブ アプリケーション | Win32 アプリ、C/C++ 実行可能ファイル、デスクトップ アプリ |
| `SE.P.TMCore.NetApp` |マネージド アプリケーション | .NET アプリ、C# サービス、F# プログラム |
| `SE.P.TMCore.ThickClient` |シッククライアント |デスクトップ GUI アプリ、WPF、WinForms、Electron |
| `SE.P.TMCore.BrowserClient` |ブラウザクライアント | SPA、JavaScript アプリ、WebAssembly |
| `SE.P.TMCore.WebServer` |ウェブサーバー | IIS、Apache、Nginx、Express、Kestrel |
| `SE.P.TMCore.WebApp` |ウェブアプリケーション | ASP.NET、Django、Rails、Spring MVC |
| `SE.P.TMCore.WebSvc` |ウェブサービス | REST API、SOAP、GraphQL エンドポイント |
| `SE.P.TMCore.VM` |仮想マシン | VM、コンテナ、Docker |
| `SE.P.TMCore.Win32Service` | Win32 サービス | Windows サービス、`ServiceBase` |
| `SE.P.TMCore.KernelThread` |カーネルスレッド |カーネル モジュール、ドライバー、リング 0 コード |
| `SE.P.TMCore.Modern` | Windows ストアのプロセス | UWP アプリ、Windows ストア アプリ、サンドボックス アプリ |
| `SE.P.TMCore.PlugIn` |ブラウザと ActiveX プラグイン |ブラウザ拡張機能、ActiveX、BHO プラグイン |
| `SE.P.TMCore.NonMS` | Microsoft 以外の OS で実行されるアプリケーション | Linux アプリ、macOS アプリ、Unix プロセス |

### 1.2 外部インタラクターのタイプ| TMT ID |名前 |識別するコード パターン |
|----------|------|--------------------------|
| `SE.EI.TMCore.Browser` |ブラウザ |ブラウザ クライアント、ユーザー エージェント、Web UI コンシューマー |
| `SE.EI.TMCore.AuthProvider` |認可プロバイダ | OAuth サーバー、OIDC プロバイダー、IdP、SAML |
| `SE.EI.TMCore.WebSvc` |外部 Web サービス |外部 API、ベンダー サービス、SaaS エンドポイント |
| `SE.EI.TMCore.User` |人間のユーザー |エンドユーザー、オペレーター、管理者 |
| `SE.EI.TMCore.Megaservice` |メガサービス |大規模なクラウド プラットフォーム (Azure、AWS、GCP サービス) |
| `SE.EI.TMCore.WebApp` |外部 Web アプリケーション |サードパーティ Web アプリ、外部ポータル |
| `SE.EI.TMCore.CRT` | Windows ランタイム | WinRT API、Windows ランタイム コンポーネント |
| `SE.EI.TMCore.NFX` | Windows .NET ランタイム | .NET フレームワーク、CLR、BCL |
| `SE.EI.TMCore.WinRT` | Windows RT ランタイム | Windows RT プラットフォーム、ARM Windows アプリ |

### 1.3 データストアの種類

| TMT ID |名前 |識別するコード パターン |
|----------|------|--------------------------|
| `SE.DS.TMCore.CloudStorage` |クラウドストレージ | Azure BLOB、S3、GCS |
| `SE.DS.TMCore.SQL` | SQL データベース | PostgreSQL、MySQL、SQL Server、SQLite |
| `SE.DS.TMCore.NoSQL` |非リレーショナル DB | MongoDB、CosmosDB、Redis、Cassandra |
| `SE.DS.TMCore.FS` |ファイルシステム |ローカル ファイル、NFS、共有ドライブ |
| `SE.DS.TMCore.Cache` |キャッシュ | Redis、Memcached、メモリ内キャッシュ |
| `SE.DS.TMCore.ConfigFile` |設定ファイル | `.env`、`appsettings.json`、YAML 構成 |
| `SE.DS.TMCore.Cookie` |クッキー | HTTP Cookie、セッション Cookie |
| `SE.DS.TMCore.Registry` |レジストリ ハイブ | Windows レジストリ、システム構成ストア |
| `SE.DS.TMCore.HTML5LS` | HTML5 ローカル ストレージ | `localStorage`、`sessionStorage`、インデックス付き DB |
| `SE.DS.TMCore.Device` |デバイス |ハードウェア デバイス、USB、周辺機器ストレージ |

### 1.4 データ フローの種類| TMT ID |名前 |識別するコード パターン |
|----------|------|--------------------------|
| `SE.DF.TMCore.HTTP` | HTTP | `fetch()`、`axios`、`HttpClient`、TLS を使用しない REST |
| `SE.DF.TMCore.HTTPS` | HTTPS | TLS で保護された REST、`https://` エンドポイント |
| `SE.DF.TMCore.Binary` |バイナリ | gRPC、Protobuf、生のバイナリ プロトコル |
| `SE.DF.TMCore.NamedPipe` |名前付きパイプ |名前付きパイプ経由の IPC |
| `SE.DF.TMCore.SMB` |中小企業 | SMB/CIFS ファイル共有 |
| `SE.DF.TMCore.UDP` | UDP | UDP ソケット、データグラム プロトコル |
| `SE.DF.TMCore.SSH` | SSH | SSH トンネル、SFTP、SCP |
| `SE.DF.TMCore.LDAP` | LDAP | LDAP クエリ、AD ルックアップ |
| `SE.DF.TMCore.LDAPS` | LDAP | TLS 経由の安全な LDAP |
| `SE.DF.TMCore.IPsec` | IPsec | VPN トンネル、IPsec で保護された接続 |
| `SE.DF.TMCore.RPC` | RPC または DCOM | COM+、DCOM、RPC 呼び出し、WCF net.tcp |
| `SE.DF.TMCore.ALPC` |アルＰＣ |高度なローカル プロシージャ コール、Windows IPC |
| `SE.DF.TMCore.IOCTL` | IOCTL インターフェイス |デバイスI/O制御、ドライバ通信 |

### 1.5 信頼境界の種類

**線の境界:**

| TMT ID |名前 |コードインジケータ |
|------|------|------|
| `SE.TB.L.TMCore.Internet` |インターネットの境界 |パブリック エンドポイント、API ゲートウェイ |
| `SE.TB.L.TMCore.Machine` |マシンの境界 |プロセス境界、VM 分離 |
| `SE.TB.L.TMCore.Kernel` |カーネル/ユーザーモード |ドライバー、リング 0/3 トランジション |
| `SE.TB.L.TMCore.AppContainer` |アプリコンテナ | UWP サンドボックス、アプリ コンテナー |

**境界線の境界:**

| TMT ID |名前 |コードインジケータ |
|------|------|------|
| `SE.TB.B.TMCore.CorpNet` |コープネット |企業ネットワーク、VPN 境界 |
| `SE.TB.B.TMCore.Sandbox` |サンドボックス |サンドボックス化された実行環境 |
| `SE.TB.B.TMCore.IEB` | Internet Explorer の境界 | IE ゾーン、IE セキュリティ設定 |
| `SE.TB.B.TMCore.NonIEB` |他のブラウザの境界 | Chrome、Firefox、Edge のセキュリティ コンテキスト |

---

## 2. 信頼境界の検出

コードが交差する場合は信頼境界 (`subgraph`) を作成します。

|境界タイプ |コードインジケータ |
|---------------|---------------|
| **インターネット/公共** |パブリック エンドポイント、API ゲートウェイ、ロード バランサー |
| **機械** |プロセス境界、ホスト分離 |
| **カーネル/ユーザー モード** |カーネルコール、ドライバー、システムコール |
| **AppContainer** | UWP サンドボックス、コンテナー化されたアプリ |
| **コーポネット** |企業ネットワーク境界、VPN |
| **サンドボックス** |サンドボックス化された実行環境 |

---

## 3. データ フローの検出

フローを識別するには、次のパターンを探してください。|フロータイプ |コードパターン |
|----------|------|
| **HTTP/HTTPS** | `fetch()`、`axios`、`HttpClient`、REST 呼び出し |
| **SQL データベース** | ORM クエリ、SQL 接続、`DbContext` |
| **メッセージ キュー** |パブリッシュ/サブスクライブ、キュー送信/受信、Dapr パブリッシュ/サブスクライブ |
| **ファイル I/O** |ファイルの読み取り/書き込み、BLOB のアップロード/ダウンロード |
| **gRPC** | Protobuf 呼び出し、gRPC ストリーム |
| **名前付きパイプ** |名前付きパイプ経由の IPC |
| **SSH** | SSH トンネル、SFTP、SCP 転送 |
| **LDAP/LDAPS** |ディレクトリ クエリ、AD ルックアップ |

---

## 4. コード分析チェックリスト

コードを分析するときは、以下を体系的に特定します。

1. **エントリ ポイント** → 外部インタラクター + インバウンド フロー
   - API コントローラー、イベント ハンドラー、Webhook エンドポイント

2. **サービス/ロジック** → プロセス
   - ビジネス ロジック クラス、サービス層、ワーカー

3. **データ アクセス** → データ ストア + フロー
   - リポジトリクラス、DBコンテキスト、キャッシュクライアント

4. **外部呼び出し** → 外部インタラクター + アウトバウンドフロー
   - HTTP クライアント、SDK 統合、サードパーティ API

5. **セキュリティ境界** → 信頼境界
   - 認証ミドルウェア、ネットワークセグメント、展開ユニット

6. **Kubernetes ポッドの構成** → サイドカーのコロケーション
   - Helm チャート、K8 マニフェスト、デプロイメント YAML を探します
   - 共通サイドカー: Dapr、MISE、Envoy、Istio プロキシ、Linkerd、ログ コレクター
   - **`diagram-conventions.md` ルール 1 のルールを適用します** - ホスト ノードに注釈を付け、スタンドアロンのサイドカー ノードを作成しない

---

## 5. 命名規則

引用符ルールを含む完全な表については、[diagram-conventions.md](./diagram-conventions.md) の命名規則セクションを参照してください。

---

## 6. 出力ファイル

柔軟性を最大限に高めるために **2 つのファイル**を生成します:

### ファイル 1: ピュアマーメイド (`.mmd`)
- 生の Mermaid コードのみ、マークダウン ラッパーなし
- 用途: CLI ツール、エディター、CI/CD、ダイレクト レンダリング

### ファイル 2: マークダウン (`.md`)
- ` ```mermaid ` コードフェンスの中の人魚
- 要素、フロー、および境界の概要テーブルを含める
- 用途: GitHub、VS Code、ドキュメント

### フォーマットの比較

|フォーマット |拡張子 |目次 |最適な用途 |
|----------|----------|----------|----------|
|ピュアマーメイド | `.mmd` |生の図コード | CLI、エディタ、ツール |
|マークダウン | `.md` |図+表 | GitHub、ドキュメント、表示 |