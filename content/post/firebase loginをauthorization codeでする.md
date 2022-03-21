+++
author = "すかい"
title = "firebase loginをauthorization codeでする"
date = "2018-08-15"
description = "firebase loginをauthorization codeでする"
tags = [
    "Firebase",
]
+++

SSHしてるリモート上でfirebase loginしようとしたらログイン後のコールバックがlocalhostになってるので認証出来ず困ったのでそのメモ

オプションに--no-localhostをつけるとコードベースで認証できる
表示されたURLを適当なマシンで開いてアカウントの認証を済ませるとブラウザ上で認証用のコードが表示されるのでコンソールに貼り付けて終わり

```
$ firebase login --no-localhost
? Allow Firebase to collect anonymous CLI usage and error reporting information? No

Visit this URL on any device to log in:
https://accounts.google.com/o/oauth2/auth[省略]

? Paste authorization code here: [ブラウザに表示されたコードを貼り付ける]

✔  Success! Logged in as [username]@gmail.com
```

## 参考記事

- [Stackoverflow How to login to firebase-tools on headless remote server?](https://stackoverflow.com/questions/43828039/how-to-login-to-firebase-tools-on-headless-remote-server)
