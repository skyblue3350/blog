+++
author = "すかい"
title = "pyenvで3.7入れて試す"
date = "2018-06-28"
description = "pyenvで3.7入れて試す"
tags = [
    "Python",
]
+++

## はじめに

3.7リリースされてたので試そうかなと思ってpyenvで3.7インストールしたのでメモ
きれいな環境で1からpyenvやら置いて試したので色々前提パッケージがなくてエラった

## 環境

- Ubuntu18.04
- pyenv 1.2.5

## 環境構築

### 前提パッケージ

必要なパッケージを用意します．
足りないとインストール時に下記にまとめたエラーが出て詰まります．

```
$ sudo apt-get install zlib1g-dev libffi-dev libbz2-dev libreadline-dev libssl-dev libsqlite3-dev
```

## インストール

```
$ pyenv install 3.7.0
Downloading Python-3.7.0.tar.xz...
-> https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
Installing Python-3.7.0...
Installed Python-3.7.0 to /path/to/.pyenv/versions/3.7.0
```

## 使ってみる

適当なディレクトリで

```
$ pyenv local 3.7.0
$ python -V
Python 3.7.0
```

あとは適当に

## エラーとか

### zipimport.ZipImportError: can't decompress data; zlib not available

必要なパッケージを入れます

```
$ sudo apt-get install zlib1g-dev
```

### ImportError: No module named _ctypes

https://github.com/pyenv/pyenv/issues/785
にある通りlibffiの問題らしいので

```
$ sudo apt-get install libffi-dev
```

しましょう

### WARNING: The Python bz2 extension was not compiled. Missing the bzip2 lib?

```
$ sudo apt-get install libbz2-dev
```

### WARNING: The Python readline extension was not compiled. Missing the GNU readline lib?

```
$ sudo apt-get install libreadline-dev
```

### ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib?

```
$ sudo apt-get install libssl-dev
```

### WARNING: The Python sqlite3 extension was not compiled. Missing the SQLite3 lib?

```
$ sudo apt-get install libsqlite3-dev
```
