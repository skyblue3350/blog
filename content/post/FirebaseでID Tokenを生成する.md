+++
author = "すかい"
title = "FirebaseでID Tokenを生成する"
date = "2018-08-29"
description = "FirebaseでID Tokenを生成する"
tags = [
    "Firebase",
]
+++

FirebaseでID Tokenを生成する方法です．
Firebase Functionsで認証の仕組みを使った時にこれフロント作らないとデバッグ出来ないのか…？みたいな気持ちになったので生成方法メモしておきます．

## 必要なもの

- ウェブ API キー
プロジェクトの設定の全般から取得できます
- サービスアカウントの認証情報
プロジェクトの設定のサービスアカウントから新しい秘密鍵の生成で持ってこれます
- ログインしたいユーザーのuid
プロジェクトのAuthenticationのユーザーuidから持ってきます

Functionsはこんな感じのなんか認証情報でアクセス許可したりしなかったりする感じ

```js
app.use((req, res, next) => {
  const idToken = req.headers.authorization.replace("Bearer ", "");
  const decodedToken = await admin.auth().verifyIdToken(idToken);
  if (よしなに判定){
    // 認証失敗
    res.status(401).send("認証いるよ");
  } else {
    next();
  }
});
```

ここでBearerヘッダに入れるidTokenってどうやって入手するの…？となるわけですがフロントがあるなら
ログイン認証後に

```js
const idtoken = await firebase.auth().currentUser.getIdToken();
```

とかやると取れます（多分

ただフロントは今回全く書いてなかったので使わずに生成する方法です．
まずカスタムトークンが必要になるのでこれを生成します．
https://firebase.google.com/docs/auth/admin/create-custom-tokens?hl=ja
この辺に各言語でのやり方が書いてありますが今回はPythonで説明します．

以下のようなコードを使います．
必要部分は事前に取得したもので埋めて下さい．

```py
import firebase_admin
from firebase_admin import credentials
from firebase_admin import auth

# 管理コンソールのサービスアカウントから取得する
cred = credentials.Certificate("service_account.json")
firebase_admin.initialize_app(cred)

# 管理コンソールのアカウント一覧から取得する
uid = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# ユーザー情報の取得
user = auth.get_user(uid)
print(user.uid, user.email)

# カスタムトークンの生成
token = auth.create_custom_token(uid)

print(token)
# Firebase Auth REST APIを叩いてID Tokenを生成
token = input(">>")

# ユーザー情報の取得
user = auth.verify_id_token(token)
print(user)
```

カスタムトークンが表示されたらFirebase Auth Rest APIを使います．
https://firebase.google.com/docs/reference/rest/auth/

```
$ curl 'https://www.googleapis.com/identitytoolkit/v3/relyingparty/verifyCustomToken?key=[API_KEY]' \
-H 'Content-Type: application/json' \
--data-binary '{"token":"[Pythonで取得したカスタムトークン]","returnSecureToken":true}'
```

レスポンスにあるidTokenが目的のものです．
コンソールでPythonが入力待ちになってるのでここに貼り付けると使えるはずです．

全部Pythonで書けば良かったですけど取り急ぎ試すだけ試したかったので雑ですが以上
