+++
author = "すかい"
title = "DiscordでBotをつくる"
date = "2016-12-04"
description = "DiscordでBotをつくる"
tags = [
    "Discord",
    "Python",
]
+++

## はじめに

DiscordのAPIを利用してオウム返しをするBotを作成してみます.

## 環境

- Windows7 64bit
- Python3.5 64bit

## 環境構築

discord用のライブラリがあるのでそれを利用します.

```
$ python3 -m pip install discord
```

## コード

こんな感じのコードでメッセージを受信してオウム返し出来ます.

asyncioの関係で3.4と3.5では文法が変わります.
こんな感じ
@asyncio.coroutine → async
yield from → await

まぁドキュメントに書いてあるんですけどね

```py
import discord

client = discord.Client()

@client.event
async def on_ready():
    print("-"*20)
    print("ユーザー名：", client.user.name)
    print("ユーザーID：", client.user.id)
    print("-"*20)

@client.event
async def on_message(message):
	if not message.author.id == client.user.id:
		await client.send_message(message.channel, message.content)
		print("投稿しました")

	print("投稿者：", message.author)
	print("メッセージ：", message.content)

client.run(os.environ["DISCORD_API_TOKEN"])
```

結構簡単に出来るのが良いですね

追記　2018-07-12
検索で来る人が結構いるみたいなのでTOKENの取り方とBotをサーバーに参加させる方法

DISCORD_API_TOKENは以下のURLから発行できます
https://discordapp.com/developers/applications/me

ここから新しいアプリ -> アプリを作成からアプリを作ります
アプリのアイコンの下辺りにBotの項目があるのでBotユーザーを作成から作成します
確認のダイアログが出てくるので実行をクリックして続行します
トークンの横にあるクリックして表示から取得できます

作ったBotはOAUTH2 URL GENERATORの項目からbotだけ有効にしてURLをコピーしてアクセスすると自分が所属するサーバーにBotを参加させることができます
