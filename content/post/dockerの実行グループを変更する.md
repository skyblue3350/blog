+++
author = "すかい"
title = "dockerの実行グループを変更する"
date = "2018-07-23"
description = "dockerの実行グループを変更する"
tags = [
    "Docker",
]
+++

## はじめに

dockerコマンドを実行する際に実行ユーザーがdockerグループに所属していない場合sudoをつけて実行する必要があります．
ユーザーをdockerグループに所属させれば済む話ではありますが既にある程度の数のユーザーとそのユーザーが所属するグループがある場合いちいちdockerグループにユーザーを追加させるのは面倒です．
今回はdockerの実行グループを変更して解決します．

今回はserviceファイルに追記しますがdaemon.jsonに書いた方が良い気がします…

## 共通でやっておくこと

まず目的のグループのIDをidコマンドかgetent辺りを使って確認します．

```
$ getent group hoge
hoge:*:9999:foo,bar
```

## serviceファイルに追記する場合

次にdocker.serviceを編集します．
Gオプションでデフォルトグループから変更出来るので変更します．
グループ名でも指定できますが当該グループがLDAPで管理されたグループの場合参照エラーが発生してサービスが起動しなかったためグループIDで指定しています．

```
$ sudo vi /usr/lib/systemd/system/docker.service
- ExecStart=/usr/bin/dockerd
+ ExecStart=/usr/bin/dockerd -G 9999
```

サービスを再起動します

```
$ sudo service docker restart
```

これでグループhogeに所属しているユーザーはコマンドを実行できるようになったはずです．

## daemon.jsonを利用する場合

/etc/docker/daemon.jsonがない場合は作成
何かしら書いている場合は整合性を保って以下を追記

```json
{
    "group": "9999"
}
```

あとはdockerサービスを上げ直して終了

```
$ sudo service docker restart
```

## 参考記事

以下の記事が参考になりました．

- [low order magic docker -G and non-local groups](http://pvh.wp.sanbi.ac.za/2015/10/15/docker-g-and-non-local-groups/)
