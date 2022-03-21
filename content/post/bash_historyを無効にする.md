+++
author = "すかい"
title = "bash historyを無効にする"
date = "2018-03-02"
description = "bash historyを無効にする"
tags = [
    "Ubuntu",
]
+++

Ubuntuでbash historyを無効にしたい時の話
一部のコマンドだけとかはわりと記事出て来るけど全部は需要ないのかすぐ見つからなかったのでメモ．

setコマンドを使ってhistoryを無効にできる．
のでこうなる．

```
$ set +o history
```

ただこれだと入力したターミナルでのみ無効になるので次回以降も記録されないようにする．

```
$ echo 'set +o history' >> ~/.bashrc
```

最後に既に記録されているhistoryを削除しておしまい

```
$ rm ~/.bash_history
```

ログアウトしてコマンド打って再ログインしてみて記録されてないかチェックして問題なければおっけー
