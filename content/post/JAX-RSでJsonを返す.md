+++
author = "すかい"
title = "JAX-RSでJsonを返す"
date = "2016-11-14"
description = "JAX-RSでJsonを返す"
tags = [
    "JAX-RS",
    "Java",
    "NetBeans",
]
+++

## はじめに

JAX-RSで

```java
@Path("test")
@GET
public Map hoge(){
    Map<String, String> data = new HashMap<>();
    data.put("key", "value");
    return data;
}
```

とかしたい時の話

## 依存性の追加

jersey-serverとjersey-jsonを追加する
追加してからメニューの実行->プロジェクトをビルド（F11）する
pom.xmlに追記するか依存性の追加から

```xml
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-server</artifactId>
    <version>1.8</version>
</dependency>
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-json</artifactId>
    <version>1.8</version>
</dependency>
```

## コード

冒頭と同じだけど

`WebApplication.java`

```java
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/api")
public class WebApplication extends Application {
}
```

`Hoge.java`

```java
@Path("/")
public class Hoge {
    @Path("test")
    @GET
    public Map hoge(){
        Map<String, String> data = new HashMap<>();
        data.put("key", "value");
        return data;
    }
}
```

してテスト　おわり
