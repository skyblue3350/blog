+++
author = "すかい"
title = "nvidia-dockerで特定のバージョンが動作しない"
date = "2018-04-08"
description = "nvidia-dockerで特定のバージョンが動作しない"
tags = [
    "Docker",
]
+++

Nvidia-docker2で特定のバージョン以降が動作しない
例えばCUDA9.0のイメージは

```
$ docker run --runtime=nvidia --rm nvidia/cuda:9.0-devel-ubuntu16.04 nvidia-smi
Sun Apr  8 03:27:41 2018
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.111                Driver Version: 384.111                   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 980 Ti  Off  | 00000000:01:00.0  On |                  N/A |
|  0%   36C    P8    14W / 250W |     32MiB /  6075MiB |      1%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 980 Ti  Off  | 00000000:02:00.0 Off |                  N/A |
|  0%   36C    P8    15W / 250W |      2MiB /  6078MiB |      1%      Default |
+-------------------------------+----------------------+----------------------+
```

のようにコンテナの起動とGPUの情報の取得ができるが

```
$ docker run --runtime=nvidia --rm nvidia/cuda:9.1-devel-ubuntu16.04 nvidia-smi nvidia-smi
docker: Error response from daemon: OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:402: container init caused \"process_linux.go:385: running prestart hook 1 caused \\\"error running hook: exit status 1, stdout: , stderr: exec command: [/usr/bin/nvidia-container-cli --load-kmods configure --ldconfig=@/sbin/ldconfig.real --device=all --compute --utility --require=cuda>=9.1 --pid=11413 /var/lib/docker/overlay2/028a6f3b9995b2357c899949cda07047f8858364ca666c7047f652e338b108f2/merged]\\\\nnvidia-container-cli: requirement error: unsatisfied condition: cuda >= 9.1\\\\n\\\"\"": unknown.
```

このような表示になって起動することができない

Nvidia-Dockerの動作要件にはGPUドライバのバージョンが絡んでいるようで当該マシンでのNvidiaのドライバのバージョンによって動かないことがある

```
$ dpkg -l | grep nvidia
～関係のない表示は省略～
ii  nvidia-384                            384.111-0ubuntu0.16.04.1                   amd64        NVIDIA binary driver - version 384.111
```

のように当該マシンではver384がインストールされていることがわかる
GPUドライバとバージョンの関係性はレポジトリのwikiにまとまっている

- [NVIDIA/nvidia-docker CUDA Requirements](https://github.com/NVIDIA/nvidia-docker/wiki/CUDA#requirements)

384ではCUDA9.0までしか対応していないため9.1は動作させることができない
ので何らかの方法でバージョンを上げる必要がある

Ubuntuならレポジトリが提供されているのでそこから入れるのが楽

```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update
$ sudo apt-get install nvidia-387
```

もちろん公式サイトからGPUドライバを落としてきてビルドしても良い
ちょっと面倒ではあるが基本的なビルドツールが入ってればそんなに手こずらずに入るはず
