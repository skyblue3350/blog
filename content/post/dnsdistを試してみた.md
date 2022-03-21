+++
author = "すかい"
title = "dnsdistを試してみた"
date = "2018-03-11"
description = "dnsdistを試してみた"
tags = [
    "dnsdist",
]
+++

## はじめに

DNSロードバランサのdnsdistを試してみました．
というのもPowerDNSを試すのがメインだったのですがBINDのようにアクセス元に応じた振り分けが欲しくて困ったときに見つけた．
こちらのスライドで存在を知りました．ありがとうございます．

dnsdistとNSDとUnboundでBINDのふりをさせる話
https://dnsops.jp/bof/20161201/bind2other.pdf
というわけで使ってみます．

## 環境

- OS
Ubuntu16.04
- 内部ネットワーク
192.168.1.0/24

## インストール

Ubuntu公式レポジトリから提供されているdnsdistはver1.0.0α版と古いのでdnsdistの公式レポジトリを登録して使います．
有名どころなディストリビューションは提供されています．
https://repo.powerdns.com/

今回は「Ubuntu16.04 dnsdist - version 1.2.X」に従ってレポジトリを追加します．

```
$ sudo cat < EOS >> /etc/apt/preferences.d/dnsdist
deb [arch=amd64] http://repo.powerdns.com/ubuntu xenial-dnsdist-12 main
EOS
$ sudo cat < EOS >> /etc/apt/sources.list.d/pdns.list
Package: dnsdist*
Pin: origin repo.powerdns.com
Pin-Priority: 600
EOS
```

鍵を登録してインストールします．

```
$ curl https://repo.powerdns.com/FD380FBB-pub.asc | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install dnsdist
インストールすると自動でサービスが上がるみたいです．

$ sudo service dnsdist status
● dnsdist.service - DNS Loadbalancer
   Loaded: loaded (/lib/systemd/system/dnsdist.service; enabled; vendor preset: enabled)
   Active: active (running) since 土 2018-03-10 01:56:11 JST; 5min ago
     Docs: man:dnsdist(1)
           http://dnsdist.org
 Main PID: 4911 (dnsdist)
   CGroup: /system.slice/dnsdist.service
           └─4911 /usr/bin/dnsdist --supervised --disable-syslog -u _dnsdist -g _dnsdist

 3月 10 01:56:11 dns-outside dnsdist[4906]: Unable to read configuration from '/etc/dnsdist/dnsdist.conf'
 3月 10 01:56:11 dns-outside dnsdist[4906]: Configuration '/etc/dnsdist/dnsdist.conf' OK!
 3月 10 01:56:11 dns-outside dnsdist[4906]: Configuration '/etc/dnsdist/dnsdist.conf' OK!
 3月 10 01:56:11 dns-outside dnsdist[4911]: Unable to read configuration from '/etc/dnsdist/dnsdist.conf'
 3月 10 01:56:11 dns-outside dnsdist[4911]: Listening on 127.0.0.1:53
```

ログに出てる通り設定ファイルがないので作成していきます．

## 設定

今回は

- hoge.comの問い合わせは内外問わず返答する（返答する権威DNSは予め構築済み）
- internalへの問い合わせは内部のみ返答
  というルールで作ってみます．
  configファイルの位置は/etc/dnsdist/dnsdist.confです．

まずは問い合わせを転送するバックエンドのDNSを登録します．
中身は適当なので良い感じに読み替えて下さい．
poolが転送するグループ名でnameがあとでコンソールから状況を確認する時に使われる名称です．

```
newServer({address="192.168.1.2:53", pool="external", name="ex-master"})
newServer({address="192.168.1.3:53", pool="internal", name="in-master"})
```

オプションにweightとかあるので良い感じに振り分けることもできるみたいです．
その他オプションについては下記にまとまっています．
https://dnsdist.org/reference/config.html#newServer

次に転送するルールを記述します．

```
-- 外部
external = newSuffixMatchNode()
external:add(newDNSName("hoge.com"))
-- 内部
internal = newNMG()
internal:addMask("192.168.1.0/24")

-- 転送ルール
addAction(SuffixMatchNodeRule(external), PoolAction("external"))
addAction(NetmaskGroupRule(internal), PoolAction("internal"))
addAction(AllRule(), RCodeAction(dnsdist.REFUSED))
```

マッチしたルールに従って指定されたpoolのDNSへ問い合わせが行われて結果が転送されます．
RCodeの一覧はドキュメントにまとまっています．
https://dnsdist.org/reference/constants.html#rcode

ルールはいくつか組み合わせることもできるので内部からhoge.comへの問い合わせの場合だけ違うサーバーから返答させるとかも出来ます．
この辺に説明があります．
https://dnsdist.org/rules-actions.html?highlight=andrule#combining-rules
例えば上記の場合なら

```
addAction(AndRule({NetmaskGroupRule(internal), SuffixMatchNodeRule(external)}, PoolAction("other"))
```

みたいな感じで臨機応変にルールが作れるので良い感じです．
最後にアクセス元のルールを作成します．

```
addACL("0.0.0.0/0")
addLocal("0.0.0.0:53")
```

configの文法チェックをしたら再起動して適用します．

```
$ dnsdist --check-config
Configuration '/etc/dnsdist/dnsdist.conf' OK!
$ sudo service dnsdist restart
```

あとはdigって上手く動いてればおっけーです．

## コンソール

dnsdistはコンソールから設定の変更や状況の確認ができるらしいので試してみます．
https://dnsdist.org/guides/console.html

configファイルに下記を追加してコンソールを有効にします．

```
controlSocket("192.168.1.10:5199")
```

次に実際にコンソールへアクセスしてみます．

```
dnsdist -c 192.168.1.10:5199
>> showServers()
～設定したバックエンドの一覧が表示される～
```

使えるコマンドとかはリファレンス参照です．
https://dnsdist.org/reference/config.html#showServers

## その他

### ヘルスチェック

https://dnsdist.org/guides/downstreams.html#healthcheck
1秒ごとにDNSのクエリが飛んできてて何かなーと思って調べたら1秒ごとにバックエンドの状況をチェックしているようです．

### NetmaskGroupRuleがnilとか怒られる

```
$ dnsdist -V
1.0.0
```

1.0.0入れてました（追加するレポジトリ間違えてた）
新しいバージョンが落とせるレポジトリを追加して新しいバージョンを入れましょう…

```
$ dnsdist -V
dnsdist 1.2.1 (Lua 5.1.4)
Enabled features: dnscrypt libsodium protobuf re2 systemd
```

というわけで記事書いてる時に使ったバージョンはこんな感じです．
