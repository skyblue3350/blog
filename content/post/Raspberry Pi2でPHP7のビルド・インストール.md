+++
author = "すかい"
title = "Raspberry Pi2でPHP7のビルド・インストール"
date = "2016-07-05"
description = "Raspberry Pi2でPHP7のビルド・インストール"
tags = [
    "PHP",
    "Raspberry Pi 2",
]
+++

## はじめに

PHP7が出たので自鯖のPHPを5.3から7へ移行しました
自鯖で稼働しているPHP関連のものはowncloudとWordpressですが移行後特にこれといった問題は起きていません

PHP7のインストールからMySQLの接続確認まで行います
インストールした環境はRaspberry Pi2 Wheezy上になります
Debian系やその他OSでも前提パッケージ名が違う程度だと思います

## 前提パッケージのインストール

```
$ sudo apt-get install libxml2 libxml2-dev pkg-config libssl-dev libreadline5 libreadline-gplv2-dev libpspell-dev apache2-dev
```

libcurl4-openssl-devもいるかもしれません

## コンパイル　インストール

適当に公式からアーカイブファイルを落として展開する
その後

```
$ sudo sh ./configure \
--with-readline \
--enable-pcntl \
--with-gettext \
--enable-phpdbg \
--enable-phpdbg-webhelper \
--enable-mbstring \
--enable-zip \
--enable-bcmath \
--enable-ftp \
--enable-exif \
--enable-calendar \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--with-curl \
--with-mcrypt \
--with-iconv \
--with-pspell \
--with-gd \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--with-openssl \
--enable-soap \
--with-zlib-dir=/usr \
--with-zlib=/usr \
--with-bz2=/usr \
--with-apxs2 \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-mysql-sock=/var/run/mysqld/mysqld.sock
$ sudo make -j `nproc`
$ sudo apt-get remove php*
$ sudo make install
```

nprocはそのPC上の最大を表すらしい　知らなかった
おそらくconfigureは前提パッケージを全て入れていれば通るはず
トラブった時は以下のページが参考になった
https://blog.flowl.info/2015/how-to-install-php7-on-ubuntu-or-debian/

## 各種設定

### 設定ファイルのコピー

パスを調べてからそこにファイルを配置する

```
$ php -i | grep php.ini
$ sudo cp ./hoge/php-7.0.3/php.ini-production /usr/local/lib/php.ini
```

```
$ vi /usr/local/lib/php.ini
必要な設定をする
必要な項目を検索して適宜書き換え
expose_php = On → Offに
;date.timezone = → date.timezone = 'Asia/Tokyo'
;error_log = php_errors.log → error_log = "適当なパス/error.log"//対象ファイルの所有者をdaemonに権限を755にしておく
mysqli.default_socket = → mysqli.default_socket = "パス"
pdo_mysql.default_socket = → pdo_mysql.default_socket = "パス"
display_errors = Off → display_errors = stderr
```

ソケットファイルの位置は/etc/mysql/my.cnfに書いてあるので環境に合わせる

```
$ php -v
PHP 7.0.3 (cli) (built: Feb 15 2016 16:56:09) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
```

インストール出来ていることを確認

### Apacheの設定
--with-apxs2の設定をしているため

```
$ cat /etc/apache2/mods-available/php7.load
LoadModule php7_module /usr/lib/apache2/modules/libphp7.so
```

というファイルが生成されているはず
同じディレクトリにconfファイルを作成する

```
$ vi /etc/apache2/mods-available/php7.conf
SetHandler application/x-httpd-php
```

一度設定を無効にして再度有効にしてからApacheを再起動する

```
$ sudo a2dismod php7
$ sudo a2enmod php7
$ sudo service apache2 restart
```

あとは適当にファイルを作成して

```php
<?php phpinfo(); ?>
```

と書いてテスト

### MySQLとの接続の確認

```php
<?php $pdo = new PDO('mysql:dbname=DB名;localhost;charset=utf8', 'ユーザー名', 'パスワード'); ?>
```

実行してエラーが出ないことを確認する
undefinedだったらコンパイル時のオプション忘れ、No such fileならソケットの位置が怪しいです

以上終わり　半日掛かった
でも引っかかったところ全部メモったから同じ環境の人は特に問題なく出来るはず…

## 参考サイト

- [あぱーブログ Apache HTTP/2＋PHP7＋MySQL5.7 インストールメモ](https://blog.apar.jp/linux/3798/)
- [k-holyのPHPとか諸々メモ mysqliのコンストラクタで"No such file or directory"のエラー](http://k-holy.hatenablog.com/entry/2014/05/28/192707)
- [Qiita CentOS 6 の環境にPHP7をインストールしてApacheで動かすまで](http://qiita.com/ssaita/items/9e0170251d45ed1b8818)
- [SOFTEL 【php】MySQLに接続するときにエラー発生](https://www.softel.co.jp/blogs/tech/archives/2187)
- [FLOW How to install PHP7 on Ubuntu or Debian](https://blog.flowl.info/2015/how-to-install-php7-on-ubuntu-or-debian/)

## 感想

必要なパッケージを探しコンパイルを通しでめんどくさいのでおとなしく誰かの作ってくれたパッケージを感謝しながら使う方が楽です
使うのが目標ならこの辺を参考にすると良いかも
コンパイルはエラー出るから良いですが既に環境が整えてあるowncloudとかWordpressでエラーが出るのがなかなか困りました
Wordpressはwp-config.phpでdebugをtrueにするとエラーが追いやすいです
owncloudはクライアントで一度ユーザーをログアウトさせてからログインさせるとうまく行きます

## メモ

記事内容と重複することもあるがエラー内容とかも兼ねてメモ

### configure: error: Cannot find OpenSSL's

openssl-devを入れれば良いと出てくるがRaspberry Piには該当パッケージがなかった
代わりにlibssl-devを入れる
https://www.raspberrypi.org/forums/viewtopic.php?f=33&t=14465

### 途中でbuild/shtoolのパーミッションがないと怒られた場合

```
$ chmod 755 build/shtool
```

### php -v できるがデータベースの接続に失敗する
### PDOException' with message 'SQLSTATE[HY000] [2002] No such file or directory

mysqlのサービスが立ち上がってるのを確認したあとsockファイルの位置を確認する

```
$ php --ri mysqli
$ php --ri pdo_mysql
```

php.iniに

```
mysqli.default_socket = "パス"
pdo_mysql.default_socket = "パス"
```

と記述してからテストする
mysqlはPHP7から無くなったので不要
Apacheもreloadしておく
