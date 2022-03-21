+++
author = "すかい"
title = "Google Cloud Datastore Emulatorを使う"
date = "2018-08-18"
description = "Google Cloud Datastore Emulatorを使う"
tags = [
    "GCP",
    "Google Cloud Database",
]
+++

Google Cloud Datastore Emulatorを使うことでクラウド上のDatastoreを使わずにローカルでDatastoreを建てて開発することができるようになります．
Firebase Datastoreも実質これなので使ってみました．

## 環境

```
$ gcloud version
Google Cloud SDK 212.0.0
beta 2018.07.16
bq 2.0.34
cloud-datastore-emulator 2.0.1
core 2018.08.13
gsutil 4.33
kubectl
$ firebase -V
4.1.0
```

## 環境構築

前提パッケージのjreを入れておきます．

```
$ sudo apt install openjdk-8-jre
```

Google Cloud Datastore Emulatorをインストールします．

```
$ gcloud components install cloud-datastore-emulator
```

## 使い方

gcloudでデフォルトプロジェクトを指定していない場合は``--project```でプロジェクトを指定しておきます．

```
$ gcloud beta emulators datastore start --project hoge
Executing: /Google/Cloud SDK/google-cloud-sdk/platform/cloud-datastore-emulator/cloud_datastore_emulator create --project_id=hoge ~/.config/gcloud/emulators/datastore
[datastore] Aug 17, 2018 7:03:38 PM com.google.cloud.datastore.emulator.CloudDatastore$CreateAction$1 apply
[datastore] INFO: Provided project_id to Cloud Datastore emulator create command, which is no longer necessary.
[datastore] Created new Cloud Datastore project in '~/.config/gcloud/emulators/datastore'.
Executing: /Google/Cloud SDK/google-cloud-sdk/platform/cloud-datastore-emulator/cloud_datastore_emulator start --host=localhost --port=8081 --store_on_disk=True --consistency=0.9 --allow_remote_shutdown ~/.config/gcloud/emulators/datastore
[datastore] Aug 17, 2018 7:03:39 PM com.google.cloud.datastore.emulator.CloudDatastore$FakeDatastoreAction$8 apply
[datastore] INFO: Provided --allow_remote_shutdown to start command which is no longer necessary.
[datastore] Aug 17, 2018 7:03:40 PM com.google.cloud.datastore.emulator.impl.LocalDatastoreFileStub <init>
[datastore] INFO: Local Datastore initialized:
[datastore]     Type: High Replication
[datastore]     Storage: ~/.config/gcloud/emulators/datastore/WEB-INF/appengine-generated/local_db.bin
[datastore] Aug 17, 2018 7:03:40 PM com.google.cloud.datastore.emulator.impl.LocalDatastoreFileStub load
[datastore] INFO: The backing store, ~/.config/gcloud/emulators/datastore/WEB-INF/appengine-generated/local_db.bin, does not exist. It will be created.
[datastore] Aug 17, 2018 7:03:40 PM io.gapi.emulators.netty.NettyUtil applyJava7LongHostnameWorkaround
[datastore] INFO: Applied Java 7 long hostname workaround.
[datastore] API endpoint: http://localhost:8081
[datastore] If you are using a library that supports the DATASTORE_EMULATOR_HOST environment variable, run:
[datastore]
[datastore]   export DATASTORE_EMULATOR_HOST=localhost:8081
[datastore]
[datastore] Dev App Server is now running.
```

ログにもある通り環境変数を設定するとエミュレータ環境を見るようになります．
自動で環境変数を生成してくれるコマンドがあるので使います．

```
$ $(gcloud beta emulators datastore env-init)
```

あとは普通にdatastoreに接続するとエミュレータ環境にアクセスすれば良いです
ただjsonとか使ってると環境変数を見ないので注意します

```py
from google.cloud import datastore

# こちらだとクラウド側にアクセスしてしまう
# clinet = datastore.Client.from_service_account_json("/path/to/json", project="hoge")
client = datastore.Client()
key = client.key("sample")

entity = datastore.Entity(key=key)
entity["created"] = datetime.now()
entity["name"] = "taro"
client.put(entity)
```
