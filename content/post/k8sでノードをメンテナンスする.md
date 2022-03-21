+++
author = "すかい"
title = "k8sでノードをメンテナンスする"
date = "2018-08-28"
description = "k8sでノードをメンテナンスする"
tags = [
    "kubernetes",
]
+++

k8sでノードを切り離したい時やメンテナンスしたい時のメモ

```
$ kubectl get node -o wide
NAME             STATUS                        ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kube01        Ready                         <none>    195d      v1.9.3    <none>        Ubuntu 16.04.3 LTS   4.4.0-112-generic   docker://17.12.0-ce
kube02        Ready                         <none>    195d      v1.9.3    <none>        Ubuntu 16.04.3 LTS   4.4.0-112-generic   docker://17.12.0-ce
kubemaster   Ready                         master    195d      v1.9.3    <none>        Ubuntu 16.04.5 LTS   4.4.0-112-generic   docker://18.6.0
$ kubectl get pod -o wide --all-namespaces | grep kube02
動いてるコンテナ一覧が見える
```

というわけでノードに配置されているコンテナを追い出す
Deployment系は多分再配置が行われるがPOD単体のものなどは多分行われないので事前に確認しておくこと

```
$ kubectl drain --ignore-daemonsets --force kube02
```

メンテが終わったら再スケジュールするようにする

```
$ kubectl uncordon kube02
```
