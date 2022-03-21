+++
author = "すかい"
title = "PythonでRAWデータをjpgに変換する"
date = "2018-03-17"
description = "PythonでRAWデータをjpgに変換する"
tags = [
    "Python",
]
+++

## はじめに

記事にしたか忘れましたが半年程前にSonyのα6000を購入しました．
撮影してるとRAWで書き出したファイルが積み重なっていくわけですがどうせなら一括処理とかできないかなと思ってrawpyを使って画像変換を試してみました．

## 環境

- Windows10
- Python3

## インストール

必要なライブラリを入れます．

```
$ pip install opencv-python numpy rawpy imageio lensfunpy
```

## 処理

### 画像の読み込み->書き出し

```py
filename = "DSC00209.ARW"
raw = rawpy.imread(filename)
# raw -> numpy array
rgb = raw.postprocess(
    bright=1.3,
    use_camera_wb=True,
)
imageio.imsave("output.jpg", rgb)
```

### レンズの歪みの補正

このままだと放射状歪み？をしているので補正します．
補正は使用しているレンズごとに変わってくるので決め打ちかEXIFから取得した値で適宜変える必要があります．
lensfunpyというライブラリが良い感じにしてくれるのでこれを使います．
自分のカメラだと以下のようになります．

```py
# レンズ補正用設定読み込み
db = lensfunpy.Database()
cam = db.find_cameras("SONY", "ILCE-6000")[0]
lens = db.find_lenses(cam, "SONY", "E 16-50mm f/3.5-5.6 OSS")[0]
```

探し方ですが

```py
cam = db.find_cameras("SONY", None)
for c in cam:
    print(c)
```

とかすると良い感じに見えるので参考にしつつ探します．
レンズも同様の方法で探します．
最後にこのレンズのデータを画像に適用して終了です．
先ほどnumpyの配列にするところまでは済んでるのでこんな感じ．

```py
height, width = rgb.shape[0], rgb.shape[1]
mod = lensfunpy.Modifier(lens, cam.crop_factor, width, height)
# 焦点距離　F値　フォーカス距離（メートル）
mod.initialize(16, 3.5, 10)
undist_coords = mod.apply_geometry_distortion()
im_undistorted = cv2.remap(rgb, undist_coords, None, cv2.INTER_LANCZOS4)

imageio.imsave("output.jpg", im_undistorted)
```

lensfunpy.Modifier.initializeについては下記のドキュメント見た方が早いです．
https://letmaik.github.io/lensfunpy/api/lensfunpy.Modifier.html#lensfunpy.Modifier.initialize

というわけで一応変換してjpgとして書き出すところまでできました．
RAWとJPGの同時書き出しにしてるので比較して見ましたがなんか色合いが同じにならないので気になってます．

同時書き出しにしてるならそもそもこのスクリプト要る？って話ではありますがお勉強なので…

## おまけ

実際に出力してみたサンプル

![](/images/2018-03-17-001.gif)
