---
title: "2. モダンな環境を構築しよう"
---

# Django-Channels環境構築 with Docker

本章において構築した環境は以下のリポジトリのchapter2フォルダに格納しています。

[Uno-Takashi/Django-Channels-Book: 『django-channelsで作る非同期通信アプリケーション入門』のサンプルコード置き場](https://github.com/Uno-Takashi/Django-Channels-Book)

本章では、Dockerで構築したDjango環境に対して、django-channelsを追加し、asgiアプリケーションとして動作させる設定を行います。

## 必須ソフトインストール

本書では以下のアプリケーションがローカル環境にインストールされている必要があります。
各アプリケーションのインストール方法については解説を行いませんが、公式のインストール手順と共に記載します。

- [Docker](https://www.docker.com/)
  - [Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)
  - [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/)
  - [Git - Downloads](https://git-scm.com/downloads)

## 環境の概要

何はともあれ環境構築を行っていきます。
本書では、スケーラビリティと環境差分について考慮して[Docker](https://www.docker.com/)、[Docker Compose](https://docs.docker.com/compose/)で開発します。

また、実運用を見越してウェブサーバーやasgiサーバーを導入した本格的な構成の環境を構築し使用します。

とはいえ、本書はDjang Channelsのチュートリアルであって、Djangoのチュートリアルではありません。そのため、Djangoの環境構築方法については説明せずに、Djangoの最低限の設定とつなぎこみを行ったコンテナ環境を配布します。

2章では、配布したDocker環境を基にして、そこからdjango-channelsの環境を構築していく方法を示します。

### 使用技術・目標構成

ひとまず、主要な技術としては以下の技術を用いた構成を目指します。

- ウェブサーバー
  - [nginx](https://www.nginx.com/)
- データベース
  - [MySQL](https://www.mysql.com/jp/)
- KVS
  - [Redis](https://redis.io/)
- ウェブアプリケーション
  - [Python](https://www.python.org/) ( 3.10^ )
  - django( 4.0^ )
  - channels
  - gunicorn
  - uvicorn
  - poetry

上記の構成技術を用いて、2章終了時には次のような構成で動作するコンテナを構築します。

![構成](/images/django-channels-book/structure.png)

これらの必要なコンテナが、1コマンドで立ち上げられるように[Docker Compose](https://docs.docker.com/compose/)を使用します。
`docker-compose up -d`で立ち上げたら、コンテナが自動で立ち上がり、サービスにアクセス可能な状態を構築することを考えます。

このうち、nginx、mysql、redisは[Docker Hub](https://hub.docker.com/)にて公式イメージが提供されています。そのため、公式のイメージをそのまま使い、設定ファイルを入れ込むだけで完成します。

djangoコンテナはpythonのベースイメージから必要なライブラリをpoetryを用いてインストールして作成するために、`Dockerfile`を作成しています。

## Dockerコンテナ立ち上げ

ひとまず、最低限の設定を行っている状態のコンテナを立ち上げてみましょう。

### Githubからプロジェクトをクローン

まず、Gitを使ってサンプルプロジェクトをクローンします。

[Uno-Takashi/Django-Channels-Book: 『django-channelsで作る非同期通信アプリケーション入門』のサンプルコード置き場](https://github.com/Uno-Takashi/Django-Channels-Book)

作業用の任意のディレクトリに移動したのち以下のコマンドを実行します。

```
git clone https://github.com/Uno-Takashi/Django-Channels-Book.git
```

すると中にはChapterごとに分けられたディレクトが存在します。

```
cd Django-Channels-Book
ls
```

それぞれのディレクトリにははChapterが終わった時の成果物が格納されています。
従って、本章で使う最低限の環境はChapter1になりますのでcdコマンドでChapter1に移動します。

(Chapter2には『2章が終わった直後』のコードが格納されていますので注意してください)

```bash
cd Chapter1
```

軽くtreeコマンドで中身を確認してみましょう。

```bash
tree -L 3
.
├── .gitignore
├── Django
│   ├── .env.django
│   ├── .gitignore
│   ├── Dockerfile
│   ├── channels_tutorial
│   ├── db.sqlite3
│   ├── entorypoint.sh
│   ├── gunicorn.conf.py
│   ├── manage.py
│   ├── pyproject.toml
│   └── static
├── MySQL
│   ├── .env.mysql
│   ├── .gitignore
│   ├── data
│   └── my.cnf
├── Redis
│   └── .redis.env
├── docker-compose.yml
└── nginx
    ├── .gitignore
    ├── mime.types
    └── nginx.conf
```

### コンテナ立ち上げ

ではこの環境を立ち上げてみましょう。とはいっても、Docker Composeを使ってビルドしてから立ち上げるだけです。

```
docker-compose build
docker-compose up -d
```

`docker-compose ps`を使って、立ち上がっているコンテナを表示してみると、先ほど見たディレクトリ単位で、コンテナが立ち上がっているのを確認できます。

```bash
docker-compose ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
Django              "sh entorypoint.sh"      django              running             0.0.0.0:8080->8080/tcp
MySQL               "docker-entrypoint.s…"   mysql               running             0.0.0.0:3306->3306/tcp
Redis               "docker-entrypoint.s…"   redis               running             0.0.0.0:6379->6379/tcp
nginx               "/docker-entrypoint.…"   nginx               running             0.0.0.0:80->80/tcp
```

この時、[localhost](http://localhost/)にアクセスすると、Djangoのプロジェクトを作ったばかりにデフォルト画面が表示されます。

![Django デフォルト画面](/images/django-channels-book/django_default.png)

また、80ポートからのアクセスはnginxを経由してのアクセスになりますが、8080ポートにはデバッグ用にDjangoコンテナをホストしています。そのため、[localhost:8080](http://localhost:8080/)にアクセスしても同様の画面が表示されます。

### django-channelsのインストール

立ち上げたDjangoコンテナにdjango-channelsをインストールします。
Djangoコンテナではpipではなくpoetryというパッケージマネージャーを使っています。

poetry環境では、`poerty add PACKAGE`を実行すると環境にパッケージをインストールしたのちに、pyproject.tomlファイルに必要なパッケージが書き込まれます。pyproject.tomlはpipで言うところのrequirement.txtの役割を持ち、そのプロジェクトに必要なパッケージを記載しておきます。

まずは、Djangoコンテナでpoetryを使ってdjango channelsをインストールします。

:::message alert
django-channelsというchannelsとは別のライブラリが存在します。
気を付けてください。
:::

```
docker-compose exec django poetry add channels
```

pyproject.tomlはDockerfileで読み込んでいるため、再び環境のビルドを行い、新規で立ち上げ直します。

```
docker-compose down
docker-compose build
docker-compose up -d
```

`poetry show`コマンドを使ってインストールされているパッケージを可視化してみると、ビルドしなおした環境にdjango-channelsが追加されていることが確認できます。

```bash
docker-compose exec django poetry show

black             21.12b0 The uncompromising code formatter.
cffi              1.15.0  Foreign Function Interface for Python calling C code.
channels          3.0.4   Brings async, event-driven capabilities to Django. Django 2.2 and up only.
click             8.0.4   Composable command line interface toolkit
constantly        15.1.0  Symbolic constants in Python
```

また、pyproject.tomlを確認してみると、poetry.dependenciesの最後にdjango-channelsが追加されていることが確認できます。

```toml
[tool.poetry.dependencies]
python = "^3.10"
Django = "^4.0.0"
gunicorn = "^20.1.0"
mysqlclient = "^2.1.0"
uvicorn = {extras = ["standard"], version = "^0.16.0"}
pydantic = "^1.9.0"
channels = "^3.0.4"
```

これ以外のパッケージも必要に応じて同様の手順でインストールできます。また、poetryでは自動的にバージョンによる依存関係を確認して、適切なバージョンを引っ張ってきてくれますので、中長期的な運用を見据えて導入を検討するのもよいかと思います。

ちなみに皆さんが独自に用意している環境にpipを使ってインストールしたいのであれば、以下のコマンドで可能です。

```bash
pip install channels
```
