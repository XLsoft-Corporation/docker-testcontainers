# Getting started with Testcontainers for Python

[testcontainers\-python — testcontainers 2\.0\.0 documentation](https://testcontainers-python.readthedocs.io/en/latest/) をやってみましょう。


## システム要件

Visual Studio Code（以下 VS Code）で Python コードのテストを行います。以下の環境を用意してください。本ドキュメントの執筆時 2025年 4月の最新版を使用しています。

- **Python:**  
  [Download Python \| Python\.org](https://www.python.org/downloads/) から最新版をダウンロードし、インストールします。
  - 本ドキュメントでの使用バージョン: `3.13.3`
  - 環境変数 `PATH` に Python インストールフォルダが設定されていることを確認
  - `pip` が利用できることを確認

```ps
> python --version
Python 3.13.3

> pip --version
pip 25.0.1 from C:\Program Files\Python313\Lib\site-packages\pip (python 3.13)
```

- **VS Code:**  
  [Download Visual Studio Code \- Mac, Linux, Windows](https://code.visualstudio.com/Download) からダウンロードし、インストールします。
  - 本ドキュメントでの使用バージョン: `1.96.2`
  - 拡張機能として [Python \- Visual Studio Marketplace](https://marketplace.visualstudio.com/items/?itemName=ms-python.python) をインストール

```ps
> code --version
1.99.1
7c6fdfb0b8f2f675eb0b47f3d95eeca78962565b
x64
```


## Virtual Environment の設定

必要に応じて Virtual Environment の設定を行います。

本ドキュメントでは VS Code のコマンド `>Python: Create Environment...` から venv を設定しています。


## サンプルファイルの作成

任意のサブフォルダを用意して、`sample.py` を作成します。

本ドキュメントでは `./demo/python/sample.py` を作成します。

以下のコードをペーストします。

```python
from testcontainers.postgres import PostgresContainer
import sqlalchemy

with PostgresContainer("postgres:16") as postgres:
    psql_url = postgres.get_connection_url()
    engine = sqlalchemy.create_engine(psql_url)
    with engine.begin() as connection:
        version, = connection.execute(sqlalchemy.text("SELECT version()")).fetchone()

print(version)
```

このコードでは

- Testcontainers で `PostgreSQL 16` のコンテナを用意します。
- `get_connection_url` メソッドでコンテナの URL を取得します。
- `sqlalchemy` で SQL 文 "SELECT versoin()" を発行し、結果を標準出力に表示します。

を行っています。


## Testcontainer の実行

コードを動かしてみます。実行に必要なライブラリ `testcontainers[postgres]`, `sqlalchemy` `psycopg2` をインストールします。

```ps
> pip install testcontainers[postgres] sqlalchemy psycopg2
Collecting sqlalchemy
  Downloading sqlalchemy-2.0.40-cp313-cp313-win_amd64.whl.metadata (9.9 kB)
Collecting psycopg2
  Downloading psycopg2-2.9.10-cp313-cp313-win_amd64.whl.metadata (4.8 kB)
Collecting testcontainers[postgres]
  Using cached testcontainers-4.10.0-py3-none-any.whl.metadata (7.8 kB)
Collecting docker (from testcontainers[postgres])
  Using cached docker-7.1.0-py3-none-any.whl.metadata (3.8 kB)
Collecting python-dotenv (from testcontainers[postgres])
  Using cached python_dotenv-1.1.0-py3-none-any.whl.metadata (24 kB)
Collecting typing-extensions (from testcontainers[postgres])
  Using cached typing_extensions-4.13.1-py3-none-any.whl.metadata (3.0 kB)
Collecting urllib3 (from testcontainers[postgres])
  Using cached urllib3-2.3.0-py3-none-any.whl.metadata (6.5 kB)
Collecting wrapt (from testcontainers[postgres])
  Downloading wrapt-1.17.2-cp313-cp313-win_amd64.whl.metadata (6.5 kB)
Collecting greenlet>=1 (from sqlalchemy)
  Downloading greenlet-3.1.1-cp313-cp313-win_amd64.whl.metadata (3.9 kB)
Collecting pywin32>=304 (from docker->testcontainers[postgres])
  Downloading pywin32-310-cp313-cp313-win_amd64.whl.metadata (9.4 kB)
Collecting requests>=2.26.0 (from docker->testcontainers[postgres])
  Using cached requests-2.32.3-py3-none-any.whl.metadata (4.6 kB)
Collecting charset-normalizer<4,>=2 (from requests>=2.26.0->docker->testcontainers[postgres])
  Downloading charset_normalizer-3.4.1-cp313-cp313-win_amd64.whl.metadata (36 kB)
Collecting idna<4,>=2.5 (from requests>=2.26.0->docker->testcontainers[postgres])
  Using cached idna-3.10-py3-none-any.whl.metadata (10 kB)
Collecting certifi>=2017.4.17 (from requests>=2.26.0->docker->testcontainers[postgres])
  Using cached certifi-2025.1.31-py3-none-any.whl.metadata (2.5 kB)
Downloading sqlalchemy-2.0.40-cp313-cp313-win_amd64.whl (2.1 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 16.7 MB/s eta 0:00:00
Downloading psycopg2-2.9.10-cp313-cp313-win_amd64.whl (2.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.6/2.6 MB 14.9 MB/s eta 0:00:00
Downloading greenlet-3.1.1-cp313-cp313-win_amd64.whl (299 kB)
Using cached typing_extensions-4.13.1-py3-none-any.whl (45 kB)
Using cached docker-7.1.0-py3-none-any.whl (147 kB)
Using cached urllib3-2.3.0-py3-none-any.whl (128 kB)
Using cached python_dotenv-1.1.0-py3-none-any.whl (20 kB)
Using cached testcontainers-4.10.0-py3-none-any.whl (107 kB)
Downloading wrapt-1.17.2-cp313-cp313-win_amd64.whl (38 kB)
Downloading pywin32-310-cp313-cp313-win_amd64.whl (9.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 9.5/9.5 MB 21.1 MB/s eta 0:00:00
Using cached requests-2.32.3-py3-none-any.whl (64 kB)
Using cached certifi-2025.1.31-py3-none-any.whl (166 kB)
Downloading charset_normalizer-3.4.1-cp313-cp313-win_amd64.whl (102 kB)
Using cached idna-3.10-py3-none-any.whl (70 kB)
Installing collected packages: pywin32, wrapt, urllib3, typing-extensions, python-dotenv, psycopg2, idna, greenlet, charset-normalizer, certifi, sqlalchemy, requests, docker, testcontainers
Successfully installed certifi-2025.1.31 charset-normalizer-3.4.1 docker-7.1.0 greenlet-3.1.1 idna-3.10 psycopg2-2.9.10 python-dotenv-1.1.0 pywin32-310 requests-2.32.3 sqlalchemy-2.0.40 testcontainers-4.10.0 typing-extensions-4.13.1 urllib3-2.3.0 wrapt-1.17.2
```

`sample.py` を実行します。

```ps
> python test.py
Pulling image testcontainers/ryuk:0.8.1
Container started: 1d33ce3e6890
Waiting for container <Container: 1d33ce3e6890> with image testcontainers/ryuk:0.8.1 to be ready ...
Pulling image postgres:16
Container started: 57a8bf77c6fc
Waiting for container <Container: 57a8bf77c6fc> with image postgres:16 to be ready ...
Waiting for container <Container: 57a8bf77c6fc> with image postgres:16 to be ready ...
PostgreSQL 16.8 (Debian 16.8-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
```

コンテナイメージ `postgres:16` を取得し、実行を待ち、"SELECT versoin()" の結果が得られたことが分かります。

