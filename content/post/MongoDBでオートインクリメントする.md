+++
author = "すかい"
title = "MongoDBでオートインクリメントする"
date = "2016-11-15"
description = "MongoDBでオートインクリメントする"
tags = [
    "Java",
]
+++

## はじめに

MongoDBの_idにはオリジナルなObjectIDがついてるけどこれをMySQLとかでいうようなオートインクリメントさせたい時の話
調べたら公式にドキュメントがあった

- [Create an Auto-Incrementing Sequence Field](https://docs.mongodb.com/v3.0/tutorial/create-an-auto-incrementing-field/)

のでこれをJavaで使ってみる

## ライブラリの追加

3.3.0のドライバを追加しておく

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.3.0</version>
</dependency>
```

## countersコレクションの作成

参考ドキュメントの1番相当
コンソールからmongoDBにアクセスしてコレクションを予め作成しておく

```
$ mongo
> db.counters.insert({_id: "userid", seq: 0})
WriteResult({ "nInserted" : 1 })
```

## getNextSequenceの作成

参考ドキュメントの2番相当

```java
    public Object getNextSequence(DB db){
        DBCollection counter = db.getCollection("counters");
        BasicDBObject find = new BasicDBObject();
        find.put("_id", "userid");
        BasicDBObject update = new BasicDBObject();
        update.put("$inc", new BasicDBObject("seq", 1));
        DBObject obj =  counter.findAndModify(find, update);

        return obj.get("seq");
    }
```

## 利用する

利用する時は上で作った関数を呼んでその結果を当該ドキュメントの_idとして扱うだけで良い
もしカウンタの値をリセットする時はdropして最初のinsertをもう1回すれば良い

```java
MongoClient client = new MongoClient("127.0.0.1", 27017);
DB db = client.getDB("hoge");

BasicDBObject document = new BasicDBObject();
document.put("_id", getNextSequence(db));
document.put("name", "ななし");
document.put("age",  10);

DBCollection todolist = db.getCollection("hogehoge");
todolist.insert(document);
```

## 参考記事

- [Create an Auto-Incrementing Sequence Field](https://docs.mongodb.com/v3.0/tutorial/create-an-auto-incrementing-field/)
- [Auto increment sequence in mongodb using java](http://stackoverflow.com/questions/32065045/auto-increment-sequence-in-mongodb-using-java)
