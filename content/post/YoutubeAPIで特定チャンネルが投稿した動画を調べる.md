+++
author = "すかい"
title = "YoutubeAPIで特定チャンネルが投稿した動画を調べる"
date = "2018-02-24"
description = "YoutubeAPIで特定チャンネルが投稿した動画を調べる"
tags = [
    "Youtube",
]
+++

## はじめに

YoutubeAPIで特定チャンネルの投稿一覧が取得したいけどどうすれば良い？って質問が来たので試した．
ついでに特定期間内に投稿された動画の取得方法も試してみた．
ソースコードはgistに置いておきます．
動かし方は前回の記事とか見てください．

https://gist.github.com/skyblue3350/005b65c1d2f156d872188842753c677c

## 叩くAPI

基本的にSearch: listを使えば出来ます．
https://developers.google.com/youtube/v3/docs/search/list?hl=ja

今回は適当なvtuberのチャンネルで試します．

## チャンネルの投稿動画の取得

channelIdパラメータを使って目的のチャンネルの動画だけ検索して
orderパラメータで日付順にすることで投稿日順にソートします．
maxResultsパラメータで50件まで一度に取れますがそれ以上ある場合は前回と同様にpageTokenを回して2ページ目を取得します．

## 特定期間の投稿の取得

期間の指定はpublishedAfterとpublishedBeforeパラメータから行います．
publishedAfter～publishedBeforeです．
リクエストはRFC 3339形式で送ります．
Pythonで綺麗に文字列化する方法がわからなかったのでdt.isoformat()して末尾にZを足して誤魔化しました．
あとはチャンネル投稿動画の取得と同様のパラメータを使えば良い感じに取得出来ます．
