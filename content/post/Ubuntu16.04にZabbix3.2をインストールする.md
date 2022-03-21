+++
author = "すかい"
title = "Ubuntu16.04にZabbix3.2をインストールする"
date = "2017-07-23"
description = "Ubuntu16.04にZabbix3.2をインストールする"
tags = [
    "Ubuntu",
    "Zabbix",
]
+++

## はじめに

Ubuntu16.04でZabbix3.2をインストールして使えるようにする
でもこれ良く考えたらDockerで良かったのでは…？感がある

## 環境

- Ubuntu 16.04 4.4.0-83-generic
- Zabbix 3.2

## 環境構築

### フォント

グラフ描画時に必要になるとのことなのでインストールしておく

```
# apt-get install fonts-vlgothic
```

### MySQL

#### インストール

予めインストールして設定しておく

```
# apt-get install mysql-server
# vi /etc/mysql/mysql.conf.d/mysqld.conf
character-set-server = utf8
collation-server = utf8_bin
skip-character-set-client-handshake
innodb_file_per_table
```

#### データベースとユーザー作成

zabbixで利用するデータベースを作る
DBユーザー名：hoge
DBパスワード：fuga
な場合はこんな感じ
ユーザー名はコンフィグのデフォルトがzabbixなのでその方が良いかも

```
# mysql -uroot -p
Enter password:
mysql> create database zabbix;
mysql> grant all privileges on hoge.* to zabbix@localhost identified by 'fuga' ;
mysql> exit
```

### Zabbix

#### レポジトリの追加

今回は3.2なので
http://repo.zabbix.com/zabbix/3.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.2-1+xenial_all.deb
を利用する
最新バージョンはルートから辿っていけば見つけられる

```
# wget http://repo.zabbix.com/zabbix/3.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.2-1+xenial_all.deb
# dpkg -i zabbix-release_3.2-1+xenial_all.deb
```

しょうもないミスだがUbuntu16.04のコードネームはxenialである
今回記事の設定中間違えてtrustyを追加していたので依存関係でエラーが出てハマった
間違えて追加した場合は

```
# dpkg -l | grep zabbix
ii  zabbix-release                     3.2-1+trusty                               all          Zabbix official repository configuration
# dpkg -r zabbix-release
```

とかして消してやり直しましょう

#### パッケージのインストール

zabbix以外入れてないクリーンな環境だといくつかphpのパッケージが足りず警告を出されるので一緒にインストールする

```
# apt-get install zabbix-agent zabbix-server-mysql zabbix-frontend-php php-bcmath php-mbstring php-xml
```

#### テーブル作成と初期データ登録

予め用意されてるデータを用いて初期設定を行う

```
# zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -u hoge -p zabbix
Enter password:
～少し時間かかる～
# 
```

#### 設定ファイルを編集する

初期設定ウィザードがあるので不要な気がするが先程の設定値と合わせて設定する

```
# vi /etc/zabbix/zabbix_server.conf
DBName=zabbix
DBUser=hoge
DBPassword=fuga
```

次はタイムゾーンの設定
コメントアウトされてるので外して設定する

```
# vi /etc/apache2/conf-enabled/zabbix.conf
    <IfModule mod_php7.c>
        php_value max_execution_time 300
        php_value memory_limit 128M
        php_value post_max_size 16M
        php_value upload_max_filesize 2M
        php_value max_input_time 300
        php_value always_populate_raw_post_data -1
        php_value date.timezone Asia/Tokyo ←こんな感じ
    </IfModule>
```

## 起動する

各サービスを起動する

```
# service zabbix-agent restart
# service zabbix-server restart
# service apache2 restart
```

## アクセスしてみる

http://zabbix_server_ip/zabbix
にアクセスしてみる
うまく行ってると初期設定のウィザードが立ち上がっている
この時点で足りないものがあると警告が出て来るので適宜必要なパッケージなどを揃える
追加でパッケージを入れた場合は適宜apache2を再起動すること

多分記事通り進めていると全て揃っているはず
設定が完了するとログイン画面に移動するので

- ユーザー名
Admin
- パスワード
zabbix

でログインできる

## 日本語化

右上のプロフィールアイコンからページを移動すると（/zabbix/profile.php）
言語選択出来るので日本語に変更しておく

## ゲストユーザーのログイン禁止

ゲストユーザーは今のところ使う予定がないのでログイン禁止に変更する
管理 -> ユーザーグループ -> guest （/zabbix/usergrps.php）
と移動してWebインターフェースへのアクセスを「無効」に変更する
試しにシークレットウィンドウで管理画面を開いてみてゲストログインが消えていることを確認する

## パスワードの変更

デフォルトパスワードだと気持ち悪いので変更する
管理 -> ユーザー -> Admin （/zabbix/users.php）
と移動して「パスワードを変更」ボタンからパスワードを変更しておく

## 参考記事

- [Zabbix 3.0をUbuntu 14.04にインストール(MySQL編)](http://qiita.com/atanaka7/items/9c4c8a5099c24f8f8be8)
