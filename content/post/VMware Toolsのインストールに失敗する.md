+++
author = "すかい"
title = "VMware Toolsのインストールに失敗する"
date = "2017-05-14"
description = "VMware Toolsのインストールに失敗する"
tags = [
    "VMware",
]
+++

## はじめに

普段VMware Toolsのインストールを行う時はPlayerのメニューから管理 -> VMware Toolsのインストール と移動するとインストーラーの入ったディスクがマウントされるのでそこから展開してインストーラーを実行すれば出来る
コケるとしてもgccが入ってないとかその程度のもの
ただ今回はすんなりいかなかったのでメモ

## エラー

この手のエラーが大量に出てビルドにコケる

```
/tmp/modconfig-O5xscq/vmhgfs-only/dir.c:1174:13: error: ‘struct file’ has no member named ‘f_dentry’
!(file->f_dentry->d_inode)) {
```

## 解決方法

- [Error VMware Tools Installation (Shared Folder) - Ubuntu 15.04 [SOLVED!]](https://communities.vmware.com/thread/509898)

こちらにある通り下記レポジトリのパッチをあてたデータを利用させてもらうことでインストール出来ました

- [rasa/vmware-tools-patches](https://github.com/rasa/vmware-tools-patches)

[Quick Start](https://github.com/rasa/vmware-tools-patches#quick-start) の項目の通りに実行するだけなので大した手間もなく助かりました
