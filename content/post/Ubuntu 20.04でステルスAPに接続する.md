+++
author = "すかい"
title = "Ubuntu 20.04でステルスAPに接続する"
date = "2020-07-19"
description = "Ubuntu 20.04でステルスAPに接続する"
tags = [
    "Ubuntu",
]
+++

ラズパイに Ubuntu を入れて Wi-Fi 接続しようとしまったらハマったのでメモ

## 環境

- Raspberry Pi 4
- Ubuntu 20.04
  ```
  $ cat /etc/lsb-release
  DISTRIB_ID=Ubuntu
  DISTRIB_RELEASE=20.04
  DISTRIB_CODENAME=focal
  DISTRIB_DESCRIPTION="Ubuntu 20.04 LTS"
  ```

## 接続

以下を参考に設定ファイルを作成します。

- [Network - Configuration | Server documentation | Ubuntu](https://ubuntu.com/server/docs/network-configuration)

ポイントは ssid の指定に wpa supplicant の scan_ssid を設定している点です。

```
$ sudo vim /etc/netplan/99_config.yaml
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            dhcp4: true
            optional: true
            access-points:
                "ssid\"\n  scan_ssid=1\n#":
                    password: "password"
```

※環境によっては一度ここでrebootする必要があるかもしれません。

ファイルを作成したら以下のコマンドで適用します。

```
$ sudo netplan apply
```

適用後 IP が振られていることを確認します。

```
$ ip -4 a show wlan0
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet xxx.xxx.xxx.xxx/24 brd xxx.xxx.xxx.xxx scope global dynamic wlan0
       valid_lft 258516sec preferred_lft 258516sec
```

## 接続できない時の確認

対象のデバイス名のサービスが作成されているため、そのサービスの status とログを確認します。

```
$ sudo systemctl status netplan-wpa-wlan0.service
$ sudo journalctl -u netplan-wpa-wlan0.service
Apr 01 17:40:48 ubuntu wpa_supplicant[2051]: Line 6: Invalid SSID line '"'.
Apr 01 17:40:48 ubuntu wpa_supplicant[2051]: Line 9: failed to parse network block.
Apr 01 17:40:48 ubuntu wpa_supplicant[2051]: Failed to read or parse configuration '/run/netplan/wpa-wlan0.conf'.
Apr 01 17:40:48 ubuntu systemd[1]: netplan-wpa-wlan0.service: Main process exited, code=exited, status=255/EXCEPTION
Apr 01 17:40:48 ubuntu systemd[1]: netplan-wpa-wlan0.service: Failed with result 'exit-code'
```

## なぜ ssid の指定時に scan_ssid を埋め込む必要があるのか

詳細は以下にまとまっていますが、netplan のバグのようです。

- [Bug #1866100 “netplan configuration does not allow for hidden SS...” : Bugs : netplan](https://bugs.launchpad.net/netplan/+bug/1866100)

のため、ssid の部分に後続で生成される wpa supplicant 用のconfigを差し込んでおくことで生成されるconfigに必要な設定が書き込まれるようです。

生成されたconfigを見ると以下のように scan_ssid が書き込まれていることがわかります。

```
$ sudo cat /run/netplan/wpa-wlan0.conf
ctrl_interface=/run/wpa_supplicant

network={
  ssid="ssid"
  scan_ssid=1
#"
  key_mgmt=WPA-PSK
  psk="password"
}
```

現在は修正されているようで netplan の 0.100 からは hidden オプションが追加されています。
ただ、手元で利用しているパッケージのバージョンは 0.99 であったため対象外でした。

```
$ sudo dpkg -l | grep netplan
ii  libnetplan0:arm64              0.99-0ubuntu2                     arm64        YAML network configuration abstraction runtime library
ii  netplan.io                     0.99-0ubuntu2                     arm64        YAML network configuration abstraction for various backends
```

## 参考記事

- [networking - Netplan not connecting to hidden SSID Server 18.04.1 LTS - Ask Ubuntu](https://askubuntu.com/questions/1111494/netplan-not-connecting-to-hidden-ssid-server-18-04-1-lts)
- [Bug #1866100 “netplan configuration does not allow for hidden SS...” : Bugs : netplan](https://bugs.launchpad.net/netplan/+bug/1866100)
- [Add hidden to connect to non-broadcast SSIDs by aganders3 · Pull Request #132 · CanonicalLtd/netplan](https://github.com/CanonicalLtd/netplan/pull/132)
