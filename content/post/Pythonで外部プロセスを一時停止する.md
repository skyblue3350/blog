+++
author = "すかい"
title = "Pythonで外部プロセスを一時停止する"
date = "2018-03-24"
description = "Pythonで外部プロセスを一時停止する"
tags = [
    "Python",
]
+++

Pythonで外部プロセスを一時停止する要件できたのでメモ
メモリ解析ツールとかでお世話になるのでスクリプトから使えたら便利かなって…
ダメだったけどwatchdogでファイル監視しつつ変更検知したら消される前に横取りするスクリプトで使えないかなーと思って書いてた
環境はWindows10 Python3です

使うのはpsutilで以下のレポジトリ参照です
https://github.com/giampaolo/psutil

pipで入るので入れます

```
$ pip install psutil
```

READMEに全部書いてあるので読んだ方が早いですが停止と再開のサンプルはこんな感じ
プロセスIDが1234で3秒間停止させてみる

```py
import time

import psutil

pid = 1234
p = psutil.Process(pid)
p.suspend()
time.sleep(3)
p.resume()
```

これだと不便なので特定のプロセスを3秒だけ止める（ペイント）

```py
import time

import psutil

pid = None
for proc in psutil.process_iter(attrs=["pid", "name"]):
    if proc.info["name"]  == "mspaint.exe":
        pid = proc.info["pid"]

if not pid:
    exit(1)

p = psutil.Process(pid)
p.suspend()
time.sleep(3)
p.resume()
```
