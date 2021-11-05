# poetry-versing-pluginによる動的バージョン付けを使ったCD構築[github actions]

## poetryでパッケージビルド時に動的にバージョンを付与したい

poetryでのビルド時には、`pyproject.toml`にてversionを指定する必要があります。

この仕様は、Github Actionsで自動ビルド・デプロイを構築しようとしたときに、いちいち`pyproject.toml`を書き換えるためのコミットを打たなくてはいけない部分が障壁となりえます。

もちろんtomlファイルを書き換えるためのスクリプトを作ってもいいのですがめんどくさいのでできればやりたくありませんね。

そのため、今回はpoetry-versing-pluginを使った動的バージョン付与と、それを応用した、GithubActionsからバージョン付けをしたうえでPyPIへの自動デプロイについて説明したいと思います。

## poetry-versing-pluginのインストール

### preview版poetryのインストール

poetry-versing-pluginは、poetryのpluginという機能を使って仮想環境に追加する必要があります。

poetryのplugin機能は、poetryのバージョン1.2以上で追加される機能です。

記事執筆時(2021/11/05)のpoetry latestバージョンは1.1.7であり、まだpluginに対応していません。そのため、previewバージョンのpoetryをインストールするか、すでにインストールしてあるpoetryをアップグレードする必要があります。

従って必要に応じて以下の二つのうちのどちらかを実行してください。

#### プレビュー版poetryのインストールコマンド

基本的には以下のコマンドを打てばインストール完了です。
- Linux
```bash
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py | python - --preview
```

- Windows

```
(Invoke-WebRequest -Uri https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py -UseBasicParsing).Content | python - --preview
```

その後、どちらの環境でもPATHを通す必要があるため、ターミナルの指示に従ってください。

#### プレビュー版poetryへのアップデートコマンド

すでにPATHが通ったpoetry環境が存在している場合は、一からインストールする場合と比べてやや簡単になります。

windows、Linuxのどちらの環境でも以下のコマンドを入力してください。

```
poetry self update --preview
```

### poetry-versing-pluginを追加

poetry-versing-pluginの追加は非常に簡単です。
プロジェクト内で以下のコマンドを実行すればよいだけです。

最低限必要な項目もpyproject.tomlに追加されます。

```
poetry plugin add poetry-version-plugin
```

### おまけ:Dockerfile

一応、ここまでのインストールを行ったDockerfileを記載しておきます。

```
FROM python:3.9.7-buster

RUN apt-get update
RUN apt-get install curl -y

RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py -O

RUN python install-poetry.py --preview --version 1.2.0a2
ENV PATH /root/.local/bin:$PATH
RUN poetry plugin add poetry-version-plugin

CMD [ "poetry install" ]
```

もしも、手元の環境でうまくいかないなどありましたら、デバッグの役に立ててください。

## poetry-versing-pluginの設定

pyproject.tomlの中で、poetry-versing-pluginのを記述する必要があります。
とはいえそんな難しいものではありません（だからこそこのプラグインは素晴らしいといえます）

このプラグインを追加した時点で`pyproject.toml`に`[tool.poetry-version-plugin]`という項目が追加されています。

その下に、「何を使って動的にバージョンを割り振るか」を示すパラメータを記載する必要があります。
現在では2つのパラメータに対応しており、それぞれ紹介します。

### ライブラリの__init__.pyでバージョンを指定する場合

ライブラリ内に`__init__.py`を作る場合があると思います。

この時に`__init__.py`にバージョンを記載することで、コンパイル時のライブラリのバージョンを指定する事が出来ます。

そのための設定は以下に示す通りです。

```
[tool.poetry-version-plugin]
source = "init"
```

この設定の場合、`__init__.py`に以下の追記を行うと`pyproject.toml`で指定したバージョンに関係なく`__init__.py`で指定したバージョンで上書きされてビルドを行います。

```
__version__ = "0.2.3"
```

### gitのtagでバージョンを指定する場合

個人的にはこちらがCDを構築する際にめちゃめちゃ便利で、Releaseアクションとの相性も良いように思います。


そのための設定は以下に示す通りです。

```
[tool.poetry-version-plugin]
source = "git-tag"
```

この設定ではgitのtagに合わせてくれるので、例えば、`git tag 0.0.1`と打った後でビルドを行えば作られる.whlファイルはバージョン0.0.1としてビルドされます。

## Github Actionsでリリースと同時にPyPIにパブリッシュする

### 作りたいGitHub Actionsを考える

※ 完成済みのGitHub Actionsは末尾に記載しております

GitHub Actionsでバージョンでリリースを作って、そのままPyPIに公開する方法をご紹介します。

