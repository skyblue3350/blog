+++
author = "すかい"
title = "JAX-RSを試す"
date = "2016-10-16"
description = "JAX-RSを試す"
tags = [
    "JAX-RS",
    "Java",
    "NetBeans",
]
+++

## はじめに

NetBeansでJAX-RSを動かすところまでメモ

## 環境

- NetBeans 8.2 EE版
  https://netbeans.org/downloads/?pagelang=ja
- GlassFish 4.1.1
  上のに付属してるやつ

## プロジェクト作成

1. プロジェクトを選択
  ファイル→新規プロジェクト→Maven→Webアプリケーション
2. 名前と場所
  設定値は適当に決める
  この記事の値は全部デフォルト値で作成した時の話
3. 設定
  サーバー：GlassFish Server4.1.1
  JavaEEバージョン：JavaEE 7 Web

終了押してプロジェクトを作成する

## 実装

左側のプロジェクトツリーのソース・パッケージの欄にある先ほど作成したパッケージ名の上で右クリック
新規→Javaクラス
WebApplicationとかで作成
中身を

```java
package com.mycompany.mavenproject1;
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/")
public class MyWebApplication extends Application {
}
```

適当にもう1個クラスを作成する

```java
package com.mycompany.mavenproject1;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("test")
public class Rest {
    @GET
    public String helloWorld() {
        return "Hello World!!";
    }
}
```

## 実行

ここまで作ったら保存して実行する
メニューの実行→プロジェクトを実行（F6）

勝手にブラウザが開いて
http://localhost:8080/mavenproject1/
とかに飛ばされる
http://localhost:8080/mavenproject1/test
とかに飛んでみて動作するか確認する

おわり

## メモ

web.xmlは要らなかった
要るみたいな記事がぼちぼちあって要るものとばっかり思ってた

## 参考記事

- [JavaEE 環境で JAX-RS の Hello World](http://qiita.com/opengl-8080/items/f6785cf3d732682ca7dd)
