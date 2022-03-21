+++
author = "すかい"
title = "TAITAN XのChainerセットアップ"
date = "2017-01-29"
description = "TAITAN XのChainerセットアップ"
tags = [
    "Chainer",
    "Ubuntu",
]
+++

## はじめに

UbuntuにTAITAN Xを載せた学習環境の構築メモ
何故かシステムが起動しなくなったりしたのでその辺の対処も含めて

## 環境

特筆すべきところだけ

- OS
Ubuntu 14.04.05 Server
- GPU
GPU TAITAN X（pascal）

## インストール

### OS インストール

特に問題なく終わる

### NVIDIAドライバインストール

http://www.nvidia.co.jp/Download/index.aspx?lang=jp
公式サイトから適宜環境にあったドライバを選択する

- 製品のタイプ
GeForce
- 製品シリーズ
GeForce 10 series
- 製品ファミリー
NVIDIA TAITAN X (pascal)
- オペレーティングシステム
Linux 64bit
- 言語
日本語

検索 -> ダウンロードと進み同意ボタンが出て来る画面で同意ボタンのURLをコピーしておく
インストール先で

```
# wget コピーURL
```

でインストーラーをダウンロードする

```
# chmod +x NVIDIA-Linux-x86_64-xxx.xx.run
# ./NVIDIA-Linux-x86_64-xxx.xx.run
```

あとは通常通りのインストールと同様
インストール終了後にnvidia-smiして確認する

```
# nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 375.26                 Driver Version: 375.26                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  TITAN X (Pascal)    Off  | 0000:01:00.0     Off |                  N/A |
| 23%   40C    P0    56W / 250W |      0MiB / 12189MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

正しく認識されていることを確認した後 一度再起動して問題なく立ち上がることを確認する

### CUDAインストール

[前回の記事](../gtx1070%E3%81%AEchainer%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97/)でも書いた様にver8以降でしか利用できないので8系をインストールする
Linux -> x86_64 -> Ubuntu -> 14.04 -> runfile(local)
を選択する
先程と同様にURLをコピーしてwgetしておく

```
# wget URL
# chmod +x cuda_x.x.xx_linux.run
# ./ cuda_x.x.xx_linux.run
```

いくつかの質問の後インストールが行われる
回答は環境に合わせて適宜変更すれば良いがGraphic DriverのインストールはNoにする
インストール後サマリが表示されたらパスを通して完了
bashrc等に書いても良いがユーザーが増えた場合を考えると面倒なのでシステム側で通す

```
# vi /etc/enviroment
PATH="/usr/local/cuda/bin:元々のパス群"
# nvcc -V
バージョンが出れば終了
```

### cudnnインストール

公式から落としてきてcp -a してインストール先に上書きコピーして終了

### Chainerインストール

ここまで問題なく出来てればpip installするだけ

```
# apt-get install python-dev python-pip
# python -m pip install chainer --user
# python
>>> import chainer
>>> chainer.cuda.get_device(0)
<CUDA Device 0>
>>> chainer.cuda.get_device(0).use()
```

して問題なければ終了
一応examplesとかも動かしてみて動作確認はした

## エラー

### Failed to apply ACL on /dev/dri/card1: No such file or directory

CUDAインストール時にGraphic Driverのインストールをしたところそこでフリーズしてリモートはおろかローカルからも触れなくなったのでシステムを落として再起動したところシステムの起動時のログに以下が大量に出て起動しなくなった

```
Failed to apply ACL on /dev/dri/card1: No such file or directory
...
```

GRUBでAdvance Modeからレスキューモードで起動
エラーログは出るが無視して5分程経つといつものリカバリメニューが出るので
fsckを実行した後にrootを選択してコンソール画面に移動し

```
# dpkg -l | grep nvidia
```

して当該ドライバを探し

```
# apt-get purge nvidia-xxx
```

削除した上でリブートする
