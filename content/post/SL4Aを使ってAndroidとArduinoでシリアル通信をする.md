+++
author = "すかい"
title = "SL4Aを使ってAndroidとArduinoでシリアル通信をする"
date = "2016-07-13"
description = "SL4Aを使ってAndroidとArduinoでシリアル通信をする"
tags = [
    "Android",
    "Python",
    "SL4A",
]
+++

## はじめに

AndroidとArduinoでPythonを使ってシリアル通信をしたかった
Android上でPythonを実行するアプリはいくつかあるけどQPythonのserialモジュールでは失敗した
QRコードでコード読み込めたりして便利だったけど残念

## SL4A
Android上でスクリプト言語で開発できるソフト、RubyとかPythonとかその他諸々に対応してる

## インストール

### SL4Aのインストール

昔はここで配布してたけどGoogle Codeはサービスを終了してるので拾えない
こちらの方が使えるapkファイルをビルドされているので対応するアーキテクチャのものをダウンロードしてAndroid端末にインストールしておく
https://github.com/kuri65536/sl4a/releases

### SL4Aで使うPythonのインストール

アプリ内からインストールすることもできるがこの時インストールされるのはPython2.6です
特に理由がなければ先ほどの方がPython2.7のapkファイルをビルドされてるので借りてくるのをオススメ
https://github.com/kuri65536/python-for-android/releases

### Pythonモジュールのインストール

今回の記事で必要なものはないですが地味に引っかかるところなのでメモ代わりに

eggファイルでのインストールしか対応してないのでzipとか投げてもダメです
eggで配布されてなかったらeggファイルを予め作っておく必要があります

```
$ python setup.py bdist_egg
```

dist/hoge.eggと出力されるのでそれを端末にコピってインストールします（Python2.6使用時）
2.7を使用する場合は逆に.eggだとダメなのでこれを.zipに名前を変更して使用します

## テスト

適当にハローワールドとか書いて試す
サンプルスクリプトが入ってるはずなのでその辺で試す

## Androidとシリアル通信をする

https://github.com/kuri65536/sl4a/blob/master/android/USBHostSerialFacade/samplescripts/testusbserial.py
基本的にはこれの通りです
Arduinoの場合は47行目のloads後のリストの最後がArduinoになるのでこのコードを書き換えて使用します

```py
# -*- encoding=utf-8 -*-
import android as _android
import json
import time

android = _android.Android()
l = android.usbserialGetDeviceList().result.items()
print l
 
h = json.loads(l[0][1])
ret = android.usbserialConnect(h[-1])
print ret

uuid = json.loads(ret.result)[-1]
print uuid
time.sleep(5)#すぐ通信を開始すると失敗する
while True:
	time.sleep(1)
	#読み込み
	print android.usbserialRead(uuid)
	#書き込み
	android.usbserialWrite("hoge".uuid)
```

## メモ

この環境でPyro4を使用しようとするとエラーになります
上記と同様の方法でserpentモジュールをインストールすることで解決できます
