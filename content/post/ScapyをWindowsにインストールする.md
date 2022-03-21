+++
author = "すかい"
title = "ScapyをWindowsにインストールする"
date = "2016-12-11"
description = "ScapyをWindowsにインストールする"
tags = [
    "Python",
    "Scapy",
]
+++

## はじめに

ScapyをWindowsにインストールする記事を調べると色々面倒な手順が書いてあるが最近は楽
ただそのまま使えないので便宜上使えるようにする
sniff使おうとしたら出たので他の機能を使うなら出ないかもしれない

## インストール

クローンするかリリースから最新のものを落とす
今回はv2.3.3を使います
解凍したら

```
$ python setup.py install
```

して終わり

## 修正

scapy/arch/windows/compatibility.pyを編集する
冒頭に

```py
from scapy.arch.pcapdnet import PcapTimeoutElapsed
```

を追加する
185行目をコメントアウト

```py
    if offline is None:
        # log_runtime.info('Sniffing on %s' % conf.iface)
        if L2socket is None:
            L2socket = conf.L2listen
```

終わり

## エラー

### global name 'log_runtime' is not defined

とりあえず必要ないので当該行をコメントアウトして対応

### global name 'PcapTimeoutElapsed' is not defined

import文を追加して対応
