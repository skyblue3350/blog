+++
author = "すかい"
title = "Django Rest Frameworkでソーシャルログインを実装する"
date = "2019-06-02"
description = "Django Rest Frameworkでソーシャルログインを実装する"
tags = [
    "Django",
    "Python",
]
+++

Django Rest Framework（以降DRF）でソーシャルログインを実装したい時の話
色々試したものの上手いこと設定出来ず手こずったのでまとめ

[DRFのドキュメント](https://django-rest-auth.readthedocs.io/en/latest/installation.html#registration-optional)にあるソーシャルログインを試しました．
今回はGithubでテストしました．

## 環境

- Python 3.7.3
- パッケージ

```
$ pip list
Django                        2.2.1
django-allauth                0.39.1
django-rest-auth              0.9.5
djangorestframework           3.9.4
```

## 設定

DRFが動くところまでは済んでる前提です．
上記のパッケージのインストール

```
$ pip install django-allauth django-rest-auth
```

settings.pyにインストールしたアプリを追加

```
INSTALLED_APPS = (
    ...,
    'django.contrib.sites', # 設定してない場合追加しておく
    'rest_framework',
    'rest_framework.authtoken',
    'rest_auth'
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.github',
)
SITE_ID = 1 # 同上
```

データベースに反映します

```
$ python manage.py migrate
```

Github用の認証ビューを作ります

```py
class GithubLogin(SocialLoginView):
    adapter_class = GitHubOAuth2Adapter
    callback_url = "Githubのアプリ側で設定したコールバックURLと合わせる"
    client_class = OAuth2Client
```

適当なURLと紐づけます

```py
urlpatterns = [
    url(r'^rest-auth/github/$', GithubLogin.as_view(), name='github_login'),
]
```

Djangoの管理画面に入ってSocial appsから新しくGithubを追加します．
名称は小文字で設定します．
その他はGithubで設定した値をコピペします．

今回フロントはないのでGithubのコールバックは適当なURLにしておきました．
https://github.com/login/oauth/authorize?client_id=[作成時のクライアントID]&scope=login
にアクセスするとGithub上でログインし，アクセス許可を出すと設定したコールバックURLに向かって飛ばされます
例）http://localhost:8000/auth/complete/github/?code=01234abcde
この末尾についているcodeを先程設定したビューに投げつけるとソーシャルログイン出来ます．

```
$ curl -H 'Content-Type:application/json' -d "{"code":"01234abcde"}" http://localhost:8000/rest-auth/github/
{
    "key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```
