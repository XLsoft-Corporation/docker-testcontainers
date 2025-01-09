# Getting started with Testcontainers for Java

[Getting started with Testcontainers for Java](https://testcontainers.com/guides/getting-started-with-testcontainers-for-java/) をやってみましょう。

## システム要件

Visual Studio Code（以下 VS Code）で maven のプロジェクトのテストを行います。以下の環境を用意してください。本ドキュメントの執筆時 2025 年 1 月の最新版を使用しています。

- **JDK:**  
  [Eclipse Temurin \| Adoptium](https://adoptium.net/temurin/) から最新版をダウンロードし、インストールします。（OpenJDK の実装は VS Code の Extension Pack for Java のドキュメントで紹介されていたので Temurin にしています。）
  - 本ドキュメントでの使用バージョン: `21.0.4`
  - 環境変数 `JAVA_HOME` を追加して JDK インストールフォルダを設定
  - 環境変数 `PATH` に `%JAVA_HOME%\bin` を設定

```ps
> java --version
openjdk 21.0.4 2024-07-16 LTS
OpenJDK Runtime Environment Temurin-21.0.4+7 (build 21.0.4+7-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.4+7 (build 21.0.4+7-LTS, mixed mode, sharing)
```

- **maven:**  
  [Maven – Download Apache Maven](https://maven.apache.org/download.cgi) から最新版をダウンロードし、任意のフォルダに展開します。
  - 本ドキュメントでの使用バージョン: `3.9.9`
  - 環境変数 `PATH` に maven のインストールフォルダ以下の `bin` フォルダを設定

```ps
> mvn --version
Apache Maven 3.9.9 (8e8579a9e76f7d015ee5ec7bfcdc97d260186937)
Maven home: C:\apache-maven-3.9.9
Java version: 21.0.4, vendor: Eclipse Adoptium, runtime: C:\Program Files\Eclipse Adoptium\jdk-21.0.4.7-hotspot
Default locale: ja_JP, platform encoding: UTF-8
OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
```

- **VS Code:**  
  [Download Visual Studio Code \- Mac, Linux, Windows](https://code.visualstudio.com/Download) からダウンロードし、インストールします。
  - 本ドキュメントでの使用バージョン: `1.96.2`
  - 拡張機能として [Extension Pack for Java \- Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) をインストール

```ps
> code --version
1.96.2
fabdb6a30b49f79a7aba0f2ad9df9b399473380f
x64
```

## Maven を使用した Java プロジェクトの作成

次の内容でご利用の IDE から Java プロジェクトを作成します。

> PowerShell で mvn コマンドを実行する際には `.` が誤認識されるので `""` で括る必要があります。（コマンドプロンプトやシェルでは不要です…）

```ps
mvn archetype:generate -DgroupId="com.testcontainers.demo" -DartifactId="demo" -DarchetypeArtifactId="maven-archetype-quickstart" -DarchetypeVersion="1.5" -DinteractiveMode="false"
```

上記は Maven を使用していますが、Gradle を利用することも可能です。

プロジェクトが作成されたら、次の依存関係を `pom.xml` に追加します。

```xml
<dependencies>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.3</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.6</version>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```


## ビジネスロジックの実装

顧客の詳細を管理するための `CustomerService` クラスを作成します。

> 全てのソースコードファイルは `src/main/java/com/testcontainers/demo` フォルダに作成します。

まず、次のように `Customer.java` クラスを作成しましょう。

```java
package com.testcontainers.demo;

public record Customer(Long id, String name) {}
```

JDBC 接続パラメータを保持する `DBConnectionProvider.java` クラスを作成し、データベース接続を取得するメソッドを次のように作成します。

```java
package com.testcontainers.demo;

import java.sql.Connection;
import java.sql.DriverManager;

class DBConnectionProvider {

  private final String url;
  private final String username;
  private final String password;

  public DBConnectionProvider(String url, String username, String password) {
    this.url = url;
    this.username = username;
    this.password = password;
  }

  Connection getConnection() {
    try {
      return DriverManager.getConnection(url, username, password);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
}
```

`CustomerService.java` クラスを作成し、次のコードで置き換えます。

```java
package com.testcontainers.demo;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class CustomerService {

  private final DBConnectionProvider connectionProvider;

  public CustomerService(DBConnectionProvider connectionProvider) {
    this.connectionProvider = connectionProvider;
    createCustomersTableIfNotExists();
  }

  public void createCustomer(Customer customer) {
    try (Connection conn = this.connectionProvider.getConnection()) {
      PreparedStatement pstmt = conn.prepareStatement(
        "insert into customers(id,name) values(?,?)"
      );
      pstmt.setLong(1, customer.id());
      pstmt.setString(2, customer.name());
      pstmt.execute();
    } catch (SQLException e) {
      throw new RuntimeException(e);
    }
  }

  public List<Customer> getAllCustomers() {
    List<Customer> customers = new ArrayList<>();

    try (Connection conn = this.connectionProvider.getConnection()) {
      PreparedStatement pstmt = conn.prepareStatement(
        "select id,name from customers"
      );
      ResultSet rs = pstmt.executeQuery();
      while (rs.next()) {
        long id = rs.getLong("id");
        String name = rs.getString("name");
        customers.add(new Customer(id, name));
      }
    } catch (SQLException e) {
      throw new RuntimeException(e);
    }
    return customers;
  }

  private void createCustomersTableIfNotExists() {
    try (Connection conn = this.connectionProvider.getConnection()) {
      PreparedStatement pstmt = conn.prepareStatement(
        """
        create table if not exists customers (
            id bigint not null,
            name varchar not null,
            primary key (id)
        )
        """
      );
      pstmt.execute();
    } catch (SQLException e) {
      throw new RuntimeException(e);
    }
  }
}
```

`CustomerService` クラスで何が起こっているのかを理解しましょう。

- JDBC API を使用してデータベース接続を取得するために、`connectionProvider.getConnection()` メソッドを呼び出しています。
- `createCustomersTableIfNotExists()` メソッドがあり、`customers` テーブルがまだ存在しない場合に作成します。
- 新しい customer レコードをデータベースに挿入する `createCustomer()` メソッドがあります。
- `customers` テーブルからすべての行をフェッチし、データを `Customer` オブジェクトに入力し、`Customer` オブジェクトのリストを返す `getAllCustomers()` メソッドがあります。

次に、Testcontainers を使用して `CustomerService` ロジックをテストする方法を見てみましょう。


## Testcontainers の依存関係を追加する

Testcontainers ベースのテストを作成する前に、次のように `pom.xml` に Testcontainers の依存関係を追加しましょう。

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.8</version>
    <scope>test</scope>
</dependency>
```

アプリケーションにPostgresデータベースを使用しているため、テスト依存関係として TestcontainersPostgres モジュールを追加しました。


## Testcontainers を使用してテストを記述する

`src/test/java/com/testcontainers/demo` の下に `CustomerServiceTest.java` を作成し、次のコードで置き換えます。

```java
package com.testcontainers.demo;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.List;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.PostgreSQLContainer;

class CustomerServiceTest {

  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
    "postgres:16-alpine"
  );

  CustomerService customerService;

  @BeforeAll
  static void beforeAll() {
    postgres.start();
  }

  @AfterAll
  static void afterAll() {
    postgres.stop();
  }

  @BeforeEach
  void setUp() {
    DBConnectionProvider connectionProvider = new DBConnectionProvider(
      postgres.getJdbcUrl(),
      postgres.getUsername(),
      postgres.getPassword()
    );
    customerService = new CustomerService(connectionProvider);
  }

  @Test
  void shouldGetCustomers() {
    customerService.createCustomer(new Customer(1L, "George"));
    customerService.createCustomer(new Customer(2L, "John"));

    List<Customer> customers = customerService.getAllCustomers();
    assertEquals(2, customers.size());
  }
}
```

CustomerServiceTest のコードを理解しましょう。

- Docker イメージ名 `postgres:16-alpine` を渡して PostgreSQLContainer を宣言しました。
- Postgres コンテナは、テストメソッドを実行する前に実行される JUnit 5 の `@BeforeAll` コールバックを使用して開始されます。
- すべてのテストメソッドを実行する前に実行されるコールバックメソッド `@BeforeEach` で、Postgres コンテナから取得した JDBC 接続パラメータを渡し、`CustomerService` インスタンスも作成する `DBConnectionProvider` インスタンスを作成しました。`CustomerService` のコンストラクターでは、customers テーブルがまだ存在しない場合は作成します。
- `shouldGetCustomers()` テストでは、2つの customer レコードをデータベースに挿入し、既存のすべての customer をフェッチしています。そして、customer の数をアサートします。
- 最後に、そのクラスのすべてのテストメソッドの後に実行されるコールバックメソッド `@AfterAll` で postgres コンテナを停止しています。

`CustomerServiceTest` を実行すると、ローカルでまだ利用できない場合は、Testcontainers が DockerHub から Postgres の Docker イメージをプルし、コンテナを起動してテストを実行したことがログで確認できます。

おめでとうございます！！！最初の Testcontainers ベースのテストが実行されました。


## 終わりに

Postgres データベースを使用して Java アプリケーションをテストするために、Testcontainers for Java ライブラリを使用する方法を紹介しました。

Testcontainers を使用して統合テストを作成することは、IDE から実行できる単体テストを作成することと非常によく似ていることが確認できました。また、チームメイトは誰でも、自分のコンピューターに Postgres をインストールしなくても、プロジェクトのクローンを作成してテストを実行できます。

Postgres に加えて、Testcontainers は一般的に使用される多くの SQL データベース、NoSQL データベース、メッセージングキューなどの専用モジュールを提供します。Testcontainers を使用して、テストの任意のコンテナー化された依存関係を実行できます。

Testcontainers の詳細については、https://www.testcontainers.com/ をご覧ください。


## 参考文献

- [Testcontainers container lifecycle management using JUnit 5](https://testcontainers.com/guides/testcontainers-container-lifecycle/)
- [The simplest way to replace H2 with a real database for testing](https://testcontainers.com/guides/replace-h2-with-real-database-for-testing/)
- [Getting started with Testcontainers in a Java Spring Boot Project](https://testcontainers.com/guides/testing-spring-boot-rest-api-using-testcontainers/)


