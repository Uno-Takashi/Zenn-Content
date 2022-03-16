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

とはいえ、本書はdjang-channelsのチュートリアルであって、djangoのチュートリアルではありません。そのため、djangoの環境構築方法については説明せずに、djangoの最低限の設定とつなぎこみを行った環境を配布します。

本章では、配布した環境を基にして、そこからdjango-channelsの環境を構築していく方法を示します。

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

## コンテナ作成

ひとまず、最低限の設定を行っている状態のコンテナを立ち上げてみましょう。

まずは任意の作業ディレクトリを作成しその中に移動してください

```bash
mkdir backend
cd backend
```

ディレクトリの分け方に明確なルールは存在しません。本書ではコンテナ単位でディレクトリを区切る事にします。フレームワーク名やイメージ名をそのままディレクトリ名として使用し、4つのディレクトリを作成します。

```bash
mkdir Django
mkdir nginx
mkdir mySQL
mkdir redis
```

### mysqlコンテナ設定

まずは、設定が一番簡単な、mysqlコマ
