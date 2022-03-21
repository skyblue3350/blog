+++
author = "すかい"
title = "Unable to find vcvarsall.bat"
date = "2016-09-10"
description = "Unable to find vcvarsall.bat"
tags = [
    "Python3",
]
+++

## 問題点

- Windows7 64bit
- Python3.5

環境でパッケージ入れようとしたら以下のエラーで怒られた

```
Unable to find vcvarsall.bat
```

その解決策

環境作ると毎回これやってる気がする

## 解決策

コンパイラを入れる
対応してるのは

- [How to deal with the pain of “unable to find vcvarsall.bat”](https://blogs.msdn.microsoft.com/pythonengineering/2016/04/11/unable-to-find-vcvarsall-bat/)

で確認する

- 2.6 ～ 3.2
  - [Microsoft Visual C++ Compiler for Python 2.7](https://www.microsoft.com/download/details.aspx?id=44266)
- 3.3 ～ 3.4
  - [Windows SDK for Windows 7 and .NET 4.0](https://www.microsoft.com/download/details.aspx?id=8279)
- 3.5
  - [Visual C++ Build Tools 2015](http://go.microsoft.com/fwlink/?LinkId=691126)
    or
  - [Visual Studio 2015](https://visualstudio.com/)

3.5なので[Visual C++ Build Tools 2015](http://go.microsoft.com/fwlink/?LinkId=691126)をインストールする
