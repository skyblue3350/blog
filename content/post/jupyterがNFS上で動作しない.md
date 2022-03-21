+++
author = "すかい"
title = "jupyterがNFS上で動作しない"
date = "2018-04-11"
description = "jupyterがNFS上で動作しない"
tags = [
    "Jupyter NoteBook",
]
+++

## はじめに

一つ前の記事でjupyterlabを複数人で使える環境を作ったわけですがいざ本番環境と同等の環境でテストしたら動きませんでした．
開発環境と違うのはホームディレクトリをNFSマウントしているかどうかだったのでその辺を調べたらjupyterやipythonがデータの管理にsqliteを使っていることが影響しているそうです．
sqliteはNFS上では利用できないためうんともすんとも言わない状況になるとのこと．

追記　2018年5月4日
最近この辺の課題にリトライして気づいたのですがNFSのマウント時のバージョンが3以下だと発生するようです
バージョン4でテストしたところ以下の設定はなくとも動作することを確認しました
追記　終わり

## 症状

ちなみに実行するとセルが実行中の状態

[*] print("hoge")
の状態で止まって5分くらい待たされます
5分くらい待つとログに

```
[IPKernelApp] ERROR | Failed to open SQLite history /home/sky/.ipython/profile_default/history.sqlite (database is locked).
[IPKernelApp] ERROR | History file was moved to /home/sky/.ipython/profile_default/history-corrupt.sqlite and a new file created.
```

と表示され記事冒頭の理由が発覚します．

## 対策

jupyterで生成されるファイルとipythonで生成されるファイルをそれぞれNFSボリュームの外に生成するようにすれば解決します
前者のファイルはjupyte_notebook_config.pyに以下の設定を作ることでメモリ上に作成して誤魔化します

### jupyter

ファイルパス書けば良いので適当にどっか指定した方が多分良いです
システム全体に適用したいので/etcに置いて全ユーザーに適用させます

```
$ sudo cat << EOS > /etc/jupyter/jupyter_notebook_config.py
c.HistoryManager.hist_file = ':memory:'
c.NotebookNotary.db_file = ':memory:'
EOS
```

### ipython

後者は環境変数IPYTHONDIRが定義されていない時にホームディレクトリにファイルを作成するのでこれを指定して解決します．

```
$ sudo echo "IPYTHONDIR=/tmp/jupyter/\${USER}" >> /etc/environment
$ sudo mkdir /tmp/jupyter
$ sudo chmod 777 /tmp/jupyter
```

とかしておきます
単にjupyter notebookとかlabを使ってる環境はこれで終わりです．

## おまけ

以下jupyterhubと使いたい人
jupyterhubと組み合わせて動かしてる人はこれで一見解決したように見えるもののもう1個罠があります
jupyterhubのセキュリティ上の理由からjupyterhubから立ち上がったjupyterlab等は最低限の環境変数しか提供されないためこれだとコケます
issue読んでるとenv_listにappendすればシステムの好きな環境変数追加できるよとか書いてあったんですが上手く行かなかったので普通に追加しました

```
$ sudo echo "c.Spawner.environment = {\"IPYTHONDIR\": \"/tmp/jupyter/${USER}\"}" >> /etc/jupyter/jupyterhub_config.py
```
