+++
author = "すかい"
title = "C#のソケット通信でクライアントが切断したか調べる"
date = "2016-08-22"
description = "C#のソケット通信でクライアントが切断したか調べる"
tags = [
    "C#",
    "Unity",
]
+++

あとで別記事書きますが簡単なソケット通信サーバーを書いた時に困ったのでメモ
クライアントの切断を検知するのにConnectedプロパティを使えば良さそうだなって思って痛い目にあった
信頼と実績のstackoverflowに解決策がありました

- [How to check if a socket is connected/disconnected in C#?](http://stackoverflow.com/questions/2661764/how-to-check-if-a-socket-is-connected-disconnected-in-c)

## Socketクラスの場合

```csharp
Socket client = listener.EndAcceptSocket(ar);

while (clinet.Connected){
    byte[] bytes = new byte[256];
    client.Receive(bytes);
    string s = Encoding.UTF8.GetString(bytes);

    // 何かする

    if ( client.Poll(1000, SelectMode.SelectRead) && (client.Available == 0) ){
         break;
    }
}
```

こんな感じ

余談ですがwhileの外にstring buf = "";って宣言してその変数に加算代入していくと最初に代入した値しか入ってないんですがバグですかね・・・？
結局TCPClientクラスからNetWorkStream貰ってStreamReaderで取得したんですが

## TCPClientクラスの場合

TcpClientのプロパティにClientがあり、ここからSocketクラスが取得出来るのでほぼ一緒です

```csharp
TcpListener listener = (TcpListener) ar.AsyncState;
TcpClient client = listener.EndAcceptTcpClient(ar);
NetworkStream stream = client.GetStream();
StreamReader reader = new StreamReader (stream);

while (client.Connected) {
    while(!reader.EndOfStream){
        string s = reader.ReadLine();
    }
    if ( client.Client.Poll(1000, SelectMode.SelectRead) && (client.Client.Available == 0) ){
        break;
    }
}
```
