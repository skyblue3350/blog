+++
author = "すかい"
title = "kubernetesでGPUコンテナを動かす"
date = "2018-01-13"
description = "kubernetesでGPUコンテナを動かす"
tags = [
    "kubernetes",
]
+++

## はじめに

1.7までは[こちら](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)に書いてある方法でしかGPUコンテナが扱えませんでしたが，
[k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)を使うことでk8s上でもう少し簡単にGPUコンテナを扱えるようになるみたいなので試してみました．

## 前提

基本的に前回と同じで物理マシン2台ですがk8sのバージョンだけあげています．
v1.9.1のk8s環境が構築されていることと各pod間とグローバルにつながっていることくらいです．
バージョンアップ自体はドキュメント通りやればあがりますが色々検証しているうちに動かなくなってしまったので一度k8s絡みのパッケージを削除した上でクリーンなk8s環境を作り直しています．

## 環境

### Master

```
root@kubernetes:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
root@kubernetes:~# uname -a
Linux kubernetes 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
root@kubernetes:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:52:23Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
root@kubernetes:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

### Worker

今回も1台だけです．GPUが1つ載っています．

```
root@kubernetes-node:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.2 LTS"
root@kubernetes-node:~# uname -a
Linux kubernetes-node 4.4.0-92-generic #115-Ubuntu SMP Thu Aug 10 09:04:33 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
root@kubernetes-node:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
root@kubernetes-node:~# lspci | grep NVIDIA
01:00.0 VGA compatible controller: NVIDIA Corporation GM200 [GeForce GTX 980 Ti] (rev a1)
```

## k8s-device-plugin

基本的にはREADMEに書いてある通りに設定するだけです．
最後にコンテナを配備する部分以外はGPUが搭載されているWorkerでのみ作業を行えば良いです．

### 動作要件

- NVIDIA drivers ~= 361.93
ドライバはUbuntu公式のレポジトリでもこの辺が提供されてるので心配しなくても良いですね．
- nvidia-docker version > 2.0 (see how to install and it’s prerequisites)
公式ドキュメントを見ながら入れればいけます．
- docker configured with nvidia as the default runtime.
これちょっとわかりにくいですがnvidia-dockerがv2からdockerコマンドにオプションをつける方式に変わったのでその辺周りでdockerコマンドにデフォでこのオプションが付くようにする…んだと思います．
- Kubernetes version = 1.9
これのためにk8sのバージョンアップが必要になりました…
- The DevicePlugins feature gate enabled
後述します．起動時に追加するオプションです．

### セットアップ

#### NVIDIA driver

k8s 1.7の頃にテストしてた名残でCUDAが入れてあったので特に何もしませんでしたがドライバだけ入っていれば良いみたいなのでUbuntu公式とかにある最新のドライバを入れれば良いと思います．
Nvidia公式レポジトリから入れるとかソースからビルドして入れるとか色々ありますがメンドイのでUbuntu公式レポジトリにあるので良いと思います．
現時点（2018年1月10日）では384が提供されている最新版みたいです．

```
# apt-get install -y nvidia-384
```

#### nvidia-dockerインストール

https://github.com/NVIDIA/nvidia-docker

事前に1.0をインストールしている場合所定の手順でアンインストールする必要があります（v1とv2で挙動が違うため）．
その辺はドキュメントを読んで下さい．

まずGPGキーの登録とレポジトリの登録をします．
最後にパッケージリストを更新しておきます．

```
# curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
# curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list
# apt-get update
```

あとはインストールして終わりです．

```
# apt-get install -y nvidia-docker2
# pkill -SIGHUP dockerd
```

最後にバージョンと動作確認します．

```
# nvidia-docker version
NVIDIA Docker: 2.0.1 ←ここが重要
Client:
 Version:      17.09.1-ce ← k8sの動作サポートバージョンにしとくのが無難
 API version:  1.32
 Go version:   go1.8.3
 Git commit:   19e2cf6
 Built:        Thu Dec  7 22:24:23 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.09.1-ce ← 同上
 API version:  1.32 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   19e2cf6
 Built:        Thu Dec  7 22:23:00 2017
 OS/Arch:      linux/amd64
 Experimental: false
# docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
Wed Jan 10 07:35:46 2018
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 387.26                 Driver Version: 387.26                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 980 Ti  Off  | 00000000:01:00.0 Off |                  N/A |
| 22%   33C    P8    26W / 250W |      0MiB /  6076MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

