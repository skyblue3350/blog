+++
author = "すかい"
title = "Youtube LiveStreaming APIで配信のコメントを取得する"
date = "2017-11-27"
description = "Youtube LiveStreaming APIで配信のコメントを取得する"
tags = [
    "Python",
    "Youtube",
]
+++

## はじめに

読み上げをYoutubeの配信中にもやりたいなーと思ってコメントの取得方法調べたら結構面倒なのでメモ

大まかな流れとしては
配信先のURLからライブのIDを取得 -> ライブのIDからチャットのIDを取得 -> チャットIDを使ってチャットの取得
といった感じ
ライブのIDからチャットのIDを取得するAPIにOauth認証が必要だったりで結構面倒

## 環境

- Python3

## APIキーを用意する
この辺は他のAPIと同じなので他のサイトの方を見たほうがわかりやすいです．

いつも通り[Google Cloud Platform](https://console.cloud.google.com/)へ行き新しくプロジェクトを作成し，APIの中から「YouTube Data API v3」を有効にしておく．
その上で認証情報 -> Oauth同意画面と進み必要な項目を埋めておく．（最低限サービス名だけ埋まっていれば良い）
認証情報の画面まで戻り認証情報を作成 -> OauthクライアントIDを選択する．
アプリケーションの種類は「その他」にして進める．名前は適当に決める．
作成するとIDとシークレットが表示されるが後でダウンロードするjsonに入ってるのでOKをクリックして閉じる．
OAuth 2.0 クライアント IDの部分に今回作成したキーがあるはずなのでこれを右端のアイコンからダウンロードしておく．
今回はclient.jsonとしてダウンロードしておく．

## ライブラリを用意する

httplib2とoauth2clientを使うのでこれらを入れておく．

```
$ pip install httplib2 oauth2client
```

## 認証する

一度目はお馴染みのOauth認証画面に飛ばされるが一度認証すると不要になるのでその辺込みでこんな感じ
client.jsonがさっき落としてきた認証情報でcredentials.jsonが認証済みの情報．

```py
import httplib2
from oauth2client import tools
from oauth2client import client
from oauth2client.file import Storage

credentials_path = "credentials.json"
if os.path.exists(credentials_path):
    # 認証済み
    store = Storage(credentials_path)
    credentials = store.get()
else:
    # 認証処理
    f = "client.json"
    scope = "https://www.googleapis.com/auth/youtube.readonly"
    flow = client.flow_from_clientsecrets(f, scope)
    flow.user_agent = "なんか適当に入れる"
    credentials = tools.run_flow(flow, Storage(credentials_path))
```

本当はもう少しシンプルに書ける．

```py
credentials_path = "credentials.json"
store = Storage(credentials_path)
credentials = store.get()

if credentials is None or credentials.invalid:
    f = "client.json"
    scope = "https://www.googleapis.com/auth/youtube.readonly"
    flow = client.flow_from_clientsecrets(f, scope)
    flow.user_agent = "なんか適当に入れる"
    credentials = tools.run_flow(flow, Storage(credentials_path))
```

## LiveChatIdを取得する

LiveChatIDは[LiveBroadcasts: list](https://developers.google.com/youtube/v3/live/docs/liveBroadcasts/list)を使うことで取得できる．
Web上でも試せるので一度試しておくと良い．
配信のURLが
`https://www.youtube.com/watch?v=xxxxxxxxx`
の場合実際のコードはこんな感じ．

```py
http = credentials.authorize(httplib2.Http())
url = "https://www.googleapis.com/youtube/v3/liveBroadcasts?part=snippet&id="
url += "xxxxxxxxx"
res, data = http.request(url)
data = json.loads(data.decode())

chat_id = data["items"][0]["snippet"]["liveChatId"]
print(chat_id)
```

## コメントの取得
コメントの取得は[LiveChatMessages: list](https://developers.google.com/youtube/v3/live/docs/liveChatMessages/list#try-it)から出来る．
どうもStreamingAPIとかは提供されてないっぽいので毎回リクエストを投げるしかなさそう？
pageTokenクエリを使うと前回との差分を取得出来るのでこれを使ってコメントの重複は避けられる．

```
pageToken = None
url = "https://www.googleapis.com/youtube/v3/liveChat/messages?part=snippet,authorDetails"
url += "&liveChatId=" + self.chat_id

while True:
    if pageToken:
        url += "&pageToken=" + pageToken

    res, data = self.http.request(url)
    data = json.loads(data.decode())

    for datum in data["items"]:
        print(datum["authorDetails"]["displayName"])
        print(datum["snippet"]["textMessageDetails"]["messageText"])
        print(datum["authorDetails"]["profileImageUrl"])

    pageToken = data["nextPageToken"]
    # time.sleep(3)
```

連続で回し続けるのはどうなんだろうと思ったので適当にwaitをかけると良いと思う．

## おまけ

GUI化してみた

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ぼちぼち動くようになった <a href="https://t.co/04q07mNcok">pic.twitter.com/04q07mNcok</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/934837880576454656?ref_src=twsrc%5Etfw">November 26, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 追記（2021/01/11）
こちらの件でマシュマロが来ていたのですがすぐに返し損ねてしまったので何かありましたらTwitterのDM開けておりますのでそちらでコメントいただければ対応できます。

## 参考記事

後ろの2つはコードサンプルなので読むと雰囲気がわかる．

- [LiveBroadcasts: list](https://developers.google.com/youtube/v3/live/docs/liveBroadcasts/list)
- [LiveChatMessages: list](https://developers.google.com/youtube/v3/live/docs/liveChatMessages/list#try-it)
- [github.com/youtube/api-samples](https://github.com/youtube/api-samples)
- [Python Code Samples](https://developers.google.com/youtube/v3/live/code_samples/python)
