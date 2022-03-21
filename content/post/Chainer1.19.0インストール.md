+++
author = "すかい"
title = "Chainer1.19.0インストール"
date = "2016-12-30"
description = "Chainer1.19.0インストール"
tags = [
    "Chainer",
    "Python",
]
+++

## はじめに

1.15.0→1.19.0へ更新した

## 環境

- Windows 7 64bit
- GTX 1070
- Python3.5 64bit
- Microsoft Visual Studio 14
- Cuda 8.0
- cudnn 8.0 v5.1

基本的に[前回の記事](../gtx1070%E3%81%AEchainer%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97/)と同一のもの

## 作業

```
$ python3 -m pip install -U chainer
```

して終わりかと思ったら

```
cupy\cuda\cudnn.cpp(3650): error C3861: 'cudnnAddTensor_v3': identifier not found
cupy\cuda\cudnn.cpp(4307): error C3861: 'cudnnGetFilterNdDescriptor_v5': identifier not found
cupy\cuda\cudnn.cpp(5056): error C3861: 'cudnnSetConvolutionNdDescriptor_v3': identifier not found
cupy\cuda\cudnn.cpp(6839): error C3861: 'cudnnConvolutionBackwardFilter_v3': identifier not found
cupy\cuda\cudnn.cpp(7729): error C3861: 'cudnnConvolutionBackwardData_v3': identifier not found
error: command 'VSのインストールパス\\cl.exe' failed with exit status 2
```

公式レポジトリのv1.19.0をDLしても同様のエラー

試しに1.18.0をインストールしたけどエラーは発生しなかった
とりあえずChainer公式のレポジトリからクローンした上でそちらをインストールして解決

```
$ git clone 
$ python3 setup.py install
$ python3
>>> import chainer
>>> chainer.__version__
1.19.0
```

上がらない時はインストール先に古いのが残ってるかも