そのためには毎回バージョンを付与してビルドしておく必要があるわけですが、それをお手軽にできるのがpoetry-versing-pluginの素晴らしさです。

ひとまず、今回作りたいGitHub Actionsの仕様を考えていきます。

- GitHub Actionsの実行時にリリース、ビルド共通のバージョンを与えたい
- 任意のバージョンでプロジェクトをビルドしたい
- 任意のバージョンでGitHubリリースを行い、リリースにはソースファイルのほかに.whlファイルを保持しておきたい。
- GitHubのタグ切りを行っておいてほしい
- PyPIに公開したい

このあたりを1クリックで実行できれば、CI/CDの観点から非常にうれしいです。

### 出来上がったものがこちらになります

今回は3分クッキング方式での説明になります。

先ほど考えた仕様を満たしているものを私が作成いたしましたので、そちらを基に解説を行います。

ひとまず、出来上がったものが以下の通りです。

```yaml
name: release

on:
  workflow_dispatch:
    inputs:
      version:
        description: "Next Version"
        required: true
        default: "x.y.z"
      release_note:
        description: "release note"
        required: false
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v1
        with:
          python-version: 3.9
      - name: Install Poetry
        run: |
          curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py -O
          python install-poetry.py --preview --version 1.2.0a1
      - name: Add path for Poetry
        run: echo "$HOME/.poetry/bin" >> $GITHUB_PATH
      - name: Add Poetry Plugin
        #  poetry plugin add poetry-version-plugin
        run: |
          pip install poetry-version-plugin
      - name: PyPI Settings
        run: |
          poetry config pypi-token.pypi ${{secrets.PYPI_TOKEN}}
      - name: Build Poetry
        run: |
          git tag v${{ github.event.inputs.version }}
          poetry build
          poetry publish
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.event.inputs.version }}
          release_name: Release v${{ github.event.inputs.version }}
          body: |
            ${{ github.event.inputs.release_note }}
          draft: false
          prerelease: false
      - name: Get Name of Artifact
        run: |
          ARTIFACT_PATHNAME=$(ls dist/*.whl | head -n 1)
          ARTIFACT_NAME=$(basename $ARTIFACT_PATHNAME)
          echo "ARTIFACT_PATHNAME=${ARTIFACT_PATHNAME}" >> $GITHUB_ENV
          echo "ARTIFACT_NAME=${ARTIFACT_NAME}" >> $GITHUB_ENV
      - name: Upload Whl to Release Assets
        id: upload-release-asset
        uses: actions/upload-release-asset@v1.0.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ${{ env.ARTIFACT_PATHNAME }}
          asset_name: ${{ env.ARTIFACT_NAME }}
          asset_content_type: application/x-wheel+zip
```

#### 引数によるバージョン入力

上から説明していきますと、まずworkflow_dispatchのinputにて引数を受け取っています。
workflow_dispatchは手動で実行できるトリガーであり、実行時に管理者から任意の引数を受け取ることができます。

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: "Next Version"
        required: true
        default: "x.y.z"
      release_note:
        description: "release note"
        required: false
```

ここで、x.y.zという文字列をフォームにあらかじめ入れておくことで、管理者はフォーマットを間違えることなくバージョンの入力を促します。

![](https://storage.googleapis.com/zenn-user-upload/5d33defb7821e4bc6a349877.png)

また一応リリースノートに含める文面もこちらで入力できるようにしておきます。

これらの値はそれぞれ`${{ github.event.inputs.version }}`と`${{ github.event.inputs.release_note }}`に格納されています。


#### おまじない（リポジトリのクローンとかインストール系）

次にインストール系の作業を行っています。
これは正直おまじないです。

```
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v1
        with:
          python-version: 3.9
      - name: Install Poetry
        run: |
          curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py -O
          python install-poetry.py --preview --version 1.2.0a1
      - name: Add path for Poetry
        run: echo "$HOME/.poetry/bin" >> $GITHUB_PATH
      - name: Add Poetry Plugin
        #  poetry plugin add poetry-version-plugin
        run: |
          pip install poetry-version-plugin
```

一応、上から説明していきますと。

1. `actions/checkout@v2`を使ってリポジトリをクローンする
2. `actions/setup-python@v1`にてpython環境を構築する
3. 先ほど説明したのと同じく、poetryをcurlでとってきてインストールしてパスを通す
4. poetry-versing-pluginをインストールする

ということをやっております。
まぁこういうもんだと思ってください。

#### PyPIのトークンを設定しておく

PyPIに登録を行うとトークンを作成することができます。
今回は、作成したトークンをシークレットトークン（`PYPI_TOKEN`という名前）として登録して、それをpoetryに読み込ませています。


```yaml
      - name: PyPI Settings
        run: |
          poetry config pypi-token.pypi ${{secrets.PYPI_TOKEN}}
