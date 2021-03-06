+++
author = "すかい"
title = "ESXi6.5でUbuntuがネットワークに繋がらない"
date = "2018-12-07"
description = "ESXi6.5でUbuntuがネットワークに繋がらない"
tags = [
    "ESXi",
    "Ubuntu",
]
+++

ESXi上で動かしているとあるVMでパッケージのアップデートを適用して再起動したらネットワークに繋がらなくなってしまった．
ただ，他のVMでは問題なくネットワークに接続できているので設定の問題かな？と思ってDHCPに戻したりして検証していたが，一行に接続できず参ってしまったのでその時のメモ．

## 環境

- ホスト
ESXi 6.5
- ゲスト
  - Ubuntu16.04（4.4.0-62-generic）
  問題なく接続できていたVM
  - Ubuntu16.04（4.4.0-140-generic）
  接続できなくなったVM

## 原因（推測）

技術力不足でログを追うところまではいけませんでしたが，カーネルのアップデートでドライバが対応しきれなくなったのかなと思います．
一応lsmodした感じ対応したドライバが読み込まれているようでしたが動作しませんでした．
接続できなかったVMにはE1000eのNICが接続されていたのでe1000のドライバを拾ってきて入れてやれば多分動いたと思います．
一応，手元にあったカーネルで動作チェックしたところ，こんな感じでした．

- 4.4.0-140-generic
- 4.4.0-139-generic
- 4.4.0-137-generic
  接続不可能
- 4.4.0-104-generic
- 4.4.0-62-generic
  接続可能

## 解決方法

今回は対応するドライバを入れ直す方法ではなく，ESXiのアダプタタイプを変更することで解決しました．
ESXiの管理コンソールに入り，ネットワークアダプタの設定項目を開いて当該アダプタのアダプタタイプをVMXNET 3に変更することでドライバも併せたものに変更されました．
変更したところ上記の接続不可能だったカーネルでも問題なく接続できるようになったのを確認しました．
