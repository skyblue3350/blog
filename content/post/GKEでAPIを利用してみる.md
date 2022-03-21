+++
author = "すかい"
title = "GKEでAPIを利用してみる"
date = "2018-11-27"
description = "GKEでAPIを利用してみる"
tags = [
    "Python",
    "kubernetes",
]
+++

GKEのAPIを利用してリソースの操作してみるメモ．
多分オンプレも同じですがまだ検証してないので動作確認したら追記します．

追記　2019/03/06
オンプレ環境下でも問題なく動きました

## 環境

環境は以下です．

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.7", GitCommit:"0c38c362511b20a098d7cd855f1314dad92c2780", GitTreeState:"clean", BuildDate:"2018-08-20T10:09:03Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9+", GitVersion:"v1.9.7-gke.11", GitCommit:"dc4f6dda6a08aae2108d7a7fdc2a44fa23900f4c", GitTreeState:"clean", BuildDate:"2018-11-10T20:22:02Z", GoVersion:"go1.9.3b4", Compiler:"gc", Platform:"linux/amd64"}
```

## APIの利用

### とりあえず叩いてみる

公式ドキュメント記載の方法でひとまず叩いてみます

- https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/

```
$ APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")
$ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t')
$ echo ${APISERVER}
$ echo ${TOKEN}
```

取得できたら試しに叩いてみます．

```
$ curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "xxx.xxx.xxx.xxx"
    }
  ]
}
```

ひとまず叩けました．
ちなみにAPISERVERはGKEの管理コンソールからも確認できます．
Google Cloud Console -> Kubernetes Engine -> クラスタ -> 任意のクラスタ -> クラスタのエンドポイントに記載があります．

APIの認証方式はいろいろあるようですが今回はサービスアカウントを用いた接続になっています．
具体的に追っていくと

```
$ kubectl get sa
NAME               SECRETS   AGE
default            1         7d
```

このように各ネームスペースにはネームスペースと同名のサービスアカウントが作成されています．
サービスアカウントにはそれに付随する認証情報が自動で生成されています．

```
$ kubectl get sa default -o yaml | grep token
- name: default-token-xxxxxx
```

xxxxx部分はランダムに割当られます．
次にこのsecretsの中身からtokenを取り出します．

```
$ kubectl get secrets default-token-xxxxxx -o yaml | grep token:
  token: [TOKEN]
```

これでtokenが取り出せたのであとはサンプルのcurlのようにリクエストを送ることで利用できます．

ただ，ネームスペースにあるデフォルトの権限では実際にリソースにアクセスすることができません．

権限はkubectl auth can-iを使うことで確認できます．
例えば普段ノード管理に使っているマシンからネームスペースを取得できるか確認すると

```
$ kubectl auth can-i list ns
yes
$ kubectl auth can-i list ns --as=system:serviceaccount:default:default
no
```

のようになります．
実際にアクセスしてみると

```
$ kubectl get ns --as=system:serviceaccount:default:default
Error from server (Forbidden): namespaces is forbidden: User "system:serviceaccount:default:default" cannot list namespaces at the cluster scope
$ kubectl get ns
NAME          STATUS    AGE
default       Active    7d
kube-public   Active    7d
kube-system   Active    7d
```

のようにデフォルトのサービスアカウントではアクセスできないことがわかります．
今回は専用のサービスアカウントを作成しそれに権限を与えていきます．

### 専用のサービスアカウントを作る

というわけで操作用のサービスアカウントを作成します．

```
$ kubectl create sa sample
$ kubectl get sa
NAME               SECRETS   AGE
default            1         7d
sample           1         20h
```

次にこのサービスアカウントへの権限を設定します．
権限は大きく2つに分けられます．
ネームスペース内だけで有効な「Role」とネームスペースをまたがって有効な「ClusterRole」です．
今回は，ClusterRoleを利用してみます．
以下のようにしてyamlを作成して権限を設定します．
今回は大雑把に発行しますが，細かく設定できるので適宜設定します．

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sample-cluster-role
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
```

