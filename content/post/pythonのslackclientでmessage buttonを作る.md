+++
author = "すかい"
title = "pythonのslackclientでmessage buttonを作る"
date = "2016-11-11"
description = "pythonのslackclientでmessage buttonを作る"
tags = [
    "Python",
]
+++

## はじめに

チームで共有出来るTodoリスト的なものを目指してMessage Buttonの出し方を調べてみた
結構Jsonで直にAPIを叩く記事は見つかるけどslackclientを使ったものは見かけないのでメモ

## slackclient

- [slackapi/python-slackclient](https://github.com/slackapi/python-slackclient)

PythonからslackのAPIが叩けるライブラリ
予めアクセストークンを用意しておく必要がある

## 投稿

- [Basic Usage Sending a message](http://slackapi.github.io/python-slackclient/basic_usage.html#sending-a-message)

```py
from slackclient import SlackClient

slack_token = os.environ["SLACK_API_TOKEN"]
sc = SlackClient(slack_token)

sc.api_call(
  "chat.postMessage",
  channel="#python",
  text="Hello from Python! :tada:"
)
```

シンプル

## ボタン付き投稿
ボタンの作り方自体は

- [Making messages more interactive with buttons](https://api.slack.com/docs/message-buttons)

に書いてある　これを参考に作る

```py
import json
from slackclient import SlackClient

slack_token = os.environ["SLACK_API_TOKEN"]
sc = SlackClient(slack_token)

attachments = [{
    "fallback": "",
    "text": u"ボタンの説明",
    "callback_id": "",
    "color": "#008000",
    "attachment_type": "default",
    "actions": [
        {
            "name": "done",
            "text": "ボタン1",
            "type": "button"
        }
    ]
}]

sd.api_call(
    "chat.postMessage",
    channel="チャンネル名",
    text="ボタン付き投稿です",
    attachments=json.dumps(attachments)
)
```

とりあえずこれでボタン付き投稿できる

詰まったポイントはjson.dumpsしておかないと行けない点
あとそもそもapi_callの引数がどういう扱いなのか知らなかった
結局可変長引数でpostデータに入れてくれるみたいなのでこんな感じになった

## ボタンの動作

ボタン付き投稿はHTTPS認証が出来るWebサーバーがないとボタンの動作が作れないらしいのでその辺調査中
アプリを作成してOauth2認証で得られるTokenだとRTMが利用出来ないんだけど…？
とりあえず今回はチームにBotを追加してそのTokenを利用してRTMでメッセージが来たら返事をするようにしてみた
そのためボタンを押しても無反応
