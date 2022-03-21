+++
author = "すかい"
title = "Dockerでマインクラフトのサーバーを建てる"
date = "2016-07-09"
description = "Dockerでマインクラフトのサーバーを建てる"
tags = [
    "Docker",
    "Minecraft",
]
+++

いろいろ投げやり

<script src="https://gist.github.com/skyblue3350/9bebfa4be9b7bfe3cef0.js"></script>

こんな感じでDockerfile書いて

docker build -t hoge/minecraft .
で終わり
適当にJavaに投げるオプションを適宜変更してビルドする

実行は

```
docker run -d -p 25565:25565 -v /hoge/fuga:/home/minecraft hoge/minecraft
```

するだけ
ボリュームで置いたファイルを呼んでるだけの手抜きイメージを走らせてる　最高に手抜き
参考サイトみたいにした方がいいと思うけどとりあえずJava動くイメージが作ってみたかっただけなんだ…

とりあえずイメージの作り方の勉強にお試しでという感じ

## 参考

- [DockerでMinecraft forgeのマルチサーバを立てる](http://qiita.com/deflis/items/9f2127e5886647306278)
