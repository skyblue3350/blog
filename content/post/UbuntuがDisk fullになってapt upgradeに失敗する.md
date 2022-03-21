+++
author = "すかい"
title = "UbuntuがDisk fullになってapt upgradeに失敗する"
date = "2018-10-06"
description = "UbuntuがDisk fullになってapt upgradeに失敗する"
tags = [
    "Ubuntu",
]
+++

## はじめに

色々あって放置してたあるUbuntu Serverですが自動更新でカーネルが積み重なった結果/bootが100%になって死んでいました．
質の悪いことにあるカーネルをインストールしてる途中でDisk fullになったらしくアンインストールしようにも途中のものがあるからそっちを先に片付けろと言われうんともすんともいかない状態になりました．
その時のメモ

今考えたら--force-yesとかしたらアンインストールできたのかな？

## 現状確認

```
$ df -h | grep /boot
/dev/sda1                        472M  472M  0M  100% /boot
```

/bootがいっぱいになってる
ひとまずautoremoveで古いカーネルを消すかと思ったもののapt-get install -fしろと言ってきてどうにもならない
/bootがいっぱいなためインストール途中のカーネルインストールにこける

ググった感じこれといった解決方法を見つけられなかったのでちょっと強引だが以下の方法で解決した

## 解決まで

まず現在利用しているカーネルバージョンをチェック

```
$ uname -a
Linux hoge 4.4.0-109-generic #132-Ubuntu SMP Tue Jan 9 19:52:39 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

次にインストール済みのカーネルをチェック

```
$ dpkg -l | grep linux-image
rc  linux-image-4.4.0-101-generic       4.4.0-101.124                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
rc  linux-image-4.4.0-103-generic       4.4.0-103.126                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
ii  linux-image-4.4.0-104-generic       4.4.0-104.127                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
ii  linux-image-4.4.0-108-generic       4.4.0-108.131                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
ii  linux-image-4.4.0-109-generic       4.4.0-109.132                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
ii  linux-image-4.4.0-112-generic       4.4.0-112.135                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
ii  linux-image-4.4.0-116-generic       4.4.0-116.140                              amd64        Linux kernel image for version 4.4.0 on 64 bit x86 SMP
...
```

今起動しているより古いカーネルを消します．
今のカーネルが使えなかった時のために1つ前は残してそれより古いものを消します．

```
$ sudo rm /boot/*-4.4.0-101*
$ sudo rm /boot/*-4.4.0-103*
以下略
```

ある程度容量が開いたらapt-get install -fして詰まってるパッケージのインストールができるか試します．
できた場合はここで再起動すれば最新カーネルで起動するので古い分はautoremoveすれば消せます．

ただ，今回は空き容量が足りなかったので再起動して最新カーネルで起動しようとするとインストール途中でコケてたためカーネルがクラッシュして上がらなくなりました．
ので最新より1,2個前のカーネルを選んで起動した後に同じ作業を繰り返しました．
