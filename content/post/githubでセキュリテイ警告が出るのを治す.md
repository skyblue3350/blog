+++
author = "すかい"
title = "githubでセキュリテイ警告が出るのを治す"
date = "2018-03-18"
description = "githubでセキュリテイ警告が出るのを治す"
tags = [
    "JavaScript",
]
+++

少し前に作ったFlaskとReactで作ったWebアプリのテンプレートで依存してるライブラリにセキュリテイの脆弱性があるってgithubから通知が来たので直した

npm lsすると依存関係が見えるのでそこから探す
なにか良い探し方あるのかもしれないけど調べるより前に前行の表示でいけばいけるか…と思って実行してしまったのでもっと良い調べ方があるのかもしれない

```
$ npm ls --depth=2 | grep ssri -B 15
| | +-- prop-types@15.6.0 deduped
| | `-- warning@3.0.0 deduped
| `-- warning@3.0.0 deduped
+-- uglifyjs-webpack-plugin@1.1.8
| +-- cacache@10.0.2
| | +-- bluebird@3.5.1
| | +-- chownr@1.0.1
| | +-- glob@7.1.2
| | +-- graceful-fs@4.1.11
| | +-- lru-cache@4.1.1
| | +-- mississippi@1.3.1
| | +-- mkdirp@0.5.1 deduped
| | +-- move-concurrently@1.0.1
| | +-- promise-inflight@1.0.1
| | +-- rimraf@2.6.2
| | +-- ssri@5.2.1
```

uglifyjs-webpack-pluginの依存関係なのがわかったのでこれをアップデートしておわり

```
$ npm install --save-dev uglifyjs-webpack-plugin
```

テンプレートなのでテストもクソもないけど一応問題なさそうだった
あとは追加して終わり

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
        modified:   package-lock.json
        modified:   package.json
$ git add package*
$ git commit -m "パッケージアップデート"
$ git push origin master
レポジトリのトップページに行って消えたのを確認して終わり
```
