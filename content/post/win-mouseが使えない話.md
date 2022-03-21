+++
author = "すかい"
title = "win-mouseが使えない話"
date = "2016-11-01"
description = "win-mouseが使えない話"
tags = [
    "Node.js",
]
+++

**未解決です**

## はじめに

Electronでウィジェットを作ってる時に画面全体をドラッグで移動＋クリックで別の動作をしたいみたいな事例

まず普通にElectron側
フレームなしの背景透過状態にする

```js
"use strict";

const electron = require("electron");
const app = electron.app;

const BrowserWindow = electron.BrowserWindow;
let mainWindow;

app.on("window-all-closed", function() {
    if (process.platform != "darwin") {
        app.quit();
    }
});

app.on("ready", function() {
    mainWindow = new BrowserWindow({
        width: 400,
        height: 600,
        transparent: true,
        frame: false,
    });

    mainWindow.loadURL("file://" + __dirname + "/index.html");
    mainWindow.openDevTools();

    mainWindow.on("closed", function() {
        mainWindow = null;
    });
});
```

次にhtml側

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="utf-8">
    <title>ほげ</title>
    <style>
        html, body{
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
        }
        body{
            -webkit-app-region: drag;
            -webkit-user-select: none;
        }
    </style>
</head>
<body>
    なんか適当にコンテンツ
</body>
</html>
```

みたいな感じで開発すると中に置いたコンテンツをドラッグすることでウィンドウを移動させることが出来る
部分的に解除する（ボタンとか）場合はその要素に

```css
.button{
    -webkit-app-region: no-drag;
}
```

することでその部分だけ対象から外すことが出来る

でもデスクトップマスコット的な画面全体でドラッグさせたいしクリック等イベントを取りたい時が困る
dragを有効にしてしまうとクリックイベントやマウスオーバーのイベントが全部取られてしまうので検知できなくなってしまう

- [-webkit-app-region: drag eats all click events](https://github.com/electron/electron/issues/1354)

issueにもある通り一応他にも困ってる人がいた
ただ自作しろ的オチしかないので困る

探してたら

- [electron-drag](https://www.npmjs.com/package/electron-drag)

というパッケージを見つけて良さ気なので入れてみたけど前提パッケージのwin-mouseが上手く動かず失敗した

一応トライしたことメモ

- Could not locate the bindings file. Tried:
  ファイルがズラーっと出る
  bindings/bindings.js
  の76行目に
  ```js
  b = opts.path ? require.resolve(n) : require(n)
  ```
  とあるけどopts.path自体がundefinedでコケてるっぽいので
  ```js
  b = require.resolve(n)
  ```
  とした
- Mouse is not a constructor
  上記の対策したら今度はこのエラー
  これ以上追いきれなかったので分からず

あとはプロセス間通信でウィンドウ側でマウスダウンとアップを検知してウィンドウをゴリゴリ動かしてipcとかで動かしたらいけそうかなって思ったけど労力に見合わ無さそうなのでどうなんだろうか

issueにもあったけど今回Electronに拘る理由はないからNW.js使ってみようかと思う
