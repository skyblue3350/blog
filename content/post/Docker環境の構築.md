+++
author = "すかい"
title = "Docker環境の構築"
date = "2016-07-03"
description = "Docker環境の構築"
tags = [
    "Docker",
]
+++

## はじめに

とりあえず使ってみたので今後使うことがあった時のためにメモ

## インストール

```
$ wget -qO- https://get.docker.com/ | sh
```

アプデの時は

```
$ wget -N https://get.docker.com/ | sh
```

## ユーザーの追加

```
$ sudo usermod -aG docker user名
```

でユーザーをグループに追加
その後再起動で反映（重要）

Cannot connect to the Docker daemon. Is the docker daemon running on this host?
とか出て怒られる

## イメージの取得

とりあえず今回は試したいものがあったのでそれを拾ってくる
https://hub.docker.com　から拾ってくる時はページの右上にあるコマンドをそのまま投入する

```
$ docker pull イメージ名
```

DLとpullが完了したら確認する

```
$ docker images
```

## 使い方

### 実行

```
$ docker run -it イメージ名
```

で実行と同時にログインするからCtrl+P、Qで離脱

```
$ docker ps
```

でプロセスを確認

### 終了

```
$ docker stop ID
```

IDはプロセス確認の時見れる

### ポートフォワーディング
Docker上の仮想環境にアクセスするにはポートフォワーディングを利用する

```
$ docker run -p 8080:80 -p 2812:2812 -it イメージ名
```
