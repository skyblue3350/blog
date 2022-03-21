+++
author = "すかい"
title = "Grafana v4.6.3からv5.2.1まであげる"
date = "2018-07-16"
description = "Grafana v4.6.3からv5.2.1まであげる"
tags = [
    "Docker",
]
+++

Docker版のGrafana v4.6.3を使用していましたが使いたいダッシュボードがv5系で作られていてインポートにコケたのでアップデートしました．
そのまま素直にあげるとコケるのでその辺のメモ

まず元のdocker-compose.ymlが以下

```yaml
  grafana:
    image: grafana/grafana:4.6.3
    container_name: grafana
    restart: always
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
```

このまま動かすDockerイメージのバージョンをあげると

```
grafana              | t=2018-07-16T09:51:34+0000 lvl=info msg="Starting DB migration" logger=migrator
grafana              | t=2018-07-16T09:51:34+0000 lvl=info msg="Executing migration" logger=migrator id="Migrate all Read Only Viewers to Viewers"
grafana              | t=2018-07-16T09:51:34+0000 lvl=eror msg="Executing migration failed" logger=migrator id="Migrate all Read Only Viewers to Viewers" error="attempt to write a readonly database"
grafana              | t=2018-07-16T09:51:34+0000 lvl=eror msg="Exec failed" logger=migrator error="attempt to write a readonly database" sql="UPDATE org_user SET role = 'Viewer' WHERE role = 'Read Only Editor'"
grafana              | t=2018-07-16T09:51:34+0000 lvl=eror msg="Server shutdown" logger=server reason="Service init failed: Migration failed err: attempt to write a readonly database"
```

と出てコケます
公式ドキュメントの下記にあるように実行ユーザーが変更されたことによるものらしいので新しいユーザーに合わせてパーミッションを変更します

- [Installing using Docker Migration from a previous version of the docker container to 5.1 or later](http://docs.grafana.org/installation/docker/#migration-from-a-previous-version-of-the-docker-container-to-5-1-or-later)

以下のように書き換えて最新版のイメージで一度bashを立ち上げるように変更

```yaml
  grafana:
    image: grafana/grafana:5.2.1
    container_name: grafana
    restart: always
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
    entrypoint: bash
    tty: true
    user: root
```

次に一度コンテナをあげてドキュメントの通りにパーミッションを変更

```
$ docker-compose up -d
$ docker-compose exec grafana bash
root@4f83c452fffa:/# chown -R root:root /etc/grafana
root@4f83c452fffa:/# chmod -R a+r /etc/grafana
root@4f83c452fffa:/# chown -R grafana:grafana /var/lib/grafana
root@4f83c452fffa:/# chown -R grafana:grafana /usr/share/grafana
```

パーミッションを変更できたらentrypoint以下を削除して起動

```yaml
  grafana:
    image: grafana/grafana:5.2.1
    container_name: grafana
    restart: always
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
$ docker-compose up -d
```
