+++
author = "すかい"
title = "WSL環境構築まとめ"
date = "2018-06-10"
description = "WSL環境構築まとめ"
tags = [
    "WSL",
]
+++

## はじめに

記事ぼちぼち書いてるのでご存知の通り未だにcygwin使っていますがようやく引っ越しする気になったので詰まった点とかまとめます．

## WSLのインストール

この辺はわかりやすい記事が色々あるので他の記事読んで下さい
今回はUbuntu18.04で検証しています

### WSLの有効化

Winマークを右クリックして「アプリと機能」->「プログラムと機能」->「Windowsの機能の有効化または無効化」
と進んで「Windows subsystem for Linux」を有効化して再起動

### 開発者向け機能の有効化

Winマークを右クリックして「設定」->「更新とセキュリティ」->「開発者向け」から開発者モードに切り替えておく

## イメージの入手

Winマークを左クリックして「store」と入力してMicrost Storeを開く
任意のイメージを検索してインストールする（Ubuntu18.04等）

## WSLの起動

Winマークを左クリックして任意のイメージ名を入力して起動する
しばらく初期設定が走ったあと任意のユーザー名とパスワードを設定します

## WSLの設定

### ユーザーのホームディレクトリの変更

FSの都合上かなり深いところにWSLのホームディレクトリがあります
そのためWin側で都合が良い場所にホームディレクトリを移します
Win側でストレージは/mnt/[DriveName]
となっているので適宜好きな場所に変更します
例えば「C:\wsl/」としたい場合

```
$ sudo vi /etc/passwd
- sky:x:1000:1000:,,,:/home/sky:/bin/bash
+ sky:x:1000:1000:,,,:/mnt/c/wsl:/bin/bash
```

のように変更しWSLを再起動します

### マウント方法の変更

/mnt下のドライブはマウント時にメタデータオプションが無効の状態でマウントされています
sshなどのファイルのパーミッションが重視されるものを利用しようとするとパーミッション周りでエラーがでるのでメタデータを有効にした状態でマウントするようにします

```
$ sudo vim /etc/wsl.conf
[automount]
enable = true
root = /mnt/
options = "metadata"
mountFsTab = false
```

## 環境構築

WSL固有でトラブったやつとかだけまとめます

### Docker

どちらの方法を使用するにせよWSL上にDockerが必要なので予めインストールしておきます

#### ホスト側にDockerを入れて使う

ホスト側にDockerを入れてTCPを有効にしWSL側から操作します
ただし以下に記載があるようにEnterprise、Professional、Educationでないと使うことができません

- [Windows 10 上に Hyper-V をインストールする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)

ただ一度設定したら楽に使えるのでエディションの問題がなければありだと思います
ホスト側でインストール後
Expose daemon on tcp://localhost:2375 without TLS
を有効にしておきます
その上で

```
$ echo "export DOCKER_HOST='tcp://0.0.0.0:2375'" >> ~/.bashrc
$ source ~/.bashrc
```

してホスト側のDockerを参照するように変更します

セキュリティ的に宜しくないので証明書を入れるなりした方が良いです
https://docs.docker.com/engine/security/https/

#### WSL側で頑張る

Homeエディションの場合は上記の選択肢は取れないのでこちらの選択肢か諦めて別の方法を模索することになります

普通にWSL上でDockerをインストールしてサービスをあげようとするとiptables周りでエラーが吐かれてあがりませんがWSLを管理者権限で上げてサービスをあげると解決できるそうです
そのあとでcgroupfs-mountしてないとコケるので先にしておきます
Winキー -> PowerShell -> 右クリックから管理者権限で立ち上げ

```
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\WINDOWS\system32> bash
$ sudo cgroupfs-mount
$ sudo service docker start
$ sudo service docker status
 * Docker is running
```

あとは通常通りのWSLからでもDockerが使えるようになります
毎回sudoつけるのは不便なのでdockerグループに入れておきます

```
$ sudo usermod -aG docker $USER
```

ただホストマシンが再起動する度に管理者権限で立ち上げてDockerサービスを上げ直さないといけないというデメリットがあります
なにか良い感じに解決出来る方法があったら教えて下さい

## 補足

開発者モードにするとOpenSSH Serverがホスト側であがります
止める場合はサービスの一覧にOpenSSH SSH Serverがあるので無効か手動にしておけば止まります
過去にやってあったので起動していませんでしたが今どうなのか知らないので一応

## エラーまとめ

### apt-getがコケる

セキュリティソフトによっては通信がブロックされるため使用しているアプリケーションの情報を集めましょう
ちなみにSEP（SymantecEndpointProtection）はSEP14MP2から公式対応しているのでapt-getはちゃんと通ります
それより前のバージョンは通信が遮断されてしまいます
利用しているSEP14はMP1だったためMP2にアップデートして解決しました
下記ページに案内がありますがダウンロードページに飛ぶ->対応言語のアップデーターを探す->ダウンロードしたzipの中に細かいバージョンから最新バージョンにあげるexeが入ってるのでそれぞれの環境に応じて実行して適用って感じです

- [Firewall blocks updates to the Windows 10 Linux sub-system feature.](https://support.symantec.com/en_US/article.TECH236946.html)

### sshできない

sshしようとすると

```
Bad owner or permissions on /mnt/c/wsl/.ssh/config
```

みたいなこと言われてchmodしようがchownしようがダメ

- [Microsoft/WSL Enable using host's .ssh folder #731](https://github.com/Microsoft/WSL/issues/731)

にある通りメタデータオプションをつけてマウントすれば解決します
自動でやる方法は環境構築のところに書きましたが毎回手動でも一応できます

```
$ umount /mnt/c
$ mount -t drvfs C: /mnt/c -o metadata
```

詳細は
https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-17063

追記
wsl.confでmountFsTab = trueとしておけばfstabを読みにいくので特定のストレージのみマウントしたい場合は

```
[automount]
enable = false
mountFsTab = true
```

としてfstabでマウントすれば良いらしいです
fstabのマウント->automountの順で処理されるので特定のストレージだけオプションつけてマウントとかもできるみたいです
https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-17093

### Error starting daemon: Devices cgroup isn't mounted

管理者権限で立ち上げたWSLからcgroupfs-mountを実行した上でDockerサービスをあげます

```
$ sudo cgroupfs-mount
$ sudo service docker start
```

## 参考記事

- [WSL(Bash on Windows)でDockerを使用する](https://qiita.com/yoichiwo7/items/0b2aaa3a8c26ce8e87fe)
- [WSL上でDocker Engineが動くようになっていたっぽいという話](https://qiita.com/yanoshi/items/dcecbf117d9cbd14af87)
