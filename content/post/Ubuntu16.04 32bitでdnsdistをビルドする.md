+++
author = "すかい"
title = "Ubuntu16.04 32bitでdnsdistをビルドする"
date = "2018-10-23"
description = "Ubuntu16.04 32bitでdnsdistをビルドする"
tags = [
    "Ubuntu",
]
+++

## はじめに

様々な事情があってどうしようもないUbuntu16.04 32bit環境にdnsdistを入れることになった時の話．
公式で提供しているレポジトリは64bit向けしか提供されていないので自分でビルドする必要がある．

## 環境

- Ubuntu 16.04 32bit

## 前提パッケージのインストール

一部不要なパッケージがある気がするが最終的に入れてパッケージ

```
$ sudo apt-get install autoconf automake libtool git make gcc g++ libsodium-dev pkg-config ragel fstrm libssl-dev libsystemd-dev libprotoc-dev protobuf-compiler libprotobuf-dev protobuf python-pip python-virtualenv
```

## dnsdistのインストール

レポジトリをクローンしてから当該バージョンに切り替える．

```
$ git clone https://github.com/PowerDNS/pdns.git
$ cd pdns
$ git checkout refs/tags/dnsdist-1.3.2
```

ビルド用のディレクトリに移動してからビルドしてインストールする

```
$ cd pdns/dnsdistdist/
$ autoreconf --install
$ make
$ ./configure
$ sudo make install
```

バージョンを確認する．

```
$ dnsdist --version
dnsdist 0.0.HEAD.ga63826e (Lua 5.1.5)
Enabled features: ebpf libsodium protobuf recvmmsg/sendmmsg systemd
```

ついでに有効化されているものも見れるので不足しているものがあったらリビルドする．

次にサービスを起動するが，Ubuntu 16.04 32bit のsystemdにはバグがあってRestrictAddressFamiliesが付いているとサービスが起動できないのでコメントアウトする．
systemdのバージョンをあげようかと思ったもののかなり根が深そうで簡単にいきそうになかったので諦め．

```
$ sudo vi /usr/local/lib/systemd/system/dnsdist.service
- RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
+ # RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
```

あとは普通にdnsdistを設定するだけなので省略．

## 参考記事

- [Installing dnsdist From git](https://dnsdist.org/install.html#from-git)
- [PowerDNS/pdns pdns/pdns/README-dnsdist.md](https://github.com/PowerDNS/pdns/blob/master/pdns/README-dnsdist.md)
- [systemd/systemd RestrictAddressFamilies= broken on 32-bit #4575](https://github.com/systemd/systemd/issues/4575)
