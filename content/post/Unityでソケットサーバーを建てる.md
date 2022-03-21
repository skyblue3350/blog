+++
author = "すかい"
title = "Unityでソケットサーバーを建てる"
date = "2016-10-30"
description = "Unityでソケットサーバーを建てる"
tags = [
    "Csharp",
    "Unity",
]
+++

## はじめに

ソケットサーバー作るよ！
Unityでソケット通信したい！って思って調べたらあんまり情報が見つからなかったので泣いた
のでメモ

## バージョン

- Unity 5.3

## コード

幾つかポート開けて見たかったので適当に継承して使うクラスを作ってみた
C#良く分からんので指摘があったら是非
try-catch今書いたからおかしいかもしれない

`SocketServer.cs`

```csharp
using UnityEngine;
using System;
using System.IO;
using System.Net;
using System.Net.Sockets;
using System.Collections.Generic;

public class SocketServer : MonoBehaviour {
    TcpListener listener = null;
    List<TcpClient> clients = new List<TcpClient>();
    readonly object lockObj = new object();

    protected void listen(string host, int port){
        IPAddress ip = IPAddress.Parse(host);
        listener = new TcpListener(ip, port);
        listener.Start();
        listener.BeginAcceptSocket(OnRequested, listener);
    }

    public void sendMessage(string msg){
        if (clients.Count == 0){
            return;
        }
        byte[] body = System.Text.Encoding.UTF8.GetBytes(msg);

        foreach(var client in clients){
            try{
                    NetworkStream stream = client.GetStream();
                    stream.Write(body, 0, body.Length);
            }catch (Exception e){
                    clients.Remove(client);
            }
        }
    }

    protected virtual void OnApplicationQuit() {
        if (listener == null){
            return;
        }

        if (clients.Count != 0){
            foreach(var client in clients){
                client.Close();
            }
        }

        listener.Stop();

	}

    protected virtual void OnMessage(string msg){
        // Debug.Log(msg);
    }

    void OnRequested(IAsyncResult ar) {
		lock (lockObj) {
			TcpListener listener = (TcpListener)ar.AsyncState;
			TcpClient client = listener.EndAcceptTcpClient(ar);
            clients.Add(client);
			NetworkStream stream = client.GetStream();

			StreamReader reader = new StreamReader(stream);

			while (client.Connected) {
				while (!reader.EndOfStream){
					OnMessage(reader.ReadLine());
				}

				if (client.Client.Poll(1000, SelectMode.SelectRead) && (client.Client.Available == 0)) {
					clients.Remove(client);
					break;
				}
			}
			listener.BeginAcceptSocket(OnRequested, listener);
		}
	}
}
```

## 使い方

これを継承したクラスを作る

`Server.cs`

```csharp
public class Server : SocketServer {
    public string host = "127.0.0.1";
    public int port = 9090;

    void start(){
        base.listen(host, port);
    }

    protected override void OnMessage(string msg){
        Debug.Log(msg);
        sendMessage(msg);
    }
}
```

みたいな感じで使える
