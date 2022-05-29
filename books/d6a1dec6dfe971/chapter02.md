---
title: "2. モダンなChannels環境を構築しよう"
---

# Django Channels環境構築 with Docker

本章において構築した環境は以下のリポジトリのchapter2フォルダに格納しています。

[Uno-Takashi/Django Channels-Book: 『django-channelsで作る非同期通信アプリケーション入門』のサンプルコード置き場](https://github.com/Uno-Takashi/Django-Channels-Book)

2章では、**Dockerで構築したDjango環境に対して、Django Channelsを追加し、asgiアプリケーションとして動作させる**設定をします。

## 必須ソフトインストール

本書では以下のアプリケーションがローカル環境にインストールされている必要があります。
**各アプリケーションのインストール方法については解説しません**が、公式のインストール手順と共に記載します。

- [Docker](https://www.docker.com/)
  - [Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)
  - [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/)
  - [Git - Downloads](https://git-scm.com/downloads)

## 環境の概要

何はともあれ環境構築していきます。

本書では、スケーラビリティと環境差分について考慮して[Docker](https://www.docker.com/)、[Docker Compose](https://docs.docker.com/compose/)で開発します。

また、実運用を見越してウェブサーバーやasgiサーバーを導入した本格的な構成の環境を構築し使用します。

とはいえ、本書は**Django Channelsのチュートリアルであって、Djangoのチュートリアルではありません**。そのため、Djangoの環境構築方法については説明せずに、Djangoの最低限の設定とつなぎこみを行ったDockerコンテナ環境を配布します。

2章では、配布したDocker環境を基にして、そこからDjango Channelsの環境を構築していく方法を示します。

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
*図2-1 想定しているコンテナ構成図*

これらの必要なコンテナが、1コマンドで立ち上げられるように[Docker Compose](https://docs.docker.com/compose/)を使用します。
`docker-compose up -d`で立ち上げたら、コンテナが自動で立ち上がり、サービスにアクセス可能な状態を構築することを考えます。

このうち、nginx、MySQL、Redisは[Docker Hub](https://hub.docker.com/)にて公式イメージが提供されています。そのため、公式のイメージをそのまま使い、設定ファイルを入れ込むだけで完成します。

djangoコンテナはPythonのベースイメージから必要なライブラリをpoetryを用いてインストールして作成するために、`Dockerfile`を作成しています。

## Dockerコンテナ立ち上げ

ひとまず、最低限の設定をしている状態のコンテナを立ち上げてみましょう。

### Githubからプロジェクトをクローン

まず、Gitを使ってサンプルプロジェクトをクローンします。

[Uno-Takashi/Django-Channels-Book: 『django-channelsで作る非同期通信アプリケーション入門』のサンプルコード置き場](https://github.com/Uno-Takashi/Django-Channels-Book)

作業用の任意のディレクトリに移動したのち以下のコマンドを実行します。

```bash
git clone https://github.com/Uno-Takashi/Django-Channels-Book.git
```

すると中にはChapterごとに分けられたディレクトが存在します。

```bash
cd Django-Channels-Book
ls
Chapter1  Chapter2  Chapter3  Chapter4  Chapter5  LICENSE  README.md
```

それぞれのディレクトリにはChapterが終わった時の成果物が格納されています。
従って、2章で使う最低限の環境は**Chapter1**になりますのでcdコマンドでChapter1に移動します。

```bash
cd Chapter1
```

:::message alert
Chapter2には『2章が終わった直後』のコードが格納されていますので注意してください
:::

treeコマンドでChapter1ディレクトリ中身を確認してみましょう。

```bash
tree -L 3
.
├── .gitignore
├── Django
│   ├── .env.django
│   ├── .gitignore
│   ├── Dockerfile
│   ├── channels_tutorial
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

Chapter1ディレクトリを見ると、図2-1で示した構成図に書かれているコンテナ単位でディレクトリが区切られていることを確認できます。
また、コンテナ内の各種設定値は`.env`から始まるファイルに格納されています。

### コンテナ立ち上げ

ひとまず、この環境を立ち上げてみましょう。とはいっても、Docker Composeを使ってビルドしてからupするだけです。

```bash
docker-compose build
docker-compose up -d
```

`docker-compose ps`を使って、立ち上がっているコンテナを表示してみると、先ほど見たディレクトリ単位で、4つのコンテナが立ち上がっているのを確認できます。

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
*Djangoのデフォルト画面。問題なく表示されていればコンテナの立ち上げに成功している*

また、80ポートからのアクセスはnginxを経由してのアクセスになりますが、8080ポートにはデバッグ用にDjangoコンテナをホストしています。そのため、[localhost:8080](http://localhost:8080/)にアクセスしても同様の画面が表示されます。

### Django Channelsのインストール

立ち上げたDjangoコンテナにDjango Channelsをインストールします。
Djangoコンテナではpipではなくpoetryというパッケージマネージャーを使っています。

poetry環境では、`poerty add PACKAGE`を実行すると環境にパッケージをインストールしたのちに、pyproject.tomlファイルに必要なパッケージが書き込まれます。pyproject.tomlはpipで言うところのrequirement.txtの役割を持ち、そのプロジェクトに必要なパッケージを記載しておきます。

まずは、Djangoコンテナでpoetryを使ってDjango Channelsをインストールします。同時に、Redisをchannelsのバックエンドとして使うためにchannels-redisをインストールします。

```bash
docker-compose exec django poetry add channels channels-redis
```

:::message alert
django-channelsというchannelsとは別のライブラリが存在します。
間違えてインストールしないようにしましょう。
:::

pyproject.tomlはDockerfileで読み込んでいるため、再び環境のビルドを行い、新規で立ち上げ直します。

```bash
docker-compose down
docker-compose build
docker-compose up -d
```

:::message alert
ここでビルドしておかないと再び立ち上げたときに、channelsがインストールされていない状態で立ち上げってしまいます。
:::

`poetry show`コマンドを使ってインストールされているパッケージを可視化してみると、ビルドしなおした環境にDjango Channelsが追加されていることが確認できます。

```bash
docker-compose exec django poetry show

black             21.12b0 The uncompromising code formatter.
cffi              1.15.0  Foreign Function Interface for Python calling C code.
channels          3.0.4   Brings async, event-driven capabilities to Django. Django 2.2 and up only.
click             8.0.4   Composable command line interface toolkit
constantly        15.1.0  Symbolic constants in Python
```

また、pyproject.tomlを確認してみると、poetry.dependenciesの最後にDjango Channelsが追加されていることが確認できます。

```diff toml: pyproject.toml
[tool.poetry.dependencies]
python = "^3.10"
Django = "^4.0.0"
gunicorn = "^20.1.0"
mysqlclient = "^2.1.0"
uvicorn = {extras = ["standard"], version = "^0.16.0"}
pydantic = "^1.9.0"
+channels = "^3.0.4"
+channels-redis = "^3.4.0"
```

これ以外のパッケージも必要に応じて同様の手順でインストールできます。また、poetryでは自動的にバージョンによる依存関係を確認して、適切なバージョンを引っ張ってきてくれますので、中長期的な運用を見据えて導入を検討するのもよいかと思います。

ちなみに皆さんが独自に用意している環境にpipを使ってインストールしたいのであれば、以下のコマンドでインストール出来ます。

```bash
pip install channels
```

### settings.pyの変更

ここからは、ようやくDjangoコンテナ内のファイルを編集していきます。
settings.pyを編集して、**channelsの有効化、Redisへの接続**をします。

#### INSTALL_APPの追加

先ほど環境にインストールしたDjango ChannelsをINSTALLED_APPSに追加します。

```diff python: settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
+   'channels'
]
```

#### Redisバックエンドへの接続

Django Channelsでは、接続状態など一時的に保持すべき情報を監理するためのバックエンドを必要とします。設定しまえば、開発時にバックエンドの存在を意識することはほとんどありません。

共通のバックエンドを参照する方式を用いることで、ウェブアプリケーションやウェブサーバを多重化した構成においても、非同期通信を実現します。

公式ドキュメントでは、Django公式が開発しているchannels_redisを使用する方法を記載しています。本書でも公式ドキュメントに準拠しchannels_redisを使用します。

settings.pyにて、バックエンドに使用するChannelLayerと、接続に必要なコンフィグを入力します。
docker-composeでは、コンテナ名で名前解決できるため『Redis』を指定し、割り当てているポート番号を指定します。

今回は以下のコードをsettings.pyに追記することで、Redisをバックエンドに使用するための設定が完了します。

```diff python: settings.py
+CHANNEL_LAYERS = {
+   "default": {
+       "BACKEND": "channels_redis.core.RedisChannelLayer",
+       "CONFIG": {"hosts": [("redis", 6379)]},
+   }
+}
```

:::message
[channels_postgres](https://github.com/danidee10/channels_postgres)などのサードパーティー製ライブラリを用いることで、Redis以外をバックエンドに用いることができます。
:::

:::message alert
バックエンドにインメモリを用いることができますが、スケーラビリティの観点から、本番環境での使用は想定されていません。インメモリはマルチインスタンス環境では最適な動作を行えません。テストなど限定された用途で用いるべきです。
:::

### asgiアプリケーションとして起動

現状ではまだ、wsgiアプリケーションとして動作しています。gunicornの設定が記載してある、gunicorn.conf.pyを見ると、`wsgi_app = "channels_tutorial.wsgi:application"`と記載されており、wsgi.pyを参照しているのが見て取れます。

また、`django-admin startproject APP_NAME`を使用したデフォルトではデバッグモードもwsgiアプリケーションとして動作しています。

そのため、asgiアプリケーションとして動作させるための設定する必要があります。

#### asgi.pyの参照

asgi.py自体は`django-admin startproject APP_NAME`によってプロジェクトを作成した時に自動的に作成されています。ただし、どこからも参照されていないので無意味なファイルです。

gunicorn.conf.pyでも同様にasgi.pyを参照していません。そこでguicorn.conf.pyの、`wsgi_app = "channels_tutorial.wsgi:application"`を`wsgi_app = "channels_tutorial.asgi:application"`に書き換える事で、asgi.pyを参照出来るようになります。

ただ、この変更だけではサーバの立ち上げ時エラーになってしまいます。なぜなら、gunicornはasgiサーバーではないからです。

そのため、ワーカーとしてuvicornを指定する必要があります。
従って、gunicorn.conf.pyに以下の変更する事で、asgiアプリケーションとしてDjangoを起動できます。

```diff python: settings.py
-wsgi_app = "channels_tutorial.wsgi:application"
+wsgi_app = "channels_tutorial.asgi:application"
+worker_class = "uvicorn.workers.UvicornWorker"
```

ここまで設定すると『なんでgunicorn使ってるねん!!全部uvicornでええじゃん』って話になってしまいます。しかしながら、**gunicornを経由してuvicornを使用**することにメリットが存在します。

gunicornは枯れた技術であるため、安定感のあるワーカー管理には定評があります。worker_classにuvicornを指定することにより、各ワーカーは最新技術を搭載したのuvicornで動作し、その管理は枯れた技術であるgunicornで動作します。これにより、UvicornとGunicornのいいとこどりが出来るのです。

:::message
この辺の技術動向は執筆時(2022年4月)での話であり、時代のトレンドに合わせた方法を読者には選択してもらいたいです。
:::

#### settigs.pyに追記

次に、setting.pyのAIGI_APPLICATIONにasgi.pyを指定します。

```diff python: settings.py
-WSGI_APPLICATION = 'channels_tutorial.wsgi.application'
+ASGI_APPLICATION = 'channels_tutorial.asgi.application'
```

#### wsgi.pyの削除

必須ではありませんが残しておく理由もありませんので、wsgi.pyは削除しておきましょう。

#### AIGIとして動作していることを確認

runserverを行った場合に、asgiアプリケーションとして動作していることを確認します。

```diff env: .env.django
-DEBUG=0
+DEBUG=1
```

デバッグモードで再起動すると、DjangoコンテナがASGI/Channelsとして起動していることを確認できます。

```bash
docker-compose down
docker-compose up -d
docker-compose logs django
Django  | Watching for file changes with StatReloader
Django  | Performing system checks...
Django  |
Django  | System check identified no issues (0 silenced).
Django  | May 04, 2022 - 01:50:01
Django  | Django version 4.0.4, using settings 'channels_tutorial.settings'
Django  | Starting ASGI/Channels version 3.0.4 development server at http://0.0.0.0:8080/
Django  | Quit the server with CONTROL-C.
```

また、RedisがChannelsのバックエンドとして設定されているかについて確認します。Django shellに入ります。

```bash
docker-compose exec django poetry run python manage.py shell
```

shellに対して以下のコマンドを入力します。

```pycon
>>> import channels.layers
>>> channel_layer = channels.layers.get_channel_layer()
>>> channel_layer
<channels_redis.core.RedisChannelLayer object at 0x7f422b7a7c10>
```

この状態で、channel_layerを取得できていれば、channelsはRedisをバックエンドとして問題無く動作しています。
