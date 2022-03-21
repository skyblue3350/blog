+++
author = "すかい"
title = "kubernetes 1.8を構築する"
date = "2017-11-12"
description = "kubernetes 1.8を構築する"
tags = [
    "kubernetes",
]
+++

## はじめに

kubernetes 1.8を構築してみた
swapが有効だと裏でサービスがコケて上手く行かなかったり権限周りでダッシュボードが動かなかったりと詰まりポイントが多かったのでメモ書き程度に残しておく

追記
1.8と1.9.2と1.9.3で同じ方法でセットアップできるのを確認しました．
アップグレードはドキュメント読んでやってください．

## ドキュメント

公式を読みます

- [kubernetes setup Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

## 環境

- Ubuntu 16.04 x 2
  2台とも物理マシン

### Master

```
root@kubernetes:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
root@kubernetes:~# uname -a
Linux kubernetes 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

### Worker

```
root@kubernetes-node:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.2 LTS"
root@kubernetes-node:~# uname -a
Linux kubernetes-node 4.4.0-92-generic #115-Ubuntu SMP Thu Aug 10 09:04:33 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

## 環境構築

途中までは一緒

### 共通部分

#### docker

Dockerはk8sの公式の方法ではなくDocker公式の方法でk8sが対応してる最新を入れます
1.8時点でDocker17.03まで対応してるとのことなのでその辺を入れます

```
# apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# apt-key fingerprint 0EBFCD88
# add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
# apt-get update
# apt-get install docker-ce=17.03.2~ce-0~ubuntu-xenial
# docker -v
Docker version 17.03.2-ce, build f5ec1e2
# service docker start && service docker status
```

#### kubernetes

Googleのレポジトリ登録して関連パッケージを入れる

```
# apt-get update
# apt-get update && apt-get install -y apt-transport-https
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
# apt-get update
# apt-get install -y kubelet kubeadm kubectl
```

### Master

スワップを切っておきます（1.8かららしいです）

initする時にあとでflannelを入れる関係でipレンジを指定しておきます

```
root@kubernetes:~# swapoff -a
root@kubernetes:~# kubeadm init --pod-network-cidr=10.244.0.0/16
～中略～
You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

initが終了するとクラスタ追加用のURLが生成されて終了するので控えておく
tokenは基本的に24時間で期限切れになるので作業時間が開くと期限切れになって使えなくなる（なった
そういう場合は生成済みのものがないか確認した上で生成する

```
root@kubernetes:~# kubeadm token list
TOKEN     TTL       EXPIRES   USAGES    DESCRIPTION   EXTRA GROUPS

root@kubernetes:~# kubeadm token create
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --ttl 0)
81b4ea.9fa80ebbc22fa190

root@kubernetes:~# kubeadm token list
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION   EXTRA GROUPS
81b4ea.9fa80ebbc22fa190   23h       2017-11-03T14:05:44+09:00   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
```

これだけだとtokenしか取れないので他の引数がわからないのでtoken生成時にjoin用のURLを表示するオプションがある

```
# kubeadm token create --print-join-command
```

って感じです

あとkubectlを叩くためのコマンドのconfigファイルが生成されるのでコピーしておく
kubeadm initした時に最後に書いてある

```
# mkdir -p $HOME/.kube
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# chown $(id -u):$(id -g) $HOME/.kube/config
```

最後に使用するネットワークを設定する
色々選べるけどひとまず情報が多そうなflannelにしておく

```
root@kubernetes:~# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
root@kubernetes:~# kubectl apply -f kube-flannel.yml
root@kubernetes:~# service kubelet restart
```

って感じでおっけー

ホストマシンもクラスタの1台として使う場合は以下のコマンドを投入して動くようにしておく．

```
root@kubernetes:~# kubectl taint nodes --all node-role.kubernetes.io/master-
node "kubernetes" untainted
```

一応podの確認をしておく

```
root@kubernetes:~# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   etcd-kubernetes                      1/1       Running   0          12m       192.168.227.7   kubernetes
kube-system   kube-apiserver-kubernetes            1/1       Running   0          12m       192.168.227.7   kubernetes
kube-system   kube-controller-manager-kubernetes   1/1       Running   0          11m       192.168.227.7   kubernetes
kube-system   kube-dns-545bc4bfd4-mkx62            3/3       Running   0          12m       10.244.0.2      kubernetes
kube-system   kube-flannel-ds-45tc8                1/1       Running   0          11m       192.168.227.7   kubernetes
kube-system   kube-proxy-lzvp7                     1/1       Running   0          12m       192.168.227.7   kubernetes
kube-system   kube-scheduler-kubernetes            1/1       Running   0          12m       192.168.227.7   kubernetes
```

dnsはクラスタがいれば動きますがMasterだけの時はPendingかなんかになってた気がします．
それ以外はRunningになってるはず．

### Worker

こっちもswapが有効だと起動しなくて詰まる
さっき生成したtokenを使って登録する

```
root@kubernetes-node:~# swapoff -a
root@kubernetes-node:~# kubeadm join --token [token文字列] 192.168.227.7:6443 --discovery-token-ca-cert-hash sha256:[sha256文字列]
～中略～
Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.
```

って出れば追加完了

```
[discovery] Failed to connect to API Server "192.168.227.7:6443": there is no JWS signed token in the cluster-info ConfigMap. This token id "9ddffd" is invalid for this cluster, can't connect
```

とか出てる場合は多分tokenの期限切れなので再生成する
追加できてるかMaster側で確認する

```
root@kubernetes:~# kubectl get nodes -o wide
NAME         STATUS    ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kubernetes   Ready     master    2h        v1.8.2    <none>        Ubuntu 16.04.3 LTS   4.4.0-62-generic   docker://Unknown
kubernetes-node        Ready     <none>    2m        v1.8.2    <none>        Ubuntu 16.04.2 LTS   4.4.0-98-generic   docker://1.12.6
```

## Dashboardの設定

コンソールからでもいいけどやっぱりWebUIからも見たい
ymlを突っ込んでproxyすれば見えるらしい

```
root@kubernetes:~# kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
root@kubernetes:~# kubectl proxy --address=192.168.227.7 -p 8001
Starting to serve on 192.168.227.7:8001
```

あとはポートフォワーディングすれば見える

```
$ ssh -L 8001:k8s.internal:8001 接続先
```

一部の記事では
http://localhost:8001/ui/
にアクセスすると自動でリダイレクトされるって書いてあるんだけど
https://github.com/kubernetes/dashboard/issues/2404#issuecomment-337134575
ここに書いてあるようにリダイレクトの実装にはまだバグがあるらしくエラーになって見えないので直にURLを入れて開く
READMEに書いてある通りに以下のURLを開く
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

どうも1.8から追加された権限周りの影響でそのまま使えないらしい
設定すれば使えるとのことだがIssueにあった説明がいまいちわからないので試行錯誤中

## トラブルシューティング

### kubelet が上がらない

```
# service kubelet start
# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since 日 2017-11-05 20:00:53 JST; 2s ago
     Docs: http://kubernetes.io/docs/
  Process: 7317 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELE
 Main PID: 7317 (code=exited, status=1/FAILURE)

