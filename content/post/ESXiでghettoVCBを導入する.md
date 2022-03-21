+++
author = "すかい"
title = "ESXiでghettoVCBを導入する"
date = "2018-05-13"
description = "ESXiでghettoVCBを導入する"
tags = [
    "ESXi",
]
+++

## はじめに

ESXiでバックアップを取るためにghettoVCBを導入する

## 環境

- ESXi 6.5

## 事前準備

### SSHの有効化

SSHを有効にしないと各種操作が行えないので予めSSHを有効にしておく
WebUIにログインし
ホスト->アクション->サービス->SSHの有効化
からSSHを有効化にしておく

## インストール

データストアブラウザから
https://github.com/lamw/ghettoVCB/blob/master/vghetto-ghettoVCB.vib
上記をダウンロードしたものをアップロードしておく

SSHでログインし

```
# esxcli software vib install -v /path/to/vghetto-ghettoVCB.vib -f
```

としてインストールを行う
インストールが出来た場合

```
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: virtuallyGhetto_bootbank_ghettoVCB_1.0.0-0.0.0
   VIBs Removed:
   VIBs Skipped:
```

と結果が表示される

次にシェルスクリプトをコピーする

```
# cp /opt/ghettovcb/bin/ghettoVCB.sh /path/to/
```

あとは環境に合わせて編集しdry-runしてテストした後に実際に実行してみて問題ないかテストする
ひとまず必要なのはバックアップデータの書き出し先なのでここだけ変更してみる

```
- VM_BACKUP_VOLUME=/vmfs/volumes/mini-local-datastore-hdd/backups
+ VM_BACKUP_VOLUME=/vmfs/volumes/ds2/backup
```

dry-runして問題ないか確認する

```
# ./ghettoVCB.sh -m VM名 -d dryrun
Logging output to "/tmp/ghettoVCB-2018-05-13_02-00-31-1436833.log" ...
2018-05-13 02:00:31 -- info: = ghettoVCB LOG START =
～略～
2018-05-13 02:00:32 -- info: ###### Final status: OK, only a dryrun. ######

2018-05-13 02:00:32 -- info: = ghettoVCB LOG END =
```

とでたら問題なく実行されている
dry-runを外して実行すれば実際にバックアップが行われる
ひとまずここまで

## 単一環境のバックアップ

dry-runの項目で利用してる-mオプションでVM名を指定することでできる

## すべての環境のバックアップ

-aオプションで稼働中のすべてのVMを対象にすることができる

## 特定VMのバックアップ

-f filenameで指定したVMのみバックアップを行う

## 特定VM以外のバックアップ

-e filenameで指定したファイルに除外リストを書いておくことで除外できる
-aと組み合わせて利用すると良いっぽい

## 詰まったところ

### エラーでコケる

```
# esxcli software vib install -v vghetto-ghettoVCB.vib -f
 [MetadataDownloadError]
 Could not download from depot at zip:/var/log/vmware/vghetto-ghettoVCB-offline-bundle.zip?index.xml, skipping (('zip:/var/log/vmware/vghetto-ghettoVCB-offline-bundle.zip?index.xml', '', "Error extracting index.xml from /var/log/vmware/vghetto-ghettoVCB-offline-bundle.zip: [Errno 2] No such file or directory: '/var/log/vmware/vghetto-ghettoVCB-offline-bundle.zip'"))
        url = zip:/var/log/vmware/vghetto-ghettoVCB-offline-bundle.zip?index.xml
 Please refer to the log file for more details.
```

インストールする際はフルパスで書かないといけないだけでしたが少しハマりました

```
# esxcli software vib install -v /vmfs/volumes/hoge/vghetto-ghettoVCB.vib -f
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: virtuallyGhetto_bootbank_ghettoVCB_1.0.0-0.0.0
   VIBs Removed:
   VIBs Skipped:
```
