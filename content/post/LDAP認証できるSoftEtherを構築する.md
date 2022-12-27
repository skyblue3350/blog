+++
author = "すかい"
title = "LDAP認証できるSoftEtherを構築する"
date = "2022-12-27"
description = "LDAP認証できるSoftEtherを構築する"
tags = [
    "SoftEther",
]
+++

LDAP のアカウントで認証して使える SoftEther を構築してみたのでメモ

## 構成

SoftEther に radius 認証と連携できる仕組みがあるので LDAP をバックエンドにした FreeRadius を使ってユーザー管理を委任した構成にします。
動くところまでしか見ていないので参考程度に…。

## 環境

AWS の ec2 に Ubuntu をおいて検証しました。

```
lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.1 LTS
Release:        22.04
Codename:       jammy

docker -v
Docker version 20.10.12, build 20.10.12-0ubuntu4
docker-compose -v
docker-compose version 1.26.0, build d4451659
```

## 設定

### SoftEther の設定

まずは SoftEther 単体で構築します。

- [siomiz/softethervpn](https://hub.docker.com/r/siomiz/softethervpn/)

サーバ管理用のパスワードがあるとリモートから全ての設定が行えるので扱いに注意が必要です。
後ほど freeradius の設定を投入するのに使います。

```yaml
version: "3"
services:
  vpn:
    image: siomiz/softethervpn
    cap_add:
      - NET_ADMIN
    ports:
      - 443:443
    environment:
      SPW: sample_spw_password
```

あとは README に従って諸々のファイルを用意します。

証明書を生成します。

```
sudo docker run --rm siomiz/softethervpn gencert > .env
```

コンテナを起動して接続用の情報をチェックします。
環境変数でユーザ名を指定していない場合ログにそのまま出ています。

```
sudo docker-compose up -d
sudo docker-compose logs | head -n 5
Attaching to vpn_vpn_1
vpn_1  | # ========================
vpn_1  | # user1234
vpn_1  | # 1234.1234.1234.1234.1234
vpn_1  | # ========================
```

AWS のセキュリティグループからインバウンドルールを追加して 443/tcp を開放しておきます。
クライアント側からポート開放チェックをして疎通しているか確認します。

```
# VM 上
nc -v -w 1 localhost 443
Connection to localhost (127.0.0.1) 443 port [tcp/https] succeeded!

# クライアント上
nc -v -w 1 203.0.113.1 443
Connection to 203.0.113.1 (203.0.113.1) 443 port [tcp/https] succeeded!
```

下記ページからクライアントを落としておきます。

- [SoftEther VPN ダウンロード](https://ja.softether.org/5-download)

下記設定で接続設定を追加します。

- 接続設定名
  - 任意
- ホスト名
  - VM のグローバル IP
- ポート番号
  - 443
- 仮想 HUB 名
  - DEFAULT
- プロキシの種類
  - 直接 TCP/IP 接続
- ユーザー認証
  - 種類
    - 標準パスワード認証
  - ユーザー名
    - user1234
  - パスワード
    - 1234.1234.1234.1234.1234

この後の作業が楽なので VPN を繋いだまま作業します。

### LDAP の設定

今回は以下の設定で LDAP を構築します。

- dc
  - dc=sample, dc=com
- ou
  - People
- cn
  - Group

イメージは以下を利用します

- [osixia/openldap](https://hub.docker.com/r/osixia/openldap/)

```yaml
  ldap:
    image: osixia/openldap:latest
    environment:
      LDAP_ORGANISATION: "sampleorg"
      LDAP_DOMAIN: "sample.com"
      LDAP_ADMIN_PASSWORD: "sample_ldap_password"
    ports:
      - 389:389
```

起動して中身を確認しておきます。

```
sudo docker-compose up -d
sudo docker-compose exec ldap slapcat
dn: dc=sample,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: sampleorg
dc: sample
structuralObjectClass: organization
entryUUID: e00880d0-19ec-103d-821e-673cb810be11
creatorsName: cn=admin,dc=sample,dc=com
createTimestamp: 20221227044421Z
entryCSN: 20221227044421.597972Z#000000#000#000000
modifiersName: cn=admin,dc=sample,dc=com
modifyTimestamp: 20221227044421Z
```

ldif 書いても良いのですが、LDAP 触っていたのがだいぶ前で思い出せないので GUI から設定します。
お好きな LDAP クライアントから以下の設定で接続します。

- Host
  - VM の IP
- Base
  - dc=sample,dc=com
- Account
  - username
    - cn=admin,dc=sample,dc=com
  - password
    - sample_ldap_password

接続したら OrganizationalUnit と Group を最初に記載した通り作成しておきます。
OrganizationalUnit 配下に User を追加しておきます。
また、当該ユーザの attribute に userPAssword が付与されていることを確認しておきます。

### freeradius の設定

以下のイメージを利用します。

- [irasnyd/freeradius-ldap](https://hub.docker.com/r/irasnyd/freeradius-ldap)

接続先設定で楽したいので先程の docker-compose に追記して利用します。

```yaml
version: "3"
services:
  ldap:
  ...
  radius:
    image: irasnyd/freeradius-ldap:latest
    ports:
      - "1812:1812/udp"
      - "1813:1813/udp"
    environment:
      - "LDAP_HOST=ldap"
      - "LDAP_USER=cn=admin,dc=sample,dc=com"
      - "LDAP_PASS=sample_ldap_password"
      - "LDAP_BASEDN=dc=sample,dc=com"
      - "LDAP_USER_BASEDN=ou=People,dc=sample,dc=com"
      - "LDAP_GROUP_BASEDN=ou=Groups,dc=sample,dc=com"
      - "RADIUS_CLIENT_CREDENTIALS=127.0.0.1:password1234"
```

コンテナを起動しておきます。

```
sudo docker-compose up -d
```

radtest コマンドを利用して freeradius <-> ldap 間の接続テストを行います。
今回はホスト側に用意して検証します。

```
sudo apt-get -y install freeradius
sudo systemctl stop freeradius
sudo systemctl disable freeradius
```

今回は LDAP 構築時に username と password を hoge にしたアカウントを用意しておいたのでそのアカウントを利用して動作確認します。

```
# 別コンソールで開いておく
sudo docker-compose logs ldap -f

# 作成した LDAP アカウントで接続
radtest hoge hoge 127.0.0.1 1812 password1234
```

うまくいくと以下のようなログが出ます。

```
radtest hoge hoge 127.0.0.1 1812 password1234
Sent Access-Request Id 88 from 0.0.0.0:41979 to 127.0.0.1:1812 length 74
        User-Name = "hoge"
        User-Password = "hoge"
        NAS-IP-Address = 127.0.0.1
        NAS-Port = 1812
        Message-Authenticator = 0x00
        Cleartext-Password = "hoge"
Received Access-Accept Id 88 from 127.0.0.1:1812 to 127.0.0.1:41979 length 20
```

失敗すると最後の行が以下のようになり Reject されます。

```
(0) -: Expected Access-Accept got Access-Reject
```

自分の場合はタイポしていたので LDAP 側のログを見ていて気付きました。

```
ldap_1    | 63aa936c conn=1010 op=3 SRCH base="ou=Poeple,dc=sample,dc=com" scope=2 deref=0 filter="(uid=hoge)"
```

また `RADIUS_CLIENT_CREDENTIALS` で事前に鍵共有して認証していますが、ここの設定がずれていると以下のようなログが出るので設定を見直す必要があります。
以下の場合は `RADIUS_CLIENT_CREDENTIALS=192.0.2.29:password1234` にする必要があります。

```
radius_1  | Tue Dec 27 06:32:41 2022 : Error: Ignoring request to auth address * port 1812 as server default from unknown client 192.0.2.29 port 60803 proto udp
```

よりセキュアにする場合は `LDAP_RADIUS_ACCESS_GROUP` 等を利用して特定ユーザーのみグループに入れて利用可能等にすると良さそうです。

### SoftEther の設定変更

radius サーバが用意できたのでつなぎこみます。
先程 VPN を張れるようにした際にリモートから設定投入できるようになっているので GUI で設定してしまいます。

SE-VPN サーバ管理ツールを起動して新しい接続設定の作成から設定を追加します。

- 接続設定名
  - 任意
- ホスト名
  - VM のグローバル IP
- ポート番号
  - 443
- プロキシの種類
  - 直接 　TCP/IP 接続
- 管理モード
  - サーバー管理モード
- 管理パスワード
  - sample_spw_password（最初に設定した SPW 変数の中身）

接続したら仮想 HUB の管理を開きます。

認証サーバの設定へ進み、以下の設定を追加します。

- Radius サーバの設定
  - 認証を使用する
    - チェック
  - ホスト名
    - VM のローカル IP
  - ポート番号
    - 1812
  - 共有シークレット
    - password1234(RADIUS_CLIENT_CREDENTIALSと対応したもの)
  - 共有シークレットの確認入力
    - 同上
  - 再試行間隔
    - 500 msec

ユーザーの管理へ進み、新規作成から以下のユーザーを追加します。

- ユーザー名
  - *
- 認証方式
  - 　RADIUS 認証

追加したら一度 VPN 接続を切って LDAP で追加したユーザーIDとパスワードでログインして接続出来たら OK です。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">おー…できた<br>LDAP のアカウントで VPN 張れる <a href="https://t.co/ClsMd68p4i">pic.twitter.com/ClsMd68p4i</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1607637677737906177?ref_src=twsrc%5Etfw">December 27, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
