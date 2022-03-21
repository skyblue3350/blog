+++
author = "すかい"
title = "k8sでPod内からAPIを叩く"
date = "2019-03-08"
description = "k8sでPod内からAPIを叩く"
tags = [
    "Python",
    "kubernetes",
]
+++

k8sでAPIを叩く記事をだいぶ前に書きましたが最終的にPodとして配置することになりました．
この場合はtokenをベタに書かなくてももっと手軽に認証できるので試します．

環境はk8s v1.9.3で検証しています．

Goだと下記みたいなやつをPythonでやりたいというお話です．
https://github.com/kubernetes/client-go/tree/master/examples/in-cluster-client-configuration

ちなみにexampleの中にありました．
https://github.com/kubernetes-client/python/blob/master/examples/in_cluster_config.py

ほぼ[前回のGKEでAPIを利用してみる](../gke%E3%81%A7api%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B/)と同じです．
PodからAPIを利用するためにServiceAccountが必要となるのでまずServiceAccountを作成します．

今回以下の条件で作成します

- Namespace
ns_sample
- ServiceAccount
sa_sample
- ClusterRole
cr_sample
- ClusterRoleBinding
crb_sample

この条件でapi.ymlを作ってリソースを作成します．

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ns_sample

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa_sample
  namespace: ns_sample

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cr_sample
  namespace: magi

rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: crb_sample
  namespace: ns_sample

subjects:
  - kind: ServiceAccount
    name: sa_sample
    namespace: ns_sample
roleRef:
  kind: ClusterRole
  name: cr_sample
  apiGroup: rbac.authorization.k8s.io
```

リソースを作成します

```
$ kubectl create -f api.yml
```

各リソースの状態に関しては前回の記事を参考に確認します．
最後にこれをPodから利用します．
Pod内部からの認証は下記コードでできます．

```py
from kubernetes import client, config

config.load_incluster_config()
```

このコードを実行するPodにServiceAccountを紐付ければ自動でSecretを読み込み認証が行われます．

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample
spec:
  serviceAccountName: sa_sample
  containers:
  - name: sample
    image: sample/image
```

既存の認証との切り替えは環境変数にKUBERNETES_SERVICE_HOSTがあるかを基準にすると良さそうです

```py
if os.environ.get("KUBERNETES_SERVICE_HOST"):
    configuration = config.load_incluster_config()
```
