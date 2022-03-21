+++
author = "すかい"
title = "自分向けgitメモ"
date = "2016-07-14"
description = "自分向けgitメモ"
tags = [
    "git",
]
+++

## はじめに

いろいろ毎回やらかすのでメモしておく
ある程度まとまったらマークダウンメモの方に移動する

## clone 編

### ユーザー情報の確認と設定

```
$ git config --global --list
user.email=hoge@fuga.com
user.name=hoge
```

このままコミットするとこのユーザー情報でコミットされる
違うユーザー情報で使用してるレポジトリとかでやらかすと恥ずかしい
cloneしたら意識して設定するようにする

```
$ cd プロジェクト
$ git config user.name hogehoge
$ git config user.email hogehoge@fuga.com
$ git config --list
user.email=hogehoge@fuga.com
user.name=hogehoge
```

## commit 編

### コミットメッセージ間違えた

Winのコマンド・プロンプトでコミットメッセージ書いてると時々盛大にミスする

```
$ git commit --amend
```

してから一番上の行を編集する　文字化けしてても別に気にしなくて良い
編集後にきちんとプレビューで表示されてればおっけー

### 空コミットしたい

最初は空コミットした方が気兼ねなく操作できるようになるので良いと思う

```
$ git commit --allow-empty -m "sky commit"
```

みたいな感じ

### 一時的にファイルを避難したい

空コミットし忘れた時とかその他諸々で一時的にファイルを元の状態に戻す

```
$ git stash save
```

して一時的に避難
いろいろ作業が終わって戻す時

```
$ git stash list
リストが出る
$ git stash apply stash@{0}
```

あとはコミットするなりお好きに

## 巻き戻す編

コミットIDまで巻き戻す fオプションつけてpushする

```
$ git reset --hard <コミットID>
$ git push -f hogehoge
```
