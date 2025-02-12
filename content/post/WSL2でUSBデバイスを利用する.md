+++
author = "すかい"
title = "WSL2でUSBデバイスを利用する"
date = "2025-02-10"
description = "WSL2でUSBデバイスを利用する"
tags = [
    "arduino",
    "lilygo",
    "wsl",
]
+++

[H716でサンプルプログラムを動かす](./H716でサンプルプログラムを動かす) で購入したlilygoのH716をWSL2で使う際にそのままでは使えなかったのでメモです。

WSL2ではUSBデバイスをそのまま使用することができないので、WinとWSL2の間でデバイスを共有する必要があります。

下記の公式ドキュメント通りではありますが、手順をまとめます。

- [USB デバイスを接続する](https://learn.microsoft.com/ja-jp/windows/wsl/connect-usb)

## usbipd-win のインストール

リリースから最新のバージョンをダウンロードします。
インストーラーを実行してインストールします。

https://github.com/dorssel/usbipd-win/releases/

## デバイスの共有

以下のコマンドでデバイスの一覧を確認できます。
H716接続前と接続後で比較することでデバイスのバスIDを特定します。

```
usbipd list
BUSID  VID:PID    DEVICE                                                        STATE
9-1    303a:1001  USB シリアル デバイス (COM6), USB JTAG/serial debug unit      Not shared
```

管理者権限でpowershellを開き、以下のコマンドでデバイスを共有します。
デバイスバスIDが `9-1` の場合、以下のように実行します。

```
usbipd bind --busid 9-1
```

再度 list で確認すると、`Shared` になっていることを確認します。

```
usbipd list
BUSID  VID:PID    DEVICE                                                        STATE
9-1    303a:1001  USB シリアル デバイス (COM6), USB JTAG/serial debug unit      Shared
```

この手順は一度実施すれば、次回以降は不要です。

## デバイスの接続

接続はデバイスがPCに接続される度に行う必要があります。

以下のコマンドでデバイスを接続します。

```
usbipd attach --busid 9-1
```

listで確認すると、`Attached` になっていることを確認します。

```
usbipd list
BUSID  VID:PID    DEVICE                                                        STATE
9-1    303a:1001  USB シリアル デバイス (COM6), USB JTAG/serial debug unit      Attached
```

WSL2でデバイスが認識されていることを確認します。

```
lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 011: ID 303a:1001 Espressif USB JTAG/serial debug unit
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

デバイスの読み書き権限を付与しておきます。

```
sudo chmod 777 /dev/ttyACM0
```
