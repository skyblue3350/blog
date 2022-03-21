+++
author = "すかい"
title = "Pythonで動画情報を取得する"
date = "2016-09-15"
description = "Pythonで動画情報を取得する"
tags = [
    "Python",
]
+++

## はじめに

Pythonで動画情報の取得をしたかった時に色々モジュール試してみたけど大抵どれも裏でffmpegかffprobe使っててPythonで完結してるものを見つけられなかったのでメモがてら使い方を書いておく

## hachoir

ファイルのメタデータの取得・編集が出来る
対応ファイルフォーマットは以下参考
http://hachoir3.readthedocs.io/metadata.html

### Python2

#### インストール

pipで入れる
幾つかにパッケージが分かれてて既に厄介　今回はhachoir_metadataを使うので

```
$ pip install hachoir_core hachoir_metadata hachoir_parser
```

#### 使い方

```py
# -*- coding: utf-8 -*-

from hachoir_parser import createParser
from hachoir_metadata import extractMetadata

filepath = u"ファイルパス"
parser = createParser(filepath)

meta = extractMetadata(parser)
print meta.exportPlaintext()
print meta.get("duration")
```

再生時間はtimedeltaで返ってくる

素直に添字で取れれば良いのにgetメソッド使わないといけなかったりで微妙に使いづらい
`meta.dict["_Metadata__data"].keys()`
とか使えば一応添字の一覧が見れる　なぜkeysメソッドとかがないのか
Githubで探すとexportPlaintextの結果を「:」でsplitしたりしててみんな困ったんだろうなって
ドキュメント見ても良い感じのやり方を見つけられなかったのでぼちぼち元のコードを見つつ・・・

### Python3

#### インストール

pipでは入らないのでソースコードを直接落として使う
レポジトリは
https://bitbucket.org/haypo/hachoir3/overview
にあるのでcloneするかファイル→ダウンロードで最新のコードを落としてくる
必要なのはhachoirディレクトリだけ今回は試すだけなのでそれだけディレクトリにコピーした

#### 使い方

```
from hachoir.metadata import extractMetadata
from hachoir.parser import createParser

filepath = u"ファイルパス"
parser = createParser(filepath)
meta = extractMetadata(parser)
duration = meta.get("duration")
print(duration)
```

構造が変わってたから少し困った
ドキュメント見ても3のドキュメントを名乗ってるわりに中のコードは2のサンプルだったりで辛い

## ffprobe

一応試したのでメモ

### Python2

2では試してないけど3とほぼ同じはず

### Python3

#### インストール

公式から落としてきてパスがあるとこに置くだけ（Win

#### 使い方

subprocessを使って出力結果をパースする
本当は[FORMAT][/FORMAT]で囲まれてるとこだけ正規表現とかで抜いた方が良い

```py
import subprocess
cmd = "ffprobe"
filepath = "ファイルパス"
p = subprocess.Popen(
    "%s %s -hide_banner -show_entries format" % (cmd, filepath),
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)
out, err = p.communicate()

d = {}
for i in out.split(b"\n")[1:-1]:
    if i:
        tmp = [x.decode("utf-8") for x in i.strip().split(b"=")]
        d[tmp[0]] = "".join(tmp[1:])
```

戻り値がbytes型なので扱いがちょっと面倒
辞書内包表記で書こうとしたけど綺麗に書けなかったのでやめた

```py
d = datetime.timedelta(
  seconds=int(d["duration"])),
)
```

とかでdeltatimeにできる

## まとめ

結局ffprobe使うことにしました
前者は対応フォーマットさえ多ければ良かった・・・mp4使えないのが痛い

## 余談
使ったのは頼まれて作った字幕生成スクリプトです
動画の再生時間と作成時間を見て監視カメラ的な日時の字幕を自動生成する感じ
