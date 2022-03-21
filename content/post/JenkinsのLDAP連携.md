+++
author = "すかい"
title = "JenkinsのLDAP連携"
date = "2017-04-11"
description = "JenkinsのLDAP連携"
tags = [
    "Docker",
    "Jenkins",
]
+++

## はじめに

前回GitlabをLDAP連携したので今度はJenkinsでもLDAP連携してみる
こちらも例の如くDocker上で動かす

## 環境

- Docker version 17.03.1-ce
- Docker-compose version 1.12.0
- jenkinsci/jenkins:2.54-alpine

## 構築

### Jenkinsの起動

何はともあれdocker-compose.ymlを書く

```yaml
version: "2"

services:
    jenkins:
        restart: always
        image: jenkinsci/jenkins:2.54-alpine
        ports:
        - "50000:50000"
        - "8080:8080"
        volumes:
        - /home/jenkins:/var/jenkins_home
```

実行する

```
$ docker-compose up -d
```

8080ポートをブラウザ上で開いたら初期登録のウィザードが始まるのでウィザードに従っていく
最初のパスワードはブラウザ上で指定された場所に一時キーが書き込まれたファイルがあるので確認する

```
$ docker exec -it jenkins_jenkins_1 bash
# cat /var/jenkins_home/secrets/initialAdminPassword
```

確認したキーをブラウザ上に貼り付けユーザー作成や各種プラグインのインストール等を済ませる
これでひとまず終わり

## LDAP 設定

Jenkinsの管理 -> グローバルセキュリティの設定 と移動する
JenkinsのユーザーデータベースからLDAPに変更し

- サーバー
  LDAP認証を提供しているサーバーを指定する
  例）ldap://192.168.1.123
  [高度な設定]ボタンをクリック
- root DN
  対象となるLDAPグループのDNを入力する
  例）dc=hoge,dc=fuga,dc=com
- User search base
  ユーザーの検索に使われる
  LDAP側の設定に合わせる
  例）ou=people
- User search filter
  ユーザー名の検索に使用される
  LDAP側の設定に合わせる
  例）uid={0}
- 権限管理
  この時点ではまだ「全員に許可」にしておく
  ログイン方式を1つしか選択出来ない？ようなので設定漏れがあった際にログイン出来なくなってしまうため

ここまで設定したら保存する
右上のログインからLDAPユーザーでログインし問題ないことを確認したら権限管理を「ログイン済みユーザー」等に変更する

## トラブル

### コンテナが立ち上がらない

ログを見てみると

```
file.log: Permission denied
```

で権限がないのでコケている
コンテナ内の実行ユーザー（uid:1000）の書き込み権限がないため発生するとのこと
ボリュームに指定したディレクトリの所有者を変更するか権限を出す

```
$ sudo chown 1000 /home/jenkins
```

### 設定が正しく行えずログイン出来なくなった

設定ファイルを編集し先程の設定を無効にしてコンテナを立ち上げ直す

```
$ docker exec -it jenkins_jenkins_1 bash
# vi /var/jenkins_home/config.xml
- <useSecurity>true</useSecurity>
+ <useSecurity>false</useSecurity>
# exit
$ docker-compose up -d
```

あとは再度設定すれば良い

## 参考記事

- [Jenkinsでログイン設定してログインできなくなった時の対処](http://qiita.com/wappy100/items/053fd584cfcd62159082)
