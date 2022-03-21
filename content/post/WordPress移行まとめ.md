+++
author = "すかい"
title = "WordPress移行まとめ"
date = "2016-07-06"
description = "WordPress移行まとめ"
tags = [
    "WordPress",
]
+++

## 概要

当初ラズパイで建ててたサーバー（Apache）から物理マシンのサーバー（nginx）へ移行しました
その時のメモ

## 環境

引越元環境
サーバー：Apache
URL：http://sky-net.pw/blog

引越先環境
サーバー：Nginx
URL：http://blog.sky-net.pw

## 作業
### 再インストール

引越自体はファイルを移動させるのみ
その後適切なパーミッションと所有者に変更する

### DB移行

#### エクスポート

もしサイトのURLを変更する場合は先に元環境で設定→一般→サイトURLを予め移転先へ変更してからMySQLのエクスポートをする
元の状態のままエクスポートして移転先にインポートしたらCookieが無効になっていてどうのでログインすることが出来なかった
エクスポート自体は

```
$ mysqldump -h ホスト名 -u ユーザー名 -p DB名 > 書き出しファイル名
```

で出来る　設定値はconfig.php見ればわかる

#### インポート

インポートは予め受け入れ先DBを作成した上でインポート

```
$ mysql -u root -p
mysql> create database DB名 default character set utf8 default collate utf8_general_ci;
mysql> create user ユーザー名@ホスト名 IDENTIFIED BY パスワード;
mysql> grant all privileges on DB名.* TO ユーザー名@ホスト名;
mysql> exit
```

```
$ mysql -h ホスト名 -u ユーザー名 -p DB名 < ファイル名
```

ここまででトップページの表示は問題なく行えるようになったはず

### パーマリンクの設定

nginxでは.htaccessが有効にならないのでconfで色々書いて解決する

```
中略
        location / {
                try_files $uri $uri/ @wordpress;
        }

        location ~ \.php$ {
                try_files $uri @wordpress;
                fastcgi_pass    unix:/var/run/php5-fpm.sock;
                fastcgi_index   index.php;
                fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include         fastcgi_params;
        }

        location @wordpress {
                fastcgi_split_path_info ^(.+\.php)(.*)$;
                fastcgi_pass    unix:/var/run/php5-fpm.sock;
                fastcgi_index   index.php;
                fastcgi_param   SCRIPT_FILENAME $document_root/index.php;
                include         fastcgi_params;
        }
中略
```

見つからなかったらindex.phpに投げる感じ

これで移行完了
