---
title: "この本について"
---

# この本について

この本を読んでいただく上で、必要な知識やソフトウェア、読了後の目標について説明いたします。

## 本書の対象者

本書は**Djangoについて、入門レベルの知識を身に着けている**ことが前提となります。MVTの基礎、静的なサイトの配信、データベースの参照など**Djangoについての基本知識は解説しません。**

本書はDjangoについてある程度の知識を有する人が、非同期通信を含んだサービスを実装するための知識を最速で得られることを目標としています。

## 必須ソフトウェア

本書では以下のアプリケーションがローカル環境にインストールされている必要があります。
**各アプリケーションのインストール方法については解説しません**が、公式のインストール手順と共に記載します。これらのソフトウェアがインストールされていれば、本書は**Linux、Mac、Windowsで動作可能なサンプル**を提供しています。

- [Docker](https://www.docker.com/)
  - [Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)
  - [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/)
  - [Git - Downloads](https://git-scm.com/downloads)

## ソースコード

以下のリポジトリには各章ごとに区切られたソースコードを格納しています。

@[card](https://github.com/Uno-Takashi/Django-Channels-Book)

フォルダは章の終了時を示しております。例えば、Chapter3フォルダにはChapter3が終了した状態のコード(Chapter4の開始時のコード)が格納されています。

```bash
cd Chapter{X}
docker-compose up -d
```

## 本書で構築する環境

本書では、本番運用を見越して、[Docker Compose](https://docs.docker.com/compose/)を用いて、[nginx](https://www.nginx.co.jp/)、[Redis](https://redis.io/)、[MySQL](https://www.mysql.com/jp/)など、本番環境での運用も可能な構成での開発環境を提供します。

なぜなら、WebSocketを本番環境で運用していく場合、Djangoだけではなくウェブサーバー等も同時に設定する必要があるためです。そのため、本番環境に近い環境でチュートリアルを行う事で、実際のサービスに導入する際の障壁の多くを経験することを目指します。

## 読了後に得られる知識

本書を読了後には以下の知識を習得出来ます。
