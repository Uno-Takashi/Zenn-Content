---
title: "2. モダンな環境を構築しよう"
---

# Django環境構築 with Channels

本章において構築した環境は以下のリポジトリのchapter2フォルダに格納しています。

## 環境の概要

まずは、何はともあれ環境構築を行っていきます。

websocketを実行したい場合、ウェブサーバーやウェブアプリケーションにも設定を行う必要があります。

本書では本番環境を見据え、django（ウェブアプリケーション）だけでなくウェブサーバやデータベースまで組み込んだ環境をDockerを用いて構築します。

### 使用技術

ひとまず、主要な技術としては以下の構成を目指します。

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
