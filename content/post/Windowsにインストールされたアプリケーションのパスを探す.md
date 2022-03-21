+++
author = "すかい"
title = "Windowsにインストールされたアプリケーションのパスを探す"
date = "2018-12-15"
description = "Windowsにインストールされたアプリケーションのパスを探す"
tags = [
    "Python",
]
+++

今書いてるスクリプトでWindowsで特定のアプリケーションのインストール場所を探す用事があったのでメモ．
Uninstall用のレジストリを探していって見つける．

<script src="https://gist.github.com/skyblue3350/409420cd55c0591c5bcbd07211a37298.js"></script>

レジストリキーを総なめしていって一致する名前を見つけたらそのキーのInstallLocationを取得してるだけ．

```
$ python find_app.py
Beat Saber
C:\Program Files(x86)\Steam\steamapps\common\Beat Saber
```

って感じで取れる．
