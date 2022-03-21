+++
author = "すかい"
title = "Pyro4の使い方"
date = "2016-07-12"
description = "Pyro4の使い方"
tags = [
    "Pyro",
    "Python",
]
+++

## Pyro4

異なるアーキテクチャ間でリモートでメソッドコールできるライブラリ
あんまり日本語の情報がないのでメモ程度に書いておく

日本語の情報はほとんどないから諦めて英語で探す
https://pythonhosted.org/Pyro4/
公式のGithubのExampleとか参考にする
https://github.com/delmic/Pyro4/

## インストール

```
$ pip install Pyro4
```

Pyroだと古いバージョンが入るので注意がいる

### ネームサーバーなし
ネームサーバーがあると裏で勝手に名前解決してくれるので楽になるけどなしでもできる

基本的に自分がクライアント側になるはず

#### ホスト側

```py
import Pyro4

def getHoge():
    return "test"
 
Pyro4.Daemon.serveSimple(
    {getHoge:'HogeMethod'},
    host="自分のIP",
    port=9090,#適当なポート
    ns=False,
)
```

#### クライアント側

```py
import Pyro4

name = "HogeMethod"
ip = "相手のIP"
port = 9090#適当なポート　合わせる
s = "PYRO:%s@%s:%s" % (name, ip, port)
remote = Pyro4.Proxy(s)
remote.getHoge()
```

みたいな感じで呼べる

### ネームサーバあり

#### ネームサーバ

用意されてるものを使う場合は

```
$ python -m Pyro4.naming
```

随時オプションをつける
WindowsだとCtrl+Cとかで抜けられないので不便
自分でコードを書くなら

```py
import Pyro4
import socket
 
host = socket.gethostbyname(socket.gethostname())#VMの仮想NIC取りに行ったりするから微妙
host = "ダメなら手動で入れる"
port = 9900
url, daemon, bcserver = Pyro4.naming.startNS(
	host = host,
	port = port,
)

print "ready", url
print "listening %s:%s" % (host, port)
#適当にスレッド化して中断出来るようにしておく
import threading
thread = threading.Thread(target = daemon.requestLoop)
thread.setDaemon(True)
thread.start()
while True: pass
```

確認する

```
$ python -m Pyro4.nsc list
```

引数あり

```
$ python -m Pyro4.nsc -n 192.168.1.114 -p 9900 list
--------START LIST
Pyro.NameServer --> PYRO:Pyro.NameServer@192.168.1.114:9900
    metadata: (u'class:Pyro4.naming.NameServer',)
robot --> PYRO:obj_91e53083bd114775adeaed91e73018bd@localhost:32977
--------END LIST
```

同じようにpingも飛ばせる

#### ホスト側

単一ノード上なら動くけど他端末との通信に失敗する
Pyro4.Daemon() メソッドの引数がネームサーバーに登録されるから忘れると失敗する

```py
# -*- encoding:utf-8 -*-
 
import Pyro4
import socket
 
#####################################################
#
#　リモートから呼ばれるメソッド
#
#####################################################
 
class PyroTest1(object):
	def hello(self):
		return "Hello World"
	def world(self):
		return "world"

class PyroTest2(object):
	def hello(self):
		return "H"
	def world(self):
		return "W"
 
#####################################################
 
#ローカルIP取得
def getLocalIP():
	import socket
	s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
	s.connect(("8.8.8.8",80))
	ip = s.getsockname()[0]
	s.close()
 
	return ip
 
#ネームサーバに登録する時の元情報
daemon = Pyro4.Daemon(
	host = "192.168.133.105",
	port = port,
)
 
#ネームサーバ情報
nameserver = Pyro4.locateNS(
	host = getLocalIP(),
	port = port,
)
#登録
nameserver.register("test1" ,daemon.register(PyroTest1))
nameserver.register("test2" ,daemon.register(PyroTest2))
 
print "everything ready ok"
import threading
thread = threading.Thread(target = daemon.requestLoop)
thread.setDaemon(True)
thread.start()
while True: pass
```

#### クライアント側

呼び出して終わり

```py
# -*- encoding:utf-8 -*-
 
import Pyro4
 
nameserver = Pyro4.locateNS(
	host = "192.168.1.114",
	port = 9090
)
 
uri = nameserver.lookup("test1")
print uri
obj = Pyro4.Proxy(uri)
print obj.hello()
```
