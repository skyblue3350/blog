+++
author = "すかい"
title = "Ubuntu16.04のPython3でOpenCV3を使う"
date = "2016-07-10"
description = "Ubuntu16.04のPython3でOpenCV3を使う"
tags = [
    "OpenCV",
    "Python",
    "Ubuntu",
]
+++

## OpenCVについて

画像処理するライブラリ
2系入れたいならディストリビューションごとにパッケージあるからそっち使いましょう
3系はパッケージが現時点（2016/06/24）でないから自分でビルドする
3.1.0しか検証してないけど多分ほかでも問題ないはず
Windowsは某所でwhl配布されてるからそれを使うと楽

## ダウンロード

Sourceforgeからソースを落とせみたいな記事がたくさんあるけど罠です
makeでコケるのでGithubからcloneすると良いです

```
$ cd ~
$ git clone https://github.com/Itseez/opencv.git
$ cd opencv
$ git tag
タグ一覧が出るから使いたいバージョンを探す
$ git checkout 3.1.0
ダメだったらcloneしたやつそのまま使えばいけるかもね（今回の記事ではcloneしたのをそのまま使用）
```

2016年11月24日 追記

バニラな環境でやったらハマったのでメモ

```
-D PYTHON_EXECUTABLE=$(which python)
```

`$(which python)` は単にパス探してるだけなので
バニラだとpythonじゃなくてpython3で探さないとダメだった

### ビルド

```
$ pwd
/home/hoge/opencv
$ mkdir build
$ cd build
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D WITH_FFMPEG=OFF -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D PYTHON_EXECUTABLE=$(which python) ..
ログの途中にちゃんとPythonが対象に含まれてるか確認しておく
これはpyenvでの結果だけどノーマルのPythonでもこんな感じで含まれてるはず
--   Python 2:
--     Interpreter:                 /usr/local/pyenv/shims/python (ver 3.5.1)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython3.5m.so (ver 3.5.1+)
--     numpy:                       /usr/local/pyenv/versions/3.5.1/lib/python3.5/site-packages/numpy/core/include (ver 1.10.4)
--     packages path:               lib/python3.5/site-packages
--
--   Python 3:
--     Interpreter:                 /usr/local/pyenv/shims/python3 (ver 3.5.1)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython3.5m.so (ver 3.5.1+)
--     numpy:                       /usr/local/pyenv/versions/3.5.1/lib/python3.5/site-packages/numpy/core/include (ver 1.10.4)
--     packages path:               lib/python3.5/site-packages
--
--   Python (for build):            /usr/local/pyenv/shims/python
$ make
```

コケないことを祈ろう
make InstallはOpenCVそのものが使いたい場合いるけどPythonで使うなら要らない

### ライブラリの追加

Python用のパスを通す

```
$ find . -name "*.so"
./lib/libopencv_features2d.so
./lib/libopencv_cudafilters.so
./lib/libopencv_imgcodecs.so
./lib/libopencv_cudacodec.so
./lib/libopencv_videoio.so
./lib/libopencv_cudabgsegm.so
./lib/cv2.cpython-35m-x86_64-linux-gnu.so　←これ
./lib/libopencv_cudaobjdetect.so
./lib/libopencv_objdetect.so
./lib/libopencv_imgproc.so
./lib/libopencv_core.so
./lib/libopencv_shape.so
./lib/libopencv_flann.so
./lib/libopencv_cudafeatures2d.so
./lib/libopencv_cudastereo.so
./lib/libopencv_cudalegacy.so
./lib/libopencv_cudaimgproc.so
./lib/libopencv_photo.so
./lib/libopencv_cudev.so
./lib/libopencv_video.so
./lib/libopencv_ml.so
./lib/libopencv_stitching.so
./lib/libopencv_cudawarping.so
./lib/libopencv_videostab.so
./lib/libopencv_cudaarithm.so
./lib/libopencv_highgui.so
./lib/libopencv_superres.so
./lib/python3/cv2.cpython-35m-x86_64-linux-gnu.so　←これ
./lib/libopencv_cudaoptflow.so
./lib/libopencv_calib3d.so
必要な方を適宜モジュールのパスが通ったところへコピーする
```

おわり
