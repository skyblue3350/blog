+++
author = "すかい"
title = "Ubuntu16.04でYoloを試す"
date = "2017-09-18"
description = "Ubuntu16.04でYoloを試す"
tags = [
    "Ubuntu",
]
+++

## はじめに

今更ながらにYoloを試します．
基本的には公式サイトの通りに実行するだけですが少し詰まったのでメモ．
コンソールで結果を見るだけなら公式サイト通りやって終わりです．

## 環境

- Ubuntu 16.04
- OpenCV 3.1.0

## 環境構築

### OpenCV

過去記事を参考にしつつ環境構築します

```
$ sudo apt-get install cmake libgtk2.0-dev pkg-config
$ git clone https://github.com/Itseez/opencv.git
$ cd opencv
$ git tag
$ git checkout 3.1.0
$ mkdir build && cd build
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D INSTALL_CREATE_DISRIB=ON -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D WITH_FFMPEG=OFF -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D PYTHON_EXECUTABLE=$(which python) ..
$ make
$ sudo make install
```

Python用のパッケージも作ってるけど特に意味はない
Yoloで必要なライブラリを配置する

```
$ find /usr/local/ -name "libippicv*"
$ ln -s /usr/local/share/OpenCV/3rdparty/lib/libippicv.a /usr/local/lib/
```

### Yolo

基本的に公式サイト通り

```
$ git clone https://github.com/pjreddie/darknet
$ cd darknet/
$ make
$ wget https://pjreddie.com/media/files/yolo.weights
$ ./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg
～中略～
data/dog.jpg: Predicted in 14.966523 seconds.
dog: 82%
truck: 65%
bicycle: 85%
```

できた

## エラーとか

### /usr/bin/ld: -lippicv が見つかりません

```
/usr/bin/ld: -lippicv が見つかりません
collect2: error: ld returned 1 exit status
Makefile:82: ターゲット 'libdarknet.so' のレシピで失敗しました
make: *** [libdarknet.so] エラー 1
```

OpenCVでビルドしたライブラリのパスが通ってないのでコピーすると読み込めるようになる．
シンボリックリンク貼った方が良いと思う．

```
$ find /usr/local/ -name "libippicv*"
$ sudo cp /usr/local/share/OpenCV/3rdparty/lib/libippicv.a /usr/local/lib/
```

### Unspecified error

```
OpenCV Error: Unspecified error (The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script) in cvNamedWindow, file /home/username/opencv/modules/highgui/src/window.cpp, line 527
```

エラー内容の通りパッケージを入れて再度makeする．

```
$ sudo apt-get install libgtk2.0-dev pkg-config
$ make
$ sudo make install
```

## 参考記事

- [/usr/bin/ld: cannot find -lippicv #5852](https://github.com/opencv/opencv/issues/5852)
- [Compiling error with -lippicv](http://answers.opencv.org/question/84265/compiling-error-with-lippicv/)
