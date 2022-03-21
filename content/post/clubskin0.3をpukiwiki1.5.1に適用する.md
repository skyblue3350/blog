+++
author = "すかい"
title = "clubskin0.3をpukiwiki1.5.1に適用する"
date = "2017-06-12"
description = "clubskin0.3をpukiwiki1.5.1に適用する"
tags = [
    "pukiwiki",
]
+++

## はじめに

この前dockerhubに公開したpukiwikiにclubskinを適用しようとしてハマったので軽くメモ
個人的にはLDAPで認証掛けられて左側にツリーメニューが出せてMDで書ければそちらに移行したいけどなかなか見つからないのでご存知の方がいたら教えて欲しい
自作してるけどなんかイマイチ

## 環境

- pukiwiki 1.5.1
- php 7系

## 作業

### 適用前作業

ポイントは2つで

- 文法を7系に合わせる
- 全部コピーしない

```
$ wget http://www.kazuwaya.jp/clubskin/clubskin-0.3-utf8.zip
$ unzip clubskin-0.3-utf8.zip
$ rm -rf clubskin-0.3-utf8/clubskin-0.3-utf8/lib
$ rm clubskin-0.3-utf8/clubskin-0.3-utf8/pukiwiki.ini.php
$ vi clubskin-0.3-utf8/clubskin-0.3-utf8/lib/convert_html.php
$ vi clubskin-0.3-utf8/clubskin-0.3-utf8/plugin/secedit.inc.php
$ vi clubskin-0.3-utf8/clubskin-0.3-utf8/plugin/attach.inc.php
& newとなっている部分の&を消す
& newだったり&newだったり表記ゆれしてるので適宜検索か置換コマンドで消す
```

### 適用

```
$ cp -R clubskin-0.3-utf8/clubskin-0.3-utf8/. pukiwikidir
```

あとは再度アクセスしてみて変わっているか確認する

## その他

### ajaxtreeの一部項目を非表示にする

このサイトについてとかHelpとか消したい時

```
$ vi plugin/ajaxtree.inc.php
// Ignore list
if (!defined('PLUGIN_AJAXTREE_NON_LIST')) {
    define('PLUGIN_AJAXTREE_NON_LIST', 'このサイトについて|Help|BracketName|FormattingRules|InterWiki|InterWikiName|InterWikiSandBox|MenuBar|PHP|PukiWiki|RecentChanges|RecentDeleted|SandBox|WikiEngines|WikiName|WikiWikiWeb|YukiWiki|ヘルプ');// kazuwaya
}
```

46行目付近に非表示リストが正規表現で書かれているので適宜合わせて編集する

### ソーシャルアイコンを消したい

一部古かったりhttpで取りに行ってるのでhttps化するとmixedコンテンツになって困る

```
$ vi skin/pukiwiki.skin.php
上部と下部の当該箇所を削除
```

### 上部メニューを編集したい

ソーシャルアイコンと同じ方法で当該要素がある場所を見つけてリンクを直に書き足して終わり

## 最後に

現状動くけどどっかしら動かなかったりするのではないかな…と不安になってるけど動いてるので良しとします
