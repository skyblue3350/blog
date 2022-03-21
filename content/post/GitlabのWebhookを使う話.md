+++
author = "すかい"
title = "GitlabのWebhookを使う話"
date = "2016-10-28"
description = "GitlabのWebhookを使う話"
tags = [
    "Gitlab",
]
+++

## はじめに

GitlabでWebhook使ってSlackにでも投稿しようか、という試み
Gitlabに限らずGithubにもあるのでそちらの場合は適宜ドキュメントを読む
ドキュメント読もうにもバージョンが分からん…って時は
`https://ドメイン名/help`
とか見れば分かる

※そもそもSlackへの通知は公式でサポートされてるのでわざわざ作る必要ないです

追記
Gitlabをオンプレで運用しててローカルネットへのWebhookが動かない！って場合は管理コンソールで内側ネットワークへのアクセスを許可しないと動きません．

## バージョン

- 8.5.1

## Webhookとは
コミットとか各種イベントが発生した時に向こうからURLを叩いてくれる
のでURLの用意とそこで動くWebアプリだけ用意しておけば良い感じに受け取って処理出来る

## Webhookの登録

どのバージョンのGitlabも
`https://ドメイン名/ユーザー名/レポジトリ名/hooks`
に登録画面がある
適宜必要なイベントにチェックする
今回はコミットログだけ拾えれば良いのでpushだけセレクト　URLは適当に
チャンネル名を指定してもらうついでにただ適当にURL叩かれて動いても困るので適当なクエリをひっつけて一応Gitlabからってことを確認しておく
`http://ドメイン名/パス/?channel=チャンネル名&secretkey=hogehoge`
みたいな感じにした

## アプリの作成

### コード

とりあえずFlaskでサクッと作る
Jsonのフォーマットは
https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/web_hooks/web_hooks.md
こことか読みましょう

```py
# -*- coding:utf-8 -*-

import json
import requests
from flask import Flask, request, abort
app = Flask(__name__)

secretkey = "hogehoge"

@app.route("/", methods=["POST"])
def index():
    if secretkey != request.args.get("secretkey", ""):
        abort(400)

    data = json.loads(request.data)

    temp = u"""レポジトリ「{repo}」に{name}がコミットしました
{msg}
{url}"""
    message = temp.format(
        repo=data["repository"]["name"],
        name=data["commits"][-1]["author"]["name"],
        msg=data["commits"][-1]["message"],
        url=data["commits"][-1]["url"],
    )

    target = request.args.get("channel", None)
    if not(target is None) or not(target=="general"):
        # Slackに投稿する処理
        return "ok"
    else:
        abort(400)

if __name__ == "__main__":
        app.run(debug=True, host="0.0.0.0", port=8080)
```

みたいに作る
Slackに投稿する部分は端折りました

## ハマったところ

- メソッドはPOST
GETだと思って10分くらいあれー？ってなりました
- コミットは古い方から取れる
data["commits"][0]とかすると古いのが取れます
新→旧じゃなくて旧→新なので-1して後ろを取りましょう
