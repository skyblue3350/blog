+++
author = "すかい"
title = "postfixでGmailのリレー設定"
date = "2017-01-01"
description = "postfixでGmailのリレー設定"
tags = [
    "Ubuntu",
]
+++

## はじめに

システムのメール送信をGmailでリレーして簡単に送信出来るように設定しておく
mailコマンドでメールが送れるようになるので手軽にログを送ったりするのに使える
この手の記事はたくさんあるがうまくいくものがなかなかなかったのでメモ

## 環境

- Ubuntu 16.04.1 LTS 64bit

## インストール

postfixとmailをコマンドを使うためにmailutilsのパッケージを導入する

```
$ sudo apt-get install postfix mailutils
```

インストール中のpostfixの質問では「設定なし」を選択する

## 設定

### gmail の事前設定

Gmailではアプリケーション側がパスワードを持つようなアプリケーションはそのままでは使えないのでGmail側の設定を変更する
https://www.google.com/settings/u/2/security/lesssecureapps
からアクセスをオンにしておく

### コンフィグファイルの作成

main.cfを新たに作成する

```
$ cat /etc/postfix/main.cf
relayhost = smtp.gmail.com:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
```

### ログイン情報の作成

```
$ sudo vi /etc/postfix/sasl/sasl_passwd
smtp.gmail.com:587 hoge@gmail.com:password
```

使用するユーザー名とパスワードを入力する

これをハッシュ化する 元ファイルは不要なので削除する

```
$ sudo postmap /etc/postfix/sasl/sasl_passwd
$ sudo rm /etc/postfix/sasl/sasl_passwd
```

### テスト

mailコマンドを使用してメールの送信を確認する

```
$ mail target@example.com
CC:
Subject:Test
test
[Ctrl+D]
$ mailq
Mail queue is empty
```

target@example.comのメールボックスに送信されてれば問題なし

## トラブルシューティング

mailqと/var/mail/usernameを確認する
エラーコードから原因を調べる

### 530 5.7.0

smtp_use_tls = yes
の設定がない場合に出る

### 530-5.5.1

https://www.google.com/settings/u/2/security/lesssecureapps
からアクセスをオンに変更する
