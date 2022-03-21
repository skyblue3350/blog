+++
author = "すかい"
title = "SWFバイナリのパース"
date = "2016-11-27"
description = "SWFバイナリのパース"
tags = [
    "Python",
]
+++

## はじめに

今までバイナリに触れる機会がなかったのでとりあえずPythonでバイナリを読めるようになろうかなって思った
とりあえず仕様が公開されてるSWFの画像・音声（動画）をバイナリから引っこ抜くところまでを目標にする

ちなみにバイナリについてバイナリエディタで開いてほーん…ってなるくらいしか知らない

## 仕様書

Adobeがファイルフォーマットの仕様書を公開しているので基本はこれを読みながらやる
http://wwwimages.adobe.com/content/dam/Adobe/en/devnet/swf/pdf/swf-file-format-spec.pdf
読み方としては15ページのInteger Typesは先に目を通してから27ページのSWFのフォーマットについて読むと良い
それ以外は読み飛ばして必要に応じて読めば良い

## レポジトリ

とりあえずの作業スペース
https://github.com/skyblue3350/swf

## 作業

### 最低限のバイナリの読み込み

とりあえずバイナリエディタみたいに16進数で表示してみる
structモジュールとかあるけど使わないで書いてみる

```py
fp = open("test.swf", "rb")
while True:
    b = fp.read(1)
    if b == "": break
    print hex(ord(b))
fp.close()
```

1バイトずつ読み出してordで10進数にしてそれをhexで16進数に変換する

### ヘッダの読み込み

ヘッダは以下で構成される

- Signature
圧縮されているかどうか
- Version
このSWFファイルのバージョン情報
- FileLength
このSWFファイルの大きさ
- FrameSize
フレームの縦横の大きさ
- FrameRate
FPS
- FrameCount
フレームの数

1個ずつ読んでいく

#### Signature

Sinatureは3バイトで表される
1バイト目が圧縮 2バイト目（W）と3バイト目（S）は固定（仕様書27ページ参照）
1バイト目がFの時 未圧縮 Cの時 zlib圧縮 Zの時 LZMA圧縮
UI8が8bit→8*3で24ビット→8ビット1バイトで3バイトなのでreadで3バイト読み出す

```py
fp = open("test.swf", "rb")

signature = fp.read(3)
print "Signature", signature
fp.close()
```

```
FWS
```

未圧縮であることが分かる
もしCWSなら9バイト目以降をzlibモジュールをインポートして

```py
zlib.decompress(fp.read())
```

する

#### Version

この辺から差分だけ

```py
print ord(fp.read(1))
```

#### FileLength

これはリトルエンディアンで表される
1バイト：A
2バイト：B
だったら
BAにしてこれを2つつなげて10進数に直す

```py
little = [ hex(ord(fp.read(1)))[2:].zfill(2) for x in range(4)][::-1]
filelen = int("".join(little), 16)
print filelen
```

今後もリトルエンディアンは出てくるので関数化しておくと良い
10進数→16進数→10進数ってやり方しか思いつかなかったけどもっと簡単にできそう

ここまで読み込んだらCWSなどの圧縮時はこれ以降のバイナリが圧縮されているので展開する

```py
if signature == "CWS":
    b = fp.read()
    fp.close()
    fp = StringIO(zlib.decompress(b))
```

とかしておくと未圧縮と圧縮の差を考えなくて良くなるので多少楽
こんな感じでヘッダの残りも読む

### Tagの読み込み

SWFファイルは[Headter][Tag][Tag]...[Tag]（28ページ）
とデータが続くようになっている
Tagには種類が設定されていてENDというTagが出て来るまでひたすら読み込んでいけば良い

#### typeとlength

2バイト読み込んで最初の10ビットを10進に直したものがTagの種類
残りの6ビットがそのTagのサイズ

```py
info = "".join([ bin(ord(f.read(1)))[2:].zfill(8) for x in range(2) ][::-1])
tagtype = int(info[:10], 2)
taglength = int(info[-6:], 2)
```

またtaglength変数の中身が6ビットで表せる最大の数（2進：111111、10進：63）の時は更に4バイトリトルエンディアンで読み込んだものが実際のサイズになります
これを反映すると以下のようになります

```py
info = "".join([ bin(ord(fp.read(1)))[2:].zfill(8) for x in range(2) ][::-1])
tagtype = int(info[:10], 2)
taglength = int(info[-6:], 2)

# サイズが63なら追加4ビットを使用する
if taglength == 63:
    taglength = int("".join([ hex(ord(fp.read(1)))[2:].zfill(2) for x in range(4)][::-1]), 16)

tagbody = fp.read(taglength)
```

#### Tag全体の読み込み

tagtypeがSWFにおけるタグの種類です（仕様書235ページに一覧あり）
ファイルの末尾はEND（id:0）となるはずなのでtagtypeが0になるまでループします

```
while True:
    fileinfo = "".join([ bin(ord(fp.read(1)))[2:].zfill(8) for x in range(2) ][::-1])
    tagtype = int(fileinfo[:10], 2)
    taglength = int(fileinfo[-6:], 2)

    # TagType：0がデータの末尾
    if tagtype == 0:
        break

    if taglength == 63:
        taglength = int("".join([ hex(ord(fp.read(1)))[2:].zfill(2) for x in range(4)][::-1]), 16)

    tagbody = fp.read(taglength)
```

これでひとまずファイルの全体の読み込みが出来ました
あとは必要なデータに応じてtagbodyを良い感じに加工するとデータが抽出出来ます

実装済みの部分はレポジトリの方に反映してあるので興味があればどうぞ
この記事の解説はいい加減なので以下の参考サイトを参考に自分で書いた方が分かりやすいと思います

## 参考記事

以下2サイトは大変お世話になりました

- [SWFバイナリ解析](https://doruby.jp/users/hal_on_rails/entries/SWF_)
- [SWFバイナリ編集のススメ](http://labs.gree.jp/blog/2010/08/631/)
