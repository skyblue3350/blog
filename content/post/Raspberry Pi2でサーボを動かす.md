+++
author = "すかい"
title = "Raspberry Pi2でサーボを動かす"
date = "2016-07-24"
description = "Raspberry Pi2でサーボを動かす"
tags = [
    "Python",
    "Raspberry Pi 2",
]
+++

## はじめに

Raspberry Pi2でサーボを動かします

このカメラマウントを動かすので制御するサーボは2つになります 1つを制御する記事は多かったけど2つのサーボを制御する記事があんま見つかりませんでした ちなみにこのカメラマウントレビューにもある通りサーボ付属のホーンが使えず少し加工が必要です 大した手間じゃないですが若干面倒

## 必要なもの

- Raspberry Pi2
- SG90*2
- カメラマウント

## 作業

### 組み立て

カメラマウントは説明書もないっぽいので適当にググって先人の知恵をお借りして頑張って組み立てます

### 配線

この辺とかが実際のRaspberryPi2の画像とかあって見やすいです

- [Pin Numbering - Raspberry Pi 2 Model B](http://pi4j.com/pins/model-2b-rev1.html)

GPIOの配置はこんな感じ
https://www.raspberrypi.org/documentation/usage/gpio/README.md

サーボ側のデータは秋月電子で公開されてるPDFを読む

- [SG90 9 g Micro Servo](http://akizukidenshi.com/download/ds/towerpro/SG90.pdf)
  2ページ目に線の説明がある　実物だと色が見づらいけど束ねられてる線の真ん中が赤色

ここを参考にすると下記の通り

- オレンジ
  PWM
- レッド
  Vcc（+）
- ブラウン
  Ground（-）

やることとしては5VとGNDに繋いでPWMをクロック違うところに繋ぐだけ
GPIO図のPWM0とかPWM1とかがクロックが違うのでうっかりPWM0とPWM0に繋ぐと2つのサーボが同じ挙動して焦る（実際にやった）
実際の配置はこんな感じ

- サーボ1
  オレンジ　→　PIN32
  レッド　→　PIN2
  ブラウン　→　PIN30
- サーボ2
  オレンジ　→　PIN33
  レッド　→　PIN4
  ブラウン　→　PIN34

ジャンパワイヤで直接繋いだのでGNDとVccを1個ずつポート使ってます

## 環境構築

といってもWiringPi2-Pythonを入れるだけです

```
$ pip install git+https://github.com/Gadgetoid/WiringPi2-Python
```

## テスト

- [RaspberryPiとWiringPiでサーボを動かす](http://qiita.com/locatw/items/f15fd9df40153bbb4d27)

この辺を参考にコードを書く
GPIOへのアクセスにrootがいる　ので実行時は

```
$ sudo python gpiotest.py
```

とかする

```py
import wiringpi2

PWM_PIN = 12  #or 13
value = 50

wiringpi2.wiringPiSetupGpio()
wiringpi2.pinMode(PWM_PIN, wiringpi2.GPIO.PWM_OUTPUT)
wiringpi2.pwmSetMode(wiringpi2.GPIO.PWM_MODE_MS)
wiringpi2.pwmSetClock(400)
wiringpi2.pwmWrite(PWM_PIN, value)
```

で1個サーボが動けばおっけー
PWM_PIN変数は12と13で違うクロックが出力出来るので2つ制御するときは12と13に繋ぐ
GPIO番号が図と違うのでGPIO番号を見る時は違う図をググって参照しました

ApacheでWebから監視カメラもどき作ったらroot権必要でモメたので不要な方法を見つけました

- [RaspberryPi2でrootなしでGPIOを操作する](https://blog.sky-net.pw/post/RaspberryPi2でrootなしでGPIOを操作する)
