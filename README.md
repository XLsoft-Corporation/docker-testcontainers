# docker-testcontainers

Docker Testcontainers のサンプルです



## はじめに

Testcontainers の開発元 AtomicJar は 2023年12月に Docker 社に買収され、Testcontainers は Docker 製品の一部となりました。（[AtomicJar is now part of Docker\! \- AtomicJar](https://www.atomicjar.com/2023/12/atomicjar-is-now-part-of-docker/)）

本記事は、[testcontainers.com/](testcontainers.com/) にある [Getting Started](https://testcontainers.com/getting-started/) ドキュメントの日本語参考訳です。



## Testcontainers とは？

Testcontainers は、Docker コンテナーにラッピングされた実際のサービスを使って、ローカルの開発とテストの依存関係をブートストラップするための、簡単で軽量なAPIを提供するライブラリです。Testcontainersを使えば、モックやインメモリサービスを使わずに、本番で使うのと同じサービスに依存するテストを書くことができます。



### Testcontainers はどのような問題を解決しますか？

クラウドネイティブなインフラストラクチャとマイクロサービスは、開発者から制御を奪い、Local 環境で働くのが難しくなりました。あなたが次のアーキテクチャ内の「My Service」に取り組んでいる開発者だとします。

![](https://testcontainers.com/getting-started/images/cn-arch.png)

あなたは My Service とそのデータストア（緑色）のみを所有していますが、いくつかのダウンストリームの依存関係があります（青色）。これは、ローカル開発と統合テストに必要です。これには、次の課題があります。

- テストを実行する前に、インフラストラクチャが稼働しており、目的の状態で事前に構成されていることを確認する必要があります。
- リソース（データベース、メッセージブローカーなど）が複数のユーザーまたは CI パイプライン間で共有されている場合、テスト結果は、データの破損や構成のずれの可能性があるため、非決定論的になります。

これらの課題を回避する 1つの方法は、インメモリデータベース、組み込みサービス、モック、および運用依存関係の他の偽のレプリカに頼る事です。ただし、これらのアプローチでは、自身の問題(たとえば、インメモリサービスには、運用サービスのすべての機能があるとは限らず、わずかに異なる動作をします)をもたらします。

Testcontainers は、データベースなどのアプリケーションの依存関係を実行することで、これらの問題を解決します。Docker コンテナ内のメッセージブローカーなど、信頼性と再現性のあるテストの実行を支援します。これらの実際のサービスと通信し、テストコードにプログラム API を提供します。

前の例では、Testcontainers を使用してプロビジョニングすることで、実際の依存関係に対して「My Service」をコードから直接自由に開発およびテストできました。（注: 緑のボックスが付いている依存関係が Testcontainers でプロビジョニングされています。）

![](https://testcontainers.com/getting-started/images/cn-arch-tc.png)



### Testcontainers を使用する利点

- **オンデマンドの分離インフラストラクチャのプロビジョニング:** 事前にプロビジョニングされた統合テストインフラストラクチャを用意する必要はありません。Testcontainers は、テストを実行する前に必要なサービスを提供します。複数のビルド パイプラインが並列に実行されている場合でも、テストデータの汚染はありません。なぜなら、各パイプラインは分離された一連のサービスで実行されるからです。
- **ローカル環境と CI 環境の両方で一貫したエクスペリエンス:** 統合テストは、単体テストを実行するのと同じように、IDE から直接実行できます。変更をプッシュして、CI パイプラインが完了するのを待つ必要はありません。
- **待機戦略を使用した信頼性の高いテストセットアップ:** Docker コンテナは、テストで使用する前に起動して完全に初期化する必要があります。Testcontainers ライブラリは、コンテナ（とその中のアプリケーション）が完全に初期化されていることを確認するために、すぐに使える待機ストラテジーの実装をいくつか提供しています。Testcontainers モジュールは、特定のテクノロジーに関連する待機ストラテジーをすでに実装しています。また、いつでも独自のストラテジーを実装することも、必要に応じて複合のストラテジーを作成することもできます。
- **高度なネットワーク機能:** Testcontainers ライブラリは、コンテナーのポートをホストマシンで使用可能なランダムなポートにマッピングします。これにより、テストがこれらのサービスに確実に接続されます。Docker ネットワークを作成し、複数のコンテナを接続して、静的な Docker ネットワークエイリアスを介して相互に通信します。
- **自動クリーンアップ:**  Testcontainers ライブラリは、Ryuk サイドカーコンテナーを使ってテスト実行が完了した後、作成されたリソース（コンテナー、ボリューム、ネットワークなど）を自動的に削除します。必要なコンテナーを起動している間、Testcontainers は作成されたリソース（コンテナー、ボリューム、ネットワークなど）にラベルのセットをアタッチし、Ryuk は自動的にそれらのラベルを照合してリソースのクリーンアップを実行します。これは、テストプロセスが異常終了（SIGKILLの送信など）しても、確実に機能します。



### Docker と Docker Compose との違い

Docker と Docker Compose は、テストに必要な依存関係をスピンアップするために直接使用することもできますが、このアプローチには欠点があります。生の Docker コマンドや Docker Compose を使用して、信頼性が高く、完全に初期化されたサービス依存関係を作成するには、Docker の内部や、コンテナー内で特定のテクノロジを実行する最適な方法について十分な知識が必要です。例えば、Docker コマンドや docker-compose を直接使用して動的な「統合テスト環境」を作成すると、ポートの競合が発生したり、テスト開始時にコンテナーが完全に初期化されていなかったり、相互作用の準備ができていなかったりする可能性があります。Testcontainers ライブラリは、Docker コンテナーのフルパワーをボンネットの下で活用し、慣用的な API を介して開発者に公開します。



### サポートされている言語と前提条件

Testcontainers は、最も一般的な言語とプラットフォームをサポートします。Java、.NET、Go、NodeJS、Python、Rust、Haskell を含みます。

- [Java](https://java.testcontainers.org/)
- [Go](https://golang.testcontainers.org/)
- [.NET](https://dotnet.testcontainers.org/)
- [Node.js](https://node.testcontainers.org/)
- [Clojure](https://cljdoc.org/d/clj-test-containers/clj-test-containers/)
- [Elixir](https://github.com/testcontainers/testcontainers-elixir)
- [Haskell](https://github.com/testcontainers/testcontainers-hs)
- [Python](https://testcontainers-python.readthedocs.io/en/latest/)
- [Ruby](https://github.com/testcontainers/testcontainers-ruby)
- [Rust](https://docs.rs/testcontainers/latest/testcontainers/)

Testcontainers ベースのテストを実行するには、Testcontainers Cloud や Docker をローカルにインストールします。次のコンテナーランタイム環境が公式にサポートされています。

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Linux 上の Docker Engine](https://docs.docker.com/engine/install/)
- [Testcontainers Cloud](https://www.testcontainers.cloud/)

サポートされているコンテナランタイム環境、および既知の制限に関する詳細な情報については、 別のコンテナランタイム環境については、[このページを参照してください](https://java.testcontainers.org/supported_docker_environment/)。



### Testcontainers ワークフロー

Testcontainers は、すでに使い慣れた任意のテストライブラリで使用できます。一般的な Testcontainers ベースの統合テストは、次のように機能します。

![](https://testcontainers.com/getting-started/images/test-workflow.png)

- **テスト実行前:** Testcontainers APIを使用して、必要なサービス（データベース、メッセージングシステムなど）を Docker コンテナーとして起動します。必要なコンテナーが起動したら、これらのコンテナー化されたサービスを使用するようにアプリケーション設定を構成または更新し、オプションでテストに必要なデータを初期化します。
- **テストの実行中:** テストは、コンテナー化されたサービスを使用して実行されます。
- **テスト実行後:** Testcontainers は、テストの実行が成功したか失敗したかに関係なく、コンテナーを破棄します。



### どのようなテクノロジーでテストできますか？

Testcontainers を使用して、任意の Docker 化されたソフトウェアでテストを実行できます。Testcontainers は既存の Docker イメージ、Dockerfile、または Docker Compose ファイルからコンテナーをスピンアップできます。



### GenericContainer の抽象化

Testcontainers は Docker コンテナーを表す GenericContainer と呼ばれるプログラム的な抽象化を提供します。GenericContainer を使って、Docker コンテナーを起動する、ホスト名（マップされたポートに到達可能なホスト）、マップされたポートなどのコンテナー情報を取得する、コンテナーを停止するなどができます。

たとえば、次のように Testcontainers for Java の GenericContainer を使用できます。

```java
GenericContainer container = new GenericContainer("postgres:15")
        .withExposedPorts(5432)
        .waitingFor(new LogMessageWaitStrategy()
            .withRegEx(".*database system is ready to accept connections.*\\s")
            .withTimes(2)
            .withStartupTimeout(Duration.of(60, ChronoUnit.SECONDS)));
container.start();
var username = "test";
var password = "test";
var jdbcUrl = "jdbc:postgresql://" + container.getHost() + ":" + container.getMappedPort(5432) + "/test";
//perform db operations
container.stop();
```

GenericContainer API は、サポートされている他の言語でも使用できます。



### Testcontainers モジュール

Testcontainers は、リレーショナルデータベースを含む、一般的に使用されるインフラストラクチャの依存関係の幅広いモジュールを提供します。 NoSQLデータストア、検索エンジン、メッセージブローカーなど完全なリストについては、https://testcontainers.com/modules/ を参照してください。

テクノロジー固有の[モジュール](https://testcontainers.com/modules/)は、GenericContainer の上位の抽象化であり、定型文なしでこれらのテクノロジーを構成して実行し、関連するパラメータに簡単にアクセスできるようにします。

例えば、GenericContainer の代わりに Testcontainers [postgres モジュール](https://testcontainers.com/modules/postgresql/)の PostgreSQLContainer を以下のように使うことができます:

```java
PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:15");
postgres.start();
var username = postgres.getUsername();
var password = postgres.getPassword();
var jdbcUrl = postgres.getJdbcUrl();
//perform db operations
postgres.stop();
```

モジュールを使用すると、`getJdbcUrl()` のような便利なヘルパーメソッドを提供することで、テクノロジーとのやり取りが非常に簡単になります。また、Testcontainers モジュールは、テクノロジ固有のポートマッピングや適切な待機ストラテジーなどの実装も行います。


### 次のステップ

[https://testcontainers.com/guides/](https://testcontainers.com/guides/) で提供されているタスク指向のガイドに取り組むことで、実際に体験することができます。最後に、既存のテストスイートを Testcontainers に移行する方法について、[https://testcontainers.com/modules/](https://testcontainers.com/modules/) にある利用可能な Testcontainers モジュールをご覧ください。

Testcontainersでのテストをお楽しみください！




