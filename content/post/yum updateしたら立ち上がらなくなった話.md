+++
author = "すかい"
title = "[未解決]yum updateしたら立ち上がらなくなった話"
date = "2017-01-08"
description = "[未解決]yum updateしたら立ち上がらなくなった話"
tags = [
    "CentOS",
]
+++

## はじめに

未解決なので経緯と作業ログだけのメモです

## 経緯

年始にサーバーのアップデートを行ったところ起動しなくなった
BIOS起動→GRUB2メニュー→kernel選択→起動時のプログレスバーが100%になるも起動せず
ログは手打ちなので間違ってるところあるかも

## 環境

2TB HDD*4でマザボでRAID5を構成した上にCentOS7をインストール
6TBになるためUEFI

## 用途

- SSH
- NFS
- LDAP

## 作業ログ

### プロセスがどこで止まってるのか調べる

※十字キーで切り替えることが出来たのでこの作業しなくても見れました
kernel選択画面で任意kernelを選択した状態でeキーを押してオプションの編集画面へ入り
linuxefiで始まる行の末尾にある「 rhgb quiet 」を削除してCtrl+xで起動
しばらく待っていると以下のログまで確認出来る

```
[  OK  ] Started Show Plymouth Boot Screen.
[  OK  ] Reached target Paths.
[  OK  ] Reached target Basic System.

[  180.055751] dracut-initqueue[334]: Warning: dracut-initqueue timeout - starting timeout scripts
～以下時間経過して同じログが続く～
一定時間経つと

[--*-- ] A start job is running for dev-mapper\x2root ( 3min 0sec / no limit)
```

あとは無限にこの待機状態が続く

### 古いkernelで試す

とりあえず最新のkernelによるものかと思ったので今まで起動出来ていたバージョンのkernelでテスト
しかし同様のエラー

### レスキューモードで起動する その1

kernelの選択画面にあるレスキュー用のkernelで起動する
しかしEmergencyModeで起動する
rootのパスワードを入れてコンソールに入る

```
# systemctl --fail | grep fail
```

して起動に失敗したサービスを調べる
色々失敗してるけどboot-efiのサービスがコケてるのでとりあえずその辺のファイルを覗きにいく
/boot/efi/の下に何もないのでマウント出来てないっぽい
/etc/fstabを見た上でマウントしてみる

```
# cat /etc/fstab
～ログをあとで入れる～
# mount -a
mount: unknown filesystem type 'vfat'
```

とりあえず有効なファイルシステムを確認

```
# cat /proc/filesystem | grep vfat
```

有効化されていない様子
/boot/efiにマウントされる領域がmount出来ないっぽいので一時的にコメントアウトした状態で再度レスキューモードで起動し直す

### レスキューモードで起動する その2

#### runlevel

今度はレスキューモードで起動出来た 一応runlevelを確認する

```
# $ runlevel
N 3
```

サービスは相変わらず色々failしてる
ネットに繋がらないので

```
# ifup eno1
```

とかしてネットワークに接続する

#### ロールバック

とりあえずアップデートしたところまでパッケージのバージョンを戻したら復活するのでは？と思ったので試す

```
# yum history
```

でログを見てアップデート前を探す

```
# yum history undo xx
```

で戻す　でもダメ

#### kernelを更新

3系のkernelを利用しているので4系のkernelを導入してみる
導入するも/boot/efi以下がマウント出来てないのでgurbのコンフィグの更新でコケる

### /boot/efiが生きてるか確認

liveUSBを用意しそのOS上でマウントして確認するとファイル群は存在している
chrootしてコンフィグを更新した上で再起動
しかし4系のkernelでも立ち上がらない

### vfatがマウント出来ない原因を調べる

kernelのinitramfsにモジュールが含まれていないのでは？と思ったのでそちらを調べる
前回の記事の内容でvfat.koが含まれていることを確認
ならmodprobeで有効に出来るはずなので有効化してみる

```
# modprobe vfat
```

何もレスポンスが来ない　成功しているだけかと思ったが存在しないモジュール名でも何も返してこない
仕方ないのでinsmodで手動で有効にしてみる

```
# insmod /lib/modules/{kernelバージョン}.el7.x86_64/kernel/fs/fat/vfat.ko
Required key not available
```

セキュアブートが有効になっているため一度BIOS画面まで戻りセキュアブートを無効にして再度トライ

```
# insmod /lib/modules/{kernelバージョン}.el7.x86_64/kernel/fs/fat/vfat.ko
Unknown symbol in module
```

前提モジュールがないためエラーになるので前提モジュールを探しにmoddepを見に行く

```
# cat /lib/modules/{kernelバージョン}.el7.x86_64/modules.dep | grep vfat
kernel/fs/fat/vfat.ko: kernel/fs/fat/fat.ko
```

前提モジュールが判明したので先にこちらを有効にする

```
# insmod /lib/modules/{kernelバージョン}.el7.x86_64/kernel/fs/fat/fat.ko
# insmod /lib/modules/{kernelバージョン}.el7.x86_64/kernel/fs/fat/vfat.ko
# cat /proc/filesystems | grep vfat
        vfat
```

有効になったので先程のコメントアウトを外して再度マウント

```
# vi /etc/fstab
# mount -a
```

今度はエラーも出ない
でも再起動しても例の「A Start job」で止まるので解決せず
ここまでの作業でレスキューモードで起動しないサービスはkdumpとNFSだけになったのでこれらの起動を目指す
※この辺で丸2日くらいぶっ通しで作業して心が折れた

### NFSのエラー原因を調べる

```
# systemctl start nfs
# systemctl status nfs -l
# journalctl -xe
```

辺りでエラー原因を探す
こちらもnfsdモジュールがないためunkwon file systemで怒られてるみたいなので先程の方法でちまちまinsmodしていく
すると一応サービスが上がったので今回はここで力尽きた

## まとめ

不用意にアプデ良くない
lvmにしてスナップショット取ったり色々対策しないと行き当たりばったりはダメだなって…
今回は応急処置状態なので再度構成を練って年度の変わり目に再構築します
