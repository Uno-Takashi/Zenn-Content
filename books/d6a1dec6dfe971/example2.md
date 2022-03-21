---
title: "2. モダンな環境を構築しよう"
---

# Django-Channels環境構築 with Docker

本章において構築した環境は以下のリポジトリのchapter2フォルダに格納しています。

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
本書では、スケーラビリティと環境差分を無くすために[Docker](https://www.docker.com/)、[Docker Compose](https://docs.docker.com/compose/)での開発を行います。

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

![構成](/images/django-channels-book/structure.png)

これらの必要なコンテナが、1コマンドで立ち上げられるように[Docker Compose](https://docs.docker.com/compose/)を使用します。
`docker-compose up -d`で立ち上げたら、コンテナが自動で立ち上がり、サービスにアクセス可能な状態を構築することを考えます。

このうち、nginx、mysql、redisは[Docker Hub](https://hub.docker.com/)にて公式イメージが提供されています。そのため、公式のイメージをそのまま使い、設定ファイルを入れ込むだけで完成します。

djangoコンテナはpythonのベースイメージから必要なライブラリをpoetryを用いてインストールして作成するために、`Dockerfile`を作成します。

## Dockerコンテナ立ち上げ

ひとまず、最低限の設定を行っている状態のコンテナを立ち上げてみましょう。

### gitからプロジェクトをクローンしてくる

まずはgitを使ってサンプルプロジェクトをクローンしてきます。

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

すると、先ほどの図１で用意した、コンテナ単位でディレクトリが作成されているかと思います。

### django-channelsのインストール

立ち上げたDjangoコンテナにdjango-channelsをインストールします。
Djangoコンテナではpipではなくpoetryというパッケージマネージャーを使っています。

poetry環境では、`poerty add PACKAGE`を実行するとpyproject.tomlファイルに必要なパッケージが書き込まれます。pyproject.tomlはpipで言うところのrequirement.txtの役割を持ち、そのプロジェクトに必要なパッケージを記載しておきます。

```
docker-compose exec django 
```

皆さんが独自に用意している環境にpipを使ってインストールしたいのであれば、以下のコマンドを入力してください。

```
pip install django-channels
```
