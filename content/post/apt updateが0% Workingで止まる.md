+++
author = "すかい"
title = "apt updateが0% Workingで止まる"
date = "2018-03-30"
description = "apt updateが0% Workingで止まる"
tags = [
    "Ubuntu",
]
+++

## はじめに

表題の通りなんですが新規インストールしたマシンにNvidia-Dockerをインストールしているときにハマった
結論としてはapt-transport-httpsが入っていなかったのが原因
最初ansibleのテストしてたのでてっきり最初ansibleでなにか間違えてるのかと思ってた

## TL;DR

```
$ sudo apt install apt-transport-https
```

しとこうね

## 症状

apt updateすると以下の状態で止まる

```
$ sudo apt update
Hit:1 http://jp.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://jp.archive.ubuntu.com/ubuntu xenial-updates InRelease [102 kB]
Get:3 http://jp.archive.ubuntu.com/ubuntu xenial-backports InRelease [102 kB]
0%[Working]
```

で永遠に処理してる

## 対策

- apt cleanしてみる
  - 効果なし
- curlで試しにリソース持ってきてみる
  - できる
- 再起動
  - 効果なし
- 次に読まれそうなnvidia-dockerのレポジトリをcurlしてみる
  - 302が帰ってくる
  ん？
  ってところでapt-transport-httpsないじゃん…って気づいたという話

あとはこの辺のIssueとかですね…
https://github.com/NVIDIA/nvidia-docker/issues/642

いつも先にDocker入れてたので意識せずに使ってたので良くなかったですね
