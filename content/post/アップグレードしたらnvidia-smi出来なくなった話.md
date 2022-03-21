+++
author = "すかい"
title = "アップグレードしたらnvidia-smi出来なくなった話"
date = "2016-12-08"
description = "アップグレードしたらnvidia-smi出来なくなった話み"
tags = [
    "Ubuntu",
]
+++

## 経緯

別件で環境構築している時にupgradeやらdist-upgradeしたらカーネルが新しくなってnvidia-smiが実行できなくなりました
解決に1日掛かったので一応メモ

こんな感じ

```
$ nvidia-smi
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
もしくは
modprobe: ERROR: could not insert 'nvidia_367'
...

$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2015 NVIDIA Corporation
Built on Tue_Aug_11_14:27:32_CDT_2015
Cuda compilation tools, release 7.5, V7.5.17
```

## アップデート前環境

当初以下の環境を組んでいました

- Ubuntu14.04 LTS（64bit）
- GTX980 Ti
- nvidia-361（361.42）
- CUDA （7.5）

ドライバ CUDA 共にnvidia公式からダウンロードしたものを使用して構築しました.

## 解決策

※多分ドライバ入れ直すだけで良い

### アンインストール

一度関連するものを削除

```
$ sudo apt-get remove nvidia-*
$ sudo apt-get remove cuda-*
```

### ドライバのダウンロード・インストール

最新のドライバを落としてくる
http://www.nvidia.com/Download/index.aspx
ここから環境に合わせて選ぶ

```
$ wget http://jp.download.nvidia.com/XFree86/Linux-x86_64/375.20/NVIDIA-Linux-x86_64-375.20.run
$ chmod +x NVIDIA-Linux-x86_64-375.20.run
$ sudo ./NVIDIA-Linux-x86_64-375.20.run
```

### CUDAのインストール

CUDAはレポジトリから7.5が取れるようになってるのでそちらを利用する

```
$ sudo apt-get install nvidia-cuda-toolkit
```

## エラー解決

以下メモ

### You appear to be running an X server

XWindowが立ち上がってると怒られます
そういえばこのマシンは最初デスクトップ版をインストールしたので

```
$ sudo service lightdm stop
```

して停止した上で再度実行します

### Kernel preparation unnecessary for this kernel. Skipping...

全文は

```
ERROR: Failed to run `/sbin/dkms build -m nvidia -v 4.4.0-53-generic`: 
Kernel preparation unnecessary for this kernel. Skipping...
```

エラーログが
`/var/log/nvidia-installer.log`
にあるので確認すると以下のエラーログがある

### unrecognized command line option ‘-fstack-protector-strong’

今回ハマってたのはここだった
gccのバージョンが古い（4.8以前）だとこのエラーが出るようです.

```
$ gcc --version
gcc (Ubuntu 4.8.x-2ubuntu1~16.04) 4.8.x
$ sudo apt-get install gcc-4.9
$ sudo update-alternatives --config gcc
$ gcc --version
gcc (Ubuntu 4.9.4-2ubuntu1~16.04) 4.9.4
$ sudo ./NVIDIA-Linux-x86_64-375.20.run
```

4.9を選択した上で再度実行すると成功する
