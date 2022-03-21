+++
author = "すかい"
title = "GitlabのWebAPIを使う話"
date = "2016-10-29"
description = "GitlabのWebAPIを使う話"
tags = [
    "Gitlab",
]
+++

## WebAPI

Webhook触ったついでに試す
バージョンの差異でないことがあったりするのであれ？って思ったらドキュメントを読む
https://github.com/gitlabhq/gitlabhq/tree/master/doc/api
左上からブランチを適宜自分のバージョンに変更して読むと捗ります
適当に検索して見つけたAPI叩こうとしたら使用しようと思ったGitlabが古すぎて使えなかったりしました

## 試す

Google Chromeの拡張機能のPostmanとかが気軽に試せてオススメ
抑えるべき点はヘッダーにPRIVATE-TOKENが必要だということ
これは自分のユーザーページに飛べばあります　コピペして使います

## ハマりどころ

GET /projects/:id/repository/commitsでレポジトリ名で指定したい時
:idのところにNAMESPACE/PROJECT_NAMEで書けとドキュメントにあります
この間のスペースはエンコードした「`%2F`」で書く必要があります
つまりユーザー名がhogeでレポジトリ名がfugaの場合
/projects/hoge%2Ffuga/repository/commits
となります
