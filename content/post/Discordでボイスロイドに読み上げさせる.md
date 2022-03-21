+++
author = "すかい"
title = "Discordでボイスロイドに読み上げさせる"
date = "2017-07-22"
description = "Discordでボイスロイドに読み上げさせる"
tags = [
    "Discord",
    "Python",
]
+++

## はじめに

Discordでボイスロイドに読み上げさせてみたくなった．
のでとりあえずやってみる

## 環境

- Windows7 64bit
- Python3.5 64bit
- discord.py 0.15.1
  ※3.4を使う場合は記法が異なる部分があるので[前回の記事](../discord%E3%81%A7bot%E3%82%92%E3%81%A4%E3%81%8F%E3%82%8B)を見て下さい

## コード

### voiceroid

voiceroidを喋らせる部分は前回のコードを利用します．

<script src="https://gist.github.com/skyblue3350/1f3b461adf3efb6e0c32.js"></script>

### discord

Oauth2認証挟んでtokenを発行した方が良い気もしたけどとりあえずIDとPWで認証してAPIが使えるのでひとまずこれでいく

```py
import discord

from voiceroid import VoiceRoid

vr = VoiceRoid("VOICEROID＋ 結月ゆかり EX")
client = discord.Client()

@client.event
async def on_ready():
    print("-"*20)
    print("ユーザー名：", client.user.name)
    print("ユーザーID：", client.user.id)
    print("-"*20)

@client.event
async def on_message(message):
    print("投稿者：", message.author)
    print("メッセージ：", message.content)
    print("サーバー：", message.server)
    print("チャンネル：", message.channel)

    vr.say(message.content)

client.run("メールアドレス", "パスワード")
```

ところでvoiceroid2でUIが統合されてるっぽいから多分これ使えないですね
買いたいけどぼちぼちするので難しい
