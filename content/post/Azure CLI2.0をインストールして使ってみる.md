+++
author = "すかい"
title = "Azure CLI2.0をインストールして使ってみる"
date = "2017-11-25"
description = "Azure CLI2.0をインストールして使ってみる"
tags = [
    "Azure",
    "cygwin",
]
+++

## はじめに

Azure CLIを使いたい用事が出来たのでcygwinにAzure CLIを入れたのでその時のメモ
一般のディストリビューションならパッケージが提供されてるのでそっちをインストールした方が遥かに楽です
Storageを取得するところまでやります

## 環境

- cygwin
  apt-cygが使える前提です

## 前提パッケージのインストール

```
$ apt-cyg install python2-devel libffi-devel openssl-devel
```

## CLIのインストール

```
$ curl -L https://aka.ms/InstallAzureCli | bash
```

待ってるとインストールが終わります

```
$ az -v
azure-cli (2.0.21)
```

## ログイン

対話式でログインしてみます
表示されたURLにアクセスしてアクセスコード（例ではAB12C34DE）を入力した後に使用するアカウントでログインします．
ブラウザ上で一通り作業が終了するとログイン情報のjsonが表示されます

```
$ az login
To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code AB12C34DE to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "id": "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "isDefault": true,
    "name": "無料試用版",
    "state": "Enabled",
    "tenantId": "xxxxxxxx-0000-0000-xxxx-xxxxxxxxxx",
    "user": {
      "name": "example@example.com",
      "type": "user"
    }
  }
]
```

他にもユーザー名とパスワードによるログイン（MFAあるとダメっぽい）とかサービス プリンシパルによるログインが出来るみたいです．

```
$ az login -u johndoe@contoso.com -p VerySecret
$ az login --service-principal -u http://azure-cli-2016-08-05-14-31-15 -p VerySecret --tenant contoso.onmicrosoft.com
```

## ファイルの一覧の取得

コマンドで作成することも出来ますが予めWebの管理画面からストレージを作成して数ファイル置いておきました．
今回は

- ストレージアカウント
sample
- コンテナ
example

とします．

```
$ az storage blob list --account-name sample --container-name example
[
  {
    "content": null,
    "metadata": null,
    "name": "cat.jpg",
    "properties": {
      "AccessTierInferred": "true",
～以下略～
```

見辛いのでjqコマンドでファイル名だけ抽出します．

```
$ az storage blob list --account-name sample --container-name example | jq -r ".[].name"
cat.jpg
wine.jpg
```

取れました

## ファイルのダウンロード

cat.jpgをローカルにdownload_cat.jpgと名前をつけて保存する

```
$ az storage blob download --account-name sample --container-name example --name cat.jpg --file download.cat.jpg
$ ls
download_cat.jpg
```

良い感じ

## 参考記事

- [Azure CLI 2.0 のインストール](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Storage での Azure CLI 2.0 の使用](https://docs.microsoft.com/ja-jp/azure/storage/common/storage-azure-cli)
