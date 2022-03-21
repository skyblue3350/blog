+++
author = "すかい"
title = "Pythonから結月ゆかりを喋らせる"
date = "2016-07-08"
description = "Pythonから結月ゆかりを喋らせる"
tags = [
    "Python",
    "結月ゆかり",
]
+++

この前結月ゆかりさんをお迎えしてしまいました　ほぼノリです

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">昨日の夜ポチってしまった <a href="https://t.co/AKEhQQ9VVE">pic.twitter.com/AKEhQQ9VVE</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/707924150380154881?ref_src=twsrc%5Etfw">March 10, 2016</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

というわけでこれをPythonから操作して喋らせます

他の言語だと誰かしら書いてるようですがPythonだとどうも見つからないので自分で書きました
コード自体はこんな感じ

<script src="https://gist.github.com/skyblue3350/1f3b461adf3efb6e0c32.js"></script>

あとは

```
voiceroid.say(u"文字列")
```

とかで喋ってくれる

コード見ればわかるけどgetVolueとかメソッド実装したけど動かない
音程とかのキーボックスにキー入力までは送れるけどsubmitイベント？とかが発生しないらしく値が反映されない
そのうち解決して実装しよう

んでここまで出来たので試しにSlackにストリーミング接続して来たチャット通知を片っ端から喋らせるスクリプト書いてみたけど結構面白かった
今のところは通知用の用途での使用を考えてるのでTwitterのリプライとかSlackの通知とか読み上げてくれるBot的なものを作成しようかなぁ
