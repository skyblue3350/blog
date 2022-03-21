+++
author = "すかい"
title = "Ubuntu16.04でRuby on Railsのチュートリアルをする"
date = "2017-08-06"
description = "Ubuntu16.04でRuby on Railsのチュートリアルをする"
tags = [
    "Ruby on rails",
]
+++

## はじめに

https://railsguides.jp/getting_started.html
こちらのチュートリアルをやる上で詰まったところとかのメモ

## 環境

検証はDockerのUbuntu16.04で行いました

最終的にこうなりました

```
$ ruby -v
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]
$ rails -v
Rails 5.1.3
```

## 環境構築

### Rubyのインストール

後者4つのパッケージはいれておかないとnokogiriとsqlite3のインストール時にコケます

```
$ sudo apt-get install ruby ruby-dev build-essential zlib1g-dev libsqlite3-dev
```

### Railsのインストール

```
$ gem install rails
```

とりあえずこれだけで終わり

## プロジェクト作成

チュートリアル通り作る

```
$ rails new blog
```

## プロジェクトの実行

チュートリアル通りサーバーを立ち上げようとするとエラーで止まる

```
$ rails server
えらー
```

ので

```
$ vi Gemfile
- #  gem 'therubyracer', platforms: :ruby
+ gem 'therubyracer', platforms: :ruby
$ bundle install
```

のようにコメントを外しておく
ただこのまま実行しても

```
$ rails server
/var/lib/gems/2.3.0/gems/activesupport-5.1.3/lib/active_support/railtie.rb:22:in `rescue in block in <class:Railtie>': tzinfo-data is not present. Please add gem 'tzinfo-data' to your Gemfile and run bundle install (TZInfo::DataSourceNotFound)
…
```

と怒られるので

```
$ vi Gemfile
- gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
+ gem 'tzinfo-data'
$ bundle update
```

する
あとは普通に起動する

チュートリアル残りの部分は特に詰まることなくいけると思います

## 参考記事

- [rails server で There was an error while trying to load the gem 'uglifier'.と言われる解決方法。](http://qiita.com/pugiemonn/items/11a2bc8403e5947a8f13)
- ["The dependency tzinfo-data will be unused" on Ubuntu](https://github.com/tzinfo/tzinfo-data/issues/12)
