+++
author = "すかい"
title = "cygwinにjupyterlabを入れる"
date = "2018-04-02"
description = "cygwinにjupyterlabを入れる"
tags = [
    "Cygwin",
    "Jupyter NoteBook",
]
+++

表題の通りです
詰まったのでメモです

素直に入れるとpyzmqのインストールでコケます

```
$ pip install jupyterlab
    bundled/zeromq/src/signaler.cpp:62:25: 致命的エラー: sys/eventfd.h: No such file or directory
     #include <sys/eventfd.h>
                             ^
    コンパイルを停止しました。
    error: command 'gcc' failed with exit status 1
```

なのでソースから入れます

```
$ wget https://github.com/zeromq/libzmq/releases/download/v4.2.1/zeromq-4.2.1.tar.gz
$ tar -zxvf zeromq-4.2.1.tar.gz
$ cd zeromq-4.2.1
$ ./configure
$ make
$ make install
```

ここまで入れたら再度入れ直します

```
$ pip install jupyterlab
```

入ってので終わりだけど一応動作確認

```
$ jupyter lab --no-browser
[I 09:13:01.480 LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 09:13:01.482 LabApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=hogehoge
```

ブラウザからアクセスして確認する

## 参考記事

- [Build fail because sys/eventfd.h not available on Cygwin32](https://github.com/zeromq/pyzmq/issues/747#issuecomment-312569216)
