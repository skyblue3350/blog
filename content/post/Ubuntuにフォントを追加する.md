+++
author = "すかい"
title = "Ubuntuにフォントを追加する"
date = "2018-09-17"
description = "Ubuntuにフォントを追加する"
tags = [
    "Ubuntu",
]
+++

Ubuntuにフォントを入れたい用事があったのでトライしてみたメモ
今回は全体に入れますがユーザー単位なら/.local/share/fontsに入れるだけで後同じだと思います

## 環境

- Ubuntu 18.04
WSL環境です

## フォントの入手

今回は[青柳隷書しも](https://opentype.jp/aoyagireisho.htm)を導入してみます．
otfフォントなので少し追加でパッケージ入れたりがあります
予めDLして展開しておきます

## 導入

デフォルトではotfファイルを読み込まないのでランタイムを追加します

```
$ sudo apt install libotf0
```

otfファイルは/usr/share/fonts/opentypeに格納するのでここにディレクトリを事前に作成しフォントファイルを配置します

```
$ sudo mkdir -p /usr/share/fonts/opentype
$ sudo mv ~/path/to/aoyagireisyosimo_otf_2_01.otf /usr/share/fonts/opentype
```

あとはこれを読み込ませます

```
$ sudo fc-cache -f -v
/usr/share/fonts: caching, new cache contents: 0 fonts, 2 dirs
/usr/share/fonts/opentype: caching, new cache contents: 1 fonts, 0 dirs <- これ
/usr/share/fonts/truetype: caching, new cache contents: 0 fonts, 2 dir
...
```

のようにフォントが読み込まれます
一応フォント一覧を出して確認します

```
$ fc-list | grep aoyagireisyo
/usr/share/fonts/opentype/aoyagireisyosimo.otf: aoyagireisyo2,青柳隷書SIMO2_O:style=Regular
```

導入されてるのがわかります

## 参考記事

- [How to install OTF fonts?](https://askubuntu.com/questions/18357/how-to-install-otf-fonts)
