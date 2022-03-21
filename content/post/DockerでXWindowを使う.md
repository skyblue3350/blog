+++
author = "すかい"
title = "DockerでXWindowを使う"
date = "2018-01-18"
description = "DockerでXWindowを使う"
tags = [
    "Docker",
    "XWindow",
]
+++

## はじめに

UbuntuでDockerで建てたコンテナからXWindowを飛ばしてみたくなったのでメモ．
SSHは使わずにTCPでXWindowを飛ばします．

## 環境

今回は同一のマシン上で行います．

```
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
$ docker -v
Docker version 17.03.1-ce, build c6d412e
$ docker-compose -v
docker-compose version 1.14.0-rc1, build c18a7ad
```

## ホスト側

### 設定の変更

Ubuntuはlightdmが上がってる想定で書いてますが違う場合はそれぞれの環境に合わせて下さい．
デフォルトではTCPでListenするようになってないので設定を加えます．
ファイルがない場合は作成し，ある場合は追記します．

```
$ sudo vi /etc/lightdm/lightdm.conf
[SeatDefaults]
xserver-allow-tcp=true
```

システムを再起動して変更を適用します．
正しく設定出来ている場合は6000番のポートが開放されているはずです．

```
$ netstat -anp | grep 6000
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::6000                 :::*                    LISTEN      -
```

最後にこのXWindowサーバーにアクセス可能なクライアントの許可を出します．
とりあえず動作確認に無制限にしますが適宜制限を設けた方が良いです．

```
$ xhost +
```

## クライアント側

### ディレクトリ構成

適当に配置します．

```
$ tree xwindow-sample
xwindow-sample/
├── Dockerfile
└── docker-compose.yml
```

### イメージを作る

XWindowを飛ばすイメージを作ります．
環境変数設定するだけなのでシンプルです．
今回はxeyesを動かしてみます．

```
$ vi Dockerfile
FROM ubuntu:16.04

RUN apt-get update -y \
  && apt-get install -y x11-apps

CMD ["xeyes"]
```

### 起動する
さっきのDockerfileを元にコンテナを起動したらウィンドウが飛んでくるはずです．
DISPLAYにホストIPを指定しておきます．

```
$ vi docker-compose.yml
version: "3"

services:
  xeyes:
    build: .
    container_name: xeyes
    environment:
      DISPLAY: 192.168.1.xxx:0.0
$ docker-compose up -d --build
```
