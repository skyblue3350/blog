+++
author = "すかい"
title = "Gitlabを8系から10系までアップデートする"
date = "2018-04-16"
description = "Gitlabを8系から10系までアップデートする"
tags = [
    "Gitlab",
]
+++

## はじめに

手元にsameersbn氏作成のGitlabの環境が2つあって双方ともアップデートするの忘れてたので最新版まであげる
現在の最新リリースが10.6.4なのでここまであげる
ダウンタイムはある程度生まれるけど1秒落ちたら殺される環境でもないので許容する

## 環境

以下2環境が動いてる

- 8.17.4
- 9.2.7

## バックアップ

定時バックアップしてたら不要ですが片方別の方法でバックアップしてたので手動でバックアップ取る
この段階で一応Gitlabを落とした方が多分良いけど落とさなくてもできました（ドキュメントにも記述あるけど多分落とした方が安全です

```
$ docker-compose down gitlab
$ docker-compose run --rm gitlab app:rake gitlab:backup:create
```

## アップデート

image書き換えたら勝手にやってくれます　楽ですね
一応以下を確認してバージョン間の問題があるか事前に調べます
https://github.com/sameersbn/docker-gitlab#upgrading
今回のバージョンは問題なさそうなので最新までそのままあげます
ダメな場合は間のバージョンに一度上げてから最新まであげれば大丈夫です
docker-compose.ymlを以下のように書き換えます

```
-    image: sameersbn/gitlab:8.17.4
+    image: sameersbn/gitlab:10.6.4
$ docker-compose up -d
docker-compose logs -f gitlab
gitlab_1      | Migrating database...
gitlab_1      | Missing Rails.application.secrets.jws_private_key for production environment. The secret will be generated and stored in config/secrets.yml.
gitlab_1      | Recompiling assets (relative_url in use), this could take a while...
```

イメージが変わると多分この辺でしばらく時間が掛かって止まるので待ちます
数分待ってると

```
gitlab_1      | Clearing cache...
gitlab_1      | 2018-04-14 03:25:29,401 CRIT Supervisor running as root (no user in config file)
```

とそれぞれのプロセスが上がり始めるので動作チェックして問題なさそうなら終わりです

## レストア

だいぶ前にバックアップを手動で取ったら上手くいかなかったことがあったので検証兼ねてレストアもテストしたのでメモ
作成したバックアップと使っているdocker-compose.ymlを検証環境に運びます
一度普通にコンテナを起動して新規にクリーンな環境を作ります

```
$ docker-compose up -d
```

起動終了後にGitlabへアクセスするとrootユーザーのパスワードの設定画面が表示されるので設定します
そこまで終わったらバックアップファイルをクリーンなGitlabのディレクトリに置いてレストアします

```
$ docker-compose stop gitlab
$ mv /path/to/backup ./gitlab/backup/
$ docker-compose run --rm gitlab app:rake gitlab:backup:restore
```

レストア可能なバックアップが一覧表示されるので選びます（クリーンな環境なので運んできたやつしかないはず）
既にあるテーブルを消すか聞かれるので消した上でレストアします
これで元の環境と同じ環境が上がっていれば問題なくバックアップできてるのでなにかあった時はこの手順で元に戻せます
