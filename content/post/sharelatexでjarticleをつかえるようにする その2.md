+++
author = "すかい"
title = "sharelatexでjarticleをつかえるようにする その2"
date = "2017-04-16"
description = "sharelatexでjarticleをつかえるようにする その2"
tags = [
    "Docker",
    "sharelatex",
]
+++

## はじめに

[前回の記事](../sharelatex%E3%81%A7jarticle%E3%82%92%E3%81%A4%E3%81%8B%E3%81%88%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B/)からバージョン上がったりしてそのままじゃいけなかったので改めて検証してDockerfileを書いた
LDAPやアカウント管理機能がPro版でしか対応されてないみたいなので利用するか迷うところ
アカウントを作成するだけならadminのアカウントで/admin/registerから作成できます
ただ作成したアカウントの一覧は見れません 不便
DBを直に見に行けば見えるでしょうけど…

## 環境

- Docker version 17.03.1-ce
- Docker-compose version 1.12.0
- sharelatex/sharelatex:0.6.1

## ディレクトリツリー

data以下に各コンテナをマウントさせる

```
.
├─ data
├─ docker-compose.yml
└─ sharelatex
       ├─ Dockerfile
       └─ jlisting.sty
```

## Dockerfile

platexが使えるようにするのとソースコードを貼る際に困るのでjlistingを導入する

```
FROM sharelatex/sharelatex:0.6.1

# platex install
RUN apt-get update \
  && apt-get install texlive-lang-cjk -y \
  && apt-get clean \
  && apt-get autoremove

# latexmk
RUN cd /usr/local/texlive/2016/bin/x86_64-linux/ \
  && sed -ri "s/$latex  = 'latex %O %S';/$latex  = 'platex -shell-escape %O %S';/g" latexmk \
  && sed -ri "s/$bibtex  = 'bibtex %O %B';/$bibtex  = 'pbibtex %O %B';/g" latexmk \
  && sed -ri "s/$dvipdf  = 'dvipdf %O %S %D';/$dvipdf  = 'dvipdfmx %O -o %D %S';/g" latexmk

# jlisting
ADD jlisting.sty /usr/share/texlive/texmf-dist/tex/latex/listings/jlisting.sty
RUN mktexlsr
```

## docker-compose.xml

先程のDockerfileをビルド元に指定する

```yaml
version: '2'

services:
    sharelatex:
        build: "./sharelatex"
        restart: always
        container_name: sharelatex

        depends_on:
            - mongo
            - redis
        privileged: true
        ports:
            - 8888:80
        links:
            - mongo
            - redis
        volumes:
            - $PWD/data/sharelatex:/var/lib/sharelatex
        environment:
            SHARELATEX_MONGO_URL: mongodb://mongo/sharelatex
            SHARELATEX_REDIS_HOST: redis
            SHARELATEX_APP_NAME: Our ShareLaTeX

    mongo:
        restart: always
        image: mongo
        container_name: mongo
        expose:
            - 27017
        volumes:
            - $PWD/data/mongo:/data/db

    redis:
        restart: always
        image: redis
        container_name: redis
        expose:
            - 6379
        volumes:
            - $PWD/data/redis:/data
```

## jlisting.sty

https://osdn.net/projects/mytexpert/downloads/26068/jlisting.sty.bz2/
からダウンロードして解凍したものを配置しておく

## 起動

--buildオプションを付けておくとDockerfileに変更を加えた時にリビルドしてくれる

```
$ docker-compose up -d --build
```

## アカウント登録

ドキュメント通り

```
$ docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email hoge@fuga.com"
      パスワード設定URL
```

表示されたURLにアクセスしてアカウントを作成する

## 停止

```
$ docker-compose down
```
