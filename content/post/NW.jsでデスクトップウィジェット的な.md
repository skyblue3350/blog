+++
author = "すかい"
title = "NW.jsでデスクトップウィジェット的な"
date = "2016-11-02"
description = "NW.jsでデスクトップウィジェット的な"
tags = [
    "Node.js",
]
+++

[前回](../win-mouse%E3%81%8C%E4%BD%BF%E3%81%88%E3%81%AA%E3%81%84%E8%A9%B1/)トラブって上手くいかなかったのでNW.jsでリトライした

仕組みの都合上フロントのjsからウィンドウ操作が簡単に出来るので共存出来ました
こちらのコードをほぼそのままお借りました
変えたのは操作対象をwindowにしたくらい

- [NWjs: Chat head click not working if -webkit-app-region: drag is set](http://stackoverflow.com/questions/37185354/nwjs-chat-head-click-not-working-if-webkit-app-region-drag-is-set)

`package.json`

```json
{
  "name": "desktop",
  "version": "1.0.0",
  "description": "",
  "main": "index.html",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nw"
  },
  "window": {
    "width": 400,
    "height": 600,
    "toolbar": false,
    "transparent": true,
    "frame": false,
    "resizable": false,
    "always-on-top": true
  }
}
```

transparentとframeしか有効になってない気がする
常に全面になってないしツールバーはtrue/false時に挙動が変わらないような…

`index.html`

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="utf-8">
    <title>デスクトップマスコット</title>

    <script src="js/jquery.min.js"></script>
    <script src="js/script.js"></script>

    <style>
        html, body{
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
        }
        body{
            -webkit-user-select: none;
        }
    </style>
</head>
<body>
    適当な内容
</body>
</html>
```

`scripts.js`

```js
$(document).ready(function () {
    var wX = 0;
    var wY = 0;
    var dragging = false;
    $(window).mousedown(function (e) {
        dragging = true;
        wX = e.pageX;
        wY = e.pageY;
    });

    $(window).mousemove(function (e) {
        e.stopPropagation();
        e.preventDefault();
        if (dragging) {
            var xLoc = e.screenX - wX;
            var yLoc = e.screenY - wY;
            try {
                window.moveTo(xLoc, yLoc);
            } catch (err) {
                console.log(err);
            }
        }
    });
    $(window).mouseup(function () {
        dragging = false;
    });
    $("body").on("click", function(){
        console.log("click!");
    });
});
```

これでウィンドウでクリックとドラッグによる移動が共存出来る

別件だけど

```
$ npm install nv -g
```

したものは公式サイトでいうところのNormal版なんだろうか
というのもDevTools使おうと思ったものの開き方が分からない
公式ドキュメント読んでもSDK版ならF12で開けるよ！としか書いてない
仕方ないので公式からSDK版と落として来て
Userdir/AppData/Roaming/npm/nw/nwjs
に丸々上書きして誤魔化した
