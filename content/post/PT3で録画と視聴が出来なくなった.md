+++
author = "すかい"
title = "PT3で録画と視聴が出来なくなった"
date = "2016-11-11"
description = "PT3で録画と視聴が出来なくなった"
tags = [
    "PT3",
]
+++

## はじめに

結論はpt3_drvが消えてたのでリビルドした

Chinachuを利用して録画鯖を建ててるけど今日リアルタイム視聴しようとしたら視聴出来ないことに気付いた
ここのところ放置して利用してなかったから気づかなかった
心当たりがあるのはdist-upgradeしたことくらい
VLCでxspf開いたら何回も認証を要求される
最初はVLCで認証がコケてる？と思ったので試しにxspfで呼びに行くURLを叩いてみた

```xml
<?xml version="1.0" encoding="UTF-8"?>
<playlist version="1" xmlns="http://xspf.org/ns/0/">
<trackList>
<track>
<location>URL</location>
<title>ほげほげ</title>
</track>
</trackList>
</playlist>
```

URLをブラウザで開いて認証してみたら認証は通る
ただwatch.m2tsが空で落ちてくる
のでそもそもPT3からデータが抜けてないんじゃないかなって思った

## 試しに録画してみる

のでとりあえず録画コマンドからテストしてみる

```
$ recpt1 --b25 --strip 27 20 test.ts
using B25...
enable B25 strip
pid = 2787
Cannot tune to the specified channel
```

録画出来ない…
ということは録画前の環境構築かなって思って調べたらドライバを無効にして有効にすると録画出来るらしい

```
$ sudo modprobe -r pt3_drv sudo modprobe pt3_drv
modprobe: FATAL: Module pt3_drv not found.
```

そもそもなかった

pt3_drvをビルドする

```
$ git clone https://github.com/m-tsudo/pt3.git
$ cd pt3
$ make
$ sudo make install
$ sudo modprobe pt3_drv
```

これで解決した

## 参考記事

- [pt3_drvがapt-get dist-upgradeで消えてrecpt1が動作しなかったのを直した](http://qiita.com/ymotongpoo/items/c31ae0fc3c7378436b27)
