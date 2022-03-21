+++
author = "すかい"
title = "owncloudの移行でハマった"
date = "2016-07-07"
description = "owncloudの移行でハマった"
tags = [
    "Owncloud",
]
+++

ハマったというより自爆ですが一応メモ

ラズパイから物理鯖に鋭意移転中です
owncloudの移行をするときに9.0がリリースされていたので新規インストールで移行することにしました
今までと特に変わりなくDBを設定しインストールしました
ただサーバー側がApacheからNginxに変更していたため.htaccessが使えません
そこでNginxの設定ファイルに記述するのですがどうも参照していたドキュメントが古かったらしくクライアントからアクセスしようとすると
Error downloading https://test.hoge.com/remote.php/webdav/ -server replied: Method Not Allowed
とエラーが出て認証が進みません

というわけで設定ファイルは正しいものを参照しましょう
https://doc.owncloud.org/server/9.0/admin_manual/installation/nginx_configuration.html

これにSSLの設定を書き加えて完了です
