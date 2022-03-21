+++
author = "すかい"
title = "VMware Playerを削除する"
date = "2018-08-04"
description = "VMware Playerを削除する"
tags = [
    "VMware",
]
+++

## はじめに

Windows10でVMware Playerをアンインストールする際にちょっと手間取ったのでメモ

## 環境

- Windows10
Win7から引き継ぎで10にあげた環境
- VMware Player 12.5.5
元々7のときにインストールして使ってた
10にしてから使おうとしたらエラーで再インストールするしかなくなった

## アンインストール

### 普通にコンパネから削除

できない
アンインストールがグレーアウトしてる
一応バージョン確認できる

### インストーラーから削除

インストール時に使ってインストーラーから削除できるらしい
ただそんなものはだいぶ前に消したので拾ってくる
公式サイトに行くと最新の14.0のページしかない
昔はメジャーバージョンのメニューで過去のも選択出来た気がするんだけど見当たらないのでURLから飛ぶ
https://my.vmware.com/jp/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/14_0
を
https://my.vmware.com/jp/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/12_0
に変更すると過去のインストーラーが拾える

あとはインストーラーを起動して削除を選択すると削除できる
サービスとか止めてファイルの実体を削除しちゃってインストール情報だけ消すなら

```
$ VMware-player-12.5.5-5234757.exe /clear
```

とかで消せる

消した後に14に移行して終わり
