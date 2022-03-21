+++
author = "すかい"
title = "k8sでNFSマウント出来なくて困った話"
date = "2018-05-29"
description = "k8sでNFSマウント出来なくて困った話"
tags = [
    "kubernetes",
]
+++

## はじめに

k8sでNFSマウントしようとすると

```
Unable to mount volumes for pod "test_default(xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)": timeout expired waiting for volumes to attach/mount for pod "default"/"test". list of unattached/unmounted volumes=[nfs]
```

みたいなエラーで怒られて上手くいかなかったのでその原因の調べ方と設定時の注意点のメモ

そもそもどうやってマウントするの？みたいな話は公式のドキュメント見た方がわかりやすいのでそちらに譲ります
DockerホストでNFSをマウントして条件に合致するpodにデータボリュームでNFSの領域をマウントしてるっぽい

- https://kubernetes.io/docs/concepts/storage/volumes/#nfs
- https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs

## 環境

相変わらずオンプレです．
物理マシンを2台にUbuntu16.04をインストールし，それぞれMasterとClusterとして設定してあります．
バージョンはv1.9.3です．

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-07T12:22:21Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-07T11:55:20Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

今回はMaster側にNFS Serverを設定しましたが適宜別のマシン等でNFSを提供できるように設定しておきます．
一応exportsは制限なしで設定した上でこの記事を書いていますが仕組み的にDockerをホストしているマシンからさえマウントできれば問題ありません．

```
$ tail -n 1 /etc/exports
/opt/nfs *(rw,sync,no_subtree_check,no_root_squash)
```

## 設定ファイル

PVと作成してPVの条件に適合するPVCを作ってそれをpodにマウントしてやればOKです．
`volume.beta.kubernetes.io/storage-class` はあとでPVCがマウント可能なPVを探す際に条件として扱われるので値を合わせておく必要があります．
空欄にしておけばすべて適合する（はず

### PV

```
$ cat nfs-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  annotations:
    volume.beta.kubernetes.io/storage-class: "nfs"
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 192.168.xxx.xxx
    path: /opt/nfs
```

作成して確認します．
既にpvc出来てますけど実際は空欄か何かになっています．

```
$ kubectl create -f nfs-pv.yml
$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM               STORAGECLASS   REASON    AGE
nfs-pv      50Gi       RWX            Retain           Bound     default/nfs-pvc   slow                     20m
```

### PVC

合致するPVがないと永遠に使用可能にならないので上で作成したPVに合致するような設定を入れます．

```
$ cat nfs-pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: slow
```

こちらも作成して確認します．

```
$ kubectl get pvc
NAME        STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound     nfs-pv      50Gi       RWX            slow           38m
```

問題なく作成されていればひとまずpodでマウントできるか確認します．

```
$ cat nfs-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
spec:
  containers:
    - name: nfs-test
      image: nginx
      volumeMounts:
        - name: nfs
          mountPath: "/var/www/"
  volumes:
    - name: nfs
      persistentVolumeClaim:
        claimName: nfs-pvc
```

あとはpod作って問題なく動けば終了です．

```
$ kubectl create -f nfs-pod.yml
pod "nfs-test" deleted
$ kubectl get pod
NAME      READY     STATUS    RESTARTS   AGE
nfs-test      1/1       Running   0          42m
$ kubectl describe pod nfs-test
Events:
  Type    Reason                 Age   From                 Message
  ----    ------                 ----  ----                 -------
  Normal  Scheduled              11s   default-scheduler    Successfully assigned demo to ノード名
  Normal  SuccessfulMountVolume  10s   kubelet, ノード名  MountVolume.SetUp succeeded for volume "nfs-pvc"
  Normal  SuccessfulMountVolume  10s   kubelet, ノード名  MountVolume.SetUp succeeded for volume "default-token-xxxxxx"
  Normal  Pulling                9s    kubelet, ノード名  pulling image "nginx"
  Normal  Pulled                 6s    kubelet, ノード名  Successfully pulled image "nginx"
  Normal  Created                6s    kubelet, ノード名  Created container
  Normal  Started                6s    kubelet, ノード名  Started container
```

適当にファイルおいてみて確認しておわりです．

## 詰まったとことか

### Node には必ず nfs-common が必要

漠然とDocker自体でマウントしているイメージで用意していませんでした．
先にDockerでNFSマウントできるか試したので先入観良くない．
describeすると以下のエラーが出ます．

```
$ kubectl get describe test
～省略～
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/pods/xxxxxxxx-xxxx-xxxx-xxxxxxxx/volumes/kubernetes.io~nfs/nfs-pv --scope -- mount -t nfs 192.189.xxx.xxx:/opt/nfs /var/lib/kubelet/pods/xxxxxxxx-xxxx-xxxx-xxxxxxxx/volumes/kubernetes.io~nfs/nfs-pv
Output: Running scope as unit run-r9da60928545e43c49189115bc75d8d44.scope.
mount: wrong fs type, bad option, bad superblock on 192.189.xxx.xxx:/opt/nfs,
       missing codepage or helper program, or other error
       (for several filesystems (e.g. nfs, cifs) you might
       need a /sbin/mount.<type> helper program)

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
```

nfs-common入れればマウントできるようになるので解決します．

```
$ sudo apt-get install nfs-common
```

### タイムアウトしてしまう

以下のようなエラーでマウント出来ない

```
$ kubectl describe pod nfs-test
～中略～
Events:
  Type     Reason                 Age   From                 Message
  ----     ------                 ----  ----                 -------
  Normal   Scheduled              2m    default-scheduler    Successfully assigned demo to ノード名
  Normal   SuccessfulMountVolume  2m    kubelet, ノード名  MountVolume.SetUp succeeded for volume "default-token-xxxxx"
  Warning  FailedMount            44s   kubelet, ノード名  Unable to mount volumes for pod "nfs-test_default(xxxxx-xxxx-xxxx-xxxxxx)": timeout expired waiting for volumes to attach/mount for pod "default"/"nfs-test". list of unattached/unmounted volumes=[nfs]
```

もう少し待ってると詳細エラーが出てきます

```
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/pods/xxxxx-xxxx-xxxx-xxxxxx/volumes/kubernetes.io~xxxxx-xxxx-xxxx-xxxxxx/volumes/kubernetes.io~nfs/nfs-pvc
Output: Running scope as unit run-xxxxx-xxxx-xxxx-xxxxxx.scope.
mount.nfs: Connection timed out
```

メモし損ねましたがこの前後に実行されたmountコマンドが表示されているので実際にマウントできるかテストしましょう．
自分の場合はしょうもないですがnfs-pvのIPアドレスを打ち間違えていたことにここで気付きました．
