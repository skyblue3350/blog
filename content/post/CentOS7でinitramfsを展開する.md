+++
author = "すかい"
title = "CentOS7でinitramfsを展開する"
date = "2017-01-05"
description = "CentOS7でinitramfsを展開する"
tags = [
    "CentOS",
]
+++

## はじめに

CentOS7であるモジュールがカーネルに組み込まれているか知りたくなったが従来の方法では展開出来なかったのでまとめ

今までの手順でgunzipをcpioにパイプすると

```
gzip: initramfs.gz: not in gzip format
```

みたいに怒られてうまくいかない

## 展開方法

まずbinwalkというアプリケーションが必要なのでインストール

## インストール

https://github.com/devttys0/binwalk
からクローンしてインストールする

```
$ git clone https://github.com/devttys0/binwalk
$ cd binwalk
$ sudo python setup.py install
```

## 展開する

バイナリでgzipが始まるところ以降を展開したいので調べる

```
$ binwalk /boot/initramfs-3.10.0-514.2.2.el7.x86_64.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             ASCII cpio archive (SVR4 with no CRC), file name: ".", file name length: "0x00000002", file size: "0x00000000"
112           0x70            ASCII cpio archive (SVR4 with no CRC), file name: "kernel", file name length: "0x00000007", file size: "0x00000000"
232           0xE8            ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86", file name length: "0x0000000B", file size: "0x00000000"
356           0x164           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode", file name length: "0x00000015", file size: "0x00000000"
488           0x1E8           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/GenuineIntel.bin", file name length: "0x00000026", file size: "0x00005800"
23164         0x5A7C          ASCII cpio archive (SVR4 with no CRC), file name: "early_cpio", file name length: "0x0000000B", file size: "0x00000002"
23292         0x5AFC          ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
23552         0x5C00          gzip compressed data, maximum compression, from Unix, last modified: 2017-01-03 12:15:21
2578829       0x27598D        MySQL MISAM index file Version 6
```

23552byte以降が当該バイナリなのでこれを展開する

```
$ dd if=/boot/initramfs-3.10.0-514.2.2.el7.x86_64.img of=initramfs-output.gz bs=23552 skip=1
$ gunzip initramfs-output.gz
$ mkdir temp
$ cd temp
$ cpio -i -d -H newc --no-absolute-filenames < ../initramfs-output.gz
```

展開さえ出来れば

```
$ find -name "module.ko"
```

とかで探せる

## 参考記事

- [のぴぴのメモ RHEL7 initramfsの展開方法](http://nopipi.hatenablog.com/entry/2015/05/06/235615#%E3%82%B0%E3%83%BC%E3%82%B0%E3%83%AB%E3%81%95%E3%82%93%E3%81%AB%E8%81%9E%E3%81%84%E3%81%A6%E3%81%BF%E3%82%8B)
- [Fedora Forum [SOLVED] how to unpack initramfs on fc21](http://forums.fedoraforum.org/showthread.php?t=302534)
- [Github devttys0/binwalk](https://github.com/devttys0/binwalk)
