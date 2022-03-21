+++
author = "すかい"
title = "jupyterhubあれこれ"
date = "2018-04-11"
description = "jupyterhubあれこれ"
tags = [
    "Jupyter NoteBook",
]
+++

## はじめに

jupyterをLDAP認証と組み合わせて複数人で使えるようにしたいなと前から思ってたのをテストしたのでメモ
jupyterを複数人で利用するにはjupyterhubを利用すれば良いらしい
ただjupyterhubはjupyter notebookを複数人で利用できるようにするものでjupyterlabを複数人で利用するにはjupyterlab hubを利用しないといけないらしい
正直ドキュメント読んでも？？？状態になってしまったのでそれぞれ環境を作る

以下の2つの組み合わせでテストしました

- jupyter notebook + jupyterhub （+ ldapauthenticator）
- jupyter lab + jupyterhub （+ ldapauthenticator）

環境はUbuntu16.04です
Docker上でイメージ作った際のコマンドを普通の環境用の書き直してるので素直に動かないかもしれないのでその時はrootで作業して下さい…

## jupyterhub

### 環境構築

必要なパッケージをそれぞれ揃えます．

```
$ sudo apt-get install -y curl python3 npm nodejs-legacy
$ curl -kL https://bootstrap.pypa.io/get-pip.py | sudo python3
$ sudo npm install -g configurable-http-proxy
```

pipでjupyter notebookとhubをインストールします

```
$ sudo pip3 install jupyterhub
$ sudo pip3 install --upgrade notebook
```

デフォルトの認証ではシステムにいるユーザーが利用されます
説明用にデモユーザーを作ります

```
$ sudo useradd testuser
$ sudo passwd testuser
Enter new UNIX password:[適当に入力]
Retype new UNIX password:[適当に入力]
$ sudo mkdir /home/testuser
$ sudo chown testuser /home/testuser 
```

### 実行

ここまで来たら実行します

```
$ sudo jupyterhub --port 8888
```

上手く行っていればアクセスしてログイン画面が出てくるのでログインして動作チェックします
問題なくnotebookが起動できればおっけーです

### 詳細設定

次に設定を生成します．

```
$ sudo jupyterhub --generate-config
Writing default config to: jupyterhub_config.py
```

/root辺りに生成されると思うので

```
c.JupyterHub.port = 8888
```

とかにしてみて再度動作テストします

## jupyterlab hub

### 環境構築

だいたい同じですがnpmの要求が5以上とかになってる関係で新しいnodejs入れたりします

```
$ curl -sL https://deb.nodesource.com/setup_9.x | sudo bash
$ curl -kL https://bootstrap.pypa.io/get-pip.py | sudo python3
$ sudo apt-get update
$ sudo apt-get install nodejs
$ npm install -g configurable-http-proxy
```

pip3で必要なライブラリを揃えます．

```
$ sudo pip3 install jupyterlab jupyterhub
$ jupyter labextension install @jupyterlab/hub-extension
```

先程の方法でconfigを生成します

```
$ sudo jupyterhub --generate-config
Writing default config to: jupyterhub_config.py
```

生成したファイルに以下の設定を追記します

```
c.Spawner.default_url = '/lab'
```

### 実行

同じように

```
$ sudo jupyterlab
```

してみて同じようにWebからアクセスしてみてテストする
システムにいるユーザーでログインしてjupyterlabが起動すればおっけー

## ldapauthenticator

jupyterhubでもjupyterlab hubでも同じ

### 環境構築

追加で以下をインストール

```
$ sudo pip3 jupyterhub-ldapauthenticator
```

先程生成したconfigファイルに以下を追記する
環境に合わせて適宜変更する

```
c.JupyterHub.authenticator_class = "ldapauthenticator.LDAPAuthenticator"
c.LDAPAuthenticator.server_address = "LDAP認証サーバーのDNSまたはIP"
c.LDAPAuthenticator.server_port = 389
c.LDAPAuthenticator.use_ssl = False ※デフォルトがTrueなのでハマります
c.LDAPAuthenticator.bind_dn_template = "uid={username},ou=people,dc=example,dc=jp"
```

起動テストして問題なくログインできればおっけーです
ダメな時は立ち上げた時のログを見ると結構詳細に出ててわかります
SSLがデフォで有効だったのでハマったけどすぐ気付けました
