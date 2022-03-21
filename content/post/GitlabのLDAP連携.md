+++
author = "すかい"
title = "GitlabのLDAP連携"
date = "2017-04-02"
description = "GitlabのLDAP連携"
tags = [
    "Docker",
	"Gitlab",
]
+++

## はじめに

sameersbn/docker-gitlabでLDAP連携するメモ

## 環境

- Docker-compose version 1.11.2, build dfed245
- Docker version 17.03.1-ce, build c6d412e
- sameersbn/docker-gitlab 8.17.4

## 設定

レポジトリにあるdocker-compose.xmlを参考に記述する

- [docker-gitlab/docker-compose.yml](https://github.com/sameersbn/docker-gitlab/blob/master/docker-compose.yml)

※余談ですがバックアップがdailyになっててテスト環境でテストしてたら死にました

## 設定値例

- LDAPホスト：192.168.1.30
- LDAPポート：389
- 認証ユーザーグループ：dc=hoge,dc=fuga,dc=foo,dc=co,dc=jp
- ユーザー名：uid

必要部分だけ抜粋

```
～略～
  gitlab:
    restart: always
    image: sameersbn/gitlab:8.17.4
    ～中略～
    environment:
    - DEBUG=false
    ～中略～
    - LDAP_ENABLED=true
    - LDAP_LABEL=えるだっぷ
    - LDAP_HOST=192.168.1.30
    - LDAP_PORT=389
    - LDAP_BASE=dc=hoge,dc=fuga,dc=foo,dc=co,dc=jp
    - LDAP_UID=uid
```

LDAP_LABELはトップページのログイン方式選択のラベルで表示される
LDAP_BASEは””とかで囲む必要はない　これでハマったので今回の記事書いたところもある
LDAP_UIDはLDAP側ユーザー名として設定してるものに合わせる

## 確認

コンテナを起動して確かめる

```
$ cd gitlab
$ docker-compose up -d
```

このように表示されてればとりあえずおっけー
あとはログインしてみて正しくログイン出来れば終わり

## 設定値の確認

ダメそうだったらコンテナの中に入って確認してみる

```
$ docker exec -it gitlab bash
# vi config/gitlab.yml
```

とかしてldapで検索すると設定値が確認出来るのでミスがないか確認する
ここで””つけてるのが反映されてるのを見つけて解決した

## おまけ

通常のユーザー登録を行えないようにする
今回はLDAPユーザー向けの提供なのでユーザー登録は禁止にします
臨時でアカウント等が要る場合は管理者から通常アカウントを作成すれば良いです（その場合はログイン時にStandardのタブに切り替えてログインします）
管理者アカウントでログイン → Settings → Sign-up Restrictionsと移動しSign-up enabled のチェックを外してセーブして終わりです
URLは/admin/application_settingsです

Overviewの画面でSign upが無効になってることを確認します
