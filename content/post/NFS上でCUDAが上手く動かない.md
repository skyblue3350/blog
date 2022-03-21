+++
author = "すかい"
title = "NFS上でCUDAが上手く動かない"
date = "2018-05-04"
description = "NFS上でCUDAが上手く動かない"
tags = [
    "Docker",
]
+++

## 症状

Nvidia Docker2で作ったコンテナの/homeにNFSマウントした状態でChainerを使うと

```
CUDARuntimeError: cudaErrorUnknown: unknown error
```

と怒られて動かない

## 解決方法

NFSマウントを外すと上手く動くのでNFSによるものらしい
ので以下のようにNFSのバージョンを明示的に最新の4を指定する
デフォルトだとどうも3でマウントしてるっぽい

```yaml
version: "3"

services:
  sample:
    build: .
    tty: true
    volumes:
      - nfs:/home

volumes:
  nfs:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.xxx,hard,rsize=1048576,wsize=1048576,nfsvers=4
      device: :/home
```

## その他試したこと

- [Problem with CUDA 8 with 381.09 drivers on Ubuntu 16.04, GTX 1080Ti](https://devtalk.nvidia.com/default/topic/1003878/problem-with-cuda-8-with-381-09-drivers-on-ubuntu-16-04-gtx-1080ti/)
  CUDA_CACHE_PATHを指定することで他の場所に作れるらしい
  元の位置は~/.nv
  変更したがダメだった
  投稿者と同じくドライバが384系だったので390系まであげたものの解決せず

## メモ

多分Dockerに限らず起きる気がする
Dockerを使っていないベタな環境で似たような環境がありますがそちらではこのエラーが出ないので比較したらそちらはNFSがバージョン4でマウントされてました