次にこのClusterRoleとサービスアカウントの紐づけを行う「ClusterRoleBinding」を設定します．
今回はClusterRoleなのでClusterRoleBindingになりますが，Roleとのヒモ付を行う場合は「RoleBinding」になります．

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sample-cluster-role-binding
subjects:
  # サービスアカウントとのヒモ付
  - kind: ServiceAccount
    name: sample
    namespace: default
# ClusterRoleとのヒモ付
roleRef:
  kind: ClusterRole
  name: sample-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

それぞれのyamlファイルを用いてリソースを作成します．

```
$ kubectl create -f clusterrole.yaml
$ kubectl create -f culsterrolebinding.yaml
```

これで紐づけができたはずなのでパーミッションを確認してみます．

```
$ kubectl auth can-i get ns --as=system:serviceaccount:default:sample
yes
```

問題なく設定されていますね．
kubectlでも試してみます．

```
$ kubectl get ns --as=system:serviceaccount:default:sample
NAME               SECRETS   AGE
default       Active    7d
kube-public   Active    7d
kube-system   Active    7d
```

問題ないですね．
次に，API経由でのアクセスを試します．
適当に取得します．

```
$ kubectl get sa sample -o yaml | grep token
- name: sample-token-xxxxxx
$ kubectl get secrets sample-token-xxxxxx -o yaml | grep token:
  token: [TOKEN]
```

これをbase64デコードして利用するので，適当にカットして使います．

```
$ TOKEN=$(kubectl get secrets sample-token-xxxxxx -o yaml | grep token: | cut -f4 -d " " | base64 -d)
```

さっきと同じようにcurlでAPIを叩いてみます．

```
$ curl $APISERVER/api/v1/namespaces --header "Authorization: Bearer $TOKEN" --insecure | grep \"name\"
        "name": "default",
        "name": "kube-public",
        "name": "kube-system",
```

取得できましたね．
defaultのTOKENに差し替えて試すと権限不足のエラーが出ます．

### API Clientから利用してみる

次に公式提供のClientから利用してみます．

```py
import os

from kubernetes import client, config

configuration = client.Configuration()
configuration.verify_ssl = False
configuration.host = os.environ.get("APISERVER")
configuration.api_key["authorization"] = os.environ.get("TOKEN")
configuration.api_key_prefix["authorization"] = "Bearer"

apiClient = client.ApiClient(configuration)
v1 = client.CoreV1Api(apiClient)

for ns in v1.list_namespace().items:
    print(ns.metadata.name)
```

を実行すると

```
default
kube-public
kube-system
```

のように取得できるはずです．

## エラーとか

- Error from server (Forbidden): error when creating hoge.yaml
ClusterRoleを作ろうとしたらエラーが出た．
GKEではデフォルトで自分のアカウントに作成権限がないらしいので付与する必要があるらしい．

```
$ gcloud info | grep Account
Account: [example@example.com]
$ kubectl create clusterrolebinding [なんか適当に名前をつける] \
  --clusterrole=cluster-admin \
  --user=example@example.com
```

のようにして自分にcluster-admin権限を付与する．

- verifyがFalseだけど良いの？
  良くない．
  secretsにあるcertをファイルに落として
  ```
  configuration.ssl_ca_cert = "/path/to/cert/file"
  ```
  とかしたらいけるはず（試してない

## 参考記事

- [How To Setup Kubernetes API Access Using Service Account](https://devopscube.com/kubernetes-api-access-service-account/)
- [Kubernetes: RBACの設定におけるAPIリソース](https://qiita.com/tkusumi/items/300c566a74b6b64e7e89)
- [KubernetesのService Accountについて調べてみた](https://qiita.com/knqyf263/items/ecc799650fe247dce9c5)
