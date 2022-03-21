+++
author = "すかい"
title = "RaspberryPi2でrootなしでGPIOを操作する"
date = "2016-11-06"
description = "RaspberryPi2でrootなしでGPIOを操作する"
tags = [
    "Raspberry Pi 2",
]
+++

## はじめに

RaspberryPi2のApache上でWebからGPIOを操作する際に詰まったのでメモ
前回の方法で使ってるWiringPi2だとGPIOを利用する際に

```py
wiringpi2.wiringPiSetupGpio()
```

するとroot権が必要とのエラーが出て止まります

```py
wiringpi2.wiringPiSetupSys()
```

だとroot権なしで実行出来ますがこれだとdigitalWriteは出来ますがPWM等を制御することは出来ない様です

というわけで使うモジュールを変えました
元々入ってるRPi.GPIOで簡単にroot権なしでPWM制御出来たのでコードを載せておきます

```py
import RPi.GPIO as GPIO

PIN1 = 13

GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN1, GPIO.OUT)
servo = GPIO.PWM(PIN1, 50)

getDuty = lambda angle: (1.0 + angle/180.0)/20.0*100.0

while True:
        i = raw_input()
        if i == "e":
                break
        servo.start(getDuty(int(i)))

servo.stop()
GPIO.cleanup()
```

これでmod_wsgi使いつつFlaskでWebから操作出来る監視カメラもどきが出来ました
前はrootでapp.runして強引に動かしてました

もしダメだったらApacheの実行ユーザー（www-data）をgpioグループに追加するとうまくいくと思います

```
$ sudo cat /etc/apache2/envvars | grep APACHE_RUN_USER
とかして確認する
$ sudo gpasswd -a www-data gpio
$ sudo service apache2 restart
```
