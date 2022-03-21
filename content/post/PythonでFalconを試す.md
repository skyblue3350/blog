+++
author = "すかい"
title = "PythonでFalconを試す"
date = "2016-07-16"
description = "PythonでFalconを試す"
tags = [
    "Falcon",
    "Python",
]
+++

## Falcon

- [公式サイト](https://falconframework.org/)

> Falcon is a very fast, very minimal Python web framework for building microservices, app backends, and higher-level frameworks.

というわけで高速で最小なWebフレームワークです
とりあえず実際にApache上で動かすところまで試したのでメモだけ
Windows上でテストしてUbuntuのApacheで動かしてみた

こちらのQiitaの記事が参考になります　というかほぼ丸パクリです

- [Falconで光速のWeb APIサーバーを構築する](http://qiita.com/icoxfog417/items/913bb815d8d419148c33)

## インストール

```
$ pip install falcon
```

おわり

## テスト

<script src="https://gist.github.com/kgriffs/90837585fa4aca475229.js"></script>

公式のサンプルに参考記事の

```py
if __name__ == "__main__":
    from wsgiref import simple_server
    httpd = simple_server.make_server("127.0.0.1", 8000, app)
    httpd.serve_forever()
```

の部分を借りてきて実行してlocalhost:8000/quoteを見て問題なければOK

## Apache上で動かす

とりあえずこんな感じの想定で
設置場所：`/var/falcon/fuga`
URL：`http://serverurl/falcon/fuga`

さっきのコードをmain.pyって名前にして/var/falcon/fuga/main.pyに配置

```
$ sudo apt-get install apache2 libapache2-mod-wsgi
$ sudo vi /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
        ServerName hoge.com
        DocumentRoot /var/www/html
        <Directory "/var/www/html">
                Require all granted
        </Directory>
        <Directory "/var/falcon/fuga">
                Require all granted
        </Directory>

        WSGIDaemonProcess hoge.com user=www-data group=www-data threads=5
        WSGIProcessGroup hoge.com
        WSGIScriptReloading On
        WSGIPassAuthorization On
        WSGIScriptAlias /falcon/fuga /var/falcon/fuga/app.wsgi
</VirtualHost>
```

そんでもって
/var/falcon/fuga/app.wsgi
をつくる

```py
# -*- coding:utf-8 -*-

import sys, os

import logging
logging.basicConfig(stream = sys.stderr)

sys.path.insert(0, os.path.abspath(os.path.dirname(__file__)))

from main import app as application
```

`http://serverurl/falcon/fuga/quote`

にアクセスしてみて実行できてるか確認する
実際に置く時の設定方法が見つけられなかったけどこれで良いんだろうか・・・
riot少し触ってみたから組み合わせてなんか作ってみたいかな