```

シークレットトークン登録の方法は[こちら](https://docs.github.com/ja/actions/security-guides/encrypted-secrets)を参照してください。

ここでトークンを登録しておくことで、後はpoetryのデフォルト機能で勝手にライブラリを登録公開してくれます。

あらかじめPyPIでプロジェクトなどを作っておく必要もありません。


#### poetryでプロジェクトをビルド、パブリッシュする

次にビルドとパブリッシュです
とはいえここまでで準備が整っているのでとても簡単なコマンドで実行可能です。

```yaml
      - name: Build Poetry
        run: |
          git tag v${{ github.event.inputs.version }}
          poetry build
          poetry publish
```

ながれとしては、
1. git tag v+{最初に入力したバージョン}(v0.0.0みたいなフォーマット)でタグを切る
2. ビルドを行う
3. パブリッシュする（先ほど指定したトークンのアカウントで自動的に公開される）

という感じです。
この時ビルドされた物はちゃんと入力したバージョンとしてビルドされていますので、パブリッシュした先でも当然入力したバージョンで公開されます。

poetryが関係する部分はこちらまでとなります。

#### GitHubのリリースを作成する

`actions/create-release@v1`を使って、GitHubのリリースを作っていきます。

```
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.event.inputs.version }}
          release_name: Release v${{ github.event.inputs.version }}
          body: |
            ${{ github.event.inputs.release_note }}
          draft: false
          prerelease: false
```

まずは.whlファイルの追加などは行わず、リリースを作っていきます。

特徴としては先ほどpoetryでリリースしたバージョンと同じバージョンを`tag_name`に与えています。また、`release_name`にRelease v+{バージョン}としており、バージョンに一致したタグとリリースが作成されます。

このとき、後で追加のファイルをアップロードするときに使うので、唯一`id: create_release`としてidを振っています。

#### ビルドした.whlファイルをリリースに入れておく

最後にビルドした.whlファイルをリリースに組み込んでおきます

![](https://storage.googleapis.com/zenn-user-upload/b9bac76b3234109906810dbf.png)

こうしておくことで、こんな感じでリリースからダウンロードすることができます。
ちょっとかっこいいですね(データの保存の意味でも重要）

```
      - name: Get Name of Artifact
        run: |
          ARTIFACT_PATHNAME=$(ls dist/*.whl | head -n 1)
          ARTIFACT_NAME=$(basename $ARTIFACT_PATHNAME)
          echo "ARTIFACT_PATHNAME=${ARTIFACT_PATHNAME}" >> $GITHUB_ENV
          echo "ARTIFACT_NAME=${ARTIFACT_NAME}" >> $GITHUB_ENV
      - name: Upload Whl to Release Assets
        id: upload-release-asset
        uses: actions/upload-release-asset@v1.0.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ${{ env.ARTIFACT_PATHNAME }}
          asset_name: ${{ env.ARTIFACT_NAME }}
          asset_content_type: application/x-wheel+zip
```

実は、この段階でビルドされた.whlファイルの名前を知りません。バージョンがファイル名に含まれてしまうため毎回固定の名前というわけでは無いからです。

とはいえ、ビルド後に格納されているディレクトリがデフォルトでは`dist/`であるため、その中にある.whlファイルをとってくればいいのです。

今回はとってきたPATHとファイル名を環境変数（${{ env.ARTIFACT_PATHNAME }},${{ env.ARTIFACT_NAME }}）として保存することで次に行う`actions/upload-release-asset@v1.0.2`に伝えています。

そして最後に`actions/upload-release-asset@v1.0.2`で、PATH、ファイル名とリリースアクションのidを各パラメータに与えることで、リリースにファイルを追加することができます。


これにて、github actionsの解説は終了です。
これでリリースをしたいタイミングでActionsからrun workflowsをぽちっと実行すれば、勝手にPyPIのリリースまでやってくれます。

![](https://storage.googleapis.com/zenn-user-upload/03fd9c2db928e35b737f05d8.png)

## 謝辞

これらの方法は私が所属するチームが参加した「PyTorch Annual Hackathon 2021」にて提出したプロダクト「[prompt2slip](https://github.com/SecHack365-Fans/prompt2slip)」で構築した方法になります。

構築に関わったチームメンバーに感謝するとともに、
当該リポジトリも観ていただけるとうれしく思います。

