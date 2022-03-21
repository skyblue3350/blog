+++
author = "すかい"
title = "VIRB Editでプレビューできない"
date = "2021-12-12"
description = "VIRB Editでプレビューできない"
tags = [
    "VIRBEdit",
]
+++

GoPro で撮影した動画に Garmin サイコンで取ったデータをオーバーレイで載せてみたい！と思って VIRB Edit を触ったところ、VIRB Editで動画や画像がプレビューできない事象に遭遇したのでメモ

## 環境

- 発生バージョン
5.4.3
- GPU Driver
30.0.14.9709

## 事象

動画や画像を読み込ませてプレビューしようとすると以下のようにエラー表示が出てしまい何も映らず…

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">何もわからん…になった <a href="https://t.co/4hFOOTNblU">pic.twitter.com/4hFOOTNblU</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1469945592269123586?ref_src=twsrc%5Etfw">December 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

エラー文の案内ではグラボのドライバ更新が案内されているがグラフィックカードのドライバは今日時点で最新のものを適用済み。

https://forums.garmin.com/ などを彷徨ってみましたが類似事象の報告はなし。

解消のため色々触っていたところ UI 表示が日本語以外だと問題なく動作することに気付き、インストール先（例 C:\Program Files\Garmin\VIRB Edit ）にある ja ディレクトリの名前を適当に変更し読み込めないようにすることで標準設定の英語で起動するようにできました。

追記1
検証で利用していた動画は再生できたのですが、GoPro で撮影した動画を読み込ませると同じエラーが発生してしまったため使用を諦めました…

追記2
上記に加えてVFRだとうまく再生できなさそうだったのでCFRに変換したところ再生はできました。
ただ、VFR -> CFR にした際に GoPro の GPS データの Stream が失われてしまうので位置合わせを手動でやる必要がでてきます…
