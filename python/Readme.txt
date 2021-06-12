コンテナ起動
下記コマンドにて、コンテナを起動する。

ターミナル
$ docker-compose up -d --build

コンテナへ接続
ターミナル
$ docker-compose exec python3 bash
各種ライブラリをインストール (任意)
ターミナル(python3コンテナ)
$ python -m pip install numpy
$ python -m pip install pandas
$ python -m pip install matplotlib
$ python -m pip install networkx
$ python -m pip install pyyaml
$ python -m pip install xlsxwriter
$ python -m pip install tornado
インストールしたライブラリの確認
ターミナル(python3コンテナ)
$ python -m pip list


https://qiita.com/reflet/items/4b3f91661a54ec70a7dc

要らなくなったら...
利用が終わって不要になったらコンテナごと下記コマンドにて削除する。

ターミナル
$ docker-compose down
以上です。
