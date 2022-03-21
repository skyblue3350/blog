+++
author = "すかい"
title = "Python3.5でexe作る話"
date = "2016-09-09"
description = "Python3.5でexe作る話"
tags = [
    "Python",
]
+++

## はじめに

2系の時はpy2exeに頼りっきりだった
インストーラー欲しい時だけ他のものに浮気してたけどzipオプション有効にすれば一塊になって配布しやすかったので

3.5から事情が変わったらしいので少しトラブりました故にメモ
記事書いてる時点での話なので日が経ってる様なら状況が変わってるかもしれません

- Python3.5 64bit
- Windows7 64bit

環境でのお話

### py2exe

今までお世話になってた
2系の時はインストーラーを落としてきてインストールした気がする
3系からpipで入るみたいですが3.4までしかサポートしてません

ソース1：[Is there a py2exe version that's compatible with python 3.5?](http://stackoverflow.com/questions/32963057/is-there-a-py2exe-version-thats-compatible-with-python-3-5)
3.5から内部仕様が変わって動かないとのこと
run-py3.5-win-amd64.exeに変更したら動くかと思ったけどそんなに甘くなかった

### cx_Freeze

pipから入れると

```
error: file '~\build\cx-freeze\cxfreeze-postinstall' does not exist.
```

pipから入るのはバージョンが古い（4.3.4）です　公式サイトに騙されました

ソース2：[Missed cxfreeze-postinstall script in source distribution](https://bitbucket.org/anthony_tuininga/cx_freeze/issues/56/missed-cxfreeze-postinstall-script-in)
新しいものなら大丈夫なのでレポジトリをクローンしてきてそこからインストールします
もしくはソース1で紹介されてるwhlを落として使えば良いと思います
https://github.com/sekrause/cx_Freeze-Wheels
