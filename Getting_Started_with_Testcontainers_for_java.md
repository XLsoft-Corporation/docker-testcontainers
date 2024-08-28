# Getting started with Testcontainers for Java

[Getting started with Testcontainers for Java](https://testcontainers.com/guides/getting-started-with-testcontainers-for-java/) をやってみましょう。

## システム要件

Visual Studio Code（以下 VS Code）で maven のプロジェクトのテストを行います。以下の環境を用意してください。本ドキュメントの執筆時 2024年8月の最新版を使用しています。

- **VS Code:**  
[Download Visual Studio Code \- Mac, Linux, Windows](https://code.visualstudio.com/Download) からダウンロードし、インストールします。
  - 本ドキュメントでの使用バージョン: `1.92.2`
- **JDK:**  
[Eclipse Temurin \| Adoptium](https://adoptium.net/temurin/) から最新版をダウンロードし、インストールします。（OpenJDK の実装は VS Code の Extension Pack for Java のドキュメントで紹介されていたので Temurin にしています。）
  - 本ドキュメントでの使用バージョン: `21.0.4`
  - 環境変数 `JAVA_HOME` を追加して JDK インストールフォルダを設定
  - 環境変数 `PATH` に `%JAVA_HOME%\bin` を設定
- **maven:**  
[Maven – Download Apache Maven](https://maven.apache.org/download.cgi) から最新版をダウンロードし、任意のフォルダに展開します。
  - 本ドキュメントでの使用バージョン: `3.9.9`
  - 環境変数 `PATH` に maven のインストールフォルダ以下の `bin` フォルダを設定




### Windows での注意点

PowerShell で mvn コマンドを実行する際には `.` が誤認識されるので `""` で括る。（コマンドプロンプトでは不要…）

```powershell
mvn archetype:generate -DgroupId="com.mycompany.app" -DartifactId="my-app" -DarchetypeArtifactId="maven-archetype-quickstart" -DarchetypeVersion="1.5" -DinteractiveMode="false"
```


## Create a Java project with Maven

次の内容で Java プロジェクトを作成します。

```powershell
mvn archetype:generate -DgroupId="com.testcontainers.demo" -DartifactId="demo" -DarchetypeArtifactId="maven-archetype-quickstart" -DarchetypeVersion="1.5" -DinteractiveMode="false"