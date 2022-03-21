+++
author = "すかい"
title = "Azure Computer Vision APIをcurlで試す"
date = "2017-11-27"
description = "Azure Computer Vision APIをcurlで試す"
tags = [
    "Azure",
]
+++

## はじめに

Azure Computer Vision APIをcurlで試してみます．
普通に適当な言語でコードを書いた方が楽なんですがちょっと諸事情あってシェルスクリプト書かないといけなくなったのでメモ．

## 事前準備

予めコンソールの方でAPIを有効化しておきAPIキーを取得しておきます．

## curlで取得してみる

1行で書くとわかりにくいのでシェルスクリプトにしておきます．

スクリプト内のLOCATION変数はAPI有効化の時に選択したロケーションと一致している必要があります．

```
IMAGE="./sample.jpg"
LOCATION="southeastasia"
VISUALFEATURES="Categories"
DETAILS=""
LANGUAGE="en"
APIKEY="取得したキー"

curl -s -X POST \
  "https://${LOCATION}.api.cognitive.microsoft.com/vision/v1.0/analyze?visualFeatures=${VISUALFEATURES}&details=${DETAILS}&language=${LANGUAGE}" \
  -H "Content-Type: application/octet-stream" \
  -H "Ocp-Apim-Subscription-Key: ${APIKEY}" \
  --data-binary "@${IMAGE}"
```

レスポンスはjsonなのでjqコマンドとかでパースすると良い感じです．
カテゴリ名とスコアが欲しかったら

```
$ curl ～省略～ | jq -r '.categories[] | "\(.name), \(.score)"'
abstract_nonphoto, 0.23828125
abstract_texture, 0.53515625
```

とかでとれます．

## 参考記事

- [Computer Vision cURL Quick Starts](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/quickstarts/curl)
