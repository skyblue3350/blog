+++
author = "すかい"
title = "Gitlab pagesをサブディレクトリで公開する"
date = "2017-06-25"
description = "Gitlab pagesをサブディレクトリで公開する"
tags = [
    "Gitlab",
]
+++

## はじめに

Gitlab CE版でも8.17辺りからGitlab Pagesが使えます．
前提としてGitLab Runnerが使える必要があります．
※そちらの設定も詰まったのでぼちぼち時間ある時に書きます．

本来Gitlab pagesはサブドメイン上で利用される機能ですが今回これを諸般の事情からサブディレクトリで動かします．
今回も使うのはsameersbn氏のsameersbn/docker-gitlabです．

## 検証環境

- docker
  Docker version 17.03.1-ce, build c6d412e
- docker-compose
  docker-compose version 1.13.0, build 1719ceb
- Gitlab
  9.2.7 CE
- Gitlab Runner（Dockerイメージを使用）
  gitlab/gitlab-runner:v9.2.0


## 設定方法

### pages の設定

docker-compose.ymlに以下を追記します．

```
    - GITLAB_PAGES_ENABLED=true
    - GITLAB_PAGES_DOMAIN=pages.hoge.jp
    - GITLAB_PAGES_HTTPS=false
    - GITLAB_PAGES_PORT=80
```

本来であれば*.pages.hoge.jpを本コンテナまでリバースプロキシで繋げば終わりです.
今回は適当な名前をつけておきます.

### DNSの設定

内向きだけでいいので
*.pages.hoge.jp
をコンテナのホストのIPを向くように設定します.

### リバースプロキシの設定

今回はサブドメイン上ではなくサブディレクトリ上で公開します.
本来であればhttp://username.domain/projectname/で公開されるものをhttp://domain/pages/username/projectname/で公開します.
リバースプロキシを以下のように設定します.

どうするのがお作法的に良いのかわからないのでひとまず正規表現でゴリ押しな感じになってます.

```
    location ~ ^/pages/(?<user>[\w-]+)/(?<project>[\w-]+)(?<path>\S*)$ {
        resolver 192.xxx.xxx.xxx;
        proxy_redirect off;
        proxy_set_header Host $user.pages.hoge.jp;
        proxy_pass http://$user.pages.hoge.jp:10080/$project/$path;
    }
```

### タスクの登録

予めタスクランナーをプロジェクトで有効にしておくか新規登録しておいて下さい．
プロジェクトのルートディレクトリに以下の.gitlab-ci.ymlを配置します．
WebUIから簡単に作成出来ます．
Webでプロジェクトを開いてNew File -> Template -> .gitlab-ci.yml -> HTMLと選択すると以下のコードが自動で生成されます．

```
# This file is a template, and might need editing before it works on your project.
# Full project: https://gitlab.com/pages/plain-html
pages:
  stage: deploy
  script:
  - mkdir .public
  - cp -r * .public
  - mv .public public
  artifacts:
    paths:
    - public
  only:
  - master
```

これでmasterブランチにコミットすると自動でレポジトリ内のデータを公開する仕組みが出来ました．
テスト用にindex.htmlをプロジェクトのルートに作成しておきます．

### タスクの実行

手動で実行するかmasterブランチにコミットした時点でタスクが実行されます．
.gitlab-ci.ymlをコミットした時点でpipelinesに登録したタスクが実行されているかと思います．

![](/images/2017-06-25-002.png)

### テスト

http://domain/username/projectname/index.html
にアクセスしてみてれればおっけー

## Q&A

### 動作確認どうやったの

Hostヘッダを見ているのでcurlでヘッダを付けた上で検証しました．

```
$ curl -H "Host: pages.hoge.jp" コンテナホストIP:10080/username/projectname
```

### ちゃんとデプロイされてるのか知りたい

$GITLAB_PAGES_DIRで指定したディレクトリにデータがあれば大丈夫です．
デフォルトは$GITLAB_SHARED_DIR/pagesです．
$GITLAB_SHARED_DIRのデフォルト値は/home/git/data/sharedなので，
/home/git/data/shared/pages/username/projectnameにデータが上がっているはずです．

### デバッグどうやったの

#### フロント側

- [[備忘録]nginxでデバッグ出力](http://qiita.com/cyclon2joker/items/c55eeb4bec9782f31264)

フロント側のNginxはこちらの記事の方法でまず正規表現の動作確認やらしました．

#### バックエンド側

コンテナ内nginxのログは/var/log/gitlab/nginx/gitlab_pages_access.logにアクセスが来るのでそこを見ます．

```
$ docker-compose exec gitlab bash
# less -F /var/log/gitlab/nginx/gitlab_pages_access.log
```

## 参考記事

- [variable capture in Nginx location matching](https://stackoverflow.com/questions/13706658/variable-capture-in-nginx-location-matching)
- [nginx 'proxy_pass' cannot have URI part in location?](https://stackoverflow.com/questions/21662940/nginx-proxy-pass-cannot-have-uri-part-in-location)

## 最後に

```
# nginx -t && service nginx restart
```

テスト環境とはいえいきなりリスタートするよりこうした方が精神衛生上良い（今更

それはさておきひとまずRunnerと連携したページ公開が出来るようになったので次はSphinxのレポジトリを置いてコミット->自動ビルド->自動公開までいけるといいな
