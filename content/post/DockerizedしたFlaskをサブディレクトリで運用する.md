+++
author = "すかい"
title = "DockerizedしたFlaskをサブディレクトリで運用する"
date = "2017-09-03"
description = "DockerizedしたFlaskをサブディレクトリで運用する"
tags = [
    "Flask",
    "Nginx",
    "Python",
]
+++

## はじめに

DockerizedしたFlaskをサブディレクトリ等からリバースプロキシでアクセスするとリダイレクト周りでおかしくなって辛いことになります
ベタに実行しないで大人しくApacheかNginxから動くイメージを作れば良いんですけど…

## 設定

### Nginx

フロントのNginxからリバースプロキシするときに必要な情報を提供してあげます

```
    location /hoge/fuga/service/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Script-Name /hoge/fuga/service;
        proxy_pass http://192.168.xxx.xxx:8888;
    }
```

### Flask

http://flask.pocoo.org/snippets/35/
snippetsにあるコードを少し変更して使います
先程もらってきた値を参照してFlask側で使用するようにします
これで実行時にremoteの正しいIPを見るようになりURL周りの挙動も直ります
挙動が怪しいときはprint(environ, "\n", file=sys.stderr)辺りのコメントアウトを外してdocker log見れば直せます（適当

`reverseproxy.py`

```py
import sys

class ReverseProxy(object):
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        # print(environ, "\n", file=sys.stderr)

        script_name = environ.get("HTTP_X_SCRIPT_NAME", "")
        if script_name:
            environ["SCRIPT_NAME"] = script_name
            path_info = environ["PATH_INFO"]
            if path_info.startswith(script_name):
                environ["PATH_INFO"] = path_info[len(script_name):]

        server = environ.get('HTTP_X_FORWARDED_SERVER', '')
        if server:
            environ['HTTP_HOST'] = server

        addr = environ.get('HTTP_X_FORWARDED_FOR', '')
        if addr:
            environ['REMOTE_ADDR'] = addr

        scheme = environ.get("HTTP_X_SCHEME", "")
        if scheme:
            environ["wsgi.url_scheme"] = scheme

        # print(environ, "\n", file=sys.stderr)

        return self.app(environ, start_response)
```

`app.py`

importしたものをインスタンス化したオブジェクトに突っ込んで終わりです

```py
from flask import Flask
from reverseproxy import ReverseProxy

app = Flask(__name__)
app.wsgi_app = ReverseProxy(app.wsgi_app)
```
