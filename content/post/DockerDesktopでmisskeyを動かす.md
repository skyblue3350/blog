+++
author = "すかい"
title = "DockerDesktopでmisskeyを動かす"
date = "2023-07-18"
description = "DockerDesktopのk8s上でmisskeyを動かしてみる"
tags = [
    "misskey",
    "kubernetes",
]
+++

Docker Desktop上でkubernetesを起動して、その上にmisskeyを構築してみます。

## バージョン

- Docker Desktop
  - 4.21.1
- docker
  - 24.0.2
- k8s
  - 1.27.2
- helm
  - 3.10.3

## k8sの起動

Docker Desktop Settings -> Kubernetes -> Enable Kubernetes にチェックを入れて有効化しておきます。

適当にPodの起動状態を確認しておきます。

```
kubectl get pod -n kube-system
NAME                                     READY   STATUS    RESTARTS        AGE
coredns-5d78c9869d-2tn9p                 1/1     Running   2 (4m2s ago)    6d19h
coredns-5d78c9869d-647kd                 1/1     Running   2 (4m2s ago)    6d19h
etcd-docker-desktop                      1/1     Running   2 (4m2s ago)    6d19h
kube-apiserver-docker-desktop            1/1     Running   2 (4m2s ago)    6d19h
kube-controller-manager-docker-desktop   1/1     Running   2 (4m2s ago)    6d19h
kube-proxy-qdzs4                         1/1     Running   2 (4m2s ago)    6d19h
kube-scheduler-docker-desktop            1/1     Running   2 (4m2s ago)    6d19h
storage-provisioner                      1/1     Running   4 (3m29s ago)   6d19h
vpnkit-controller                        1/1     Running   2 (4m2s ago)    6d19h
```

## ingressの設定

公式ドキュメントを参考にingressを動作させるのに必要なマニフェストをapplyします。

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

## helmを使ってmisskeyを構築

configを書き出します。

```bash
helm inspect values TrueCharts/misskey > misskey-config.yaml
```

URLをDocker Desktopで使われているものに変更しておきます。

```diff
misskey:
  # Final accessible URL seen by a user. ONCE YOU HAVE STARTED THE INSTANCE, DO NOT CHANGE THE URL SETTINGS AFTER THAT!
- url: "https://example.tld/"
+ url: "http://kubernetes.docker.internal/"
  # ID generation method. 'aid' recommended.
  id: "aid"
```

applyします

```bash
helm install misskey -n misskey -f misskey-config.yaml TrueCharts/misskey
```

Podの起動状況を確認して起動したことを確認します。

```
kubectl get pod -n misskey
NAME                       READY   STATUS    RESTARTS      AGE
misskey-77f675c9bb-j5wgg   1/1     Running   1 (32m ago)   29h
misskey-postgresql-0       1/1     Running   1 (32m ago)   31h
misskey-redis-0            1/1     Running   1 (32m ago)   29h
```

## ingressの設定

port-forwardで中身を見ても良いですが、せっかくなのでingressで公開します。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: misskey-ingress
  namespace: misskey
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: kubernetes.docker.internal
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: misskey
            port:
              number: 3003
```

applyします。

```
kubectl apply -f misskey-ingress.yaml
```

kubernetes.docker.internalにアクセスして動作することを確認します。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">適当にローカルで動かすだけなら思ったより簡単だな…となるなど <a href="https://t.co/tzLgjO90Qg">pic.twitter.com/tzLgjO90Qg</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1680814692061814784?ref_src=twsrc%5Etfw">July 17, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

絵文字の登録なども問題なさそうでした。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">適当に遊んでる <a href="https://t.co/iTDZfKNDHT">pic.twitter.com/iTDZfKNDHT</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1680823275302301697?ref_src=twsrc%5Etfw">July 17, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 参考リンク

- [Kubernetes/TrueNASを使ったMisskey構築](https://misskey-hub.net/docs/install/kubernetes.html)
- [Ingress-Nginx Controller Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/#docker-desktop)
