+++
author = "すかい"
title = "FFmpegでエンコードした動画をTwitterにあげる"
date = "2016-11-20"
description = "FFmpegでエンコードした動画をTwitterにあげる"
tags = [
    "FFmpeg",
]
+++

## はじめに

ShadowPlayで撮影したゲームの一シーンとかTwitterにあげたいなって時の話
GUIツール入れるのめんどくさいのでFFmpegで任意部分をカットしてエンコードする

## 環境

- Windows 7 64bit
- ffmpeg version N-81609-g7b3bc36

## 動画の切り出し

10分30秒から2分切り出す場合

- 入力ファイル
input.mp4
- 出力ファイル名
output.mp4

```
$ ffmpeg -ss 00:10:30 -i input.mp4 -t 120 output.mp4
```

こんな感じで切り出せる
iオプションよりssオプションを前に持ってくることでシーク時間が早くなる
逆にするとファイルの最初から読み出すため動画が長ければ長い程シークに時間を取られる

YouTube等の動画サイトにあげる時はこれで良いがTwitterにあげる際にはもう1つオプションを追加する必要がある

```
$ ffmpeg -ss 00:10:30 -i input.mp4 -t 120 -pix_fmt yuv420p  output.mp4
```

これでTwitterの公式サイトの方でD&Dすると動画をアップロードすることが出来る
少し前からFFmpegがCUDA対応してるからそれ込みで使えば多少長くても結構早めに切り出せるかな
こんな感じ

```
$ ffmpeg -ss 00:10:30 -i input.mp4 -t 120 -vcodec nvenc -pix_fmt yuv420p  output.mp4
```

もし動画連結するなら

```
$ ffmpeg -i input_1.movie -i input_2.movie -filter_complex "concat=n=2:v=1:a=1" output.mp4
```

## 参考記事

- [FFmpegで素早く正確に動画をカットする自分的ベストプラクティス](http://qiita.com/kitar/items/d293e3962ade087fd850)
- [Twitterに動画を投稿する際に行ったffmpegの設定](http://bibouroku.viratube.com/2016/02/24/twitter%E3%81%AB%E5%8B%95%E7%94%BB%E3%82%92%E6%8A%95%E7%A8%BF%E3%81%99%E3%82%8B%E9%9A%9B%E3%81%AB%E8%A1%8C%E3%81%A3%E3%81%9Fffmpeg%E3%81%AE%E8%A8%AD%E5%AE%9A/)
