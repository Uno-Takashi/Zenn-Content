---
title: "VSCodeとDockerで作る再配布可能なZenn執筆環境（Remote Container+）"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenncli","docker","VSCode","markdown","github"]
published: false
---

## zenn-cliめっちゃべんりですね

こんにちは。

最近Zennデビューしました、U-Notです。

Qiitaから移民して驚いたのですが、[Zenn CLI](https://zenn.dev/zenn/articles/install-zenn-cli)がめちゃめちゃ便利であると同時に、先駆者の皆様が志向を凝らした環境を色々と共有してくれていて、とてもありがたいです。

ただ、自由度の高さ故に、ある程度共通化されている設定を導入するだけでも結構大変なうえに、ローカル環境を汚しそうだと感じました。

なので、今回はVSCodeの拡張機能である[Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)を使って、再配布性、機能性、再現性、容易性に優れた環境の構築と配布します。

今回構築する環境には、以下の機能が含まれます。

- VSCode
    - ほかのワークスペースを汚さないように配慮した設定ファイル
    - [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)を推奨拡張機能に設定
    - [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)を導入することでコンテナ立ち上げ時の手順の簡略化
    - 記事ファイルの作成コマンドをタスクとして容易に実行可能
    - markdown執筆に必要な拡張機能のインストールと設定
- textlintによる日本語の自動校正
- Dockerコンテナによって隔離された環境
    - Zenn-cliをインストール済み
    - textlintでローカルに必要なソフトのインストール済み
    - コンテナを立ち上げた瞬間自動でプレビュー用のサーバーを起動したままにする

これらの条件を満たした、執筆環境を以下のリポジトリで配布しています。

[Zenn-Content-VSCode-Template: VSCodeのRemote - Containersを使用して、Zennの執筆環境を構築したテンプレート](https://github.com/Uno-Takashi/Zenn-Content-VSCode-Template)

先ほど、再配布性と容易性に優れていると記載しましたが、このリポジトリはVSCode以外に[Docker](https://docs.docker.com/get-docker/)及び[Docker Compose](https://docs.docker.com/compose/install/)と[Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)がローカルにインストールされていれば直ちに執筆環境を構築できます。

そして、この3つソフトウェア以外はローカルにインストールする必要がないため、環境を汚すことがありません。

それぞれのインストールが必要であれば、以下のリンクから適宜インストールしてください。

- [Dockerのインストール](https://docs.docker.com/get-docker/)
- [Docker Composeのインストール](https://docs.docker.com/compose/install/)
- [Remote - Containersのインストール](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

[Remote Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)はVSCodeの拡張機能であるため、拡張機能のタブから検索すればインストールできます。また、後で解説いたしますのでこちらでインストールしなくても問題ありません。

もしも、構築方法はどうでもよいので、このリポジトリの環境を使いたいという事でしたら、[こちらの章](#作ったテンプレートで執筆を始める)まで飛ばしてください。

これ以降はしばらく、どのように設定を作るのかについて説明します。

まず事前準備として、GitHubで`README.md`以外は空のリポジトリを作り、ローカルにクローンしてください。

## DockerfileでZenn-cli環境を構築

まず、VSCode云々の話は一旦忘れて、DockerにZenn-cli環境を作ります。



## Remote Containersとは？

Remote Containersとは、VSCodeのプラグインです。

Remote Containerでは、Dockerコンテナ環境でVSCodeを開き、コンテナ内でのみ有効な拡張機能のインストールと設定を作ることができる拡張機能です。また、Dockerコンテナですから、ローカル環境を汚すことなくコンテナ内でPiPYパッケージやnpmパッケージをインストールできます。

![Remote - Container](https://storage.googleapis.com/zenn-user-upload/9867fb4f14db656f56c9e23d.png)

Remote Containersは、Dockerコンテナの持つ再配布性と環境分離能力をVSCodeのワークスペースに拡張できます。

ざっくりと言ってしまえば、各プロジェクトごとに実行環境と拡張機能とエディタ設定を作れるという事です。

### インストールと推薦事項に追加

まず、ローカル環境にRemote Containersをインストールします。

![Remote Containersのインストール](https://storage.googleapis.com/zenn-user-upload/1d4650f3547c8fe6055de12c.png)

VSCodeの拡張機能欄から検索しインストールします。

同時にワークスペースの「推薦事項に追加する」をしておきます。

![推薦事項に追加](https://storage.googleapis.com/zenn-user-upload/e1a65d3456955019f90e0887.png)

こうしておくと、Remote Containersをインストールしていない人がワークスペースを開いたときに、右下にホップアップが現れインストールを促します。

## 作ったテンプレートで執筆を始める

構築済みのテンプレートを以下のリポジトリからクローンするか、

[Zenn-Content-VSCode-Template: VSCodeのRemote - Containersを使用して、Zennの執筆環境を構築したテンプレート](https://github.com/Uno-Takashi/Zenn-Content-VSCode-Template)

## 参考文献

- [Developing inside a Container using Visual Studio Code Remote Development](https://code.visualstudio.com/docs/remote/containers)