#### docker configured with nvidia as the default runtime

https://github.com/nvidia/nvidia-container-runtime

Dockerのデフォルトランタイムを変更します．
不要かもしれませんが一応入れます．

```
# apt-get install nvidia-container-runtime
```

次にデフォルトランタイムを変える方法が3つ程あるので適当に選択します．
ちなみに2つ以上同時にやると同じパラメータを2重に設定しようとして既にそのパラメータは設定済みですってエラーが出て詰みます．
今回はドキュメントの「Daemon configuration file」を選択．

```
# tee /etc/docker/daemon.json <<EOF
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF
# pkill -SIGHUP dockerd
# systemctl daemon-reload
# systemctl restart docker
```

追記　2018-04-06
"default-runtime": "nvidia",
を記事中のconfigに書くのを忘れていたため追加しました
追記終わり

動作確認します．先程nvidia-dockerのテストした時のコマンドを確認します．

```
# docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

良く見るとランタイムをきちんと指定していますね．
正しく設定出来ていれば指定しなくてもデフォルトのランタイムがこのランタイムに変更されているはずです．

```
# docker run --rm nvidia/cuda nvidia-smi
～同じなので略～
```

#### The DevicePlugins feature gate enabled

kubeletの起動時にデバイスプラグインを有効にする必要があるため有効にします．
当該ファイルは/etc/systemd/system/kubelet.service.d/10-kubeadm.confにあります．
空気を読んで追加します．
この時言うまでもないですがKUBELET_EXTRA_ARGSが既に宣言されている場合は後に書いた方が優先されてしまいます．
その場合はその行に--feature-gates=DevicePlugins=trueだけ追加しましょう．

```
# vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_EXTRA_ARGS=--feature-gates=DevicePlugins=true"
```

最後にこの変更を適用させるためにkubeletを再起動します．

```
# systemctl daemon-reload
# systemctl restart kubelet
```

kubeletの起動ログをチェックし正しくプラグインが読み込まれているか確認します．

```
# journalctl -u kubelet | grep feature
～省略～
1月 06 17:05:03 ホスト名 kubelet[15771]: I0106 17:05:03.167655   15771 feature_gate.go:220] feature gates: &{{} map[DevicePlugins:true]}
```

大丈夫そうですね．

## k8s-device-pluginのコンテナの配備

この作業はMasterで行います

```
# kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.9/nvidia-device-plugin.yml
```

しばらく待ってGPUノードのClusterで当該コンテナが起動しているか確認します．

```
# kubectl get pods --all-namespaces -o wide | grep nvidia
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE       IP               NODE
kube-system   nvidia-device-plugin-daemonset-hb9l8   1/1       Running   0          3d        10.244.1.3       kubernetes-node
```

これで環境構築終わりです．
実際に使ってみます．

## 実際にGPUコンテナを動かす

適当なファイル名で以下のymlを作ります．

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: cuda-container
      image: nvidia/cuda:9.0-cudnn7-devel ← GPU利用可能なイメージを選択する（devel付きでないとnvccが入ってない GPUドライバのみ）
      tty: true ← nvidiaのデフォルトコンテナはCMDがないのでttyで開いておかないと即時クラッシュする
      resources:
        limits:
          nvidia.com/gpu: 1 ← 必要なGPUの数を指定する
```

いつも通りPodを作成します．

```
# kubelet create -f 作成したyml
```

Podの起動を確認します．

```
# kubectl get pods --all-namespaces -o wide | grep gpu-pod
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE       IP               NODE
default       gpu-pod                                1/1       Running   0          3d        10.244.1.7       lotus
```

コンソールにアクセスしてmnistを動かしてみます．

```
# kubectl exec -it gpu-pod /bin/bash
root@gpu-pod:/# apt-get install python-pip git
root@gpu-pod:/# pip install chainer
root@gpu-pod:/# git clone https://github.com/chainer/chainer
root@gpu-pod:/# cd chainer
root@gpu-pod:/# git checkout v3
root@gpu-pod:/# cd examples/mnist/
root@gpu-pod:/# python train_mnist.py -g 0
GPU: 0
# unit: 1000
# Minibatch-size: 100
# epoch: 20
～中略～
19          0.0124792   0.107687              0.996549       0.9801                    38.4196
20          0.0113122   0.119661              0.996782       0.9821                    40.4097
```

無事学習が回りました．