11月 05 20:00:53 kubernetes systemd[1]: kubelet.service: Unit entered failed state.
11月 05 20:00:53 kubernetes systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

みたいになってる場合は多分swapが有効になってる
一応手動であげてみる

```
# /usr/bin/kubelet
～中略～
I1105 20:05:43.082110    7680 manager.go:216] Machine: {NumCores:4 CpuFrequency:3504000 MemoryCapacity:2097143808 HugePages:[{PageSize:1048576 NumPages:0} {PageSize:2048 NumPages:0}] MachineID:70f9f6adc07ef97bd4c9d2815959e520 SystemUUID:7F8E4D56-FB2A-7013-6F5B-51ABE73A6A74 BootID:50f740f4-e7d3-46c6-a5e1-9d560a26706d Filesystems:[{Device:/dev/mapper/kubernetes--vg-root DeviceMajor:252 DeviceMinor:0 Capacity:18381979648 Type:vfs Inodes:1148304 HasInodes:true} {Device:/dev/sda1 DeviceMajor:8 DeviceMinor:1 Capacity:494512128 Type:vfs Inodes:124928 HasInodes:true} {Device:tmpfs DeviceMajor:0 DeviceMinor:19 Capacity:209715200 Type:vfs Inodes:255999 HasInodes:true}] DiskMap:map[252:1:{Name:dm-1 Major:252 Minor:1 Size:2147483648 Scheduler:none} 8:0:{Name:sda Major:8 Minor:0 Size:21474836480 Scheduler:deadline} 252:0:{Name:dm-0 Major:252 Minor:0 Size:18811453440 Scheduler:none}] NetworkDevices:[{Name:ens160 MacAddress:00:0c:29:3a:6a:74 Speed:10000 Mtu:1500}] Topology:[{Id:0 Memory:2097143808 Cores:[{Id:0 Threads:[0] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2}]}] Caches:[{Size:8388608 Type:Unified Level:3}]} {Id:2 Memory:0 Cores:[{Id:0 Threads:[1] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2}]}] Caches:[{Size:8388608 Type:Unified Level:3}]} {Id:4 Memory:0 Cores:[{Id:0 Threads:[2] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2}]}] Caches:[{Size:8388608 Type:Unified Level:3}]} {Id:6 Memory:0 Cores:[{Id:0 Threads:[3] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2}]}] Caches:[{Size:8388608 Type:Unified Level:3}]}] CloudProvider:Unknown InstanceType:Unknown InstanceID:None}
I1105 20:05:43.082493    7680 manager.go:222] Version: {KernelVersion:4.4.0-62-generic ContainerOsVersion:Ubuntu 16.04.3 LTS DockerVersion:1.12.6 DockerAPIVersion:1.24 CadvisorVersion: CadvisorRevision:}
W1105 20:05:43.082800    7680 server.go:232] No api server defined - no events will be sent to API server.
I1105 20:05:43.082809    7680 server.go:422] --cgroups-per-qos enabled, but --cgroup-root was not specified.  defaulting to /
error: failed to run Kubelet: Running with swap on is not supported, please disable swap! or set --fail-swap-on flag to false. /proc/swaps contained: [Filename                         Type            Size    Used    Priority /dev/dm-1                               partition      2097148 0       -1]
```

最後にswapを無効にしろって書いてあるので無効にする

```
# swapoff -a
```

### 最初からやりたい時

Masterでの作業詰まってちょっと最初からやりたいって時
パッケージをpurgeしたらいけるでしょって思ったけど色々ダメだった

```
# kubeadm reset
# kubeadm init
```

したらいけた
