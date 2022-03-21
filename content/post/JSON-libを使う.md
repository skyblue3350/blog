+++
author = "すかい"
title = "JSON-libを使う"
date = "2016-10-16"
description = "JSON-libを使う"
tags = [
    "JSON-lib",
    "Java",
    "NetBeans",
]
+++

## はじめに

前回と同じく使い方で困ったのでメモ

## 環境

- NetBeans 8.2
というか1個前の記事[JAX-RSを試す](../jax-rs%E3%82%92%E8%A9%A6%E3%81%99/)と同じ

## インストール

プロジェクトツリーのプロジェクトファイル→pom.xmlを開く
以下を追加する

```xml
    <dependencies>
        // ここから
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
        // ここまで
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

記述元はここから[Json Lib » 2.4](https://mvnrepository.com/artifact/net.sf.json-lib/json-lib/2.4)
ただこれだけだと動かない

```
<classifier>jdk15</classifier>
```

が要る

- [JSON-lib Frequently Asked Questions](http://json-lib.sourceforge.net/faq.html)

> How do I configure Json-lib as a dependency with Maven2 ?
> As Json-lib comes in two flavors (for the time being) you'll have to add to your dependency declaration, like the following:

だそうです。

最後にメニューの実行→プロジェクトをビルド（F11）からビルドし直す
classifierの記述が抜けてると怒られる

## 使い方

実際に使ってみる

```java
import net.sf.json.JSONArray;

boolean[] boolArray = new boolean[] { true, false, true };
JSONArray jsonArray = JSONArray.fromObject(boolArray);
System.out.println(jsonArray.toString());
```

こんな感じ

## 参考記事

- [[maven2][memo]json-libは普通にdependencyに足すだけだとダウンロードに失敗する件](http://d.hatena.ne.jp/tanamon/20091206/1260106485)
- [JSON-lib Frequently Asked Questions](http://json-lib.sourceforge.net/faq.html)
