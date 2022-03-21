+++
author = "すかい"
title = "LINE BOTメモ書き"
date = "2016-07-31"
description = "LINE BOTメモ書き"
tags = [
    "LINEBOT",
]
+++

## はじめに

最初の公開時に登録して放置してあったLINEBOTを触り始めました
公開時の情報しか持ってなかったので現時点での情報整理

## 必要なもの

- HTTPS通信出来る鯖
herokuとかGASとかがお手軽？
あと自分の動かしたいコードが動かせる鯖
前は固定IPな必要があったりと色々不便だったけど改善してたらしい

## メモ

- 固定IPはオプションになった
  必要なら登録出来るが任意になった
  > These settings will take effect immediately.
  > Registering an IP address is optional.
  > You may choose to register IP addresses for security purposes.

- HTTPSの認証局
API公開時はletsencryptだとダメだったけど現在は対応済みだったらしい
ちょうど2週間くらい前に相談受けたのでその時に知ってれば良かった

## ドキュメント

良く探すやつだけまとめ

- [メッセージの種類](https://developers.line.me/bot-api/api-reference#sending_message_text)

とりあえず自鯖でテスト中 Apache+Flaskで動かしてるけどFalconとか使えば良かった
