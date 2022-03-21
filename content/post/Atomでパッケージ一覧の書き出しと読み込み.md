+++
author = "すかい"
title = "Atomでパッケージ一覧の書き出しと読み込み"
date = "2016-11-07"
description = "Atomでパッケージ一覧の書き出しと読み込み"
tags = [
    "Atom",
]
+++

## はじめに

Atomでパッケージ一覧の書き出しと読み込みのメモ
環境を別にPCに移すだけならUserDir/.atomを丸々コピーするのもあり

apmコマンドを使う　多分デフォでパスが通ってる
Winだと

```
UserDir/AppData/Local/atom/bin/apm.cmd
```

とかインストールしたところに入ってる

## 書き出し

```
$ apm list --installed --bare > package-list.txt
```

一覧がバージョンと共に書き出される

## 読み込み

```
$ apm install --packages-file package-list.txt
```

## スター付き

アカウントあればスター付きのやつを拾ってきたりすることも出来るっぽいけど前者で事足りたのでメモ程度
詳しくは参考元を

```
$ apm star --installed
```

## 参考
- [Installed packages list into single file](https://discuss.atom.io/t/installed-packages-list-into-single-file/12227)
