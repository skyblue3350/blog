+++
author = "すかい"
title = "GTX1070のChainerセットアップ"
date = "2016-08-09"
description = "GTX1070のChainerセットアップ"
tags = [
    "Chainer",
    "Python",
]
+++

## はじめに

- Windows7 64bit
- GPU **GTX1070**
- Python2 64bit

環境でのChainerのGPU処理を有効にしてインストールする方法のメモ
ちゃんと学習するところまでやってなかったので大幅に加筆修正した（2016/09/06）

## メモ

> nvcc fatal : Value 'sm_61' is not defined for option 'gpu-architecture'

CUDA7.5がGTX1080をサポートしてない GTX1070も同様っぽい
8.0からサポートっぽいです

ソースは下記

- [When the CUDA Toolkit will support GTX1070 Graphics Card???
train_mnist on Chainer 1.10 + Ubuntu 16.04 + Cuda V7.5](https://devtalk.nvidia.com/default/topic/949823/cuda-setup-and-installation/when-the-cuda-toolkit-will-support-gtx1070-graphics-card-/post/4925608/#4925608)

[公式](https://developer.nvidia.com/cuda-toolkit)を確認すると

> Out of box performance improvements on Tesla P100, supports GeForce GTX 1080

とのこと

## インストール

### Python

今回は2系の64bitを使う
既に32bitが居座ってたのでpython64.batみたいなファイルを置いて誤魔化して使ってるんですが良い解決方法ご存知の方いたら教えて下さい

```
@PATH=64bit版のインストールパス\;%PATH%
python %*
```

としておいて

```
$ python64 hoge
```

することで切り替えています

### CUDA8.0

上記の通りGTX1070/1080では7.5が使用出来ないので8.0をインストール
8.0は開発者として登録が必要なので各項目適当に埋めて登録する
※2016/09/06の記事作成時は登録してすぐDL出来ました（cuDNNも）
https://developer.nvidia.com/cuda-toolkit
上記URLから対応するプラットフォームのものをDLして解凍、インストール
インストール時にグラフィックカードのドライバを入れなおされて画面が消えて戻ってこなくなりました
大人しく強制的に再起動して800x600の画面でドライバ入れなおしました

### cuDNN

https://developer.nvidia.com/rdp/cudnn-download
DLして解凍したらCUDAのインストールパスに上書き

##Microsoft Visual C++ Compiler for Python 2.7
https://www.microsoft.com/en-us/download/details.aspx?id=44266
Chainerをインストールする時に必要になる
インストールしてないと

```
error: Microsoft Visual C++ 9.0 is required (Unable to find vcvarsall.bat). Get it from http://aka.ms/vcpython27
```

みたいなエラーで怒られる
インストールするだけでパス通す必要はなかった

### PyCuda

Chainer1.2までしか使用してないらしいので不要だと思う　一応
http://www.lfd.uci.edu/~gohlke/pythonlibs/#pycuda
pycuda-2016.1.2+cuda7518-cp27-cp27m-win_amd64.whlをダウンロード

```
$ python64 -m pip install pycuda-2016.1.2+cuda7518-cp27-cp27m-win_amd64.whl
```

### Chainer

既にインストール済みの場合はアンインストールする

```
$ python64 -m pip uninstall chainer
```

インストールする

```
$ python64 -m pip install --upgrade --no-cache-dir chainer
```

## サンプルを動かす

いつものところからサンプルを借りてくる

```
$ git clone https://github.com/pfnet/chainer.git
$ cd chainer/examples/mnist
$ python64 train_mnist.py --gpu 0
GPU: 0
# unit: 1000
# Minibatch-size: 100
# epoch: 20
```

おっけー

## トラブルシューティング

### fatal error : Microsoft Visual Studio configuration file 'vcvars64.bat' could not be found for installation at ～～

http://stackoverflow.com/questions/18727964/nvcc-exe-linking-error-microsoft-visual-studio-configuration-file-vcvars64-bat
ここを参考に設定する
自分の場合Visual Studio Express 2011と 2012両方インストールしてた
上記設定後もエラーが出てエラーログ見てるとVS2012が使用されてるみたいなので困ってたけどVS120COMNTOOLSって環境変数を見て使用するものを決めてるらしいのでここをVS2011のパスに設定して解決
