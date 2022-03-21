+++
author = "すかい"
title = "Camera Remote APIを利用してILCE-6000のライブビューを取得する"
date = "2018-12-14"
description = "Camera Remote APIを利用してILCE-6000のライブビューを取得する"
tags = [
    "Python",
]
+++

存在だけ知って触ったことのなかったCamera Remote APIを触ってみました．

## 環境
- ILCE-6000
α6000
本体ソフトウェア Ver3.20

## ドキュメントとか

https://developer.sony.com/ja/develop/cameras/
公式ページのDownload SDKからドキュメント落とせるので基本的にはそれを見てHTTPを叩きます．
カメラがAPIに対応しているかは以下から確認できます．
https://developer.sony.com/ja/develop/cameras/api-information/supported-features-and-compatible-cameras

## 接続する

まず，同じネットワークにつなぎます．
カメラ側を宅内のネットワークにつなげれば良いのですがAPの登録はできても接続し続けておく方法がわからなかった．
のでAP化する方法で進める．

## カメラのAP化

MENU -> 設定 -> アプリ一覧からリモートコントロールを選択．
カメラのモニタにAP名とパスワードが出るのでPCを接続．

## カメラのIPを調べる

SSDPを使ってカメラのIPアドレスを教えてもらいます．
ただ，カメラをAPとして使っている場合は返ってくる値に変化はないのであんまりやる意味ないです．
上手く返ってくるパケットが捕まえられなかったので今回はWiresharkを使って戻ってきたパケットを読みました．

```py
import socket

request = "\r\n".join([
	"M-SEARCH * HTTP/1.1",
	"HOST: 239.255.255.250:1900",
	"MAN: \"ssdp:discover\"",
	"MX: 1",
	"ST: urn:schemas-sony-com:service:ScalarWebAPI:1"])

sock = socket.socket(socket.AF_INET , socket.SOCK_DGRAM)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL , 2)

msg_bytes = bytearray(request, "utf8")
sock.sendto(msg_bytes, ("239.255.255.250", 1900))
sock.close()
```

送ってしばらくすると4種類くらいのフォーマットでカメラから応答が来るのでそこからXMLを返してくれるエンドポイントへアクセスします．
多分レスポンスの中に以下のURLがあります．

```
192.168.122.1:61000/scalarwebapi_dd.xml
```

これにアクセスするとこんな感じで欲しい情報が取れます．

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">何故かスクリプトから引っ張ってこれないのでパケットキャプチャして見た情報から叩いたけど一応取れた <a href="https://t.co/VJCglZxtgT">pic.twitter.com/VJCglZxtgT</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1073264341527420928?ref_src=twsrc%5Etfw">December 13, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ドキュメントにある<ACTION_URL>/cameraの<ACTION_URL>にあたる部分が

```xml
<av:X_ScalarWebAPI_ActionList_URL>http://192.168.122.1:8080/sony</av:ScalarWebAPI_ActionList_URL>
```

となります．

## APIを叩いてみる

APIは基本的にPOSTリクエストで要求をjsonでbodyに格納して送ります．
対象のカメラの使用可能なAPIのリストはgetAvailableApiListで取得できるのでこんな感じで取れます．

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ないのか… <a href="https://t.co/E0bevA2fMK">pic.twitter.com/E0bevA2fMK</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1073509401913257985?ref_src=twsrc%5Etfw">December 14, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## ライブビューを取得する

本題のライブビューです．
ドキュメントP.278のSample Sequenceにも説明がある通りライブビューの取得には
startRecMode -> startLiveview -> レスポンスにあるURLから画像取得
といった手続きをする必要があります．

画像は独自のストリームで流れてきます．
ストリームの仕様はP.272のStreaming data formatで詳しく解説されています．

<script src="https://gist.github.com/skyblue3350/90105ab2bc56876e54c4ceb99ad904c8.js"></script>

あとはこんな感じで取得できます．
開いている間はひたすら同じStreamが流れてくるのでパースしていけば良いです．

## サイズの変更

ただ，これだとサイズが640x424の画像しか取れないのでもう少し大きいサイズが欲しくなります．
startLiveviewWithSizeを使えば適宜サイズを変更できるとのことでしたが，このカメラだとダメなようです．

## 余談

Webカメラ的なノリで使えないかなーと思ってトライしましたがダメそうでした．
大人しくHDMIの外部出力をキャプチャボードとかで取った方が良さそうです．
そもそも長時間の映像撮影は向いてないとのことで，大人しくWebカメラ買った方が良いみたいです．
