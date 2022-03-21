+++
author = "すかい"
title = "LunaMultiplayerをDockerで起動する"
date = "2018-07-06"
description = "LunaMultiplayerをDockerで起動する"
tags = [
    "Docker",
]
+++

## はじめに

KSPのマルチMOD LunaMultiplayerのサーバー側をDockerコンテナで立ち上げる方法のメモ
monoのruntimeだけあれば動きますがUbuntuの公式レポから提供されてるmonoはバージョンが古く新しいの入れるのが面倒そうだったので…
ちなみに古いバージョンだとエラー吐いて動きませんでした

## docker-compose.yml

```yaml
version: "3"

services:
  ksp:
    image: mono:latest
    volumes:
      - ./LMPServer:/LMPServer
    ports:
      - 8800:8800
    command: mono /LMPServer/Server.exe
```

あとはzip落としてきて展開してコンテナあげて終わりです

```
$ wget [リリースページの最新版のZip]
$ unzip LunaMultiplayer-Release.zip
$ docker-compose up -d
```
