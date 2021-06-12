ビルド：
docker-compose build

イメージ確認：
docker-compose images

docker-compose ps

★接続
docker-compose exec app bash



実行
docker-compose exec app go run main.go



---
title: DockerでGoの開発環境を構築する
tags: Go Docker docker-compose
author: uji_
slide: false
---
Go言語の勉強を始めたので備忘録がてら書いてみます。
今回は環境構築のみですが、Go言語でのAPIサーバーの開発編を続編で書けたらいいなと思ってます。

# なぜDockerで？

Dockerでの環境構築は、様々なメリットが存在します。

1. どのOSでも簡単に環境構築が出来る
2. ホストOSと開発アプリが隔離されるので安全
3. デプロイやテストが楽になる
4. ホストOSにインストールするソフトウェアが減る

Dockerは最高です。

# そもそもDockerとは？

コンテナ型の仮想環境を扱うためのプラットフォームです。
コンテナと呼ばれる軽量、高速に動作する仮想マシンのようなものを作成、配布、実行することができます。
詳細についてはさくらナレッジの記事がわかりやすいのでそちらをご覧ください。
>[さくらナレッジ
Docker入門（第一回）～Dockerとは何か、何が良いのか～](https://knowledge.sakura.ad.jp/13265/)

# Dockerのインストール
Windows と Mac は docker hub の公式ページでインストーラーがダウンロードできます。

- [windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
- [mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

Linux系は`apt-get`や`yum`を使用し、インストール出来ます。

詳細については、以下の記事が参考になります。
> - [Dockerインストールメモ @n-yamanaka](https://qiita.com/n-yamanaka/items/ddb18943f5e43ca5ac2e)
> - [LinuxにDockerをインストールする @yoshiyasu1111](https://qiita.com/yoshiyasu1111/items/f2cab116d68ed1a0ce13)

# 環境構築

はじめに、端末で作業ディレクトリを作成し、そこに移動します。

```shell
mkdir work # workディレクトリ作成
cd work # 作成したworkディレクトリに移動
```

## Dockerコンテナの設定を書く

Dockerは、docker hubで配布されているDockerイメージをもとに、コンテナのインスタンスを作成することが出来ます。
今回は、配布されているDockerイメージをそのまま使うのではなく、`Dockerfile`を駆使してカスタマイズしたイメージを作成します。

また、docker-composeというツールを使用することで、コンテナのビルドから連携までの設定をyml形式のファイルベース(`docker-compose.yml`)に記述し、自動化できます。

作業ディレクトリに`Dockerfile`と`docker-compose.yml`を作成します。
作成したファイルをお好きなエディタで開き、編集していきます。

```Dockerfile:Dockerfile
# ベースとなるDockerイメージ指定
FROM golang:latest
# コンテナ内に作業ディレクトリを作成
RUN mkdir /go/src/work
# コンテナログイン時のディレクトリ指定
WORKDIR /go/src/work
# ホストのファイルをコンテナの作業ディレクトリに移行
ADD . /go/src/work
```

```yaml:docker-compose.yml
version: '3' # composeファイルのバーション指定
services:
  app: # service名
    build: . # ビルドに使用するDockerfileがあるディレクトリ指定
    tty: true # コンテナの起動永続化
    volumes:
      - .:/go/src/work # マウントディレクトリ指定
```

`Dockerfile`、`docker-compose.yml`はもっと色々遊べるので、調べてみてください！

## buildする
docker-composeのコマンドを使用して、コンテナを作成します。
> [Docker Compose - docker-compose.yml リファレンス @zembutsu](https://qiita.com/zembutsu/items/9e9d80e05e36e882caaa)

`Dockerfile`、`docker-compose`がある作業ディレクトリで

```shell
docker-compose build
```

を実行。
すると`docker-compose`による`Dockerfile`で構成したDockerイメージの作成が始まります。
正常に終了するとイメージファイルが用意され、Dockerコンテナの立ち上げが可能になります。

下記のコマンドで、用意されているイメージファイルを確認できます。

```shell
docker-compose images
```

## Dockerコンテナ起動

同じディレクトリで

```shell
docker-compose up -d
```

を実行するとdockerが起動します。
`-d`オプションをつけることでバックグラウンドでの起動となります。

コンテナの状態確認は、以下のコマンドで出来ます。

```shell
docker-compose ps
```

## 開発してみる

Goで Hello, World! を書いて実際に動かしてみます。
ホストの`work`の中に`main.go`ファイルを作成し、以下のコードを書きます。

```go:main.go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

ホストの`work`ディレクトリはコンテナの`work`ディレクトリにマウントされているので、作成した`main.go`はコンテナからもアクセスが出来ます。
以下のコマンドで、main.goをコンテナの環境で実行できます。

```shell
docker-compose exec app go run main.go
# docker-compose exec サービス名 コマンド
```

**Herro, World!**
できましたか？

コンテナのシェルに入って実行することも可能です。

```
docker-compose exec app /bin/bash # コンテナのシェル起動
go run main.go
```

これで環境構築は完了です！
作業ディレクトリをそのままGithubでソース共有すれば、Dockerさえあれば誰でも簡単に環境が構築できてしまうのです...！
