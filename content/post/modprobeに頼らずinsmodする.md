+++
author = "すかい"
title = "modprobeに頼らずinsmodする"
date = "2018-08-29"
description = "modprobeに頼らずinsmodする"
tags = [
    "CentOS",
]
+++


だいぶ前に書いた記事 [[未解決]yum updateしたら立ち上がらなくなった話](../yum-update%E3%81%97%E3%81%9F%E3%82%89%E7%AB%8B%E3%81%A1%E4%B8%8A%E3%81%8C%E3%82%89%E3%81%AA%E3%81%8F%E3%81%AA%E3%81%A3%E3%81%9F%E8%A9%B1/) の後日談です．
このマシン実は未だにレスキューモードで強引に動かしています．

今回ようやく移行機材が揃ったので移行先へファイルを移すためにNFSマウントしようとしたらモジュールが有効になってないので

```
mount.nfs4: No such device
```

と出てマウント出来ませんでした．
mountコマンドに-vをつけて実行した感じnfsv4が有効になってないようなのでこれを手動で有効にします．
普通の場合なら単に

```
$ modprobe nfs
```

として終わりですがレスキューモードだとmodules.depが読めないからか使えないので人力でやります．

```
$ insmod /lib/modules/3.10.0-327.22.2.el7.x86_64/kernel/fs/nfs/nfsv4.ko
Unknown symbol in module
```

と出て前提モジュールがないので調べます．

```
$ cat /lib/modules/3.10.0-327.22.2.el7.x86_64/modules.dep | grep nfsv4.ko:
kernel/fs/nfs/nfsv4.ko: kernel/net/dns_resolver/dns_resolver.ko kernel/fs/nfs/nfs.ko kernel/fs/lockd/lockd.ko kernel/fs/nfs_common/grace.ko kernel/net/sunrpc/sunrpc.ko kernel/fs/fscache/fscache.ko
```

で必要なものを1個ずつinsmodして繰り返していきます．
今回はdns_resolver.ko以外は

```
$ insmod /lib/modules/3.10.0-327.22.2.el7.x86_64/kernel/fs/nfs/nfs.ko
File exists
```

と出て有効化してあったので

```
$ insmod /lib/modules/3.10.0-327.22.2.el7.x86_64/kernel/net/dns_resolver/dns_resolver.ko
$ insmod /lib/modules/3.10.0-327.22.2.el7.x86_64/kernel/fs/nfs/nfsv4.ko
```

して終わりでした．
一応確認して無事NFSマウントできました．

```
$ cat /proc/filesystems | grep nfs
nodev   nfsd
nodev   nfs
nodev   nfs4
$ mount -t 192.168.xxx.xxx:/path/to/dir /mnt/path/to/dir -v
```
