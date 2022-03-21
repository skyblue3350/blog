+++
author = "すかい"
title = "pyvenvで環境が作れない"
date = "2018-04-11"
description = "pyvenvで環境が作れない"
tags = [
    "Python",
    "Ubuntu"
]
+++

## はじめに

Ubuntu16.04 Python3.5環境下での出来事

新しく作った環境でいつも通り環境を作ろうとすると

```
$ python3 -m venv .
The virtual environment was not created successfully because ensurepip is not
available.  On Debian/Ubuntu systems, you need to install the python3-venv
package using the following command.

    apt-get install python3-venv

You may need to use sudo with that command.  After installing the python3-venv
package, recreate your virtual environment.

Failing command: ['/home/sky/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']
```

怒られる

## 対処

ここまではいつも通りなのでパッケージを入れる

```
$ sudo apt-get install python3-venv
```

これで問題なく生成できるはずですが今回はダメでした
正確には直接コンソールから実行する分には上手くいくのですがリモートからSSHで実行すると最初のエラーが出ます．
環境変数かlocale辺りで詰まってるっぽいので調べたらlocaleによって上手くいかないことがあるみたいです．

- [pyvenv not working because ensurepip is not available](https://stackoverflow.com/questions/39539110/pyvenv-not-working-because-ensurepip-is-not-available)

というわけで一時的に変えたら上手く行きました

```
$ LANG=C python3 -m venv .
$ source bin/activate
(example) $ which python ←それぞれ切り替わってるか確認
/example/bin/python
(example) $ which pip
/example/bin/pip
```

あとは確認した時のメモ
docker execで入った時のlocale

```
$ locale
LANG=
LANGUAGE=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=
sshで入った時のlocale

locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=ja_JP.UTF-8
LANGUAGE=
LC_CTYPE="ja_JP.UTF-8"
LC_NUMERIC="ja_JP.UTF-8"
LC_TIME="ja_JP.UTF-8"
LC_COLLATE="ja_JP.UTF-8"
LC_MONETARY="ja_JP.UTF-8"
LC_MESSAGES="ja_JP.UTF-8"
LC_PAPER="ja_JP.UTF-8"
LC_NAME="ja_JP.UTF-8"
LC_ADDRESS="ja_JP.UTF-8"
LC_TELEPHONE="ja_JP.UTF-8"
LC_MEASUREMENT="ja_JP.UTF-8"
LC_IDENTIFICATION="ja_JP.UTF-8"
LC_ALL=
```
