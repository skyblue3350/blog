+++
author = "すかい"
title = "iCloudにスクリプトから画像をアップロードする"
date = "2017-05-30"
description = "iCloudにスクリプトから画像をアップロードする"
tags = [
    "Python",
]
+++

## はじめに

AppleTVにはスクリーンセーバーにアルバムの写真を流す機能がある
今回は複数人でこのアルバムに写真を追加してAppleTVのスクリーンセーバーに流したくない？という話になったけどアルバムに追加するにはWeb経由（iCloud）からかApple端末のアルバムの共有機能を使って写真を追加するしかない
そこでスクリプトから写真をアップロードできるようにできたらSlackやらで専用チャンネルを用意しそこに写真をあげてもらうことで簡単に写真の追加が出来るんじゃないかな？と思ったのでトライしたメモ

余談ですがpngの画像のアップロードはまた事情が異なるみたいです
また、肝心のアルバムへの追加はできていません

## iCloud

写真の追加、アルバムの編集等の操作はWebから行うことができる
https://www.icloud.com/

今回はこれを利用してみることにしたがAPIなどは非公開のようなので適当にちょろめのDeveloperToolを使って解析した内容を元にスクリプトを書いてみた

余談ですがこちらのライブラリ使うとiCloudの他の情報が欲しい場合は便利です
写真のダウンロードしか出来なかったので今回は使用しませんでした
https://github.com/picklepete/pyicloud

## ログイン

必要なヘッダを用意してIDとパスワードをなげるとAPIの一覧が入ったjsonが返ってくる

```py
import json
import requests

s = requests.Session()

data = s.post(
    "https://setup.icloud.com/setup/ws/1/login",
    headers={
        "Origin": "https://www.icloud.com"
    },
    data=json.dumps({
        "apple_id": "めーるあどれす",
        "password": "ぱすわーど"
    })
).json()
```

## 写真のアップロード

アップロード先のAPIはjsonの中に入っているのでまずURLを抽出します

```
url = data["webservices"]["uploadimagews"]["url"]
```

必要な各種パラメータを用意します

```
params = {
	"lastModDate": 1397975488598,
	"timezoneOffset": -540,
	"filename": "test.jpg",
	"dsid": data["dsInfo"]["dsid"]
}
```

最後に用意したデータでAPIを叩きます

```
p = s.post(url + "/upload",
    params=params,
    headers={
        "Origin": "https://www.icloud.com"
    },
    data=open("test.jpg", "rb").read()
)

print(p.text)
```

## アルバムに写真を追加する

ここでコケました
良い感じに解決出来た方がいたら教えてください
