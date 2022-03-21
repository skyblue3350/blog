+++
author = "すかい"
title = "Raspberry PiでUSBヘッドセットを使う"
date = "2018-02-10"
description = "Raspberry PiでUSBヘッドセットを使う"
tags = [
    "Raspberry Pi 2",
]
+++

## はじめに

Raspberry PiでUSBヘッドセットを使いたいって相談を受けて試してみたのでメモ．
Raspberry Pi2で検証しましたが基本的にはどれでも同じはず．
Raspberry Pi2はクリーンインストールしたキレイな状態です．

## 環境

- Raspberry Pi 2
  ```
  pi@raspberrypi:~ $ cat /etc/debian_version
  9.3
  pi@raspberrypi:~ $ lsb_release -a ※lsb-releaseパッケージ入れました
  No LSB modules are available.
  Distributor ID: Raspbian
  Description:    Raspbian GNU/Linux 9.3 (stretch)
  Release:        9.3
  Codename:       stretch
  ```
- ヘッドセット（バッファロー BUFFALO BSHSUH12BK）

## 環境構築

### パッケージのアップデート

いつも通りアプデします．

```
pi@raspberrypi:~ $ sudo apt-get update -y
pi@raspberrypi:~ $ sudo apt-get upgrade -y
```

### スピーカーで音を聞くまで

Raspberry Piを起動した状態でUSB接続します．
認識したか確認します．大体末尾の番号のやつがそうです．

```
pi@raspberrypi:~ $ lsusb
Bus 001 Device 005: ID 046d:c52b Logitech, Inc. Unifying Receiver ←USBキーボード
Bus 001 Device 004: ID 0d8c:013c C-Media Electronics, Inc. CM108 Audio Controller ←認識された
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Alsa側に認識された確認します．

```
pi@raspberrypi:~ $ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 0: ALSA [bcm2835 ALSA], device 1: bcm2835 ALSA [bcm2835 IEC958/HDMI]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio] ←認識されてる
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

デバイスの優先度を確認します．

```
pi@raspberrypi:~ $ cat /proc/asound/modules
 0 snd_bcm2835
 1 snd_usb_audio
```

USBデバイスの優先度が低いので上にあげます．
デフォルトの状態であればファイルがないので作成します．

```
pi@raspberrypi:~ $ sudo vi /etc/modprobe.d/alsa-base.conf
options snd slots=snd_usb_audio,snd_bcm2835
options snd_usb_audio index=0
options snd_bcm2835 index=1
```

適用するために再起動します．

```
pi@raspberrypi:~ $ sudo reboot
```

確認します．

```
pi@raspberrypi:~ $ cat /proc/asound/modules
 0 snd_usb_audio ←優先度があがった
 1 snd_bcm2835
```

ここでusbが消えてる場合はconfig書き間違えてる可能性が高いです．
コメントアウトしたりして元に戻るようなら記述を見直しましょう．
音を流してみて聞こえるかテストします．

```
pi@raspberrypi:~ $ aplay /usr/share/sounds/alsa/Front_Center.wav
Playing WAVE '/usr/share/sounds/alsa/Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

Front Centerで音声が聞こえたらOKです．

### マイクを使う

多分スピーカーが鳴る状態まで来てると使えるはずです．
今回はヘッドセットだったからかもですけど．
録音はCtrl+Cで終了します．

```
pi@raspberrypi:~ $ arecord -f S16_LE -r 44100 test.wav
Recording WAVE 'test.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Mono
[Ctrl+C]
Aborted by signal Interrupt...
arecord: pcm_read:2103: read error: Interrupted system call
pi@raspberrypi:~ $ ls
test.wav
```

再生してみます．

```
pi@raspberrypi:~ $ aplay test.wav
Playing WAVE 'test.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Mono
```

録音した音が入ってればおっけーです．

## エラーあれこれ

### /proc/asound/modulesからsnd_usb_audioが消えた！

多分alsa-base.confの記述が間違っています．
自分の場合はslotsの部分がslotになってました．

```
- options snd slot=snd_usb_audio,snd_bcm2835
+ options snd slots=snd_usb_audio,snd_bcm2835
```
