+++
author = "すかい"
title = "k8sでingress-nginxを設定する"
date = "2018-03-29"
description = "k8sでingress-nginxを設定する"
tags = [
    "kubernetes",
]
+++

## はじめに

オンプレk8sでingress-nginxを設定する記事があんまり見つからないので苦労したのでメモ

## 環境

Master Clusterともに同じ環境です
説明の都合上
Masterは192.168.1.10
Workerは192.168.1.11
で説明します

```
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-07T11:55:20Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-07T12:22:21Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-07T11:55:20Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

## インストール

基本的にymlをドキュメント通り入れるだけです

```
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml \ 
    | kubectl apply -f  - 
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml \
    | kubectl apply -f -
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml \
    | kubectl apply -f -
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml \
    | kubectl apply -f -
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml \
    | kubectl apply -f -
```

1.9だとRBACが有効になっているのでRBACの設定も投入します

```
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml \
    | kubectl apply -f -
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/with-rbac.yaml \
    | kubectl apply -f -
```

デフォルトのままだと外部からアクセスすることができないため外部からアクセスする手段を提供する必要があります
現在ここから先のベストな解がわかっていなくてちょっと困っていますが外部からアクセスできるところまではいけたのでメモしておきます

```
$ kubectl edit deployment nginx-ingress-controller -n ingress-nginx
～省略～
      dnsPolicy: ClusterFirst
+      hostNetwork: true
      restartPolicy: Always
```

これでClusterの全てにそれぞれNginxがホストネットワークモードで配備されるので192.168.1.0/24下から

```
$ curl 192.168.1.11 -H "Host: Ingressで設定したホスト名"
```

とすることで閲覧できるようになっているはずです

## 気になること

hostNetwork: trueとするのが正しいのかちょっと気になる

素直にやるならNodePortで80と443を公開すれば良いはず…（[これとか](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/provider/baremetal/service-nodeport.yaml)）
ですが入れても外部から見せるためにはExternalIPを明示的に指定する必要があってこれじゃない感があります
hosPortで80と443だけ公開するのが綺麗な手段に見えるんですが理解不足もあって上手くいってないです

type: LoadBalancerを使えばExternal IPが自動で付与されるらしいですがクラウドサービス側から提供されるものらしいのでオンプレ環境で使うのは大変そうです
[zlabjp/nghttpx-ingress-lb](https://github.com/zlabjp/nghttpx-ingress-lb)とかもあるのでこちらを使った方が良いのかもしれないですね

## 参考記事

- [kubernetes / ingress-nginx](https://github.com/kubernetes/ingress-nginx/tree/master/deploy)
- [Qiita 快適な kubernetes オンプレミス環境を構築する(5. nginx-ingress-controllerを使ったIngress環境のセットアップ)](https://qiita.com/kaishuu0123/items/031f071b970e3be5666b)

## 関係してそうなページ

- [kubernetes/ingress-nginx nginx-ingress port binding not working #1911](https://github.com/kubernetes/ingress-nginx/issues/1911)
- [kubernetes/kubernetes HostPort seemingly not working #23920](https://github.com/kubernetes/kubernetes/issues/23920)
