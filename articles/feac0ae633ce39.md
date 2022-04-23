---
title: "django-crontabで環境変数が参照出来ずに色々と躓いた話"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker","Django","Python","crontab"]
published: true
---

# django-crontabで環境変数が参照出来ずに色々と躓いた話

django-crontabの設定で1日を無駄にしたため、備忘録として躓きポイントを残しておきます。

特に環境変数周りは落とし穴になるかと思います。

## django-crontabとは？

djangoをいじっていて、定期的に実行したい操作があったとします。
例えば、『データベース上で1年以上保管したデータは捨てる』といった処理です。

一般的には定期実行にcronを使う事になりますが、pythonから離れて設定する必要があるうえに、djangoのカスタムコマンドを作る手間なども存在します。

cronのインストールさえ終わっていれば、手軽に定期実行を可能にするのが、django-crontabというライブラリです。

ただ、エラーが発生するとdjangoのエラーなのか、crontabのエラーなのか、定期実行タスクのエラーなのかの切り分けが難しいという問題があります。

## 躓きポイント

個人的に躓いていったポイントをひたすら列挙します。

### cronのインストールと起動

環境によってはcronのインストールを行います。
何となくデフォルトでインストールと有効化がされている気がしますが、インストールと有効化を行う必要があります。

Dockerfileであれば、以下の2行を追加する必要があります。

```bash
RUN apt-get update
RUN apt-get install -y cron
```

また、コンテナ起動後に`service cron start`を実行する必要があります。
コマンド or エントリーポイントとして記載しておきましょう。

### printが出力されない

単にテスト用に`print("hello")`だけをする関数を定期タスクとして登録した場合、標準出力として表示されるような気がしますが、そんなことはありません。

出力結果を確認したい場合、ジョブ登録時にログファイルとして出力先のファイルを指定する必要があります。

```python
CRONJOBS = [
    ("* * * * *", "d_party.cron.my_scheduled_job", ">> /var/log/cron.log"),
]
```

また、デバッグ用にエラー結果も確認したい場合は、setting.pyでsuffixを追加しておく必要があります。

```python
CRONTAB_COMMAND_SUFFIX = "2>&1"
```

### 実行時のディレクトリが違いimport出来ない

この問題は`crontab -l`を実行して登録されているコマンドをコピペして実行したら動くけど、定期実行ではエラーが出てしまうという状況の原因その１でした。

crontabでは、rootユーザーのホームディレクり上で実行されます。

djangoプロジェクトをrootユーザーのホームディレクトリ以外にマウントしている場合、正しくimport出来ません。

環境変数のPYTHONPATHにディレクトリを登録するなどの回避方法があります。

dockerfileで行う場合は以下のように配置ディレクトリを入れる登録しておけます。

```dockerfile
ENV PYTHONPATH /usr/src/app
```

### 環境変数が参照できない

この問題は`crontab -l`を実行して登録されているコマンドをコピペして実行したら動くけど、定期実行ではエラーが出てしまうという状況の原因その２でした。

pythonで環境変数を参照するコードを書いていた場合、cronで実行するときに環境変数が参照できない問題があります。

これは、django-crontabやdjangoに起因する問題ではなく、cronの仕様に起因する問題です。

cronは最小限の環境変数しか設定されていないshellで実行されます。

自分は、環境ごとに差し替える必要のある値はsettings.pyに直接書かずに、.envファイルとして書いておき、それをDockerfileやdocker-compose.ymlでコンテナの環境変数に設定して、settings.pyでは環境変数を設定変数に読み出す方式を採用していました。

django-crontabでは変数をdjangoの環境下で動作させるために、これらのsettigs.pyやmanatge.pyを実行しています。

その中に環境変数を読みだしている部分があると、正しく動作しないという問題に遭遇しました。

自分は[django-dotenv](https://github.com/jpadilla/django-dotenv)を使って、manage.pyの中で、環境変数を読み込む方式を採用しました。

こうすることで、cronの実行時にも環境変数が自動的に設定されます。

```diff python
+import dotenv

def main():
+   dotenv.load_dotenv("/env_files/.env.django", verbose=True, override=True)

```

一応注意点としては、env_fileはルートディレクトリからパスを書くことをお勧めします。
先ほど言ったようにrootユーザーのホームディレクトリで実行されるため、思わぬ副作用を生み出さないためです。

## まとめ

躓きポイントは色々とありますが、まずはエラーとprintを確認できるように設定して、そのエラーをつぶしていくという流れになるかと思います。

本記事で皆さんの躓きが少しでも減ってくれればうれしいです。
