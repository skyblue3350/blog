+++
author = "すかい"
title = "sharelatexでjarticleをつかえるようにする"
date = "2017-02-19"
description = "sharelatexでjarticleをつかえるようにする"
tags = [
    "Docker",
    "sharelatex",
]
+++

## ShareLatex とは

Web上でLatexが書けるオープンソースプロジェクト
Webサービスとしても提供されてるけどオープンソースなので自前のサーバーで動かすこともできる
ただCloud LaTeXと違ってjarticleが使えないのでその辺もなんとかします
大体参考元記事のままです

~~というか大人しくDockerfile書いた方が早いです~~

## 環境

- Docker
1.13.1 build 092cba3
- Docker-compose
1.11.1 build 7c5d5e4

## 環境構築

### イメージの用意

公式がDockerイメージを提供しているのでこちらを利用します

```
$ mkdir sharelatex
$ cd sharelatex
$ wget https://raw.githubusercontent.com/sharelatex/sharelatex/master/docker-compose.yml
$ docker-compose up -d
```

80番にフォワーディングされてるので一度アクセスして動作することを確認します
起動直後は502 bad gatewayが出ますがしばらく待てばログイン画面がでます

### アカウント設定

docker execで管理用アカウントのIDを発行します

```
$ docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email joe@example.com"
```

登録用 URL
登録用URLにアクセスしてパスワードを設定しサンプルプロジェクトを作って問題なく動作することを確認します

### イメージの編集

コンテナ内に入って作業します

```
$ docker exec -it sharelatex bash
```

#### platexのインストール

何はともあれplatexがないことには始まらないのでインストールします

```
# apt-add-repository ppa:texlive-backports/ppa
# apt-get install texlive-lang-cjk
```

ここまででかなり時間かかるのでここで一度適当にコンテナをcommitしておくと失敗した時のダメージが軽くなります

#### latexmkの編集

```
# vi /usr/local/texlive/2016/bin/x86_64-linux/latexmk
-$latex  = 'latex %O %S';
+$latex  = 'platex -shell-escape %O %S';

-$bibtex  = 'bibtex %O %B';
+$bibtex  = 'pbibtex %O %B';

-$dvipdf  = 'dvipdf %O %S %D';
+$dvipdf  = 'dvipdfmx %O -o %D %S';
```

#### clsiの編集

参考サイトまんまです

```
# vi /var/www/sharelatex/clsi/app/coffee/LatexRunner.coffee
-               else if compiler == "xelatex"
-                       command = LatexRunner._xelatexCommand mainFile
+               else if compiler == "platex"
+                       command = LatexRunner._platexCommand mainFile

-       _xelatexCommand: (mainFile) ->
+       _platexCommand: (mainFile) ->
                LatexRunner._latexmkBaseCommand.concat [
-                       "-xelatex", "-e", "$pdflatex='xelatex -synctex=1 -interaction=batchmode %O %S'",
+                       "-pdfdvi", "-e", "$latex='platex -synctex=1 -interaction=batchmode %O %S'",
```

```
# vi /var/www/sharelatex/clsi/app/coffee/RequestParser.coffee
-      VALID_COMPILERS: ["pdflatex", "latex", "xelatex", "lualatex"]
+      VALID_COMPILERS: ["pdflatex", "latex", "platex", "lualatex"]
```

#### web の編集

```
# vi /var/www/sharelatex/web/app/coffee/Features/Project/ProjectOptionsHandler.coffee
-       VALID_COMPILERS: ["pdflatex", "latex", "xelatex", "lualatex"]
+       VALID_COMPILERS: ["pdflatex", "latex", "platex", "lualatex"]
```
```
# vi /var/www/sharelatex/web/app/views/project/editor/left-menu.pug
-                                       option(value='xelatex') XeLaTeX
+                                       option(value='platex') platex
```

#### 変更の適用

足りないパッケージのインストールとgrunt install（不要？）します

```
# apt-get install libkrb5-dev
# grunt install
```

一度コンテナから抜けコンテナ停止後イメージとして書き出します

```
# exit
$ docker stop sharelatex redis mongo
redis
mongo
sharelatex
$ docker commit sharelatex custom_sharelatex
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
custom_sharelatex       lastest             xxxxxxxxx         2 days ago          4.01 GB
mongo                   latest              xxxxxxxxx         9 days ago          402 MB
sharelatex/sharelatex   latest              xxxxxxxxx         11 days ago         2.39 GB
redis                   latest              xxxxxxxxx         2 weeks ago         184 MB
```

platex入れたりでサイズが大きくなり過ぎなので後で要対策です

先程作成したイメージを利用するようにcomposeファイルを編集します

```
$ vi docker-compose.yml
version: '2'
services:
    sharelatex:
        restart: always
        image: custom_sharelatex
```

これで終わり
再度起動します

```
$ docker-compose up -d
```

してしばらく待つと起動してくるので試しにプロジェクトを作成し
Menu -> Compilerから先程作成したらplatexを選択して文章を書いてテストします

```latex
\documentclass{jarticle}
\begin{document}
これはサンプルドキュメントです。
\section{ほげほげ}

\end{document}
```

RecompileするとPDFが出来ました

![](/images/2017-02-19-001.png)

## 参考記事

- [ShareLatexでjarticleを使えるようにする](http://qiita.com/ishigaki/items/3191e1a330dfb4e11289)
- [日本語対応ShareLatexを自前サーバーで運用する](http://ken1row.net/archives/47)
- [sharelatex/sharelatex Dependencies](https://github.com/sharelatex/sharelatex/wiki/Dependencies)
- [sharelatex/sharelatex-docker-image](https://github.com/sharelatex/sharelatex-docker-image)
