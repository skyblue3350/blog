+++
author = "すかい"
title = "dnsdistでコンソールからサーバーを操作する"
date = "2018-09-03"
description = "dnsdistでコンソールからサーバーを操作する"
tags = [
    "dnsdist",
]
+++

dnsdistで管理してるキャッシュDNSサーバーがダウン判定されて復旧しているが再判定とかないのかダウンのままになっているので手動でUpに戻す

dnsdistのコンソールにアクセスする
前回の記事とかで設定済みの前提

```
$ dnsdist -c xxx.xxx.xxx.xxx:5199
```

まず現在のサーバーの状態を確認する

```
> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   powerdns-master      xxx.xxx.xxx.xxx:53                 up     0.0       0   1  1   41116479      12   0.0   0.3           0 auth
1   powerdns-recursor    yyy.yyy.yyy.yyy:5353                 down     0.0       0   1  1      96579       5   0.0 124.7           0 recursor
All
```

downになっているサーバーを取得してupにする

```
> getServer(1)
powerdns-recursor
> getServer(1):setUp()
```

再確認する

```
> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   powerdns-master      xxx.xxx.xxx.xxx:53                 up     4.0       0   1  1   41121037      12   0.0   0.3           0 auth
1   powerdns-recursor    yyy.yyy.yyy.yyy:5353                   UP     0.0       0   1  1      96583       5   0.0 124.3           0 recursor
All
```

終わり

ハマりどころとしては関数は「:」を使いデータに対しては「.」を使うという点
https://dnsdist.org/reference/config.html?highlight=setup#functions-and-types

```
getServer(0).order=12         -- set order of server 0 to 12
getServer(0):addPool("abuse") -- add this server to the abuse pool
```

> The . means order is a data member, while the : means addPool is a member function.

とのこと
